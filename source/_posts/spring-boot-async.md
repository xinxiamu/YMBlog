---
title: spring-boot异步调用
date: 2017-11-10 16:22:18
categories: spring-boot
tags: spring-boot-async
---

在一些场景中，为了快速响应页面，把一些对数据一致性要求没那么严格的业务逻辑放到异步执行，可以有效的提交系统性能。
在spring-boot中，通过简单的注解 `@Async` 就可以实现，非常优雅，不用再像以前自己直接new线程。下面是其使用方式：

## 1. 启用异步操作功能
很简单，只需要在主类中添加注解`@EnableAsync` 即可。

    package com.ymu.demo.async;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.scheduling.annotation.EnableAsync;
    
    @EnableAsync
    @SpringBootApplication
    public class SpringBootAsyncApplication {
    
    	public static void main(String[] args) {
    		SpringApplication.run(SpringBootAsyncApplication.class, args);
    	}
    }

## 2. 定义处理类，并添加方法

    package com.ymu.demo.async;
    
    import org.springframework.scheduling.annotation.Async;
    import org.springframework.scheduling.annotation.AsyncResult;
    import org.springframework.stereotype.Component;
    
    import java.util.Random;
    import java.util.concurrent.Future;
    
    @Component
    public class Task {
    
        public static Random random =new Random();
    
        @Async
        public void webTest() throws Exception {
            System.out.println("开始测试异步");
            long start = System.currentTimeMillis();
            Thread.sleep(random.nextInt(10000));
            long end = System.currentTimeMillis();
            System.out.println("完成测试异步，耗时：" + (end - start) + "毫秒");
        }
    
        @Async
        public Future<String> doTaskOne() throws Exception {
            System.out.println("开始做任务一");
            long start = System.currentTimeMillis();
            Thread.sleep(random.nextInt(10000));
            long end = System.currentTimeMillis();
            System.out.println("完成任务一，耗时：" + (end - start) + "毫秒");
            return new AsyncResult<>("任务一完成");
        }
    
        @Async
        public Future<String> doTaskTwo() throws Exception {
            System.out.println("开始做任务二");
            long start = System.currentTimeMillis();
            Thread.sleep(random.nextInt(10000));
            long end = System.currentTimeMillis();
            System.out.println("完成任务二，耗时：" + (end - start) + "毫秒");
            return new AsyncResult<>("任务二完成");
        }
    
        @Async
        public Future<String> doTaskThree() throws Exception {
            System.out.println("开始做任务三");
            long start = System.currentTimeMillis();
            Thread.sleep(random.nextInt(10000));
            long end = System.currentTimeMillis();
            System.out.println("完成任务三，耗时：" + (end - start) + "毫秒");
            return new AsyncResult<>("任务三完成");
        }
    }

只需要在方法上添加注解`@Async`。方法webTest是无返回值的，其他的是有返回值，返回的数据类型为Future类型，其为一个接口。具体的结果类型为AsyncResult,这个是需要注意的地方。通过其返回类型，可以检测异步线程执行的情况。

## 3. 测试

    package com.ymu.demo.async;
    
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.test.context.junit4.SpringRunner;
    
    import java.util.concurrent.Future;
    
    @RunWith(SpringRunner.class)
    @SpringBootTest
    public class SpringBootAsyncApplicationTests {
    
        @Autowired
        private Task task;
    
        @Test
        public void contextLoads() {
    
            System.out.println("===============load context");
        }
    
        @Test
        public void test() throws Exception {
    
            long start = System.currentTimeMillis();
    
            Future<String> task1 = task.doTaskOne();
            Future<String> task2 = task.doTaskTwo();
            Future<String> task3 = task.doTaskThree();
    
            while(true) {
                if(task1.isDone() && task2.isDone() && task3.isDone()) {
                    // 三个任务都调用完成，退出循环等待
                    break;
                }
                Thread.sleep(1000);
            }
    
            long end = System.currentTimeMillis();
    
            System.out.println("任务全部完成，总耗时：" + (end - start) + "毫秒");
    
        }
    
    }

---
    package com.ymu.demo.async;
    
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RestController;
    
    @RestController
    public class IndexController {
    
        @Autowired
        Task task;
    
        @GetMapping("/index")
        public String index() throws Exception {
            task.webTest();
            return "index";
        }
    }
    
## 4. 基于@Async调用中的异常处理机制
   在上面的异步调用中，调用者是无法感知异步线程的执行成功与否的。所以当要对异步线程执行异常做处理的时候，可以按下面方法来：
   
1. 自定义实现AsyncTaskExecutor的任务执行器。
2. 配置由自定义的TaskExecutor替代内置的任务执行器。      

自定义的TaskExecutor

    
