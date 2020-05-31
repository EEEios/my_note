Version 5.2.6.RELEASE

# 一、核心技术

## 1.1. IOC / DI 和 Beans
org.springframework.beans 和 org.springframework.context.packages是spring IOC容器的基础。

BeanFactory接口提供呢能够管理任何类型对象的高级配置机制

ApplicationContext是BeanFactory的子接口，增加了：
- 消息资源处理（用于国际化）
- 时间发布
- 应用程序层特定上下文

Spring中，构成应用程序主干并由Spring IOC容器管理的对象称为bean。 

## 1.2. 容器总览
org.springframework.context.ApplicationContext接口表示Spring IOC容器，负责通过读取配置文件的元数据来实例化、配置、组装bean。
> 配置数据用xml、java注解、java代码表示。

ClassPathXmlApplicationContext 或 FileSystemXmlApplicationContext 是ApplicationContext接口的实现，通过创建Context可以读取xml进行bean的元数据的配置。

下图是spring中的配置过程。

![](http://img.doze9097.top//20200529125944.png)

 ### 1.2.1. 配置中的元数据

通过 配置中的元数据 告诉Spring如何实例化、配置和组装一个对象。

spring容器本身 和 元数据的编写 完全分离

> 文档用XML作为介绍，元数据的配置还可以使用：
>
> 1. 注解（Annotation-based configuration）
> 2. Java配置类（Java-based configuration）：通常需要使用 @Configuration，@Bean，@Import， @DependsOn 注解



XML配置样例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- id: 标识单独的bean -->
        <!-- class: 说明bean的对象类型，需要用完全限定名称 -->
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```



在配置文件中引入其他配置文件

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

### 1.2.2. 实例化容器

实例化语句：

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

services.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        
        <!-- name：当前bean的属性名称 -->
        <!-- ref：引用的bean的名称 -->
        
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

daos.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```

### 1.2.3. 使用容器

ApplicationContext是一个维护不同beans和它们以来的注册表的高级工厂

通过 `T getBean(String name, Class<T> requiredType)`可以获取实例化的bean

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```



还可以通过`GenericApplicationContext`结合reader delegates进行使用

```java
enericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```



## 1.3. Bean总览

bean通常包含一下数据：

- 包限定类名
- bean行为配置：例如scope，lifecycle，callbacks
- bean的依赖项
- 新创建的对象中的其他配置



如下表：

| Property                 | Explained in…                                                |
| :----------------------- | :----------------------------------------------------------- |
| Class                    | [Instantiating Beans](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-class) |
| Name                     | [Naming Beans](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-beanname) |
| Scope                    | [Bean Scopes](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-scopes) |
| Constructor arguments    | [Dependency Injection](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-collaborators) |
| Properties               | [Dependency Injection](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-collaborators) |
| Autowiring mode          | [Autowiring Collaborators](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-autowire) |
| Lazy initialization mode | [Lazy-initialized Beans](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-lazy-init) |
| Initialization method    | [Initialization Callbacks](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean) |
| Destruction method       | [Destruction Callbacks](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean) |

Application还允许用户创建容器外的bean。

这是通过`getBeanFactory()`方法访问ApplicationContext的BeanFactory来完成的，该方法返回BeanFactory的`DefaultListableBeanFactory`实现。

`DefaultListableBeanFactory`通过`registerSingleton(..)`和`registerBeanDefinition(..)`方法支持这种注册。

### 1.3.1. Bean的命名

bean可以拥有多个标识，通过`id` 和 `name`进行定义。

bean通常只有一个标识，如果有多个标识则会作为bean的别名。

`id`：定义bean的标识，只有一个

`name`：定义bean的别名，多个可以通过 逗号、引号、空格进行分隔

> bean id的唯一性由容器执行，不再由XML解析器执行



如果不提供命名，spring将会给bean自动命名。不命名的bean通常作为 inner bean 和 autowriting collaborators

> 自动命名规则：
>
> - 类名开头只有一个大写字母，则首字母小写
> - 其他情况保持不变
>
> 详见 `java.beans.Introspector.decapitalize`



#### 在bean定义外给bean增加别名

在xml配置中，可以使用如下语句赋予别名

```xml
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```

在Java配置类中，可以使用 `@Bean` 指定别名



### 1.3.2. 实例化Beans

- 通常容器会反射调用bean对应calss的构造方法
- 当请求一个拥有静态工厂方法的类的对象时，在少数情况下，容器会请求静态工厂方法创建bean。从静态工厂方法中返回的bean可能是同一类或完全不同。

#### （1）使用容器进行实例化

取决于指定bean的ioc类型，可能需要为bean创建一个无参构造方法。

#### （2）使用静态工厂方法实例化

需要通过`<bean>`中的`factory-method`指定类的内部静态工厂方法。如下:

```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

#### （3）使用实例化工厂方法实例化

需要使用`factory-bean`指定 工厂bean ，并且使用`factory-method`指定方法

```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

### 1.3.2.1 确定bean运行时的类型

元数据中定义的类只是初始引用，可能与声明的工厂方法结合使用或者作为一个FactoryBean类，这可能导致Bean运行时的类型有所不同，甚至在实例工厂的方法中没有被设置。

获得Bean运行时的类型的推荐方法是使用 `BeanFactory.getType` 并传入指定的Bean名称。



## 1.4 依赖

### 1.4.1. 依赖注入（Dependency Injection/DI）

依赖注入是 对象通过构造方法参数、方法工厂的参数、在实例化或从方法工厂返回时的设置定义依赖。在容器创建bean时，会将依赖注入。这个过程完全是逆向的（即控制反转[IOC]），是由bean自身通过定向使用构造方法或服务类定位方式去实例化或定位它的依赖。

#### 1.4.1.1. 基于构造器的DI

基于构造器的DI由构造器请求调用构造方法并传入一些参数实现，每个参数对应一个依赖。这种方法和在静态工厂方法中传入参数几乎一样，示例代码如下：

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

**构造器参数解析**

如果在bean定义的构造函数参数中不存在歧义，则构造函数参数的顺序与在实例化bean时提供这些参数的顺序相同。并且在配置中不用指定索引，如下：

```java
package x.y;

public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```

```xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>

    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```



如果是简单类型，spring不能识别值的类型，例如`<value>true</value>`

用下方的类作为示例：

```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```



**1. 通过类型进行匹配**

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```



**2. 通过索引进行匹配**

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```



**3. 通过参数名称进行匹配**

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

使用这种形式，需要在启用调试标记的模式下编译。

不启用可以在对应类中，添加`@ConstructorProperties`注解并指明参数名称，如下：

```java
package examples;

public class ExampleBean {

    // Fields omitted
		
    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```



#### 1.4.1.2. 基于setter的依赖注入

该方式支持在通过构造方法传入依赖之后，继续进行依赖注入。

可以以`BeanDefinition`的形式配置依赖，将其与`PropertiesEditor`实例共同使用进行属性值形式的切换。

但是，大多数Spring用户并不直接使用这些类(也就是说，编程化的)，而是使用XML bean定义、带注解的组件(即使用@Component、@Controller等注解)，或基于java的@Configuration类中的@Bean方法。然后，在内部将这些类转换为BeanDefinition的实例，并用于加载整个Spring IoC容器实例。



#### 1.4.1.3. 基于构造器？基于setter？

基于构造器（Spring团队推荐）：

​	必要/强制性的依赖。

​	对参数的程序校验（programmatic validation）更好

​	可以确保运行的应用组件是不可改变的对象并确保必要的依赖不为null

基于setter：

​	可选的依赖。

​	通过在setter上标注`@Required`注解表明一个必要的依赖。

​	以后可以重新配置或重新注入



#### 1.4.1.4. 依赖的解析过程

容器对bean执行依赖解析的步骤如下：

1. ApplicationContext通过配置的所有bean的元数据进行初始化。配置形式可以是XMl，java代码，注解
2. 对于每一个bean，它的依赖关系用属性，构造器参数或者静态工厂中的方法的参数这些形式表示。这些依赖将会在bean创建时传入。
3. 每个属性或构造器参数是一个待设置值的定义（definition），或者一个容器中其他bean的引用
4. 每个属性或构造器参数都是一个值从指定格式转换成属性或构造器参数的实际类型(actual type)。默认情况下，Spring可以将以字符串格式提供的值转换为所有内置类型，如int、long、string、boolean等。

spring容器会对每个创建的bean的配置进行校验。然而，bean的属性在bean没有被创建前是不会被设置的。在创建容器时，将创建单例作用域的bean并将其设置为预实例化(缺省值)。作用域是在Bean中定义的。换言之，bean只有在被使用时才会被创建。创建一个bean可能会使与之关联的bean也被创建，因为会创建并分配bean和其依赖项的依赖项。需要注意的是，这些依赖项之间的解析不匹配可能会在第一次创建相关bean的时候出现。

> **循环依赖**
>
> 如果主要使用构造器注入，可能会出现无法解析的循环依赖。
>
> 例如：A类需要通过构造器注入B类实例，同时，B类需要通过构造器注入A类。如果A类和B类互被互相注入，Spring IOC容器会阻止循环依赖并，并抛出一个`BeanCurrentlyInCreationException`异常

你可以完全相信Sprng。他会阻止配置问题，比如在容器载入时候引用一个不存在的beans和循环依赖。Spring会在完全被创建时尽可能玩设置属性和解析依赖。这意味着已经正确加载的spring容器以后当你请求一个对象时，如果在创建该对象或它的一个依赖项时出现问题可以生成异常。例如，bean在有确实或无效的属性时会抛出异常。一些配置项的潜在延迟可见性（potentially delayed visibility）是Applicatioin在默认情况下预实例化单例bean的原因。在实际需要这边bean之前，需要花费一些前期时间和内存，你可以在创建ApplicationContext时及时发现配置问题。你可以修改这个默认行为，这样就可以懒加载单例bean，而不是预先实例化。

如果不存在循环依赖，且有一个或多个**协作**（collaboration）的bean被注入到一个依赖bean，每一个**协作**的bean会在它完全配置后被注入依赖的bean。这意味着，如果A依赖于B，则spring IOC 容器会在请求A的setter时完全配置好B，换言之，这个bean会被实例化（如果不是一个预实例化的单例），它的依赖项会被设置，并且会调用相关的生命周期方法（例如配置的init方法或InitializingBean的回调方法）。

#### 1.4.1.5. 依赖注入例子

以下是基于XML的数据配置和基于setter的DI（依赖注入）。

一小部分的XML配置指明了bean的定义如下：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

下方为相关联的`ExampleBean`类:

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    // setter ------------------------------------------
    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```

在前面的例子中，声明了setter去匹配xml文件中特定的属性。



以下是例子使用了基于构造器的DI：

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下为对应的`ExampleBean`类：

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    //构造器
    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```

bean中指定的构造函数参数用作ExampleBean构造函数的参数。



spring可以通过调用静态工厂方法替代构造器实例化一个对象：

```xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下为对应的`ExampleBean`类：

```java
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
}
```

静态工厂方法的参数由`<constructor-arg>`的元素提供，正如使用构造器那样。工厂方法返回的类不必和静态工厂方法所属的类相同（虽然示例中是）。一个示例（非静态）工厂方法本质上形式相同（除了用`factory-bean`属性替代`class`属性），所以我们不在这讨论其细节。



### 1.4.2. 依赖和配置中的细节

如上节所述，你可以定义bean属性和构造器参数作为引用去管理其他beans（collaborators）或

作为内联定义的值。为实现该目标，spring基于xml的元数据配置可以通过`<property/>`和`<constructor-arg>`元素支持子元素。

#### 1.4.2.1. 直接值(Straght Values)（Primitives，Strings，and so on）

`<property/>`中的value属性指明了易于读者阅读形式的属性或构造器参数。Spring的转换服务（conversion service）被用于将这些值从String转换成属性或参数的实际类型。下面是配置了不同类型的值的例子：

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="masterkaoli"/>
</bean>
```

下面的示例使用`p-namespace`进行更简洁的XML配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- p:属性名称="[对应值]" -->
    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="masterkaoli"/>
</beans>
```

以上的形式虽然更简洁，但是拼写错误将会在运行时而不是编写时被发现，除非使用支持定义bean时自动填充属性的IDE（诸如Intellij IDEA 或者 Spring Tool for Eclipse）

你可以用如下方式配置一个`java.util,Properties`实例如下：

```xml
<bean id="mappings"
    class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```

spring容器会用JavaBeans`PropertyEditor`机制将`<value/>`中的文本转换成`java.util.Properties`实例。这是一个不错的方法，并且也是spring团队更喜欢使用嵌套的`<value/>`而不是value属性样式的几个地方之一。

**idref 元素**

`idref`元素只是一种简单的防错(error-proof)方法，可以将容器中另一个bean的id（字符串值-不是引用）传递给`<constructor-arg>`或者`<property/>`元素。如下例：

```xml
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```

上面的例子等价于：

```xml
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```

第一种形式会比第二种更好，因为使用`<idfer>`可以让容器在部署的时候校验引用的bean是否存在。在第二种形式中，通过`targetName`属性传递给`client`Bean的值没有经过校验。拼写错误会在`client`Bean创建时才会被发现（很可能是致命错误）。如果`client`Bean是一个原型(prototype)Bean，那么产生的错误和异常只有可能在部署容器之后很久才会被发现。

> `<idref>`中的`local`属性在4.0 beans XSD中不再被支持。当升级到4.0时，请将`idref local`修改为`idref bean`

一个`<idref/>`元素体现价值的地方是在一个`ProxyFactoryBean`bean配置AOP拦截器时，能够防止拦截器ID(interceptor ID)的拼写错误。



#### 1.4.2.2. 引用其他beans (协作者[Collaborators])

`ref`是`<constructor-arg/>`或`<property>`中最后一个定义的元素。在此，你可以设置一个bean的指定属性的值，被其他容器所管理的其他bean（协作者）引用。被引用的bean是对应属性被设置的bean的依赖项，并且被引用的bean在被使用前需要被初始化。（如果协作者是一个单例bean，它可能已经被容器初始化了）。所有引用最终都是对其他对象的引用。作用域和验证取决于你是否同通过`bean`或者`parent`指明其他对象的ID或name。

通过`bean`属性的`<ref />`指定目标bean是最通用的形式，同时允许了同一容器或父容器中创建对任何bean的引用，不管是否在同一个xml文件中。`bean`中的值可以与`id`属性或者`name`属性其一相同。下面为使用`ref`的例子：

```xml
<ref bean="someBean"/>
```

通过`parent`属性知名一个目标bean创建了一个对当前容器的父容器的bean的引用。`parent`属性中的值可以和目标bean的`id`属性或者`name`属性其一相同。目标bean必须在父容器中。当使用一个容器层次结构并且想将一个父容器中的一个bean作为与parent bean的相同name的代理时，应该主要使用这种类型的引用。下面的两个清单显示了如何使用parent属性：

```xml
<!-- in the parent context -->
<bean id="accountService" class="com.something.SimpleAccountService">
    <!-- insert dependencies as required as here -->
</bean>
```

```xml
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> 
        <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

#### 1.4.2.3. 内部bean

内部bean是一个在`<property/>`或者`<constructor-arg/>`中定义的`<bean/>`元素，如下：

```xml
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

一个内部bean不一定要定义ID或者name。如果制定了，容器也不会将其作为一个标识。容器在创建时还会忽略其作用域标志，因为内部bean总是匿名的，也总是由外部bean创建的。单独访问内部bean，或者将内部bean注入到协作bean而不是封装在bean中是不可能的。

作为一个特例，有可能收到一个从自定义作用域的销毁回调（destructioin callback），例如，用于单例的bean中包含请求（request-scope）作用域的内部bean。内部类的创建与其包含的bean绑定到一起，当时销毁回调让它加入到了请求域的生命周期。这不是常见的情况。内部类通常仅它们所包含的bean的作用域。

#### 1.4.2.4. 集合 （Collection）

`<list/>`，`<set/>`，`<map/>`，`<props/>`分别对应Java`Collection`中的List，Set，Map和Properties。如下：

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

map中的key和value，或set的value，可以是以下任何类型：

```xml
bean | ref | idref | list | set | map | props | value | null
```

**集合的合并**

spring容器也支持合并集合。一个应用程序开发认源可以定义一个父级的`<list/>`，`<map/>`，`<set/>`或者`<props/>`元素和继承并覆写父元素的子级的`<list/>`，`<map/>`，`<set/>`或者`<props/>`元素。也就是说，子集合的值是父级和子级集合通过子集合元素覆写指定父集合元素进行合并的结果。

这个关于合并的小节将讨论父子bean的机制（the parent-child bean mechanism）。不熟悉父级和子级bean定义的读者可能希望在继续之前先阅读[相关章节](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-child-bean-definitions) 。

下面的示例演示集合合并：

```xml
<beans>
    
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    
    <bean id="child" parent="parent">
        <property name="adminEmails">
        <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```

留意`child` bean的`adminEmails` property的`<props/>`中的`merge = true`属性。当`child` bean被容器解析并实例化时，生成的bean具有一个包含了子级的`adminEmails`集合合并父级的`adminEmails`的结果的`adminEmails` `Properties`集合。下面的列表展示了结果：

```yml
administrator=administrator@example.com //父级的
sales=sales@example.com	//子级的
support=support@example.co.uk //同id子级的覆盖了父级的
```

子级的`Properties`集合继承了所有父级的`<props/>`元素，并且子级的`support`的值覆盖了父级的`support`的值。

这个合并操作类似地适用于`<list/>`，`<map/>`和`<set/>`集合类型。在`<list/>`这种有序集合下，列表集合类型中的语义（ the semantics associated with the `List` collection）会被保持，父元素的值先于所有子元素。在例如`Map`，`Set`和`Properties`集合类型中，不存在顺序。因此，容器内部使用的`Map`，`Set`，`Properties`没有排序相关的语义。

**集合合并的限制**

你不能合并不同的集合类型（例如一个`Map`和一个`List`）。如果你仍这样使用，会抛出一个适当的异常。`merge`属性必须在更低级的、继承的、子级的定义中。在父级集合中指定`merge`属性时冗余的

，并且不会实现所要的合并。

**强类型集合**

随着Java 5中泛型的引入，你可以使用强类型集合。也就是说，可以通过定义一个仅包含（例如）`String`元素的`Collection`类型。如果你使用Spring将一个强类型集合（依赖）注入一个bean中，

就会利用到spring中类型转换（type-convert）的支持，例如一个强类型的元素实例会在被加入到集合前被转换成正确的类型。下面的类和bean定义展示了其实现：

```java
public class SomeClass {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```

```xml
<beans>
    <bean id="something" class="x.y.SomeClass">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```

当准备注入`something` bean的`accounts`属性时，可以通过反射获取有关强类型`Map<String, Float>`的元素类型的泛型信息。因此，spring的类型转换功能可以识别各种值元素是`Float`，并将字符串的值（`9,99` ,`2.75`）转换成真正的Float类型

**Null 和 空字符串**

sprin将空参数和类型的东西看作空字符串。下面基于xml的元数据配置片段将email属性设置为空字符串（"")

```xml
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

上面的例子等价于下面的代码：

```java
exampleBean.setEmail("");
```

`<null/>`元素负责处理`null`值。下面的用列表作为例子进行展示：

```xml
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

#### 1.4.2.5. 用p-namespace便捷地配置XML

可以使用p-namespace代替`<bean>`中的`<property/>`去描述协作bean（collaborating beans)的属性值，也可以两者都用。

spring支持 [通过命名空间（namespaces）](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#xsd-schemas)（namespaces）进行扩展配置，命名空间基于xml模式定义。本章讨论的bean配置格式是在XML模式文档中定义的。然而，p-namespace没有在XSD文件中定义，只存在于spring核心。

下面展示两个等价的XML片段（第一个是使用了标准的xml格式，第二个使用了p-namespaces）：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 标准格式 -->
    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="someone@somewhere.com"/>
    </bean>

    <!-- p-namespace格式 -->
    <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="someone@somewhere.com"/>
</beans>
```

这个例子展示了一个bean定义中名为`email`的p-namespace属性。这会告诉spring其中包含一个属性声明。如先前提到的，p-namespace并非一个模式定义（schema definition），因此你可以将属性的名称（property name）作为一个属性名（attribute name）。

下面这个例子包含了更多bean定义，并且有bean的引用：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="john-classic" class="com.example.Person">
        <property name="name" value="John Doe"/>
        <property name="spouse" ref="jane"/>
    </bean>

    <bean name="john-modern"
        class="com.example.Person"
        p:name="John Doe"
        p:spouse-ref="jane"/>

    <bean name="jane" class="com.example.Person">
        <property name="name" value="Jane Doe"/>
    </bean>
</beans>
```

上述例子不仅包含了使用p-namespace声明的属性值，同时用特殊格式声明了属性引用。不论是第一个bean定义中用`<property name="spouse" ref="jane" />`去创建`john`对`jane`的引用，还是第二个bean定义中用属性`p:spouse-ref="jane"`，都达到了同样的效果。在这个例子中，`soupse`是属性名称，`-ref`说明其值不是一个直接值（straight value），而是一个对bean的引用。

> 其实p-namespace并不如标准xml格式那样灵活。例如，p-namespace生命引用回和以`Ref`结尾的属性产生冲突，而标准的XML不会。我们推荐您和团队进行沟通并谨慎选择使用方法，避免出现同时使用这三种方法编写XML文档的情况。

**用c-namespace便捷地配置XML**

和p-namespace配置类似，c-namespace在Spring3.1发布，其允许使用内部的属而不是嵌套的`construtor-arg`元素配置构造器参数。

下面的例子等价于[基于构造器的依赖注入](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-constructor-injection):

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>

    <!-- traditional declaration with optional argument names -->
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg name="thingTwo" ref="beanTwo"/>
        <constructor-arg name="thingThree" ref="beanThree"/>
        <constructor-arg name="email" value="something@somewhere.com"/>
    </bean>

    <!-- c-namespace declaration with argument names -->
    <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
        c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>

</beans>
```

`c:namespace`的使用方式和`p:namespace`类似（结尾的`-ref`表示对bean的引用），主要用他们的名称去设置构造器的参数。`c:namespace`没有在XSD中定义，存在于spring内核。

在构造器参数名称不可用的情况下会出现一些少见的情况（通常在字节码在没有调试信息的情况下编译），你可以退而求其次，使用索引，如下例：

```xml
<!-- c-namespace index declaration -->
<bean id="beanOne" class="x.y.ThingOne" c:_0-ref="beanTwo" c:_1-ref="beanThree"
    c:_2="something@somewhere.com"/>
```

> 由于xml的语法，索引的表达需要在前面加"_"，因为XML属性名称不能以数字开头（尽管IDE允许这样做）。在`<constructor-arg>`中也可以使用对应的索引符号，但是不常用，因为准确的声明顺序就可以表示了。

事实上，构造器解析[机制](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-ctor-arguments-resolution)在参数匹配上是非常有效率的，除非非常重要，我们推荐您在配置中使用

`c-namespace`。

**混合属性名称**

在设置bean的属性时，只要路径中的所有组件（出了最后的属性名称）都不为null，你可以使用混合或者嵌套的属性名称，如下：

```xml
<bean id="something" class="things.ThingOne">
    <property name="fred.bob.sammy" value="123" />
</bean>
```

`something` bean有一个有sammy属性的bob属性的fred属性，并且最后的属性`sammy`的值为`123`，只有在创建后`fred`和`bob`属性不为null的情况下，`sammy`才能工作，否则会抛出一个`NullPointerException`异常。



#### 1.4.3. 使用 depends-on



