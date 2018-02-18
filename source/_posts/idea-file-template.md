---
title: idea创建类，方法注解提示模板
date: 2018-02-10 14:30:29
categories: idea
tags: idea注解模板
---

为了规范编码，在开发项目前，理应让团队每个人成员按公司编码规范定制好注释模板……

## java类的注解模板

1. 首先，打开设计：
`File->Settings->file and code template` 

{% asset_img a.png %}

`#parse("File Header.java")` 
这句：include 名字为File Header的模板进来。 

2. 定义如图： 

{% asset_img b.png %}

模板内容：

       /**
        * 功能简述:<br> 
        * 
        *
        * @author zmt
        * @create ${YEAR}-${MONTH}-${DAY} ${TIME}
        * @updateTime 
        * @since 1.0.0
        */
        
## java方法模板

在File->Settings->Editor->Live Templates下添加自定义Template Group，并在自定义Template Group下添加自定义Template

- 创建mygroup的Template,并定义名为/**的template。如下图：

{% asset_img c.png %}

模板内容：

    /**
     * 功能描述: <br>
     * 〈$END$〉
     *
     $param$
     * @return: $return$
     * @since: 1.0.0
     * @author: $user$
     * @date: $DATE$ $TIME$
     */
     
- 编辑模板的变量内容：

点击右边的Edit Variables: 

{% asset_img e.png %}

添加变量内容，如图： 

{% asset_img f.png %}

注意：$param$这么添加，否则无效：

    groovyScript("def result=''; def params=\"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList(); for(i = 0; i < params.size(); i++) {result+='* @param ' + params[i] + ((i < params.size() - 1) ? '\\n ' : '')}; return result", methodParameters())

- 选定作用域：

是应用在类上，还是方法上

{% asset_img g.png %}

- 测试：

在完成如上配置后，只需在方法内执行/**+Tab键即可生成注释，切记这里说的是方法内部，因为methodParameters()的作用域只在方法内部，这也是Intellij IDEA比较蛋疼的一点。

`注意：在方法内部操作或者在方法尾部，总之要在方法参数后面，再把注解剪切到方法头上，否则parameters无效`

{% asset_img j.png %}


     

