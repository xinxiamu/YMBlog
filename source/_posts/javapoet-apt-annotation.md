---
title: javapoet根据编译注解自动生成代码
date: 2018-05-03 20:54:06
categories: javapoet
tags: javapoet-apt-annotation
---

本文介绍利用apt技术，使用javapoet框架优雅的在编译阶段自动生成代码……

## 引入相关依赖包和插件

1.依赖包

    <!-- https://mvnrepository.com/artifact/com.google.auto.service/auto-service -->
    <dependency>
        <groupId>com.google.auto.service</groupId>
        <artifactId>auto-service</artifactId>
        <version>1.0-rc4</version>
    </dependency>
    <!--优雅生成代码-->
    <!--https://github.com/square/javapoet-->
    <dependency>
        <groupId>com.squareup</groupId>
        <artifactId>javapoet</artifactId>
        <version>1.10.0</version>
        <optional>true</optional>
    </dependency>
    
- javapoet是square公司出品的用来优雅生成代码库。网址： https://github.com/square/javapoet

- google auto-service是google公司的一个库，用来自动注册服务。

2.apt插件

    <plugin>
        <groupId>com.mysema.maven</groupId>
        <artifactId>maven-apt-plugin</artifactId>
        <version>1.0.4</version>
        <executions>
            <execution>
                <goals>
                    <goal>process</goal>
                </goals>
                <phase>generate-sources</phase>
                <configuration>
                    <outputDirectory>${project.basedir}/target/generated-sources/java</outputDirectory>
                    <processor>com.example.demo.HelloProcessor</processor>
                    <processors>
                        <processor>com.example.demo.HelloProcessor</processor>
                    </processors>
                    <showWarnings>true</showWarnings>
                </configuration>
            </execution>
        </executions>
    </plugin>
    
可以在插件中添加注解处理类。在编译时将会调用相关注解处理类做处理生成代码。

## demo演示

{% asset_img a.png %} 

1.新建注解类`HelloAnnotation`    

    package com.example.demo;
    
    import java.lang.annotation.ElementType;
    import java.lang.annotation.Retention;
    import java.lang.annotation.RetentionPolicy;
    import java.lang.annotation.Target;
    
    @Retention(RetentionPolicy.CLASS)
    @Target(ElementType.TYPE)
    public @interface HelloAnnotation {
    }


2.新建注解处理类`HelloProcessor`

    package com.example.demo;
    
    import com.google.auto.service.AutoService;
    import com.squareup.javapoet.JavaFile;
    import com.squareup.javapoet.MethodSpec;
    import com.squareup.javapoet.TypeSpec;
    
    import javax.annotation.processing.*;
    import javax.lang.model.SourceVersion;
    import javax.lang.model.element.Element;
    import javax.lang.model.element.Modifier;
    import javax.lang.model.element.Name;
    import javax.lang.model.element.TypeElement;
    import java.io.IOException;
    import java.lang.reflect.Method;
    import java.util.Collections;
    import java.util.Set;
    
    @AutoService(Processor.class)
    @SupportedAnnotationTypes(value = {"com.example.demo.HelloAnnotation"})
    @SupportedSourceVersion(SourceVersion.RELEASE_8)
    public final class HelloProcessor extends AbstractProcessor {
    
        private Filer filer;
    
        @Override
        public synchronized void init(ProcessingEnvironment processingEnv) {
            super.init(processingEnv);
    
            filer = processingEnv.getFiler(); // for creating file
            System.out.println(">>>processor init:" + filer.toString());
        }
    
        @Override
        public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
            System.out.println(">>>> process:" + annotations.size());
            for (TypeElement element : annotations) {
                /*Set<? extends Element> aa = roundEnv.getElementsAnnotatedWith(element);
                for (Element e:
                     aa) {
                    String name = e.getClass().getTypeName();
                    System.out.println(">>>name:" + name);
                    Name s = e.getSimpleName();
                    System.out.println(">>>aa:" + s.toString());
    
                    *//*Method[] m = e.getClass().getMethods();
                    for (int i = 0; i < m.length; i++) {
                        Method mm = m[i];
                        System.out.println(mm.getName());
                    }*//*
                }*/
                System.out.println(">>>>>> process：" + element.getQualifiedName().toString());
                if (element.getQualifiedName().toString().equals(HelloAnnotation.class.getCanonicalName())) {
                    // main method
                    MethodSpec main = MethodSpec.methodBuilder("main")
                            .addModifiers(Modifier.PUBLIC, Modifier.STATIC)
                            .returns(void.class)
                            .addParameter(String[].class, "args")
                            .addStatement("$T.out.println($S)", System.class, "Hello, JavaPoet!")
                            .build();
                    // HelloWorld class
                    TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
                            .addModifiers(Modifier.PUBLIC, Modifier.FINAL)
                            .addMethod(main)
                            .build();
    
                    try {
                        // build com.example.HelloWorld.java
                        JavaFile javaFile = JavaFile.builder("com.example", helloWorld)
                                .addFileComment(" This codes are generated automatically. Do not modify!")
                                .build();
                        // write to file
                        javaFile.writeTo(filer);
    
                        javaFile.writeTo(System.out);
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
            return true;
        }
    
        //采用注解
        /*@Override
        public Set<String> getSupportedAnnotationTypes() {
            return Collections.singleton(HelloAnnotation.class.getCanonicalName());
        }*/
    
        //采用注解
       /* @Override
        public SourceVersion getSupportedSourceVersion() {
            return SourceVersion.latestSupported();
        }*/
    }

-  加上注解`@AutoService(Processor.class)`，该注解为auto-service中的类，加上该注解，编译时候将会自动注册服务。在类路径下新增META-INF/services文件夹，并注册服务。

{%asset_img b.png%}

- 关键，覆盖方法`process`，并在该方法中利用javapoet库相关特性，优雅生成想要的java代码，并输出.java文件。输出路径可以在`maven`插件中配置。

3.定义使用类

    package com.example.demo;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    
    @HelloAnnotation
    @SpringBootApplication
    public class DemoApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(DemoApplication.class, args);
    //        HelloWorld.main(null);
        }
    }

在类上添加注解`@HelloAnnotation`。  
那么用`mvn install`或者运行该类编译代码时，将会调用注解处理类`HelloProcessor`进行代码生成。

----------------------------------------------------------------------------------

<<完>>