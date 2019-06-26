# springboot源码

springboot启动：

* `org.springframework.boot.SpringApplication#run(java.lang.String…)`
  1. 准备环境，包括命令行的参数和默认的应用参数，还有监听器
  2. 创建web容器：servlet类型或者reactive类型
  3. 异常报告器
  4. 准备上下文：环境，监听者，应用参数，banner
  5. 启动上下文
  6. 应用程序的Log
  7. 监听器启动
  8. 调用运行器（CommandLineRunner、ApplicationRunner）？？？

* 事件广播器：`org.springframework.context.event.ApplicationEventMulticaster#addApplicationListener`

1. 实现类`org.springframework.context.event.SimpleApplicationEventMulticaster`

2. `java.util.EventObject`

   1. ```java
      public abstract class ApplicationEvent extends EventObject {}
      ```

3. `PayloadApplicationEvent`：载荷，可以存储事件的相关属性，当事件触发的时候，备监听到的时候，可以将相关的属性取出来。

   1. ```java
      // 添加监听器
      multicaster.addApplicationListener(event -> {
        if (event instanceof PayloadApplicationEvent) {
          System.out.println("接受到 PayloadApplicationEvent :"                       + PayloadApplicationEvent.class.cast(event).getPayload());
        }else {
          System.out.println("接收到事件：" + event);
        }
      });
      ```

   2. java.lang.Class#cast：转型。

   3. ```java
      public T cast(Object obj) {
        if (obj != null && !isInstance(obj))
          throw new ClassCastException(cannotCastMsg(obj));
        return (T) obj;
      }
      ```

   4. `com.gupao.spring.SpringAnnotationDemo$$EnhancerBySpringCGLIB$$d90c9946@6f03482`

      1. 类名称中为什么会有2个$符号呢？？

4. ```java
   // 应用监听器。
   public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
     // 事件触发时候的回调方法，on开头，当。。。的时候
   	void onApplicationEvent(E event);
   }
   ```

* `org.springframework.context.support.AbstractApplicationContext#addApplicationListener`
  1. 应用中添加应用监听器。
* `org.springframework.context.ApplicationEventPublisher#publishEvent(java.lang.Object)`
  1. Spring 应用上下文发布事件

