---

title: SpringBoot
date: 2020-05-23 08:51:04
tags: springboot
category: java
---

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

![](批注 2020-05-23 145433.jpg)

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

## 二、配置文件

### 1、SpringBoot使用一个全局的配置文件，配置文件名是固定的；

application.properties

application.yml

配置文件的作用：修改SpringBoot自动配置的默认值

SpringBoot在底层都给我们自动配置好；

YAML是一个标记语言：以前的配置大都使用xxx.xml文件，而yaml以数据为中心，比json,xml更适合作配置文件

### 2、YAML语法

#### 1、k: v :标识一对键值对（空格必须有）

以空格的缩进来控制层级关系，只要左对齐的一列数据，都是一个层级的，属性和值也是大小写敏感；

#### 2、值的写法

字面量：普通的值（数字、字符串、布尔）

​	字面量直接来写，字符串默认不用加上单引号或者双引号

​	"":双引号，不会转义字符串里面的特殊字符，特殊字符会作为本身想表示的意思

​		name: "zhangsan \n lisi" 输出zhangsan 换行 lisi

​	'':单引号，会转义特殊字符，输出zhangsan \n lisi

对象、map（属性和值）（键值对）

​	k:v :对象还是k:v的形式

```yml
friends:
	lastName: zhangsan
		age: 18
```

​	行内写法

```yml
friends{lastName:zhangsan,age:18}
```

数组（list、set）

```yml
pets:
 - cat
 - dog
 - pig
```

​	行内写法

```yml
pets:{cat,dog,pig}
```

### 3、配置文件的注入和校验

#### 1、properities配置文件在idea中默认utf-8可能会乱码

```yml
person:
  name: zhangsan
  age: 18
  boss: false
  birth: 2020/12/12
  map: {k1: v1,k2: 12}
  objectList:
    - lisi
    - wangwu
  dog:
    name: mumu
    age: 2
```

javaBean

```java
/**
 * @ClassNamePerson
 * @Description 将配置文件中的每一个属性的值映射到这个组件中
 * @Author
 * @Date2020/5/23 16:08
 * @ConfigurationProperties告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定
 * prefix = "person":配置文件中哪个下面的所有属性进行一一映射
 * 只有这个组件是容器中的组件，才能用容器提供的@ConfigurationProperties功能,需要加上@Component
 * @ConfigurationProperties(prefix = "person")默认从全局配置文件中获取值
 **/
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Integer age;
    private boolean boss;
    private Date birth;
    private Map<String,Object> map;
    private List<Object> objectList;
    private Dog dog;
```

我们可以导入配置文件处理器，以后编写配置就有提示了

```xml
 <!--导入配置文件处理器，配置文件进行绑定就会有提示-->
 <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-configuration-processor</artifactId>
     <optional>true</optional>
 </dependency>
```

|                | @ConfigurationProperties   | @Value     |
| -------------- | -------------------------- | ---------- |
| 功能           | 批量注入文件的属性         | 一个个指定 |
| 松散绑定       | 支持（lastName,last-name） | 不支持     |
| SpEL           | 不支持                     | 支持       |
| JSR303数据校验 | 支持                       | 不支持     |
| 复杂类型封装   | 支持                       | 不支持     |

配置文件yml还是properties都可获取到值

如果只需要获取简单属性值可用@Value

#### 2、@PropertySource&ImportResource

@PropertySource:加载指定的配置文件，需要指定配置文件的路径

```java
/**
 * @Description 将配置文件中的每一个属性的值映射到这个组件中
 * @ConfigurationProperties告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定
 * prefix = "person":配置文件中哪个下面的所有属性进行一一映射
 * 只有这个组件是容器中的组件，才能用容器提供的@ConfigurationProperties功能,需要加上@Component
 **/
@PropertySource(value = {"classpath:person.properties"})
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Integer age;
    private boolean boss;
    private Date birth;
    private Map<String,Object> map;
    private List<Object> objectList;
    private Dog dog;
```

@ImportResource:导入Spring的配置文件，让配置文件里面的内容生效；

Spring Boot里面没有Spring的配置文件，我们自己编写的配置文件，也不能自动识别，想让Spring的配置文件生效，加载进来；@ImportResource需要标注在一个配置类上

```
@ImportResource(locations = {"classpath:beans.xml"})
导入Spring的配置文件，让其生效
```

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
<bean id="helloService" class="com.think.hello.service.HelloService"></bean>
</beans>
```

SpringBoot推荐给容器中添加组件的方式，推荐使用全注解的方式；

1、配置类===Spring配置文件

2、@Bean给容器中添加组件

```java
@Configuration
public class myAppConfig {
    //将方法的返回值添加到容器，容器中这个组件默认的id就是方法名
    @Bean
    public HelloService helloService(){
        return new HelloService();
    }
}
```

### 4、配置文件占位符

占位符后期之前配置的值，如果没有可用：指定默认值

```yml
person:
  name: zhangsan${random.uuid}
  boss: false
  age: ${random.int}
  birth: 2020/12/12
  map: {k1: v1,k2: 12}
  objectList:
    - lisi
    - wangwu
    - zhangsan
  dog:
    name: ${person.hello:hello}mumu
    age: 2
```

### 5、Profile

#### 1、多profile文件

我们在主配置文件编写的时候，文件名可用applicaton-{profile}.properties/yml

默认使用application.properties的配置

#### 2、yml支持多文档块的方式

```yml
server:
  port: 8080
spring:
  profiles:
    active: prod
---
server:
  port: 8081
spring:
  profiles: dev
---
server:
  port: 8082
spring:
  profiles: prod
---
```



#### 3、激活指定profile

##### 1、在配置文件中指定spring.profiles.active=dev

##### 2、命令行的方式

​	在启动配置里 --spring.profiles.active=dev或java -jar xxx.jar --spring.profiles.active=dev

##### 3、虚拟机参数

​	-Dspring.profiles.active=dev

### 6、配置文件的加载默认的优先级由高到低

高优先级的配置会覆盖低优先级的配置生效；

SpringBoot会从这四个位置全部加载主配置文件；互补配置；

-file:./conifg/

-file:./

-classpath:/config/

-classpath:/

我们还可用通过spring.config.location来改变默认的配置文件位置

项目打包后可用命令行参数的形式，启动项目的时候来指定配置文件的新位置，指定配置文件会和默认加载的这些配置文件共同起作用形成配置

 -jar xxx.jar --server.port=8080



### 7、自动配置原理

配置文件能配置的属性参照

[](https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#common-application-properties)

1、SpringBoot启动的时候加载主配置类，开启了自动配置功能@EnableAutoConfiguration

2、@EnableAutoConfiguration

​		利用AutoConfigurationImportSelector给容器中导入一些组件

​		可以查看selectImports（）方法的内容

```
List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
```

获取候选的配置

```
SpringFactoriesLoader.loadFactoryNames()
扫描所有jar包类路径下 META-INF/spring.factories
把扫描到的这些文件的内容包装成properties
从properties中获取到EnableAutoConfiguration.class（类名）对应的值，将其加入到容器中
```

将类路径下META-INF/spring.factories里面配置的所有EnableAutoConfiguration的值加入到了容器中

```xml
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer

# Auto Configuration Import Listeners
org.springframework.boot.autoconfigure.AutoConfigurationImportListener=\
org.springframework.boot.autoconfigure.condition.ConditionEvaluationReportAutoConfigurationImportListener

# Auto Configuration Import Filters
org.springframework.boot.autoconfigure.AutoConfigurationImportFilter=\
org.springframework.boot.autoconfigure.condition.OnBeanCondition,\
org.springframework.boot.autoconfigure.condition.OnClassCondition,\
org.springframework.boot.autoconfigure.condition.OnWebApplicationCondition

# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRestClientAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.r2dbc.R2dbcDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.r2dbc.R2dbcRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.r2dbc.R2dbcTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.ElasticsearchRestClientAutoConfiguration,\
org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration,\
org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration,\
org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration,\
org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
org.springframework.boot.autoconfigure.availability.ApplicationAvailabilityAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration,\
org.springframework.boot.autoconfigure.r2dbc.R2dbcAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketRequesterAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketServerAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketStrategiesAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.security.rsocket.RSocketSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.saml2.Saml2RelyingPartyAutoConfiguration,\
org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.reactive.ReactiveOAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration,\
org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration

# Failure analyzers
org.springframework.boot.diagnostics.FailureAnalyzer=\
org.springframework.boot.autoconfigure.diagnostics.analyzer.NoSuchBeanDefinitionFailureAnalyzer,\
org.springframework.boot.autoconfigure.flyway.FlywayMigrationScriptMissingFailureAnalyzer,\
org.springframework.boot.autoconfigure.jdbc.DataSourceBeanCreationFailureAnalyzer,\
org.springframework.boot.autoconfigure.jdbc.HikariDriverConfigurationFailureAnalyzer,\
org.springframework.boot.autoconfigure.r2dbc.ConnectionFactoryBeanCreationFailureAnalyzer,\
org.springframework.boot.autoconfigure.session.NonUniqueSessionRepositoryFailureAnalyzer

# Template availability providers
org.springframework.boot.autoconfigure.template.TemplateAvailabilityProvider=\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerTemplateAvailabilityProvider,\
org.springframework.boot.autoconfigure.mustache.MustacheTemplateAvailabilityProvider,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAvailabilityProvider,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafTemplateAvailabilityProvider,\
org.springframework.boot.autoconfigure.web.servlet.JspTemplateAvailabilityProvider

```

每一个autoconfigure类都是容器中的一个组件，都加入到容器中，用他们来做自动配置

每一个自动配置类进行自动配置功能;

以HttpEncodingAutoConfiguration为例解释自动配置原理

```
@Configuration(//表示这是一个自动配置类
    proxyBeanMethods = false
)
@EnableConfigurationProperties({ServerProperties.class})//启用configurationProperties功能，将配置文件中对应的值和xxxProperties绑定起来
@ConditionalOnWebApplication(//Spring底层@Conditional注解，根据不同条件，如果满足指定条件，整个配置里里面的配置就会生效，判断当前应用是否是web应用，是就生效不是就不生效
    type = Type.SERVLET
)
@ConditionalOnClass({CharacterEncodingFilter.class})//判断当前项目有没有这个类
CharacterEncodingFilter：SpringMVC中进行乱码解决的过滤器
@ConditionalOnProperty(//判断配置文件中是否存在某个配置server.servlet.encoding.enabled如果不存在判断也是成立的，即使我们的配置文件中不配置server.servlet.encoding.enabled=true,也是默认生效的  
    prefix = "server.servlet.encoding",//从配置文件中获取指定的值和bean的属性进行绑定
    value = {"enabled"},
    matchIfMissing = true
)
public class HttpEncodingAutoConfiguration {
```

根据当前不同的条件判断，决定这个配置类是否生效

*如果生效，这个配置类就会给容器中添加各种组件，这些组件的属性是从对应的properties类中获取的，这些类里面的每一个又是和配置文件绑定的*

```
 @Bean//给容器中添加一个组件,这个组件的某些值需要从properties中获取
 @ConditionalOnMissingBean
 public CharacterEncodingFilter characterEncodingFilter() {
 	CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
 	filter.setEncoding(this.properties.getCharset().name());
 	filter.setForceRequestEncoding(this.properties.shouldForce(org.springframework.boot.web.servlet.server.Encoding.Type.REQUEST));
 	filter.setForceResponseEncoding(this.properties.shouldForce(org.springframework.boot.web.servlet.server.Encoding.Type.RESPONSE));
 return filter;
 }
```

所有在配置文件中能配置的属性都是xxxProperties类中封装着，配置文件能配置什么就可以参照某个功能对应的这个属性类



总结 1、SpringBoot启动会加载大量的自动配置类；

​		2、我们看需要的功能有没有SpringBoot默认写好的自动配置类

​		3、我们再来看这个自动配置类中配置了哪些组件，只要我们要用的组件有，就不需要再来配置了，

​		4、给容器中自动配置类添加组件的时候，会从properties类中获取某些属性，我们就可以再配置文件总指定这些属性的值。

xxxAutoConfiguration:自动配置类

给容器中添加组件

xxxProperties：封装配置文件中相关属性



自动配置类哪个生效了

我们可以通过debug=true来让控制台打印自动配置报告，这样就可以很方便知道哪些自动配置生效了

```
============================
CONDITIONS EVALUATION REPORT
============================


Positive matches:（自动配置类启用的）
-----------------

AopAutoConfiguration matched:
- @ConditionalOnProperty (spring.aop.auto=true) matched (OnPropertyCondition)

AopAutoConfiguration.ClassProxyingConfiguration matched:
- @ConditionalOnMissingClass did not find unwanted class 'org.aspectj.weaver.Advice' (OnClassCondition)
- @ConditionalOnProperty (spring.aop.proxy-target-class=true) matched (OnPropertyCondition)

DispatcherServletAutoConfiguration matched:
- @ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet' (OnClassCondition)
- found 'session' scope (OnWebApplicationCondition)

DispatcherServletAutoConfiguration.DispatcherServletConfiguration matched:
- @ConditionalOnClass found required class 'javax.servlet.ServletRegistration' (OnClassCondition)
- Default DispatcherServlet did not find dispatcher servlet beans (DispatcherServletAutoConfiguration.DefaultDispatcherServletCondition)


Negative matches:（没有启用的，没匹配成功的）
-----------------

ActiveMQAutoConfiguration:
Did not match:
- @ConditionalOnClass did not find required class 'javax.jms.ConnectionFactory' (OnClassCondition)

AopAutoConfiguration.AspectJAutoProxyingConfiguration:
Did not match:
- @ConditionalOnClass did not find required class 'org.aspectj.weaver.Advice' (OnClassCondition)

ArtemisAutoConfiguration:
Did not match:
- @ConditionalOnClass did not find required class 'javax.jms.ConnectionFactory' (OnClassCondition)
```

