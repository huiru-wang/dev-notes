
# Spring生态

**Spring**：是一个轻量级的Web框架；提供了Web开发所需要的各种模块：SpringCore、SpringMVC、SpringJDBC、Transaction事务管理、AOP等；

1、消除了繁琐的XML配置、可以完全不配置XML；通过引用各种starter简化配置，并提供默认配置及自动装配；
- starter：**基于约定大于配置思想**，使用spi机制及自动装配原理，可以将一些通用的功能能够封装成一个独立组件并很方便的集成到不同的项目里面，简化开发，提升代码复用能力

2、提供嵌入式的web容器；高效部署；使用java命令直接运行jar包启动；

# IOC

传统开发，开发者自己创建对象、管理对象；

Spring的IOC容器可以统一管理对象，对象的创建、管理、销毁等整个生命周期交给Spring；

思想是：工程模式 + 反射机制；

工作方式：Spring在启动时创建容器（本质是ConcurrentHashMap）来存储所有的Bean，根据开发者的注解或XML配置加载Java对象，并向这些Bean注入配置好的依赖属性，减少开发者自己创建对象、维护对象间依赖关系。

## Bean的声明

首先开发者需要向Spring提供**Bean的声明**；告知要将哪些Bean注入到Spring容器中；一般的方式有：
- xml声明
- 注解声明

Spring将这些声明加载并封装为：`BeanDefinition` 对象；

此对象定义了Bean的元数据信息、含有的依赖、bean的工程、初始化、销毁方法等等；
```java
private Boolean lazyInit;        // 是否懒加载
private String[] dependsOn;      // 依赖
private volatile Object beanClass; // beanClass对象
private String initMethodName;   // 初始化方法
private String destroyMethodName;// 销毁方法
private String factoryBeanName; // factory
```

## Bean的注入方式

1. Setter
2. 构造器

## Bean的实例化和初始化

入口：`doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)`

1. Bean的实例化
2. Bean的提前暴露
3. Bean的属性填充
4. Bean的初始化
	1. 前置处理
	2. 初始化
	3. 后置处理
## 1. 实例化

Spring按照`BeanDefinition`开始创建Bean对象

调用：`createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)`方法；

判断当前Bean是否有工厂方法、是否有指定的构造函数等；如果都没有，则使用反射通过无参构造函数创建Bean的空对象；

## 2. 提前暴露

```java
boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences && isSingletonCurrentlyInCreation(beanName));
```

## 3. 属性填充

调用：`populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw)`完成属性填充；

中间遇到依赖Bean，则
填充用户定义的依赖、属性，并扫描Bean实现的Aware接口，回调Aware方法；

如果Bean需要从Spring上下文中获取一些基础设施信息，实现对应的Aware：
- BeenNameAware：获取Beanname；
- ApplicationContextAware：如果Bean要获取ApplicationContext对象；
- BeanClassLoaderAware：获取Bean的类加载器；
- EnvironmentAware：获取环境配置信息等；

## 4. 初始化Bean

调用：`initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd)`

```java
protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
  // 调用bean实现的Aware方法
  invokeAwareMethods(beanName, bean);
  
  // 执行bean的前置处理(如果有)
  wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
  
  // 执行初始化方法(如果有)
  invokeInitMethods(beanName, wrappedBean, mbd);

  // 执行bean的后置处理(如果有，SpringAOP在这里执行)
  wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
  
  return wrappedBean;
}
```

### 4.1 前置处理

执行`BeanPostProcessor`中`postProcessBeforeInitialization()`

`PropertyPlaceholderConfigurer`：XML中的占位符，在前置处理中替换为值；

@PostConstruct在前置处理后执行；

### 4.2 初始化

1、执行实现 InitializingBean 接口的`afterPropertiesSet()`方法(钩子函数)

2、执行用户自定义的`initMethod`初始化方法；

### 4.3 后置处理
执行`BeanPostProcessor`中`postProcessAfterInitialization()`
- AnnotationAwareAspectJAutoProxyCreator在此处执行`wrapIfNecessary`完成代理对象的创建；




# AOP

本质是：创建新的对象，替换原有的对象，在执行原对象方法的前后，对其进行增强；

## 代理模式原理

```java
class A {
    B b;
    void function(){
        b.func();
    }
}
```

原始对象A，调用对象B的方法，如何能够不动原来的代码，用另一个`代理对象C`来替换掉`b对象`呢？

- 继承：`C继承B`，就能够把B对象替换成C对象，原来的代码不需要改；
- 接口：B是一个接口，`c`和`b`都实现了B接口，c可以通过多态替换掉b；

## 代理的方式

- 静态代理：AspectJ静态代理框架；编译器修改`.class`文件，完成替换；

- 动态代理：`.class`文件不改变，进行对象创建时，完成替换；

  - JDK动态代理：JDk原生代理方式；需要被代理类实现接口；因为JDK创建代理类需要继承Proxy，因为Java单继承继承的位置已经被占了，所以要求必须要被代理类实现接口；

  - Cglib动态代理；第三方代理；只需要类是可继承的，需要创建子类完成替换；

## 代理效率

静态代理：性能高，功能强，复杂度高；运行时不需要额外处理；

动态代理：性能差，但是简单易用；

