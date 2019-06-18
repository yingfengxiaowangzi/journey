## 第一章 初览spring boot

* 问题：
  * 颠覆J2EE的开发方式，J2EE标准，2中颠覆的方式：IoC和DI ？ejb中没有吗？
  * JSR，java specification requests：java标准规范
  * spring对事务的抽象，统一了数据库事务和分布式事务？怎么抽象的？
    * 这里源码和思想要学习一下
* 总结：
  * spring boot 6大特性
    * 创建独立的spring应用
    * 直接嵌入web容器，不需要部署war文件
      * 那这个内存消耗怎么看？是动态的吗？
    * 提供固化的starter依赖
    * 自动装配spring和第三方类库
    * 提供运维特性：指标信息(Metrics)、健康检查、外部化配置
    * 没有代码生成？，不需要xml配置？
  * 那么这本书，是对前三大特性的理解、梳理、思想的总结。



## 第二章 理解独立的spring应用

* 问题done：

  * Maven ArcheType插件：工程类插件？

    * 构建命令：插件简称，插件目标，插件参数，交互模式参数
    * `mvn dependency:tree - Dincludes=org.springframework.*`
      * 查看依赖树，仅关注spring相关的依赖

  * Maven Wrapper文件，是一个脚本文件，作用：从Maven官方下载Maven二进制文件，

  * spring-boot:run插件

  * *.jar.orginal是原始maven打包jar文件，而*.jar是repackage命令执行后生成的fat jar

  * fat jar解压后目录信息：

    * BOOT-INF/classes
    * BOOT-INF/lib
    * META-INF，MANIFEST.MF文件
    * org/目录

  * MF文件

    * 项目引导类`thinking.in.spring.boot.firstappbygui.FirstAppByGuiApplication`被`JarLauncher`引导并执行。

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



## 第三章 理解固话的Maven依赖

