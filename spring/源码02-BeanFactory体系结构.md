`org.springframework.beans.factory.config.ConfigurableBeanFactory#setParentBeanFactory`:

* `ConfigurableBeanFactory`实现类`AbstractBeanFactory`（很多重要的实现，都在这个类中）

```java
@Override
	public void setParentBeanFactory(@Nullable BeanFactory parentBeanFactory) {
    // 请注意，父级不能更改：如果在工厂实例化时不可用，则只应在*构造函数外部设置。
    // 父级容器，属性，只能设置一次，不能修改。
		if (this.parentBeanFactory != null && this.parentBeanFactory != parentBeanFactory) {
			throw new IllegalStateException("Already associated with parent BeanFactory: " + this.parentBeanFactory);
		}
		this.parentBeanFactory = parentBeanFactory;
	}
```

```java
public interface HierarchicalBeanFactory extends BeanFactory {

	BeanFactory getParentBeanFactory();
	
	boolean containsLocalBean(String name);
}
```

```java
package java.lang.reflect;
/**
 * @since 1.5
 */
public interface Type {
    default String getTypeName() {
        return toString();
    }
}
```

* 实现：`java.lang.Class#getTypeName`



 *Singleton or Prototype design pattern*:

* 单例设计模式和原型设计模式？区别？
* 原型设计模式：是一种创建型模式。

*canonical names.*：

* 规范名称



`DefaultSingletonBeanRegistry`，注册器：保留原始信息？

* 缓存单例bean对象，map中存储。