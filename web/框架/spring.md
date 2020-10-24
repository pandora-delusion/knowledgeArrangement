[TOC]

### Spring是什么？

Spring是一个轻量级的IoC和AOP容器框架。是为Java应用程序提供基础性服务的一套框架，目的是用于简化企业应用程序的开发，它使得开发者只需要关心业务需求。常见的配置方式有三种：基于XML的配置、基于注解的配置、基于Java的配置。

主要由以下几个模块组成：

Spring Core：核心类库，提供IOC服务；

Spring Context：提供框架式的Bean访问方式，以及企业级功能（JNDI、定时任务等）；

Spring AOP：AOP服务；

Spring DAO：对JDBC的抽象，简化了数据访问异常的处理；

Spring ORM：对现有的ORM框架的支持；

Spring Web：提供了基本的面向Web的综合特性，例如多方文件上传；

Spring MVC：提供面向Web应用的Model-View-Controller实现。

### Spring的优点

（1）spring属于低侵入式设计，代码的污染极低；

（2）spring的DI机制将对象之间的依赖关系交由框架处理，减低组件的耦合性；

（3）Spring提供了AOP技术，支持将一些通用任务，如安全、事务、日志、权限等进行集中式管理，从而提供更好的复用。

（4）spring对于主流的应用框架提供了集成支持

### Spring IoC的概念

Spring实现依赖注入的手段是控制反转（IoC）

核心理念是**IoC（控制反转）**和**AOP（面向切面编程）**

容纳Bean的是Spring所提供的IoC容器

控制反转是一种通过描述（在Java中可以是XML或者注解）并通过第三方去产生或获取特定对象的方式

有了IoC容器后，把创建和查找依赖对象的控制权交给了容器，由容器进行注入组合对象，所以对象与对象之间是松散耦合，这样也方便测试，利于功能复用，更重要的是使得程序的整个体系结构变得非常灵活

在Spring中实现控制反转的是IoC容器，其实现方法是依赖注入

Spring会提供IoC容器来管理对应的资源

控制反转的最大好处在于降低对象之间的耦合

基于降低开发难度，对模块解耦，同时也更加有利于测试的原则。Spring IoC理念在各种JavaEE开发者中广泛应用

#### Spring IoC容器  

SpringIoC容器的设计主要是基于BeanFactory和ApplicationContext两个接口，其中ApplicationContext是BeanFactory的子接口之一

```Java
public interface BeanFactory {
    String FACTORY_BEAN_PREFIX = "&";

    // 获取给配置Spring IoC容器的Bean
    Object getBean(String var1) throws BeansException;

    <T> T getBean(String var1, Class<T> var2) throws BeansException;

    <T> T getBean(Class<T> var1) throws BeansException;

    Object getBean(String var1, Object... var2) throws BeansException;

    <T> T getBean(Class<T> var1, Object... var2) throws BeansException;

    boolean containsBean(String var1);
	
    // 默认情况下，Spring会为Bean创建一个单例
    // 用于判断是否单例
    boolean isSingleton(String var1) throws NoSuchBeanDefinitionException;
	// 如果判断为真，就是当你从容器中获取一个Bean，容器就为你生成了一个新的实例
    boolean isPrototype(String var1) throws NoSuchBeanDefinitionException;

    boolean isTypeMatch(String var1, Class<?> var2) throws NoSuchBeanDefinitionException;

    Class<?> getType(String var1) throws NoSuchBeanDefinitionException;
	// 获取别名
    String[] getAliases(String var1);
}
```

#### SpringIoC容器的初始化和依赖注入

注意步骤：先定义，在初始化，后依赖注入

Bean的定义分为3步：

1. Resource定位，根据XML或者注解的方式进行资源定位
2. BeanDefinition的载入，将Resource定位到的信息保存到Bean定义（BeanDefinition）中去，此时不会创建Bean的实例
3. BeanDefinition的注册，将BeanDefinition的信息发布到Spring IoC容器中去，此时仍不会创建Bean实例

这样，Bean在SpringIoC容器中被定义了，而没有被初始化，更没有完成依赖注入，不能完全使用。

lazy-init是一个配置选项，为是否初始化Spring Bean。默认为false，就是Spring IoC会自动初始化Bean；若为true，当我们用Spring IoC容器的getBean方法获取它时，它才会进行Bean的初始化，完成依赖注入。

#### Spring Bean的生命周期

![](F:\mycode\knowledgeArrangement\web\框架\bean.jpg)

1. Spring启动，查找并加载需要被Spring管理的bean，进行Bean的实例化
2. Bean实例化后对将Bean的依赖和值注入到Bean的属性中
3. 如果Bean实现了BeanNameAware接口的话，Spring将Bean的Id传递给setBeanName()方法
4. 如果Bean实现了BeanFactoryAware接口的话，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入
5. 如果Bean实现了ApplicationContextAware接口的话，Spring将调用Bean的setApplicationContext()方法，将bean所在应用上下文引用传入进来。
6. 如果Bean实现了BeanPostProcessor接口，Spring就将调用他们的postProcessBeforeInitialization()方法。
7. 如果Bean 实现了InitializingBean接口，Spring将调用他们的afterPropertiesSet()方法。类似的，如果bean使用init-method声明了初始化方法，该方法也会被调用
8. 如果Bean 实现了BeanPostProcessor接口，Spring就将调用他们的postProcessAfterInitialization()方法。
9. 此时，Bean已经准备就绪，可以被应用程序使用了。他们将一直驻留在应用上下文中，直到应用上下文被销毁。
10. 如果bean实现了DisposableBean接口，Spring将调用它的destory()接口方法，同样，如果bean使用了destory-method 声明销毁方法，该方法也会被调用。

所有的Spring IoC容器最低的要求是实现BeanFactory接口，而非ApplicationContext接口。如果采用了非ApplicationContext子类创建Spring IoC容器，那么即使实现了ApplicationContextAware的setApplicationContext方法，它也不会在生命周期之中被调用。

BeanPostProcessor接口是针对所有的Bean而言的。

### 装配Spring Bean

#### 依赖注入的三种方式

##### 构造器注入

依赖构造方法实现，在这里spring是采用**反射**的方式，通过使用构造方法来完成注入。

```xml
<bean id="role1" class="com.ssm.chapter9.pojo.Role">
    <constructor-arg index="0" value="总经理" />
    <constructor-arg index="1" value="公司管理者" />
</bean>
```

constructor-arg元素用于定义类构造方法的参数 ，index是参数位置，value是设置值。如果参数过多，应该考虑setter注入

##### 使用setter注入

最主流的注入方式，它消除了使用构造器注入时出现多个参数的可能性。首先将构造方法声明为无参数的，然后使用setter注入为其设置对应的值，也是通过反射实现的。

```xml
<bean id="role2" class="com.ssm.chapter9.pojo.Role">
    <property name="roleName" value="高级工程师" />
    <property name="note" value="重要人员" />
</bean>
```

这样spring就会通过反射调用没有参数的构造方法生成对象，同时通过反射对应的setter注入配置的值了。

##### 接口注入

当资源并非来自自身系统，而是来自于外界（比如在Tomcat中配置的数据库连接资源），通过JNDI的形式来获取它。

如果Tomcat的Web工程使用了Spring，那么可以通过Spring的机制，用JNDI获取Tomcat启动的数据库连接池，代码如下：

```xml
<bean id="dataSource" class="org.springframework.jndi.JndiObjectFactoryBean">
    <property name="jndiName">
        <value>java:comp/env/jdbc/ssm</value>
    </property>
</bean>
```

#### 装配Bean概述

在大部分场景下，我们都使用ApplicationContext的具体实现类，因为对应的Spring IoC容器功能相对强大。在Spring中提供了三种方式进行配置：

1. 在XML中显式配置
2. 在Java的接口和类中实现配置
3. 隐式Bean的发现机制和自动装配原则

使用建议：

1. 基于约定优于配置的原则，最优先的应该是通过隐式Bean的发现机制和自动装配的原则。好处是减少程序开发者的决定权

2. 在无法使用自动装配原则的情况下优先考虑Java接口和类中实现配置，好处是避免XML配置的泛滥

3. 在上述方法都无法使用的情况下，选择XML配置Spring IoC容器。

   当配置的类是你自身正在开发的工程，应该考虑Java配置为主，而Java配置又分为自动装配和Bean名称配置。在没有歧义的基础上，优先使用自动装配，这样就可以减少大量的XML配置。如果所需配置的类并不是你的工程开发的，建议使用XML的方式。

#### 通过XML配置装配Bean

XML文件的配置如下：

```XML
<?xml version='1.0' encoding='UTF-8' ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:schemaLocation="http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans-4.0.xsd">
    <!--Spring Bean配置代码-->
</beans>
```

注意引入的XSD文件，它说定义的元素将可以定义对应的Spring Bean

```xml
<bean id="role2" class="com.ssm.chapter9.pojo.Role">
    <property name="id" value="1" />
    <property name="roleName" value="高级工程师" />
    <property name="note" value="重要人员" />
</bean>
<bean id="source" class="com.ssm.chapter9.pojo.Source">
    <property name="fruit" value="a" />
    <property name="sugar" value="b" />
    <property name="size" value="big" />
</bean>
<bean id="juiceMaker2" class="com.ssm.chapter9.pojo.JuiceMaker2">
    <property name="beverageShop" value="m" />
    <property name="source" ref="source" />
</bean>
```

id属性是Spring找到的这个Bean的编号，不一定必须。class属性是一个类全限定名。property元素是定义类的属性，name是属性名称，value是其值。ref属性去引用对应的Bean

```xml
<bean id="complexAssembly" class="com.ssm.chapter10.pojo.ComplexAssembly">
    <property name="id" value="1" />
    <property name="list">
        <list>
            <value>value-list-1</value>
            <value>value-list-2</value>
            <value>value-list-3</value>
        </list>
    </property>
    <property name="map">
        <map>
            <entry key="key1" value="value-key-1" />
            <entry key="key2" value="value-key-2" />
            <entry key="key3" value="value-key-3" />
        </map>
    </property>
    <property name="props">
        <props>
            <prop key="prop1">value-prop-1</prop>
            <prop key="prop2">value-prop-2</prop>
            <prop key="prop3">value-prop-3</prop>
        </props>
    </property>
    <property name="set">
        <set>
            <value>value-set-1</value>
            <value>value-set-2</value>
            <value>value-set-3</value>
        </set>
    </property>
    <!-- 数组对象 -->
    <property name="array">
        <array>
            <value>value-array-1</value>
            <value>value-array-2</value>
            <value>value-array-3</value>
        </array>
    </property>
</bean>

<bean id="userRoleAssembly" class="com.ssm.chapter10.pojo.UserRoleAssembly">
    <property name="id" value="1" />
    <property name="list">
        <list>
            <!-- 对象引用 -->
            <ref bean="role1" />
            <ref bean="role2" />
        </list>
    </property>
    <property name="map">
        <map>
            <entry key-ref="role1" value-ref="user1" />
            <entry key-ref="role2" value-ref="user2" />
        </map>
    </property>
    <property name="set">
        <set>
            <!-- 对象引用 -->
            <ref bean="role1" />
            <ref bean="role2" />
        </set>
    </property>
</bean>
```

Spring还提供了对应的命名空间的定义，只是在使用命名空间的时候要先引入对应的命名空间和XML模式（XSD）文件

#### 通过注解装配Bean

使用注解的方式可以减少XML的配置，注解功能更加强大，它既能实现XML的功能，也提供了自动装配的功能，采用了自动装配后，程序员所要作的决断就更少了，这就是“约定优于配置”的开发原则。

Spring中提供了两种方式来让IoC容器发现Bean：

组件扫描：通过定义资源的方式，让Spring IoC容器扫描对应的包，从而把Bean装配进来

自动装配：通过注解定义，使得一些依赖关系可以通过注解完成。

##### 使用@Component装配Bean

```java
package com.ssm.chapter10.annotation.pojo;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component(value="role")
public class Role {
    @Value("1")
    private Long id;
    @Value("role_name_1")
    private String roleName;
    @Value("role_note_1")
    private String note;
    
    /** setter and getter **/
}
```

@Component代表Spring IoC会把这个类扫描生成Bean实例，value属性代表这个类在Spring中的id。如果不写这个参数，容器使用默认名。这个时候使用一个Java Config来告诉它：

```Java
package com.ssm.chapter10.annotation.pojo;
import org.springframework.context.annotation.ComponentScan;

@ComponentScan
public class PojoConfig {
    
}
```

Config类得和POJO在一个包里，@ComponentScan代表进行扫描，默认是扫描当前包的路径。

```Java
ApplicationContext context = new AnnotationConfigApplicationContext(PojoConfig.class);
```

这样Spring IoC会根据注解的配置解析对应的资源。

@ComponentScan有两个参数，basePackages和basePackageClasses，spring会根据它的配置扫描对应的包和子包

```Java
@ComponentScan(basePackageClasses={Role.class, RoleSeriviceImpl.class})
//@ComponentScan(basePackages={"com.ssm.chapter10.annotation.pojo", "com.ssm.chapter10.annotation.service"})
//@ComponentScan(basePackages={"com.ssm.chapter10.annotation.pojo", "com.ssm.chapter10.annotation.service"},
// basePackageClasses={Role.class, RoleServiceImpl.class})
public class ApplicationConfig {}
```

如果采用多个@ComponentScan去定义对应的包，每定义一个，Spring就会为所定义的类生成一个新的对象，所配置的Bean将会生成多个实例，这不是我们所需要的。但在同一个@ComponenScan中重复定义相同的包或者存在其子包定义，也不会因为同一个Bean的多次扫描，而导致一次配置生成多个对象。

##### 自动装配@Autowired

当Spring生成所有的Bean后，如果发现这个注解，它就会在Bean中查找，然后找到对应的类型，将其注入进来。自动装配是一种由spring自己发现对应的Bean，自动完成装配工作的方式。

```java
@Autowired
private Role role = null;
```

这里的@Autowired注解，表示在Spring IoC定位所有的Bean后，这个字段需要按照类型注入，这样IoC容器就会寻找资源，然后将其注入。IoC容器有时寻找失败后抛出异常，累哦通过配置项required来改变，比如@Autowired(required=false)。

还可以配置方法，比如setter方法：

```Java
public class RoleServiceImpl2 implements RoleService2 {
    @Autowired
    public void setRole(Role role) {
        this.role = role;
    }
}
```

spring大部分情况下使用接口编程，但是定义一个接口，并不一定只有一个对应的实现类。当一个接口由多个实现类是，spring就无法判断把哪个对象注入进来。

为了消除歧义性，可使用两个注解：

- 注解@Primary告诉Spring IoC容器，请优先使用该类注入。
- 注解@Qualifier告诉容器按照名称的方式注入

```Java
public class RoleController {
    @Autowired
    @Qualifier("roleService3")
    private RoleService roleService = null;
    //......
}
```

装载带有参数的构造方法类

```java
public RoleController2(@Autowired RoleService roleService) {
    this.roleService = roleService;
}
```

使用@Bean装配Bean

@Component只能注解在类上，不能注解到方法上。@Bean可以注解在方法上，并将方法返回的对象作为Spring的Bean，存放在IoC容器中。

```Java
@Bean(name="dataSource")
public DataSource getDataSource() {
    // ......
    return dataSource;
}
```

@Bean的配置项：

- name：是一个字符串数组，允许配置多个BeanName
- autowire：标志是否是一个引用的Bean对象，默认为Autowire.NO
- initMethod：自定义初始化方法
- destroyMethod：自定义销毁方法

```Java
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ImportResource;
@ComponentScan(basePackages={"com.ssm.chapter10.annotation"})
@ImportResource({"classpath:spring-dataSource.xml"})
public class ApplicationConfig {
    
}
```

@ImportResource中配置的内容是一个数组，也可以是配置多个XML配置文件，这样就可以引入多个XML所定义的Bean了。

可以使用@Import注解（XML文件中的import元素）来加载其他的配置类（XML文件）。

```Java
@ComponentScan(basePackages={"com.ssm.chapter10.annotation"})
@Import({ApplicationConfig2.class, ApplicationConfig3.class})
public class ApplicationConfig{
    
}
```

```xml
<import resourse="spring-datasource.xml" />
<context:component-scan base-package="com.ssm.chapter10.annotation" />
```

建议把第三方包，系统外的接口服务和通用的配置使用XML配置，而对于系统内部的开发则以注解方式为主

##### Profile

**使用@Profile注解配置**

```Java
@Component
public class ProfileDataSource {
    @Bean(name="devDataSource")
    @Profile("dev")
    public DataSource getDevDataSource() {
        // ......
        return dataSource;
    }
    
    @Bean(name="testDataSource")
    @Profile("test")
    public DataSource getTestDataSource() {
        // ......
        return dataSource;
    }
}
```

**使用XML定义Profile**

```xml
<?xml version="1.0" encoding="utf-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/shema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.0.xsd"
       profile="dev">
    <bean></bean>
</beans>
```

也可以在一个XML文件中配置多个profile

```xml
<beans>
    <beans profile="test">
        <!--......-->
    </beans>
    <beans profile="dev">
        <!--......-->
    </beans>
</beans>
```

启动profile

当启动Java配置或者XML配置Profile时，可以发现这两个Bean不会被加载到Spring IoC容器中，需要自行激活Profile。

激活Profile的方法有5中：

- 在使用SpringMVC的情况下可以配置Web上下文参数，或者DispatchServlet参数
- 作为JNDI条目
- 配置环境变量
- 配置JVM启动参数
- 在继承测试环境中使用@ActiveProfiles

```Java
import javax.sql.DataSource;
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=ProfileConfig.class)
@ActiveProfiles("dev")
public class ProfileTest {
    @Autowired
    private DataSource dataSource;
    @Test
    public void test() {
        System.out.println(dataSource.getClass().getName());
    }
}
```

可以配置java虚拟机的启动项，关于制定Profile的参数存在两个：

- spring.profiles.active：如果配置它，spring.profiles.default配置项将会失效
- spring.profiles.default:默认启动的Profile，如果系统没有配置Profile参数的时候，它将启动。

如果使用的是Spring MVC，那么可以设置Web环境参数或者DispatcherServlet参数来选择对应的Profile，比如在web.xml中进行配置：

```xml
<!--使用web环境参数-->
<context-param>
    <param-name>spring.profiles.active</param-name>
    <param-value>test</param-value>
</context-param>
<!--使用SpringMVC的DispatcherServlet环境参数-->
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>2</load-on-startup>
    <init-param>
        <param-name>spring.profiles.active</param-name>
        <param-name>test</param-name>
    </init-param>
</servlet>
```

##### 加载属性（properties）文件

@PropertySource注解来加载属性文件，Spring会把对应文件加载进来，在Spring环境中使用它们。

```Java
@Configuration
@PropertySource(value={"classpath:database-config.properties"}, ignoreResourceNotFound=true)
public class ApplicationConfig {
    
}

// 测试加载属性
ApplicationContext context = new AnnotationConfigApplicationContext(ApplicationConfig.class);
String url = context.getEnvironment().getProperty("jdbc.database.url");
```

Spring推荐一个属性文件解析类来进行处理，允许Spring解析对应的属性文件，并通过占位符去引用对应的配置

```java
@Configuration
@ComponentScan(basePackages={"com.ssm.chapter10.annotation"})
@PropertySource(value={"classpath:database-config.properties"}, ignoreResourceNotFound=true)
public class ApplicationConfig {
    @Bean
    // 这个Bean为了让Spring能够解析属性占位符
    public PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }
}

// 于是可以使用@Value和占位符
@Value("${jdbc.database.driver}")
private String driver = null;
```

使用<context:property-placeholder>来加载配置项

```xml
<beans>
    <context:component-scan base-package="com.ssm.chapter10.annotation" />
    <context:property-placeholder ignore-resource-not-found="true" location="classpath:database-config.properties" />
</beans>
```

location可以配置单个文件或者多个文件，多个文件之间要用逗号分隔。

##### 条件化装配Bean

Spring提供了注解@Conditional去配置，通过它可以配置一个或者多个类，只是这个类都需要实现接口Condition（org.springframework.context.annotation.Condition）

```Java
@Bean(name="dataSource")
@Conditional({DataSourceCondition.class})
public DataSource getDataSource (@Value("${jdbc.database.driver}") String driver,
                                @Value("${jdbc.database.url}") String url,
                                @Value("${jdbc.database.username}") String username,
                                @Value("${jdbc.databse.password}") String password){
    // ......
    return dataSource
}

public class DataSourceCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment env = context.getEnvironment();
        return env.containsProperty("jdbc.database.driver")
            && env.containsProperty("jdbc.database.url")
            && env.containsProperty("jdbc.database.username")
            && env.containsProperty("jdbc.database.password");
    }
}
```

##### Bean的作用域

默认情况下，Spring IoC容器只会为配置的Bean生成一个实例，而不是多个。

Spring提供4种作用域：

- 单例（singleton）
- 原型（prototype）：每次注入，或者通过IoC容器获取Bean时，Spring都会为它创建一个新的实例
- 会话（session）：在web中，在会话过程中spring只创建一个实例
- 请求（request）：在web中，在一次请求中spring会创建一个实例

```Java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class RoleDataSourceServiceImple implements RoleDataSourceService {
    
}
```

##### Spring EL

略略略

### 面向切面编程

​	现实中有一些内容并不是面向对象可以解决的，AOP编程有着重要的意义，首先它可以拦截一些方法，然后将各个对象组织成一个整体。如果一个AOP框架不需要莫问去实现累成中的方法，而是在流程中提供一些通用的方法，并可以通过一定的配置满足各种功能，比如AOP框架帮助你完成了获取数据库，你就不需要知道如何获取数据库连接功能了。

​	AOP可以说是OOP（Object Oriented Programming，面向对象编程）的补充和完善。OOP引入封装、继承、多态等概念来建立一种对象层次结构，用于模拟公共行为的一个集合。不过OOP允许开发者定义纵向的关系，但并不适合定义横向的关系，例如日志功能。日志代码往往横向地散布在所有对象层次中，而与它对应的对象的核心功能毫无关系对于其他类型的代码，如安全性、异常处理和透明的持续性也都是如此，这种散布在各处的无关的代码被称为横切（cross cutting），在OOP设计中，它导致了大量代码的重复，而不利于各个模块的重用。

#### 面向切面编程的术语

1. 切面（Aspect）

   切面就是在一个怎样的环境中工作，它可以定义各类通知、切点和引入等内容，然后Spring AOP会将其定义的内容织入到约定的流程中，在动态代理中可以理解成一个拦截器。

2. 通知（Advice）

   通知是切面开启后，切面的方法。它根据在代理对象真实方法调用前、后的顺序和逻辑分区，它和拦截器的方法十分接近。

   前置通知（before）：在原有方法或者环绕通知前执行的通知功能。

   后置通知（after）：在原有方法或者环绕通知后执行的通知功能。无论是否抛出异常，它都会被执行。

   返回通知（afterReturning）：在原有方法或者环绕通知后正常返回执行的通知功能。

   异常通知（afterThrowing）：在原有方法或者环绕通知异常后执行的通知功能。
   
   环绕通知（around）：取代当前被拦截对象的方法，提供回调原有被拦截对象的方法。
   
3. 引入（Introduction）

   允许我们在现有的类中添加自定义的类和方法。

4. 切点（Pointcut）

   告诉Spring AOP在什么时候启动拦截并织入对应的流程中，因为不是所有的开发都需要启动AOP的，它往往通过这则表达式进行限定。

5. 连接点（join point）

   对应的是具体要拦截的东西，比如通过切点的正则表达式去判断哪些方法是连接点，从而织入对应的通知。

6. 织入（Weaving）

   织入是一个生成代理对象并将切面内容放入到流程中的过程。Spring是通过JDK和CGLIB动态代理来生成代理对象的。

Spring对AOP的支持

SpringAOP是一种基于方法拦截的AOP，Spring只能支持方法拦截的AOP，在Spring中有4中方式去实现AOP的拦截功能。

- 使用ProxyFactoryBean和对应的接口实现AOP
- 使用XML配置AOP
- 使用@AspectJ注解驱动切面
- 使用Aspect注入切面

#### 使用@AspectJ注解开发SpringAOP

##### 选择连接点

```Java
package com.ssm.chapter11.aop.service;
import com.ssm.chapter11.gama.pojo.Role;
public interface RoleService {
    public void printRole(Role role);
}

package com.ssm.chapter11.aop.service.impl;
import org.springframework.stereotype.Component;
import com.ssm.chapter11.aop.service.RoleService;
import com.ssm.chapter11.game.pojo.Role;

@Component
public class RoleServiceImpl implements RoleService {
    @Override
    public void printRole(Role role) {
        //.....
    }
}
```

##### 创建切面，定义切点

```Java
package com.ssm.chapter11.aop.aspect;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
// 创建切面
public class RoleAspect {
    
    // 定义切点
    @Pointcut("execution(* com.ssm.chapter11.aop.service.impl.RoleServiceImpl.printRole(..))")
    public void print() {}
    
    @Before("print()")
    public void before() {
        //.....
    }
    
    @After("print()")
    public void after() {
        //.....
    }
    
    @AfterReturning("print()")
    public void afterReturning() {
        //.....
    }
    
    @AfterThrowing("print()")
    public void afterThrowing() {
        //.....
    }
}
```

[AspectJ指示器](https://blog.csdn.net/sinat_38393872/article/details/100593202)

```Java
@Before("execution(* com.ssm.chapter11.*.*.*.*.printRole(..)) && within(com.ssm.chapter11.aop.service.impl.*)")
public void before() {
    //.....
}
```

这里使用within去限定了execution定义的正则式下的包的匹配。

##### 启用AOP

```Java
package com.ssm.chapter11.aop.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

import com.ssm.chapter11.aop.aspect.RoleAspect;

@Configuration
// 代表启用AspectJ框架的自动代理，此时Spring才会生成动态代理对象，进而可以使用AOP
@EnableAspectJAutoProxy
@ComponentScan("com.ssm.chapter11.aop")
public class AopConfig {
    @Bean
    public RoleAspect getRoleAspect() {
        return new RoleAspect();
    }
}
```

在XML中使用aspectj-autoproxy元素达到@EnableAspectJAutoProxy的效果

```xml
<!--spring-cfg3.xml-->
<beans>
	<aop:aspectj-autoproxy />
    <bean id="roleAspect" class="com.ssm.chapter11.aop.aspect.RoleAspect" />
    <bean id="roleService" class="com.ssm.chapter11.aop.service.impl.RoleServiceImpl" />
</beans>
```

于是测试AOP流程

```Java
ApplicationContext context = new AnnotationConfigApplicationContext(AopConfig.class);
// 使用XML配置
ApplicationContext context = new ClassPathXmlApplicationContext("spring-cfg3.xml");

```

注意切点调用顺序

```
before ....
// 代码调用内容
after ....
afterReturning ....
```

##### 环绕通知

```Java
@Around("print()")
public void around(ProceedingJoinPoint jp) {
    System.out.println("around before ....");
    try {
        jp.preceed();
    } catch(Throwable e) {
        e.printStackTrace();
    }
    System.out.println("around after ....");
}
```

调用顺序如下：

```
around before ....
before ....
// code
around after ....
after ....
afterReturning ....
```

around在使用jp.preceed后会先调度前置通知（当使用XML方式时，前置通知放在jp.proceed()之前调用，估计是版本问题），然后才会反射切点方法，最后才是后置通知和返回（或者异常）通知。

##### 织入

织入是生成代理对象并将切面内容放入约定流程的过程。实际上没有接口，Spring也能提供AOP功能，所以是否拥有接口不是使用Spring AOP的一个强制要求。当类不存在接口时就无法使用JDK动态代理，Spring会采用CGLIB来生成代理对象。

在Spring中建议使用接口编程，这样的好处是使定义和实现相分离。

##### 给通知传递参数

当一个连接点为一个多参数的方法，连接点和通知改写如下：

```Java
public void printRole(Role role, int sort) {
    // ....
}

// 以前置通知为例
@Before("execution(* com.ssm.chapter11.aop.service.impl.RoleServiceImpl.printRole(..)) " 
        + "&& args(role, sort)")
public void before(Role role, int sort) {
    // ....
}
```

##### 引入

加入RoleVerifier到切面中

```Java
@DeclareParents(value="com.ssm.chapter11.aop.service.impl.RoleServiceImpl+", defaultImpl=RoleVerifierImpl.class)
public RoleVerifier roleVerifier;
```

value="com.ssm.chapter11.aop.service.impl.RoleServiceImpl+"表示对RoleServiceImpl类进行增强，也就是在RoleServiceImpl中引入一个新的接口。defaultImpl代表其默认的实现类。

这样就可以通过强制转换把roleService转化为RoleVerifier接口对象，然后就可以使用verify方法了。而RoleVerifier调用的方法就是通过RoleVerifierImpl来实现的。

@DeclareParents用于对现有类***增加新的方法\***

Spring AOP让代理对象挂到RoleService和RoleVerifier两个接口下，也就是把对应的Bean通过强制转换，让其在RoleService和RoleVerifier之间相互转换。这样就能够在原有基础的实现上再次增强Bean的功能了。

如果RoleServiceImpl没有接口，这里也会使用CGLIB动态代理

#### 使用XML配置开发Spring AOP

```
before ....
around before ....
// code
arounr after ....
after-returning ....
after ....
```

注意执行的顺序和@AspectJ的不同

#### 多个切面

Spring也支持多个切面，当有多个切面时，这些切面的执行顺序时随机的

可以采用@Order注解。

```Java
package com.ssm.chapter11.multi.aspect;
@Aspect
@Order(1)
public class Aspect1 {
    //.....
}
```

```xml
<aop:aspect ref="aspect1" order="1"></aop:aspect>
```

### Spring和数据库编程

Spring为开发者提供了JDBC模板模式，就是它自身的JdbcTemplate，可以简化许多代码的编程，但是在实际的工作中JdbcTemplate并不常用。对于支持事务，Spring还提供了TransactionTemplate，只是这些都不是常用的技术，对于持久层，工作中更多的是使用Hibernate框架和MyBatis框架。Spring并不会代替已有框架的功能，而是以提供模板的形式给与支持。

**传统JDBC代码的弊端**

太多的try catch finally语句，造成了代码的泛滥，导致了代码可读性和可维护性极具下降，从而引发信任问题。为此，Spring提供了JdbcTemplate模板，来解决这些问题

**JDBC代码失控的解决方案**

JdbcTemplate是针对JDBC代码失控提供的解决方案，体现了Spring框架的主导思想之一：给予常用技术提供模板化的编程，减少了开发者的工作量。JdbcTemplate在内部实现了数据库资源的处理，使用户无需再些任何关闭数据库资源的代码。

但是JdbcTemplate是不能支持事务的，还要引入对应的事务管理器才能够支持事务。

#### MyBatis-Spring项目

 使用MyBatis-Spring使得业务层和模型层得到了更好的分离，在Spring环境中使用MyBatis也更加简单，节省了不少代码，因为MyBatis-Spring为我们封装了它们。

配置MyBatis-Spring项目需要一下几步：

- 配置数据源
- 配置SqlSessionFactory
- 可以选择配置有SqlSessionTemplate，在同时配置SqlSessionTemplate和SqlSessionFactory的情况下，优先采用SqlSessionTemplate
- 配置Mapper，可以配置单个Mapper，也可以通过扫描的方法生成Mapper。此时Spring IOC会生成对应接口的实例，这样就可以通过注入的方式获得资源
- 事务管理

##### 配置SqlSessionFactoryBean

在MyBatis-Spring中提供了SqlSessionFactoryBean去支持SqlSessionFactory的配置。在其源码中，几乎可以配置所有关于MyBatis的组件，并且它也提供了对应的setter方法让Spring设置它们，所以完全可以通过Spring IoC容器的规则去配置它们。

比如xml配置如下：

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <!--这个是MyBatis配置文件-->
    <property name="configLocation" value="classpath:sqlMapConfig.xml" />
</bean>
```

##### SqlSessionTemplate组件

SqlSessionTemplate并不是一个必须配置的组件，但它是一个线程安全的类，也就是确保每个线程使用的SqlSession唯一且不相冲突。xml配置如下：

```xml
<bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
    <constructor-arg ref="SqlSessionFactory" />
    <constructor-arg value="BATCH" />
</bean>
```

SqlSessionTemplate要通过带有参数的构造方法去创建对象，常用的参数是SqlSessionFactory和MyBatis执行器（Executor）类型，取值范围是SIMPLE、REUSE和BATCH。

SqlSessionTemplate每次运行的时候会产生新的SqlSession，每一个方法都是独立的SqlSession，这意味着它是安全的线程。当同时配置SqlSessionTemplate和SqlSessionFactory的时候，SqlSessionTemplate的优先级大于SqlSessionFactory（毕竟SqlSessionTemplate内部维护了SqlSessionFactory）。

##### 配置MapperFactoryBean

通过MapperFactoryBean来实现我们想要的Mapper。使用了Mapper接口编程方式可以有效地在逻辑代码中擦除SqlSessionTemplate，这样代码就按照面向对象的规范进行编写了。

```xml
<!-- 配置RoleMapper对象 -->
<bean id="roleMapper" class="org.myBatis.spring.mapper.MapperFactoryBean">
	<!--RoleMapper接口将被扫描为Mapper-->
    <property name="mapperInterface" value="com.ssm.chapter12.mapper.RoleMapper" />
    <property name="sqlSessionFactory" ref="sqlSessionFactory" />
    <!--如果同时注入sqlSessionTemplate和sqlSessionFactory，则会启用sqlSessionTemplate-->
</bean>
```

有三个属性可以配置。mapperInterface是映射器的接口

这样就可以使用下面的代码来获取映射器了：

```Java
RoleMapper roleMapper = ctx.getBean(RoleMapper.class);
```

##### 配置MapperScannerConfiguration

像上面一样一个个配置Mapper会造成配置量大的问题，那么使用MapperScannerConfiguration生成大量的Mapper，从而减少工作量。

MapperScannerConfigurer的配置如下：

- basePackage：制定spring扫描哪些包，它会逐层深入扫描，如果遇到多个包可以使用半角逗号分隔。
- annotationClass：表示如果类被这个注解表示的时候，才进行扫描，使用这个注解注册对应的Mapper。在Spring中往往使用注解@Repository表示数据访问层（DAO）。
- sqlSessionFactoryBeanName：制定在Spring中定义SqlSessionFactory的Bean名称，如果sqlSessionTemplateBeanName被定义，那么它将失去作用。
- markerInterface：指定实现了什么接口就认为它是Mapper。我们需要一个公共接口来标记。

在Spring中使用@Repository表示DAO层的。

```Java
import org.springframework.stereotype.Repository;

@Repository
public intelface RoleMapper{
    // 定义方法
}
```

@Repository表示这是一个DAO层，我们还需要告诉Spring扫描哪个包，这样就会扫除对应的Mapper到Spring IoC容器中了：

```xml
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.ssm.chapter12.mapper" />
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
    <!-- 使用sqlSessionTemplateBeanName将覆盖sqlSessionFactoryBeanName的配置-->
    <property name="sqlSessionTemplateBeanName" value="sqlSessionFactory" />
    <property name="annotationClass" value="org.springframework.stereotype.Repository" />
</bean>
```

这样Spring IoC容器就知道将包命名为com.ssm.chapter12.mapper，把注解为@Repository的接口扫描为Mapper对象，存放在容器中，对于多个包的扫描可以使用半角逗号分来。

也可以使用拓展接口名的方式，比如事先定义一个接口——baseMapper

```Java
package com.ssm.chapter12.base;
public interface BaseMapper {
    // 无任何逻辑，只为标注
}

// RoleMapper如下
import com.ssm.chapter12.base.BaseMapper;
public interface RoleMapper extends BaseMapper {
    //....
}
```

接下来是Spring能扫描到这个接口：

```xml
<bean id="roleMapper" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
	<property name="basePackage" value="com.ssm.chapter12.mapper" />
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
    <property name="markerInterface" value="com.ssm.chapter12.base.BaseMapper" />
</bean>
```

这种方法用的不多

### 深入Spring数据库事务管理

#### Spring数据库事务管理器的设计

Spring中数据库事务是通过PlatformTransactionManager进行管理的。jdbcTemplate自身不能支持事务，而能够提供org.springframework.transaction.support.TransactionTemplate模板，它是Spring所提供的事务管理模板。

在TransactionTemplate代码中，我们可以看到：

- 事务的创建、提交和回滚是通过PlatformTransactionManager接口来完成的。
- 当事务产生异常时会回滚事务，在默认的实现中所有的异常都会回滚。我们可以通过配置去修改在某些异常发生时回滚或者不会滚事务。
- 当无异常时，会提交事务

```Java
package org.springframework.transaction;

// 注意，这个包现在在spring-fx下
public interface PlatformTransactionManager {
    // 获取事务状态
    TransactionStatus getTransaction(TransactionDefinition var1) throws TransactionException;
	// 提交事务
    void commit(TransactionStatus var1) throws TransactionException;
	// 回滚事务
    void rollback(TransactionStatus var1) throws TransactionException;
}
```

##### 配置事务管理器

MyBatis中用得最多的是DataSourceTransactionManager（org.springframework.jdbc.DataSourceTransactionManager）。在使用时，要加入XML的事务命名空间，配置如下：

```xml
<beans xsi:schemaLocation="...
                           http://www.springframework.org/schema/tx
                           http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
                           ...">
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">...</bean>
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>
</beans>
```

这样就将数据库事务委托给事务管理器管理了，数据库资源产生和释放如果没有委托给数据库管理器，那么就由jdbcTemplate管理，但在这里已经委托给了事务管理器，所以jdbcTemplate的数据库资源和事务已经由事务管理器处理了。

在Spring中可以使用申明式事务或者编程式事务，编程式事务几乎不用，申明式事务分为XML配置和注解配置，但XML方式也不常用。

##### 用Java配置方式实现Spring数据库事务

用Java配置方式实现Spring数据库事务,需要在配置类中实现接口TransactionManagementConfigurer的annotationDrivenTransactionManager方法。Spring会将annotationDrivenTransactionManager方法返回的事务管理器作为程序的事务管理器。

```Java
package com.ssm.chapter13.config;
@Configuration
@ComponentScan("com.ssm.chapter13.*")
@EnableTransactionManagement
public class JavaConfig implements TransactionManagementConfigurer {
    @Override
    @Bean(name="transactionManager")
    public PlatformTransactionManager annotationDrivenTransactionManager() {
        //.....
    }
}
```

使用注解@EnableTransactionManagerment后，在Spring上下文中使用事务注解@Transactional，Spring就会知道使用这个数据库事务管理器管理事务了。

#### 编程式事务

需要用到事务定义类接口——TransactionDefinition

```Java
ApplicationContext ctx = new ClassPathXmlApplicationContext("spring-cfg.xml");
JdbcTemplate jdbcTemplate = ctx.getBean(JdbcTemplate.class);
// 事务定义类
TransactionDefinition def = new DefaultTransactionDefinition();
PlatformTransactionManager transactionManager = ctx.getBean(PlatformTransactionManager.class);
TransactionStatus status = transactionManager.getTransaction(def);
try {
    // 不会自动提交事务
    jdbcTemplate.update("insert into t_role(role_name, note) " + 
                        "values('role_name_transactionManager','note_transactionManager')");
    // 提交事务
    transactionManager.commit(status);
} catch(Exception ex) {
    // 会滚事务
    transactionManager.rollback(statis);
}
```

JdbcTemplate本身的数据库资源已经由事务管理器管理，因此当它执行万insert语句时不会自动提交事务，这个时候需要使用事务管理器的commit方法，回滚事务需要使用rollback方法。

#### 声明式事务

声明式事务允许自定义事务接口——TransactionDefinition， 它由xml或者注解@Transactional进行配置。

注意Spring默认情况下会对（RuntimeException）及其子类进行回滚，在遇见Exception及其子类的时候不进行回滚。

@Transactional源码如下：

```Java
package org.springframework.transaction.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.core.annotation.AliasFor;

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {
    // aliasfor 表示互为别名，两个属性其实一个含义
    @AliasFor("transactionManager")
    String value() default "";
    
    // 定义事务管理器，是IoC容器中的一个bean的id，这个bean需要实现接口PlatformTransactionManager
    @AliasFor("value")
    String transactionManager() default "";
	// 传播行为，默认值为Propagation.REQUIRED
    Propagation propagation() default Propagation.REQUIRED;
	// 隔离级别，默认取值为数据库默认隔离级别
    Isolation isolation() default Isolation.DEFAULT;
	// 超时时间，当超时时会引发异常，默认会导致事务回滚
    int timeout() default -1;
	// 是否开启只读事务，默认为false
    boolean readOnly() default false;
	// 回滚事务的异常类定义，当方法产生异常时，才会回滚事务，否则就会提交事务
    Class<? extends Throwable>[] rollbackFor() default {};
	// 回滚事务的异常类名定义，同rollbackFor，只是使用类名称定义
    String[] rollbackForClassName() default {};
	// 当产生哪些异常不会滚事务，当产生所定义异常时，Spring将继续提高事务
    Class<? extends Throwable>[] noRollbackFor() default {};
	// 同noRollbackFor
    String[] noRollbackForClassName() default {};
}
```

这些属性将被Spring放到事务定义类TransactionDefinition中。@Transactional既可以作用于接口，接口方法以及类的方法上。但是Spring官方不建议接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效。@Transactional 注解应该只被应用到 public 方法上，这是由 Spring AOP 的本质决定的。如果你在 protected、private 或者默认可见性的方法上使用 @Transactional 注解，这将被忽略，也不会抛出任何异常。

使用声明式事务需要配置注解驱动，在代码中配置如下：

```xml
<tx:annotation-driven transaction-manager="transactionManager" />
```

使用XML配置事务管理器（略）

**事务定义器**

```Java
package org.springframework.transaction;

public interface TransactionDefinition {
    // 传播行为常量定义(7个)
    int PROPAGATION_REQUIRED = 0; // 默认传播行为
    int PROPAGATION_SUPPORTS = 1;
    int PROPAGATION_MANDATORY = 2;
    int PROPAGATION_REQUIRES_NEW = 3;
    int PROPAGATION_NOT_SUPPORTED = 4;
    int PROPAGATION_NEVER = 5;
    int PROPAGATION_NESTED = 6;
    // 隔离级别定义（5个）
    int ISOLATION_DEFAULT = -1;
    // 读未提交
    int ISOLATION_READ_UNCOMMITTED = 1;
    // 读提交
    int ISOLATION_READ_COMMITTED = 2;
    // 可重复读
    int ISOLATION_REPEATABLE_READ = 4;
    // 串行化
    int ISOLATION_SERIALIZABLE = 8;
    // 代表永不超时
    int TIMEOUT_DEFAULT = -1;
    // 获取传播行为
    int getPropagationBehavior();
	// 获取隔离级别
    int getIsolationLevel();
	// 事务超时时间
    int getTimeout();
	// 是否只读事务
    boolean isReadOnly();
	// 获取事务定义器的名称
    String getName();
}
```

​	除了异常的定义，其他关于事务的定义都可以在这里完成，而对于事务的回滚内容，她会以RollbackRuleAttribute和NoRollbackRuleAttribute两个类进行保存，这样在事务拦截器（XML配置事务管理器时使用）中就可以根据我们所配置的内容来处理事务方面的内容了。

​	@Transaction注解可以用在方法或者类上面，在Spring IoC容器初始化时，Spring会读入这个注解或者XML配置的事务信息，并保存到一个事务定义类里面（TransactionDefinition接口的子类），以备将来使用。运行时会让Spring拦截注解标注的某个方法或者类的所有方法。

​	首先Spring通过事务管理器（PlatformTransactionManager的子类）创建事务，此时会把事务定义中的隔离级别、超时时间等属性根据配置内容往事务上设置。而根据传播行为配置采取一种特定的策略。然后启动开发者提供的业务代码，Spring会通过反射的方式调度开发者的业务方法，只要发生异常，并且符合事务定义类回滚条件的，Spring就会将数据库事务回滚，否则将数据库事务提交。在整个开发过程中，只需要编写业务代码和对事务属性进行配置就可以了。

#### 选择隔离级别和传播行为

选择隔离级别的出发点在于两点：性能和数据一致性

隔离级别需要根据并发的大小和性能来做出决定。

传播行为是指方法间的调用和事务策略的问题。

在Spring中传播行为的类型，是通过一个枚举类型去定义的，这个枚举是org.springframework.transaction.annotation.Propagation，7种传播行为如下：

REQUIRED：当方法调用时，如果不存在当前事务，那么创建事务；如果或已经存在事务，则沿用。这是默认的传播行为

SUPPRORTS：当方法调用时，不存在当前事务则不启用新事务；存在当前事务就沿用。

MANDATORY：方法必须在事务内运行，如果不存在当前事务就抛出异常。

REQUIRES_NEW：无论是否存在当前事务，方法都会在新的事务中运行

NOT_SUPPORTED：不支持事务，如果不存在当前事务也不会创建事务；如果存在当前事务，则挂起它，直至该方法结束后才会恢复当前事务。（使用于不需要事务的SQL）

NEVER：不支持事务，只有在没有事务的环境中才能运行它，如果存在当前事务，则抛出异常

NESTED：嵌套事务，也就是调用方法如果抛出异常只回滚自己内部执行的SQL，而不会滚主方法的SQL

用的多的就是REQUIRES_NEW和NESTED

有些数据库不支持NESTED，所以Spring先去探测当前数据库是否能支持保存点技术。如果不支持，它就会和REQUIRES_NEW一样创建新事务去运行代码，以达到内部方法发生异常时并不回滚当前事务的目的。

#### @Transactional的自调用失效的问题

SpringAOP技术使用的是动态代理。这就意味着对于静态（static）方法和非public方法，注解@Transactional是失效的，

自调用就是一个类的方法去调用自身另外一个方法的过程。

案例如下：

```Java
package com.delusion.aop.service.impl;
import com.delusion.aop.service.RoleService;
import com.delusion.pojo.Role;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

public class ExampleServiceImpl implements RoleService {

    @Autowired
    private RoleMapper roleMapper = null;

    @Override
    @Transactional(propagation = Propagation.REQUIRED_NEW,
    isolation = Isolation.READ_COMMITTED)
    public int insertRole(Role role) {
        return roleMapper.insertRole(role);
    }

    @Override
    @Transactional(propagation = Propagation.REQUIRED,
    isolation = Isolation.READ_COMMITTED)
    public int insertRoleList(List<Role> roleList) {
        int count = 0;
        for (Role role : roleList) {
            try {
                // 调用自身类的方法，产生自调用问题
                count += 1;
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }

        return count;
    }

}
```

这样，insertRole上标注的@Transactional失效了。原因在于AOP的实现原理，这里是自己调用自己的方法，并不存在代理对象的调用，这样就不会产生AOP为我们设置的@Transactional配置的参数，于是就出现了自调用的注解失效的问题。

一种改写的方式如下：

```Java
@Override
@Transactional(propagation=Propagation.REQUIRED, isolation=Isolation.READ_COMMITTED)
public int insertRoleList(List<Role> roleList) {
    int count = 0;
    // 这里拿到的是代理对象
    RoleService service = ctx.getBean(RoleService.class);
    for (Role role : roleList) {
        try {
            service.insertRole(role);
            count += 1;
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
    return count;
}
```

从容器中获取代理对象的方法克服了自调用的过程，但是从容器中获取代理对象的方法又侵入直线，你的类需要依赖SpringIoC容器，所以还是使用另一个服务类去调用。

**错误捕捉异常**

```Java
@Autowired
private ProductsService productService;
@Autowired
private TransactionService transactionService;

@Override
@Transactional(propagation=Propagation.REQUIARED, isolation=Isolation.READ_COMMITTED)
public int doTransaction(TransactionBean trans) {
    int result = 0;
    try {
        // 减少库存
        int result = productService.docreaseStock(
            trans.getProductId, trans.getQuantity());
        // 如股票减少库存成功则保存记录
        // 本来若save抛出异常，事务回滚，结果在方法中捕获了异常，导致事务没有回滚，正常提交
        if (result > 0)
            transactionService.save(trans);
    } catch (Exception ex) {
        // 自行处理异常代码
        // 记录异常日志
        log.info();
    }
    return result;
}
```

改造如下：

```Java
@Autowired
private ProductsService productService;
@Autowired
private TransactionService transactionService;

@Override
@Transactional(propagation=Propagation.REQUIARED, isolation=Isolation.READ_COMMITTED)
public int doTransaction(TransactionBean trans) {
    int result = 0;
    try {
        // 减少库存
        int result = productService.docreaseStock(
            trans.getProductId, trans.getQuantity());
        // 如股票减少库存成功则保存记录
        if (result > 0)
            transactionService.save(trans);
    } catch (Exception ex) {
        // 自行处理异常代码
        // 记录异常日志
        log.info();
        // 自行抛出异常，让Spring事务管理流程获取异常，进行事务管理
        throw new RuntimeException(ex);
    }
    return result;
}
```

