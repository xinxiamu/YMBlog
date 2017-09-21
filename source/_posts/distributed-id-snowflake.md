---
title: 分布式ID生成-snowflake算法
date: 2017-09-20 11:06:57
categories: 分布式系统
tags: snowflake算法
---
## 应用场景
snowflake是twitter开源的分布式ID生成算法，其核心思想是：一个long型的ID，使用其中41bit作为毫秒数，10bit作为机器编号，12bit作为毫秒内序列号。这个算法单机每秒内理论上最多可以生成1000*(2^12)，也就是400W的ID，完全能满足业务的需求。
借鉴snowflake的思想，结合各公司的业务逻辑和并发量，可以实现自己的分布式ID生成算法。
#### 举例，假设某公司ID生成器服务的需求如下：
（1）单机高峰并发量小于1W，预计未来5年单机高峰并发量小于10W
（2）有2个机房，预计未来5年机房数量小于4个
（3）每个机房机器数小于100台
（4）目前有5个业务线有ID生成需求，预计未来业务线数量小于10个
（5）…
分析过程如下：
（1）高位取从2016年1月1日到现在的毫秒数（假设系统ID生成器服务在这个时间之后上线），假设系统至少运行10年，那至少需要10年*365天*24小时*3600秒*1000毫秒=320*10^9，差不多预留39bit给毫秒数
（2）每秒的单机高峰并发量小于10W，即平均每毫秒的单机高峰并发量小于100，差不多预留7bit给每毫秒内序列号
（3）5年内机房数小于4个，预留2bit给机房标识
（4）每个机房小于100台机器，预留7bit给每个机房内的服务器标识
（5）业务线小于10个，预留4bit给业务线标识
 {% asset_img a.png %}   
 这样设计的64bit标识，可以保证：
 （1）每个业务线、每个机房、每个机器生成的ID都是不同的
 （2）同一个机器，每个毫秒内生成的ID都是不同的
 （3）同一个机器，同一个毫秒内，以序列号区区分保证生成的ID是不同的
 （4）将毫秒数放在最高位，保证生成的ID是趋势递增的
 缺点：
 （1）由于“没有一个全局时钟”，每台服务器分配的ID是绝对递增的，但从全局看，生成的ID只是趋势递增的（有些服务器的时间早，有些服务器的时间晚）
 最后一个容易忽略的问题：
 生成的ID，例如message-id/ order-id/ tiezi-id，在数据量大时往往需要分库分表，这些ID经常作为取模分库分表的依据，为了分库分表后数据均匀，ID生成往往有“取模随机性”的需求，所以我们通常把每秒内的序列号放在ID的最末位，保证生成的ID是随机的。
 又如果，我们在跨毫秒时，序列号总是归0，会使得序列号为0的ID比较多，导致生成的ID取模后不均匀。解决方法是，序列号不是每次都归0，而是归一个0到9的随机数，这个地方。
 
    package com.ymu.spcselling.infrastructure.idgenerator;
    
    import lombok.extern.slf4j.Slf4j;
    
    /**
     * <p>
     * Snowflake算法是带有时间戳的全局唯一ID生成算法。它有一套固定的ID格式，如下：
     * <p>
     * 41位的时间序列（精确到毫秒，41位的长度可以使用69年）
     * 10位的机器标识（10位的长度最多支持部署1024个节点）
     * 12位的Sequence序列号（12位的Sequence序列号支持每个节点每毫秒产生4096个ID序号）
     * <p>
     * 结构如下(每部分用-分开):<br>
     * 0 - 0000000000 0000000000 0000000000 0000000000 0 - 00000 - 00000 - 000000000000 <br>
     * 优点是：整体上按照时间自增排序，且整个分布式系统内不会产生ID碰撞(由数据中心ID和机器ID作区分)
     * Author:frankwoo(吴峻申) <br>
     * Date:2017/8/29 <br>
     * Time:下午6:32 <br>
     * Mail:frank_wjs@hotmail.com <br>
     */
    @Slf4j
    public class SnowflakeIdWorker {
        //开始时间截 (从2015-01-01起)
        private static final long START_TIME = 1420041600000L;
        // 机器ID所占位数
        private static final long ID_BITS = 5L;
        //数据中心ID所占位数
        private static final long DATA_CENTER_ID_BITS = 5L;
        // 机器ID最大值31 (此移位算法可很快计算出n位二进制数所能表示的最大十进制数)
        private static final long MAX_ID = ~(-1L << ID_BITS);
        // 数据中心ID最大值31
        private static final long MAX_DATA_CENTER_ID = ~(-1L << DATA_CENTER_ID_BITS);
        //Sequence所占位数
        private static final long SEQUENCE_BITS = 12L;
        //机器ID偏移量12
        private static final long ID_SHIFT_BITS = SEQUENCE_BITS;
        //数据中心ID偏移量12+5=17
        private static final long DATA_CENTER_ID_SHIFT_BITS = SEQUENCE_BITS + ID_BITS;
        //时间戳的偏移量12+5+5=22
        private static final long TIMESTAMP_LEFT_SHIFT_BITS = SEQUENCE_BITS + ID_BITS + DATA_CENTER_ID_BITS;
        // Sequence掩码4095
        private static final long SEQUENCE_MASK = ~(-1L << SEQUENCE_BITS);
        // 上一毫秒数
        private static long lastTimestamp = -1L;
        //毫秒内Sequence(0~4095)
        private static long sequence = 0L;
        //机器ID(0-31)
        private final long workerId;
        //数据中心ID(0-31)
        private final long dataCenterId;
    
        /**
         * 构造
         *
         * @param workerId     机器ID(0-31)
         * @param dataCenterId 数据中心ID(0-31)
         */
        public SnowflakeIdWorker(long workerId, long dataCenterId) {
            if (workerId > MAX_ID || workerId < 0) {
                throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0", MAX_ID));
            }
            if (dataCenterId > MAX_DATA_CENTER_ID || dataCenterId < 0) {
                throw new IllegalArgumentException(String.format("datacenter Id can't be greater than %d or less than 0", MAX_DATA_CENTER_ID));
            }
            this.workerId = workerId;
            this.dataCenterId = dataCenterId;
            log.info(String.format("worker starting. timestamp left shift %d, datacenter id bits %d, worker id bits %d, sequence bits %d, workerid %d", TIMESTAMP_LEFT_SHIFT_BITS, DATA_CENTER_ID_BITS, ID_BITS, SEQUENCE_BITS, workerId));
        }
    
        /**
         * 生成ID（线程安全）
         *
         * @return id
         */
        public synchronized long nextId() {
            long timestamp = timeGen();
    
            //如果当前时间小于上一次ID生成的时间戳，说明系统时钟被修改过，回退在上一次ID生成时间之前应当抛出异常！！！
            if (timestamp < lastTimestamp) {
                log.error(String.format("clock is moving backwards.  Rejecting requests until %d.", lastTimestamp));
                throw new IllegalStateException(String.format("Clock moved backwards.  Refusing to generate id for %d milliseconds", lastTimestamp - timestamp));
            }
    
            //如果是同一时间生成的，则进行毫秒内sequence生成
            if (lastTimestamp == timestamp) {
                sequence = (sequence + 1) & SEQUENCE_MASK;
                //溢出处理
                if (sequence == 0) {//阻塞到下一毫秒,获得新时间戳
                    timestamp = tilNextMillis(lastTimestamp);
                }
            } else {//时间戳改变，毫秒内sequence重置
                sequence = 0L;
            }
            //上次生成ID时间截
            lastTimestamp = timestamp;
    
            //移位并通过或运算组成64位ID
            return ((timestamp - START_TIME) << TIMESTAMP_LEFT_SHIFT_BITS) | (dataCenterId << DATA_CENTER_ID_SHIFT_BITS) | (workerId << ID_SHIFT_BITS) | sequence;
        }
    
        /**
         * 阻塞到下一毫秒,获得新时间戳
         *
         * @param lastTimestamp 上次生成ID时间截
         * @return 当前时间戳
         */
        private long tilNextMillis(long lastTimestamp) {
            long timestamp = timeGen();
            while (timestamp <= lastTimestamp) {
                timestamp = timeGen();
            }
            return timestamp;
        }
    
        /**
         * 获取以毫秒为单位的当前时间
         *
         * @return 当前时间(毫秒)
         */
        private long timeGen() {
            return System.currentTimeMillis();
        }
    
        //==============================Test=============================================
        /** 测试 */
        /*public static void main(String[] args) {
            SnowflakeIdWorker idWorker = new SnowflakeIdWorker(0, 0);
            for (int i = 0; i < 1000; i++) {
                long id = idWorker.nextId();
                System.out.println(Long.toBinaryString(id));
                System.out.println(id);
            }
        }*/
    } 
