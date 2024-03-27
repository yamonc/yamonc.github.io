---
title: IOC高级-Bean与BeanDefinition
date: 2023-07-13 11:27:09
tags: [Java, Spring]
categories: [Spring]
recommend: true
locate: [天津]
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202307131125914.jpeg
comment: 是
keywords: 
---
# IOC高级-Bean与BeanDefinition

![a man standing in the middle of a desert](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202307131125914.jpeg)

## BeanDefinition概述

`BeanDefinition` 也是一种**配置元信息**，它描述了 **Bean 的定义信息**。

>bean 的定义信息可以包含许多配置信息，包括构造函数参数，属性值和特定于容器的信息，例如初始化方法，静态工厂方法名称等。子 bean 定义可以从父 bean 定义继承配置数据。子 bean 的定义信息可以覆盖某些值，或者可以根据需要添加其他值。使用父 bean 和子 bean 的定义可以节省很多输入

bean 的定义信息也是有**层次性**的（联想 `BeanFactory` 的层次性），bean 的定义信息可以继承自某个已经有的定义信息，并覆盖父信息的一些配置值。

## BeanDefinition接口的方法定义

`BeanDefinition` 整体包含以下几个部分：

- Bean 的类信息 - 全限定类名 ( beanClassName )
- Bean 的属性 - 作用域 ( scope ) 、是否默认 Bean ( primary ) 、描述信息 ( description ) 等
- Bean 的行为特征 - 是否延迟加载 ( lazy ) 、是否自动注入 ( autowireCandidate ) 、初始化 / 销毁方法 ( initMethod / destroyMethod ) 等
- Bean 与其他 Bean 的关系 - 父 Bean 名 ( parentName ) 、依赖的 Bean ( dependsOn ) 等
- Bean 的配置属性 - 构造器参数 ( constructorArgumentValues ) 、属性变量值 ( propertyValues ) 等

由此可见，`BeanDefinition` 几乎把 bean 的所有信息都能收集并封装起来，可以说是很全面了。

## :star::star::star::star::star:面试题：概述BeanDefinition

**`BeanDefinition` 描述了 SpringFramework 中 bean 的元信息，它包含 bean 的类信息、属性、行为、依赖关系、配置信息等。`BeanDefinition` 具有层次性，并且可以在 IOC 容器初始化阶段被 `BeanDefinitionRegistryPostProcessor` 构造和注册，被 `BeanFactoryPostProcessor` 拦截修改等。**

## BeanDefinition的结构

![img](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202307131009380.webp)

### AttributeAccessor

**属性的访问器**：定义用于将元数据附加到任意对象，或从任意对象访问元数据的通用协定的接口。

总结出第一个 `BeanDefinition` 的特征：**`BeanDefinition` 继承了 `AttributeAccessor` 接口，具有配置 bean 属性的功能。**

### BeanMetadataElement

**存放了 bean 的元信息**， 只有一个方法，是获取 bean 的资源来源

```java
public interface BeanMetadataElement {
    default Object getSource() {
        return null;
    }
}
```

资源来源，说白了，就是 bean 的文件 / url 路径。咱们前面写的所有示例，都是在本地磁盘上的 .class 文件加载进来的，所以对应的也就应该是 `FileSystemResource`

### AbstractBeanDefinition

到了 `BeanDefinition` 的第一个实现类了，作为 `BeanDefinition` 的抽象实现，它里面已经定义好了一些属性和功能（大部分都有了），大体包含以下内容：（只挑选部分重要属性）：

```java
    // bean的全限定类名
    private volatile Object beanClass;

    // 默认的作用域为单实例
    private String scope = SCOPE_DEFAULT;

    // 默认bean都不是抽象的
    private boolean abstractFlag = false;

    // 是否延迟初始化
    private Boolean lazyInit;
    
    // 自动注入模式(默认不自动注入)
    private int autowireMode = AUTOWIRE_NO;

    // 是否参与IOC容器的自动注入(设置为false则它不会注入到其他bean，但其他bean可以注入到它本身)
    // 可以这样理解：设置为false后，你们不要来找我，但我可以去找你们
    private boolean autowireCandidate = true;

    // 同类型的首选bean
    private boolean primary = false;

    // bean的构造器参数和参数值列表
    private ConstructorArgumentValues constructorArgumentValues;

    // bean的属性和属性值集合
    private MutablePropertyValues propertyValues;

    // bean的初始化方法
    private String initMethodName;

    // bean的销毁方法
    private String destroyMethodName;

    // bean的资源来源
    private Resource resource;

```

### GenericBeanDefinition

又看到 `Generic` 了，它代表着通用、一般的，所以这种 `BeanDefinition` 也具有一般性。`GenericBeanDefinition` 的源码实现非常简单，仅仅是比 `AbstractBeanDefinition` 多了一个 `parentName` 属性而已。

由这个设计，可以得出以下几个结论：

- `AbstractBeanDefinition` 已经完全可以构成 `BeanDefinition` 的实现了
- `GenericBeanDefinition` 就是 `AbstractBeanDefinition` 的非抽象扩展而已
- `GenericBeanDefinition` 具有层次性（可从父 `BeanDefinition` 处继承一些属性信息）

### RootBeanDefinition与ChildBeanDefinition

**root** 和 **child** ，很明显这是父子关系的意思了呀。对于 `ChildBeanDefinition` ，它的设计实现与 `GenericBeanDefinition` 如出一辙，都是集成一个 `parentName` 来作为父 `BeanDefinition` 的 “指向引用” 。不过有一点要注意， `ChildBeanDefinition` 没有默认的无参构造器，必须要传入 `parentName` 才可以，但 `GenericBeanDefinition` 则有两种不同的构造器。

### AnnotatedBeanDefinition

它可以把 Bean 上的注解信息提供出来

## 基于@Component的BeanDefinition

给 `Person` 上打一个 `@Component` 注解，然后使用 `AnnotationConfigApplicationContext` 来驱动扫描 `Person` 类

```java
public class BeanDefinitionQuickstartComponentApplication {
    
    public static void main(String[] args) throws Exception {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(
                "com.linkedbear.spring.definition.a_quickstart.bean");
        BeanDefinition personBeanDefinition = ctx.getBeanDefinition("person");
        System.out.println(personBeanDefinition);
        System.out.println(personBeanDefinition.getClass().getName());
    }
}

```

**基于 xml 解析出来的 bean ，定义来源是 xml 配置文件；基于 `@Component` 注解解析出来的 bean ，定义来源是类的 .class 文件中。**

## 基于@Bean的BeanDefinition

编写一个配置类 `BeanDefinitionQuickstartConfiguration` ，使用 `@Bean` 注册一个 Person ：

```java
java复制代码@Configuration
public class BeanDefinitionQuickstartConfiguration {
    
    @Bean
    public Person person() {
        return new Person();
    }
}
```

之后，使用这个配置类驱动 IOC 容器，并直接获取 `BeanDefinition` ：

```java
java复制代码public class BeanDefinitionQuickstartBeanApplication {
    
    public static void main(String[] args) throws Exception {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(
                BeanDefinitionQuickstartConfiguration.class);
        BeanDefinition personBeanDefinition = ctx.getBeanDefinition("person");
        System.out.println(personBeanDefinition);
        System.out.println(personBeanDefinition.getClass().getName());
    }
}
```

具体区别可以发现有这么几个：

- Bean 的类型是 Root bean （ `ConfigurationClassBeanDefinition` 继承自 `RootBeanDefinition` ）
- Bean 的 className 不见了
- 自动注入模式为 `AUTOWIRE_CONSTRUCTOR` （构造器自动注入）
- 有 factoryBean 了：person 由 `beanDefinitionQuickstartConfiguration` 的 `person` 方法创建

## :star::star::star:【原理】BeanDefinition是如何生成的（简易理解）

1. 通过 xml 加载的 `BeanDefinition` ，它的读取工具是 `XmlBeanDefinitionReader` ，它会解析 xml 配置文件，最终来到 `DefaultBeanDefinitionDocumentReader` 的 `doRegisterBeanDefinitions` 方法，根据 xml 配置文件中的 bean 定义构造 `BeanDefinition` ，最底层创建 `BeanDefinition` 的位置在 `org.springframework.beans.factory.support.BeanDefinitionReaderUtils#createBeanDefinition` 。
2. 通过模式注解 + 组件扫描的方式构造的 `BeanDefinition` ，它的扫描工具是 `ClassPathBeanDefinitionScanner` ，它会扫描指定包路径下包含特定模式注解的类，核心工作方法是 `doScan` 方法，它会调用到父类 `ClassPathScanningCandidateComponentProvider` 的 `findCandidateComponents` 方法，创建 `ScannedGenericBeanDefinition` 并返回。
3. 通过配置类 + `@Bean` 注解的方式构造的 `BeanDefinition` 最复杂，它涉及到配置类的解析。配置类的解析要追踪到 `ConfigurationClassPostProcessor` 的 `processConfigBeanDefinitions` 方法，它会处理配置类，并交给 `ConfigurationClassParser` 来解析配置类，取出所有标注了 `@Bean` 的方法。随后，这些方法又被 `ConfigurationClassBeanDefinitionReader` 解析，最终在底层创建 `ConfigurationClassBeanDefinition` 并返回。
