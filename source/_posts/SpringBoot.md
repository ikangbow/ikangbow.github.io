---
title: SpringBoot
date: 2020-05-23 08:51:04
tags: springboot
---

## Spring Boot入门

## 一、Spring Boot入门

### 1、Spring Boot简介

简化Spring应用开发的一个框架

整个Spring技术栈的一个大整合

J2EE开发的一站式解决方案

### 2、微服务

微服务：架构风格

一个应用应该是一组小型服务；可以通过HTTP的方式进行互通；

每一个功能元素最终都是一个可独立替换和独立升级的软件单元；

### 3、环境准备

#### 1、环境约束

-jdk1.8

-maven 3.x

-springboot1.5.9RELEASE

#### 2、MAVEN设置

```xml
<profile>
    <id>jdk-1.8</id>
    <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
    </activation>
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
</profile>
```

### 4、Spring Boot HelloWorld

浏览器发送hello请求，服务器接受请求并处理，响应HelloWorld字符串；

#### 1、创建一个maven工程（jar）

#### 2、导入springBoot 依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
        <exclusion>
        <groupId>org.junit.vintage</groupId>
        <artifactId>junit-vintage-engine</artifactId>
        </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-logging</artifactId>
    </dependency>
</dependencies>
```

#### 3、编写主程序，启动Spring Boot应用

```java
/**
 * @Date2020/5/23 12:53
 * SpringBootApplication来标注一个主程序
 **/
@SpringBootApplication
public class HelloWorldMainApplication {
    public static void main(String[] args) {
        //spring应用启动起来
        SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}
```

#### 4、编写相关的Controller、Service

#### 5、运行主程序测试

#### 6、简化部署

```xml
<!--可以将应用打包成一个可执行的jar宝-->
<build>
    <plugins>
        <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

##### 1、将这个应用打成jar包，直接用java -jar的命令进行执行；

##### 2、终止运行

```shell
netstat -aon|findstr "8080"
taskkill /f /pid 8976  终止jar命令运行的程序
```

### 5、Hello World探究

#### 1、pom文件

##### 1、父文件

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
他的父项目是是

```

Spring Boot的版本仲裁中心

以后我们导入依赖默认是不需要写版本号的（没有在dependencies里面管理的依赖自然需要声明版本号）

##### 2、导入的依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

spring-boot-starter-web：

###### spring-boot-starter：spring-boot场景启动器；帮我们导入了web模块正常运行所依赖的组件；

Spring Boot将所有功能场景都抽取出来，做成一个个starters(启动器)，只需要在项目里引入这些starter相关场景的所有依赖都会导入进来。要用什么功能就导入什么场景的启动器

##### 3、主程序类

```java
/**
 * @Date2020/5/23 12:53
 * SpringBootApplication来标注一个主程序,说明这是一个Spring Boot应用
 **/
@SpringBootApplication
public class HelloWorldMainApplication {

    private static Logger log= LoggerFactory.getLogger(HelloWorldMainApplication.class);

    public static void main(String[] args) {
        log.info("HelloWorldMainApplication is success!");
        //spring应用启动起来
        SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}
```

@SpringBootApplication Spring Boot应用标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
```

@SpringBootConfiguration：SpringBoot的配置类；

​	标注在某个类上，标识这是一个SpringBoot的配置类；

​	@Configuration:配置类上来标注这个注解：

​		配置类---配置文件；配置类也是容器中的一个组件；@Componet

@EnableAutoConfiguration：开启自动配置功能

​	以前需要配置的东西，SpringBoot帮我们自动配置；@EnableAutoConfiguration告诉SpringBoot开启自动配置功能；这样自动配置才能生效。

```java
@AutoConfigurationPackage
@Import({EnableAutoConfigurationImportSelector.class})
```

​	@AutoConfigurationPackage：自动配置包

​			@Import({Registrar.class})

​			Spring的底层注解@Import，给容器中导入一个组件；导入的组件由Registrar.class

​			将主配置类（@SpringBootApplication标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器；

​	@Import({EnableAutoConfigurationImportSelector.class})

​	给容器中导入组件

​	EnableAutoConfigurationImportSelector：导入那些组件的选择器；

​	将所有需要导入的组件以全类名的方式返回；

​	会给容器中导入非常多的自动配置类（xxxAutoConfiguration）;就是给容器中导入这个场景需要的所有组件，并配置好这些组件；免去了我们手动编写配置注入功能组件的工作。

```java
SpringFactoriesLoader.loadFactoryNames(
      getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
```

![批注 2020-05-23 145433.jpg]()

Spring Boot在启动的时候从类路径下的"META-INF/spring.factories"中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作。

J2EE的整体整合解决方案和自动配置都在m2\repository\org\springframework\boot\spring-boot-autoconfigure\1.5.9.RELEASE\spring-boot-autoconfigure-1.5.9.RELEASE.jar

### 6、使用Spring Initializer快速创建Spring Boot项目

IDE都支持使用Spring的项目创建向导快速创建一个Spring Boot项目；

选择我们需要的模块，向导会联网进行项目的创建；

默认生成的Spring Boot项目

1. 主程序已经生成好了，只需要实现我们自己的逻辑

2. resources文件夹的目录结构

   static:保存所有的js  css images

   templates:保存所有的页面模板；（Spring Boot默认jar包使用嵌入式的Tomcat,默认不支持JSP页面）；可以使用模板引擎（freemarker、thymeleaf）;

   application.properties:Spring Boot 的配置文件，可以修改一些默认设置；