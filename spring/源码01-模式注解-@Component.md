spring应用指定package扫描spring模式注解时，反射不适用，asm适用。



```java
public class ClassPathBeanDefinitionScanner extends ClassPathScanningCandidateComponentProvider {}
```

ClassPathBeanDefinitionScanner：一个bean定义扫描程序，用于检测类路径上的bean候选者，*使用给定的注册表（{@code BeanFactory} *或{@code ApplicationContext}）注册相应的bean定义。



BeanDefinitionRegistry: 包含bean定义的注册表的接口，例如RootBeanDefinition *和ChildBeanDefinition实例。通常由BeanFactories实现，*内部使用AbstractBeanDefinition层次结构。

`ClassUtils`工具类中使用缓存。

```java
/**
	 * Map with common Java language class name as key and corresponding Class as value.
	 * Primarily for efficient deserialization of remote invocations.
	 */
	private static final Map<String, Class<?>> commonClassCache = new HashMap<>(64);
```

类加载器：一个典型的策略是将名称转换为文件名，然后从文件系统中读取该名称的*“类文件”。

```java
// 注解过滤器：一个简单的过滤器，它匹配具有给定注释的类，*也检查继承的注释。
// 过滤器：有几个boolean返回值类型的判断方法。
public class AnnotationTypeFilter extends AbstractTypeHierarchyTraversingFilter {
	// 封装一个注解对象
	private final Class<? extends Annotation> annotationType;
}
```



`org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider#registerDefaultFilters`

```java
// 注册{@link Component @Component}的默认过滤器。
protected void registerDefaultFilters() {
		this.includeFilters.add(new AnnotationTypeFilter(Component.class));
		ClassLoader cl = ClassPathScanningCandidateComponentProvider.class.getClassLoader();
		try {
			this.includeFilters.add(new AnnotationTypeFilter(
					((Class<? extends Annotation>) ClassUtils.forName("javax.annotation.ManagedBean", cl)), false));
      // javax.annotation包可能没有配置到类路径的。如果没有配置，会抛出异常，捕获掉。
			logger.trace("JSR-250 'javax.annotation.ManagedBean' found and supported for component scanning");
		}
		catch (ClassNotFoundException ex) {
			// JSR-250 1.1 API (as included in Java EE 6) not available - simply skip.
		}
		try {
			this.includeFilters.add(new AnnotationTypeFilter(
					((Class<? extends Annotation>) ClassUtils.forName("javax.inject.Named", cl)), false));
			logger.trace("JSR-330 'javax.inject.Named' annotation found and supported for component scanning");
		}
		catch (ClassNotFoundException ex) {
			// JSR-330 API not available - simply skip.
		}
	}
```



```java
// 扫面组件，返回bean定义的set集合。
public Set<BeanDefinition> findCandidateComponents(String basePackage) {
		if (this.componentsIndex != null && indexSupportsIncludeFilters()) {
			return addCandidateComponentsFromIndex(this.componentsIndex, basePackage);
		}
		else {
			return scanCandidateComponents(basePackage);
		}
	}
```

`org.springframework.asm.ClassReader`

* 一个解析器，使{@link ClassVisitor}访问ClassFile结构，如Java *虚拟机规范（JVMS）中所定义。
* asm包？？

```java
// ASM类访问者，它查找类名并将类型实现为*以及在类上定义的注释，通过* {@link org.springframework.core.type.AnnotationMetadata}接口公开它们。
// 访问器：get方法。从。。。中get。那么有一个资源的，从类中获取注解元数据。
// 读取注解元数据的访问器。
public class AnnotationMetadataReadingVisitor extends ClassMetadataReadingVisitor implements AnnotationMetadata {
```



`MetadataReader`: 

* 用于访问类元数据的简单外观，*由ASM {@link org.springframework.asm.ClassReader}读取。

`BeanDefinition` :

* BeanDefinition描述了一个bean实例，它具有属性值，*构造函数参数值以及*具体实现提供的更多信息。

```java
public interface AnnotatedBeanDefinition extends BeanDefinition {
	// 获取此bean定义的bean类的注释元数据（以及基本类元数据）*。
	AnnotationMetadata getMetadata();
}
```



* 不需要加载类, 暴露这个bean的注解元数据，`AnnotationMetadata`

```java
// 类型过滤器
@FunctionalInterface
public interface TypeFilter {

	// 是否匹配
	boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)
			throws IOException;

}
```

`ConditionEvaluator` 

* 用于评估{@link Conditional}注释的内部类。

`java.lang.Object#getClass`

* {@code Class}对象，表示此对象的runtime *类。
* 每一个对象，都能获取他的Class对象。

`java.lang.Class#getClassLoader`

* 每一个Class对象，都能获取他的类加载器。

二维数组的Class对象名称（一维与二维数组，名称是不一样的），并且，结尾是否分号的。

* **[[Ljava.lang.String;**

* ```java
  // 基本类型：Class对象的名称。
  Class<Boolean> a = boolean.class;
  Class<Boolean> b = Boolean.class;
  System.out.println(a.getName());// boolean
  System.out.println(b.getName());// java.lang.Boolean
  ```

`org.springframework.asm.ClassReader#accept(org.springframework.asm.ClassVisitor, int)`

```java
// 使此ClassVisitor获得访问class文件的能力。有几种不同类型的class文件。
// 将访问者的能力，传递给ClassReader对象。
// 方法入参，使用final修饰入参？？
public void accept(final ClassVisitor classVisitor, final int parsingOptions) {
    accept(classVisitor, new Attribute[0], parsingOptions);
  }
```

```java
// 多值Map。
package org.springframework.util;
public interface MultiValueMap<K, V> extends Map<K, List<V>> {}
```

```java
// 附属，直接继承Map。获取map的增删改查的功能。存储的功能。
public class AnnotationAttributes extends LinkedHashMap<String, Object> {}
```



## @Component组件流程：

1. 扫描类路径下的所有组件（class文件资源的加载，asm框架），返回集合：Set<BeanDefinition>
2. 过滤，所有含有@Component注解的BeanDefinition。注解是递归扫描。
3. java字节码框架ASM。ASM框架是一个致力于字节码操作和分析的框架，它可以用来修改一个已存在的类或者动态产生一个新的类。3个核心类：
   1. ClassReader:该类用来解析编译过的class字节码文件。
   2. ClassWriter:该类用来重新构建编译后的类，比如说修改类名、属性以及方法，甚至可以生成新的类的字节码文件。
   3. ClassAdapter:该类也实现了ClassVisitor接口，它将对它的方法调用委托给另一个ClassVisitor对象。