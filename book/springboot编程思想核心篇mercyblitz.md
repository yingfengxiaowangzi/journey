> 笔记的作用：总结作者的思想和意图，理清文章的思路。阐明了哪些问题。

## 第一章 初览spring boot

* 问题：
  * 颠覆J2EE的开发方式，J2EE标准，2中颠覆的方式：IoC和DI ？ejb中没有吗？
  * JSR，java specification requests：java标准规范
  * spring对事务的抽象，统一了数据库事务和分布式事务？怎么抽象的？
    * 这里源码和思想要学习一下
  
* 总结：
  * spring boot 6大特性（5大特性？第六章）
    * 创建独立的spring应用
    * 直接嵌入web容器，不需要部署war文件
      * 那这个内存消耗怎么看？是动态的吗？
    * 提供固化的starter依赖
    * 自动装配spring和第三方类库
    * 提供运维特性：指标信息(Metrics)、健康检查、外部化配置
    * 没有代码生成？，不需要xml配置？
  * 小马哥，spring5大特性：（这5大特性构成了spring boot作为微服务中间件的基础）
    1. SpringApplication
    2. 自动装配
    3. 外部化配置
    4. Spring Boot Actuator
    5. 嵌入式Web容器
  
  * 那么这本书，是对前三大特性的理解、梳理、思想的总结。



## 第二章 理解独立的spring应用

* 问题done：

  * Maven ArcheType插件：工程类插件？

    * 构建命令：插件简称，插件目标，插件参数，交互模式参数
    * `mvn dependency:tree -Dincludes=org.springframework.*`
      * 查看依赖树，仅关注spring相关的依赖

  * Maven Wrapper文件，是一个脚本文件，作用：从Maven官方下载Maven二进制文件，

  * spring-boot:run插件

  * *.jar.orginal是原始maven打包jar文件，而*.jar是repackage命令执行后生成的fat jar

  * fat jar解压后目录信息：

    * BOOT-INF/classes

    * BOOT-INF/lib

    * META-INF -> MANIFEST.MF文件(注意文件位置)，这个文件在启动的时候被识别？？

      * java -jar命令，引导的具体启动类必须配置在MANIFEST.MF资源的Main-Class属性中：

      * jar文件规范，MANIFEST.MF资源必须在/META-INF/目录下。

      * fat jar 和war 执行模块`spring-boot-loader`

      * ```xml
        <!-- spring-boot-loader 用于源码分析 -->
        		<dependency>
        			<groupId>org.springframework.boot</groupId>
        			<artifactId>spring-boot-loader</artifactId>
        			<scope>provided</scope>
        		</dependency>
        ```

    * org/目录

  * MANIFEST.MF文件

    * 项目引导类`thinking.in.spring.boot.firstappbygui.FirstAppByGuiApplication`被`JarLauncher`引导并执行。
    * 

    `org.springframework.boot.loader.JarLauncher`

  * ```properties
    Spring-Boot-Version: 2.0.2.RELEASE
    Main-Class: org.springframework.boot.loader.JarLauncher
    Start-Class: thinking.in.spring.boot.firstappbygui.FirstAppByGuiApplication
    Spring-Boot-Classes: BOOT-INF/classes/
    Spring-Boot-Lib: BOOT-INF/lib/
    ```

* 问题undone：

  * 传统spring应用，外置容器需要启动脚本将其引导，随其生命周期回调执行spring应用上下文
    * 例如ContextLoaderListener和DispatcherServlet
    * springboot中，使用核心组件SpringApplication启动spring应用上下文。
  * java标准`java.util.jar.JarEntry`
  * java系统属性：java.protocol.handler.pkgs
    * jdk支持的协议，
    * springboot扩展了JAR协议，所以可以使用java -jar启动命令？启动一个独立的应用归档文件。
    * 扩展了URL协议，可以看源码实现机制。
  * WarLauncher，WAR增加了WEB-INF/lib-provided目录
    * 该目录可以被Servlet容器忽略，
    * 目的：打包后的war文件能够在Servlet容器中兼容运行

* 总结：



## 第三章 理解固化的Maven依赖

* 问题done：
  * spring-boot-dependencies是spring-boot-starter-parent的parent。区别？作用？
  * `mvn clean package`：可以观看详细的构建日志；明白每一步都发生了什么，使用了哪些插件构建项目；
    * maven-war-plugin
    * spring-boot-maven-plugin
  * maven-war-plugin：2.2 版本，默认打包规则是必须存在WEB-INF/web.xml文件；3.1.0调整了该默认行为；
  * `mvn dependency:tree -Dincludes=org.springframework.*`
    * 分析tomcat依赖。
* 问题undone：
  * maven，scope = import？？
  * web应用部署描述文件WEB-INF/web.xml，在springboot项目中不存在。
  * <goal>repackage</goal>??
  * 
* 总结：

## 第四章 理解嵌入式web容器

* 问题done：
* 问题undone：
  * java -jar 命令启动，修正spring boot fat jar或者fat war。如何修正的？？【第二章】
    * java -jar命令是如何找到main方法所在类的？命令的规范。
  * 传统的Servlet容器将压缩的war文件解压到对应目录，再加载该目录中的资源。
    * 而springboot可执行war文件则需要不解压当前war文件的前提下读取其中的资源。
    * 这就是为什么spring-boot-loader需要覆盖内建jar协议的URLStreamHandler实现的原因？
    * 哪里覆盖了？url文件流处理器。？
* 总结：
  * Servlet规范4.0对应Tomcat9.x
  * Servlet规范的重要版本：2.5，3.0，3.1，4.0
  * `mvn spring-boot:run`：注意，有中划线。
  * Servlet组件：Servlet、Filter、Listener
    * api可插拔性？？
    * ServletContainerInitializer生命周期的回调

```xml
<!-- WebFlux 依赖 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-webflux</artifactId>
		</dependency>
```

maven编译插件配置jdk版本：

```xml
<!-- 保持与 spring-boot-dependencies 版本一致 -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.7.0</version>
				<configuration>
					<source>${java.version}</source>
					<target>${java.version}</target>
				</configuration>
			</plugin>
```

web服务器已初始化事件：

```java
@EventListener(WebServerInitializedEvent.class)
    public void onWebServerReady(WebServerInitializedEvent event) {
        System.out.println("当前 WebServer 实现类为：" + event.getWebServer().getClass().getName());
    }
```



## 第五章 理解自动装配

* 问题done：

  * 如何装配@Configuration类：
    1. xml的`<context:component-scan>`
    2. 注解@Import
    3. @ComponentScan
  * @SpringBootApplication等价于（还是有一些差异的）
    1. @EnableAutoConfiguration
    2. @ComponentScan
    3. @Configuration
  * @SpringBootApplication属于多层次的@Component“派生”注解，所以能够被@ComponentScan识别。
    * 也就是说：？：这个类启动扫描，然后还会扫描到自身，作为bean注册到容器上下文中。
    * 低层次的注解，会获取高层次注解的能力。那么注解组合起来，功能会很强大。

  ```java
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Inherited
  @SpringBootConfiguration
  @EnableAutoConfiguration
  @ComponentScan(excludeFilters = {
  		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
  		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
  public @interface SpringBootApplication {}
  ```

  ```java
  public interface TypeFilter {
  	boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)
  			throws IOException;
  }
  ```

  ```java
  public class TypeExcludeFilter implements TypeFilter, BeanFactoryAware {
  	@Override
  	public boolean match(MetadataReader metadataReader,
  			MetadataReaderFactory metadataReaderFactory) throws IOException {
  		if (this.beanFactory instanceof ListableBeanFactory
  				&& getClass() == TypeExcludeFilter.class) {
  			Collection<TypeExcludeFilter> delegates = ((ListableBeanFactory) this.beanFactory)
  					.getBeansOfType(TypeExcludeFilter.class).values();
  			for (TypeExcludeFilter delegate : delegates) {
          // 调用自身match方法？子类也有这个方法，不然永远返回不了true。
  				if (delegate.match(metadataReader, metadataReaderFactory)) {
  					return true;
  				}
  			}
  		}
  		return false;
  	}
  }
  ```

  spring注解的多层次“派生性”：

  ```java
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Configuration
  public @interface SpringBootConfiguration {
  }
  ```

  ```java
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Component
  public @interface Configuration {
  	@AliasFor(annotation = Component.class)
  	String value() default "";
  }
  ```

  ```java
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Inherited
  @AutoConfigurationPackage
  @Import(AutoConfigurationImportSelector.class)
  // 没有派生@Configuration
  public @interface EnableAutoConfiguration {}
  ```

  

* 问题undone

  * @Configuration的CGLIB提升特性？？
    * @Configuration作用：表示一个类声明了一个或多个{@link Bean @Bean}方法，并且*可以由Spring容器处理，以便在运行时为这些bean生成bean定义和*服务请求
  * @Bean在@Component类与在@Configuration类中的差异？
    * 前者不存在CGLIB处理
    * 后者执行了CGLIB提升？？
  * CGLIB提升的实现？？
  * spring5中@Indexed，在代码编译时，向@Component和“派生”注解添加索引？减少运行时性能消耗？应该是启动时启动速度的影响吧？？

* 总结：

  * 属性别名：@AliasFor，用于桥接其他注解的属性。
    * @SpringBootApplication利用@AliasFor注解，别名了@ComponentScan注解的basePackages()属性。
    * spring的注解编程模型
  * @Bean属于编码方式的导入，并不是自动装配；
  * @ConditionalOnClass: 仅在指定的类位于类路径上时才匹配。
    * 与@Configuration配合使用
    * 与@ConditionalOnMissingBean
    * 与@Import
      * 表示要导入的一个或多个{@link Configuration @Configuration}类。
  * 开发人员自定义自动装配类，打包到jar中？？
    * `org.springframework.boot.autoconfigure.EnableAutoConfiguration`
    * 自定义类要配置在META-INF/spring.factories资源中。

```java
@AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
	String[] scanBasePackages() default {};
```



```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Documented
public @interface AliasFor {
	@AliasFor("attribute")
	String value() default "";

	@AliasFor("value")
	String attribute() default "";

	Class<? extends Annotation> annotation() default Annotation.class;
}
```



## 第6章 理解Production-Ready特性

* 问题done
* 问题undone：
* 总结：
  * metrics指标，health check， 外部配置
  * Actuator可以视为将Production-Ready的概念具体化。Spring Environment抽象与外部化配置的关系亦是如此。
  * Actuator：制动器，传动装置。
    * 作用：监控和管理spring应用。
  * `mvn spring-boot:run -Dmanagement.endpoints.web.exposure.include=beans,conditions,env`
  * 常用的Endpoints：
    1. beans：容器中的bean，包含所有Application Context层次
    2. conditions：当前应用所有配置类和自动装配类的条件评估结果（匹配和非匹配）
    3. env：环境配置信息
    4. health
    5. info：应用信息

```xml
<!-- Spring Boot Actuator 依赖 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
```

条件化注解：当应用是web应用时：

```java
Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(OnWebApplicationCondition.class)
public @interface ConditionalOnWebApplication {}
```

