### @SpringBootApplication解析

这个注解组合了@Configuration、@EnanbleAutoConfiguration和@ComponentScan。Spring Boot会自动扫描@SpringBootApplication所在类的同级包以及下级包里面的Bean。

```Java
// @EnableAutoConfiguration
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
// 这个注解表明被EbableAutoConfiguration标注的类都将继承标注中的属性
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}

// @SpringBootApplication
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
// @SpringBootConfiguration继承自Configuration注解
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    // 排除其他配置类？（可能）
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    // 继承自EnableAutoConfiguration
    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
    Class<?>[] exclude() default {};

    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
    String[] excludeName() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackages"
    )
    String[] scanBasePackages() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackageClasses"
    )
    Class<?>[] scanBasePackageClasses() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "nameGenerator"
    )
    Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

    @AliasFor(
        annotation = Configuration.class
    )
    boolean proxyBeanMethods() default true;
}
```

### Spring元数据的、注解编程

[参见这里]: https://blog.csdn.net/f641385712/article/details/88765470

### Spring Boot运行自动配置的原理

Spring Boot关于自动配置的源码在Spring-boot-autoconfigure-xxx.jar内，可以通过在运行jar时使用--debug参数

```
java -jar xx.jar --debug
```

或者在application.properties中设置属性debug=true来实现。

关于Spring Boot的运作原理，关键是@SpringBootApplication注解，而@SpringBootApplication是一个组合注解，其核心功能是由@EnableAutoConfiguration注解提供的。

```javA
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

在这里EnableAutoConfigurationImportSelector使用SpringFactoriesLoader.loadFactoryNames方法扫描具有META-INF/spring.factories文件的jar包。Spring Boot中有一种非常解耦的扩展机制：Spring Factories。这种扩展机制实际上是仿照Java中的SPI扩展机制来实现的。在Spring中也有一种类似与Java SPI的加载机制。它在META-INF/spring.factories文件中配置接口的实现类名称，然后在程序中读取这些配置文件并实例化。这种自定义的SPI机制是Spring Boot Starter实现的基础。

##### Spring Factories实现原理

在SpringFactoriesLoader类中定义了loadFactoryNames方法和loadFactories方法从指定的ClassLoader中获取spring.factories文件，并解析得到类型列表

```Java
	public static <T> List<T> loadFactories(Class<T> factoryType, @Nullable ClassLoader classLoader) {
        Assert.notNull(factoryType, "'factoryType' must not be null");
        ClassLoader classLoaderToUse = classLoader;
        if (classLoader == null) {
            classLoaderToUse = SpringFactoriesLoader.class.getClassLoader();
        }

        // 返回对应配置接口的实现类名别表
        List<String> factoryImplementationNames = loadFactoryNames(factoryType, classLoaderToUse);
        if (logger.isTraceEnabled()) {
            // 写入日志
            logger.trace("Loaded [" + factoryType.getName() + "] names: " + factoryImplementationNames);
        }

        List<T> result = new ArrayList(factoryImplementationNames.size());
        Iterator var5 = factoryImplementationNames.iterator();

        while(var5.hasNext()) {
            String factoryImplementationName = (String)var5.next();
            // 更具配置中的实现类全类名、接口和类加载器初始化这些实现类，并添加到结果列表中
            result.add(instantiateFactory(factoryImplementationName, factoryType, classLoaderToUse));
        }
		// 根据注解顺序对这些配置实现类进行排序，以安排好各个实现类的执行顺序
        AnnotationAwareOrderComparator.sort(result);
        return result;
    }

    public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
        String factoryTypeName = factoryType.getName();
        // factoryType是配置接口，通过接口名找到对应的实现类名列表并返回
        return (List)loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
    }

    private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
        MultiValueMap<String, String> result = (MultiValueMap)cache.get(classLoader);
        // 如果之前已经扫描过就直接返回结果
        if (result != null) {
            return result;
        } else {
            try {
                // 从classLoader中获取spring.factories文件
                Enumeration<URL> urls = classLoader != null ? classLoader.getResources("META-INF/spring.factories") : ClassLoader.getSystemResources("META-INF/spring.factories");
                LinkedMultiValueMap result = new LinkedMultiValueMap();
				
                // 扫描spring.factories文件中的配置，以键值对的方式保存接口和实现类
                while(urls.hasMoreElements()) {
                    URL url = (URL)urls.nextElement();
                    UrlResource resource = new UrlResource(url);
                    Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                    Iterator var6 = properties.entrySet().iterator();

                    while(var6.hasNext()) {
                        Entry<?, ?> entry = (Entry)var6.next();
                        String factoryTypeName = ((String)entry.getKey()).trim();
                        String[] var9 = StringUtils.commaDelimitedListToStringArray((String)entry.getValue());
                        int var10 = var9.length;

                        for(int var11 = 0; var11 < var10; ++var11) {
                            String factoryImplementationName = var9[var11];
                            result.add(factoryTypeName, factoryImplementationName.trim());
                        }
                    }
                }
				// 将扫描出的结果保存到缓存中，下次如果还取的话就直接返回扫描结果
                cache.put(classLoader, result);
                return result;
            } catch (IOException var13) {
                throw new IllegalArgumentException("Unable to load factories from location [META-INF/spring.factories]", var13);
            }
        }
    }
```

从代码中可以知道，SpringFactoriesLoader会扫描spring.factories，根据配置接口加载配置实现类。spring.factories的是通过Properties解析得到的，所以我们在写文件中的内容都是安装下面这种方式配置的：

```properties
com.xxx.interface=com.xxx.classname
```

如果一个接口希望配置多个实现类，可以使用','进行分割

在日常工作中，我们可能需要实现一些SDK或者Spring Boot Starter给被人使用时，
 我们就可以使用Factories机制。Factories机制可以让SDK或者Starter的使用者只需要很少或者不需要进行配置，只需要在服务中引入我们的jar包即可。

### Spring Security

#### WebSecurityConfigurerAdapter

WebSecurityConfigurerAdapter类是个适配器类，在配置的时候，只要自己写一个配置类去继承，然后编写自己所特殊需要的配置。

```Java
@Configuration
@EnableWebSecurity
public class MyWebConfig extends WebSecurityConfigurerAdapter {
    //....
}
```

#### HttpSecurity

HttpSecurity 也是 Spring Security 中的重要一环。我们平时所做的大部分 Spring Security 配置也都是基于 HttpSecurity 来配置的。HttpSecurity 继承自 AbstractConfiguredSecurityBuilder，同时实现了 SecurityBuilder 和 HttpSecurityBuilder 两个接口。

```Java
public final class HttpSecurity extends
  AbstractConfiguredSecurityBuilder<DefaultSecurityFilterChain, HttpSecurity>
  implements SecurityBuilder<DefaultSecurityFilterChain>,
  HttpSecurityBuilder<HttpSecurity> {
        //...
}
```

先讲DefaultSecurityFilterChain

##### DefaultSecurityFilterChain、SecurityChain

```Java
// Spring Security 中的过滤器链
public interface SecurityFilterChain {
    // 匹配请求
    boolean matches(HttpServletRequest var1);
	// 返回过滤器列表
    List<Filter> getFilters();
}
```

当一个请求到来时，matches方法会比较请求是否与当前链吻合，如果吻合就返回getFilters方法中的过滤器，那么当前请求会逐个经过list集合的过滤器。SecurityChain接口只有一个实现类DefaultSecurityFilterChain。这个类只是对接口的方法进行类实现，没什么可说的。

##### SecurityBuilder

```Java
package org.springframework.security.config.annotation;

public interface SecurityBuilder<O> {
    O build() throws Exception;
}
```

SecurityBuilder是用来构建过滤器链的，在HttpSecurityBuilder时，传入的泛型就是DefaultSecurityFilterChain，所以build方法就是用来构建一个过滤器链

##### HttpSecurityBuilder

这是用来构建HttpSecurity的，也是一个接口

```Java
public interface HttpSecurityBuilder<H extends HttpSecurityBuilder<H>> extends
  SecurityBuilder<DefaultSecurityFilterChain> {
// 获取配置对象，Spring Security过滤器链中的所有过滤器对象都是由xxxConfigure来配置的，这里获取配置对象
 <C extends SecurityConfigurer<DefaultSecurityFilterChain, H>> C getConfigurer(
   Class<C> clazz);
// 移除一个配置对象
 <C extends SecurityConfigurer<DefaultSecurityFilterChain, H>> C removeConfigurer(
   Class<C> clazz);
// 设置/获取由多个SecurityConfigurer共享的对象
 <C> void setSharedObject(Class<C> sharedType, C object);
 <C> C getSharedObject(Class<C> sharedType);
// 配置验证器
 H authenticationProvider(AuthenticationProvider authenticationProvider);
// 配置数据源接口
 H userDetailsService(UserDetailsService userDetailsService) throws Exception;
 // 在某个过滤器后添加过滤器
 H addFilterAfter(Filter filter, Class<? extends Filter> afterFilter);
 // 在某个过滤器前添加过滤器
 H addFilterBefore(Filter filter, Class<? extends Filter> beforeFilter);
 // 添加一个过滤器，该过滤器必须时现有过滤器链中某一个过滤器或者其扩展
 H addFilter(Filter filter);
}
```

##### AbstractSecurityBuilder

AbstractSecurityBuilder类实现了SecurityBuilder接口，该类中主要做了一件事，就是确保整个构建只被构建一次。

```Java
public abstract class AbstractSecurityBuilder<O> implements SecurityBuilder<O> {
 private AtomicBoolean building = new AtomicBoolean();
 private O object;
 public final O build() throws Exception {
  if (this.building.compareAndSet(false, true)) {
   this.object = doBuild();
   return this.object;
  }
  throw new AlreadyBuiltException("This object has already been built");
 }
 public final O getObject() {
  if (!this.building.get()) {
   throw new IllegalStateException("This object has not been built");
  }
  return this.object;
 }
 protected abstract O doBuild() throws Exception;
}
```

build方法无法被重写，而具体的构建逻辑则交给了doBuild方法有子类实现。

##### AbstractConfiguredSecurityBuilder

AbstractSecurityBuilder 方法的实现类就是 AbstractConfiguredSecurityBuilder

在 AbstractConfiguredSecurityBuilder中定义了一个枚举类，将构建的整个过程分为5个状态，可以理解为其生命周期的5个状态

```Java
private enum BuildState {
    UNBUILT(0),
 	INITIALIZING(1),
 	CONFIGURING(2),
 	BUILDING(3),
 	BUILT(4);
 	
    private final int order;
 	BuildState(int order) {
  		this.order = order;
 	}
    public boolean isInitializing() {
  		return INITIALIZING.order == order;
 	}
 	public boolean isConfigured() {
  		return order >= CONFIGURING.order;
 	}
}
```

通过AbstractConfiguredSecurityBuilder的两个初始化器，我们可以对该抽象类维护的数据结构做一个了解：

```Java
    protected AbstractConfiguredSecurityBuilder(ObjectPostProcessor<Object> objectPostProcessor) {
        this(objectPostProcessor, false);
    }

    protected AbstractConfiguredSecurityBuilder(ObjectPostProcessor<Object> objectPostProcessor, boolean allowConfigurersOfSameType) {
        this.logger = LogFactory.getLog(this.getClass());
        // 一个映射表，维护class-configurer键值对
        this.configurers = new LinkedHashMap();
        // 研究HttpSecurity再看
        this.configurersAddedInInitializing = new ArrayList();
        // 应该是处理共享配置数据的，研究HttpSecurity再看
        this.sharedObjects = new HashMap();
        // build状态，是个枚举类
        this.buildState = AbstractConfiguredSecurityBuilder.BuildState.UNBUILT;
        Assert.notNull(objectPostProcessor, "objectPostProcessor cannot be null");
        this.objectPostProcessor = objectPostProcessor;
        // 一般是false
        this.allowConfigurersOfSameType = allowConfigurersOfSameType;
    }
```

ObjectPostProcessor:是一个接口，我们在学习Spring Security Filter源码的时候，会发现，相关的Filter都是以new的形式创建的，作为Java服务端开发，我们知道，new出来的对象是不受Spring Bean生命周期管控的。为了使这些Filter实例能够被Spring Bean容器管理，Spring Security定义了一个接口ObjectPostProcessor，AutowireBeanFactoryObjectPostProcessor实现了这个接口，通过postProcess方法手把Bean注入到spring容器进行管理。ObjectPostProcessor在Spring Security中广泛应用，我们这里就以HeaderWriterFilter为例

```Java
@Override
public void configure(HttpSecurity http) throws Exception {
    http.headers()
        .addObjectPostProcessor(new ObjectPostProcessor<HeaderWriterFilter>() {
            @Override
            public HeaderWriterFilter postProcess(HeaderWriterFilter filter) {
				//....
            }
        });
}
```

在headers方法返回的HeaderConfigurer中，HeaderWriterFilter确实是new出来的，接着调用了postProcess()方法，进行后置处理objectPostProcessor是一个复合的对象后置处理器，内部维护一个ObjectPostProcessor List, 调用postProcess()时，遍历ObjectPostProcessor List逐一处理传入的对象。这个List是在默认情况下，只包含一个ObjectPostProcessor，实现就是AutowireBeanFactoryObjectPostProcessor，但是我们自己配置了http.headers().addObjectPostProcessor(…)，因此还有一个匿名ObjectPostProcessor。

AbstractConfiguredSecurityBuilder的两个关键方法如下：

```Java

	// 将配置类对象存储根据配置类种类整理到configurers中，以便后来统一初始化配置
	private <C extends SecurityConfigurer<O, B>> void add(C configurer) throws Exception {
        Assert.notNull(configurer, "configurer cannot be null");
        Class<? extends SecurityConfigurer<O, B>> clazz = configurer.getClass();
        synchronized(this.configurers) {
            if (this.buildState.isConfigured()) {
                throw new IllegalStateException("Cannot apply " + configurer + " to already built object");
            } else {
                List<SecurityConfigurer<O, B>> configs = this.allowConfigurersOfSameType ? (List)this.configurers.get(clazz) : null;
                if (configs == null) {
                    configs = new ArrayList(1);
                }

                ((List)configs).add(configurer);
                this.configurers.put(clazz, configs);
                if (this.buildState.isInitializing()) {
                    this.configurersAddedInInitializing.add(configurer);
                }

            }
        }
  	}
	// 返回配置类
	public <C extends SecurityConfigurer<O, B>> List<C> getConfigurers(Class<C> clazz) {
        List<C> configs = (List)this.configurers.get(clazz);
        return configs == null ? new ArrayList() : new ArrayList(configs);
    }

```

接下来就是doBuild方法：

```Java
	protected final O doBuild() throws Exception {
        synchronized(this.configurers) {
            this.buildState = AbstractConfiguredSecurityBuilder.BuildState.INITIALIZING;
            this.beforeInit();
            this.init();
            this.buildState = AbstractConfiguredSecurityBuilder.BuildState.CONFIGURING;
            this.beforeConfigure();
            this.configure();
            this.buildState = AbstractConfiguredSecurityBuilder.BuildState.BUILDING;
            O result = this.performBuild();
            this.buildState = AbstractConfiguredSecurityBuilder.BuildState.BUILT;
            return result;
        }
    }

	// 预留方法，没有实现
    protected void beforeInit() throws Exception {
    }
    protected void beforeConfigure() throws Exception {
    }

    protected abstract O performBuild() throws Exception;
	
	// 挨个调用配置类对象放入init方法进行初始化
    private void init() throws Exception {
        Collection<SecurityConfigurer<O, B>> configurers = this.getConfigurers();
        Iterator var2 = configurers.iterator();

        SecurityConfigurer configurer;
        while(var2.hasNext()) {
            configurer = (SecurityConfigurer)var2.next();
            configurer.init(this);
        }

        var2 = this.configurersAddedInInitializing.iterator();

        while(var2.hasNext()) {
            configurer = (SecurityConfigurer)var2.next();
            configurer.init(this);
        }

    }
	// 挨个调用配置类对象的configure方法进行配置
    private void configure() throws Exception {
        Collection<SecurityConfigurer<O, B>> configurers = this.getConfigurers();
        Iterator var2 = configurers.iterator();

        while(var2.hasNext()) {
            SecurityConfigurer<O, B> configurer = (SecurityConfigurer)var2.next();
            configurer.configure(this);
        }

    }
```

preformBuild方法是真正的过滤器链构造方法，留给子类实现。

##### HttpSecurity

HttpSecurity做的事情，就是进行各种各样的configurer配置

```java
	private <C extends SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity>> C getOrApply(C configurer) throws Exception {
        C existingConfig = (SecurityConfigurerAdapter)this.getConfigurer(configurer.getClass());
        return existingConfig != null ? existingConfig : this.apply(configurer);
    }
```

这里的apply方法（在AbstractConfiguredSecurityBuilder方法中）最终会调用到AbstractConfiguredSecurityBuilder.add方法，将当前配置收集起来。

然后getOrApply方法会在HttpSecurity的各种配置方法中被调用

```Java
public CorsConfigurer<HttpSecurity> cors() throws Exception {
    return getOrApply(new CorsConfigurer<>());
}

public CsrfConfigurer<HttpSecurity> csrf() throws Exception {
    ApplicationContext context = getContext();
    return getOrApply(new CsrfConfigurer<>(context));
}

public ExceptionHandlingConfigurer<HttpSecurity> exceptionHandling() throws Exception {
    return getOrApply(new ExceptionHandlingConfigurer<>());
}
```

HttpSecurity的初始化器如下：

```Java
public final class HttpSecurity extends AbstractConfiguredSecurityBuilder<DefaultSecurityFilterChain, HttpSecurity> implements SecurityBuilder<DefaultSecurityFilterChain>, HttpSecurityBuilder<HttpSecurity> {
    private final HttpSecurity.RequestMatcherConfigurer requestMatcherConfigurer;
    // 过滤器列表
    private List<Filter> filters = new ArrayList();
    private RequestMatcher requestMatcher;
    private FilterComparator comparator;

    public HttpSecurity(ObjectPostProcessor<Object> objectPostProcessor, AuthenticationManagerBuilder authenticationBuilder, Map<Class<? extends Object>, Object> sharedObjects) {
        super(objectPostProcessor);
        this.requestMatcher = AnyRequestMatcher.INSTANCE;
        this.comparator = new FilterComparator();
        Assert.notNull(authenticationBuilder, "authenticationBuilder cannot be null");
        // AuthenticationManagerBuilder对象是共享的
        this.setSharedObject(AuthenticationManagerBuilder.class, authenticationBuilder);
        Iterator var4 = sharedObjects.entrySet().iterator();

        while(var4.hasNext()) {
            Entry<Class<? extends Object>, Object> entry = (Entry)var4.next();
            this.setSharedObject((Class)entry.getKey(), entry.getValue());
        }

        ApplicationContext context = (ApplicationContext)sharedObjects.get(ApplicationContext.class);
        this.requestMatcherConfigurer = new HttpSecurity.RequestMatcherConfigurer(context);
    }
    
    // .....
}
```

RequestMatcher接口只声明了一个方法matches(HttpServletRequest var1)。结合上文的SecurityFilterChain接口我们不难猜测RequestMatcher负责FilterChain的匹配工作。AnyRequestMatcher是RequestMatcher的实现类，它的matches永远返回true。FilterComparator实现了Comparator<Filter>接口，其具体实现后面讨论。

阅读HttpSecurity的源码我们可以看出，其配置都是通过配置类实现的，配置类后面详细讨论，而在配置类中创建了具体的Filter。这些配置类的整理、初始化和配置都交给了父类AbstractConfiguredSecurityBuilder处理，这在上文中已经详细讨论过。

关于Filter的两个重要的方法addFilterAfter和addFilterBefore涉及comparator，希望留在后面讲。

注意addFilter方法：

```Java
	public HttpSecurity addFilter(Filter filter) {
        Class<? extends Filter> filterClass = filter.getClass();
        if (!this.comparator.isRegistered(filterClass)) {
            throw new IllegalArgumentException("The Filter class " + filterClass.getName() + " does not have a registered order and cannot be added without a specified order. Consider using addFilterBefore or addFilterAfter instead.");
        } else {
            this.filters.add(filter);
            return this;
        }
    }
```

这个 addFilter 方法的作用，主要是在各个 xxxConfigurer 进行配置的时候，会调用到这个方法，（xxxConfigurer 就是用来配置过滤器的），把 Filter 都添加到 fitlers 变量中

最终在HttpSecurity的performBuild方法中，构建出来一个过滤器链：

```Java
	protected DefaultSecurityFilterChain performBuild() throws Exception {
        Collections.sort(this.filters, this.comparator);
        return new DefaultSecurityFilterChain(this.requestMatcher, this.filters);
    }
```

献给过滤器排序，然后构造DefaultSecurityChain对象。

##### 配置类

设置HttpSecuity相应的配置有很多，它们都调用了getOrApply方法，这个方法会调用apply方法，传入对应的配置类对象，如下：

```Java
	public <C extends SecurityConfigurerAdapter<O, B>> C apply(C configurer) throws Exception {
        // 添加容器的objectPostProcessor，用来将new出来的filter注入到bean中
        configurer.addObjectPostProcessor(this.objectPostProcessor);
        configurer.setBuilder(this);
        // 管理configurer
        this.add(configurer);
        return configurer;
    }
```

对于配置类，大部分配置类的继承关系如下面的例子所示：

```Java
public class CorsConfigurer<H extends HttpSecurityBuilder<H>> extends AbstractHttpConfigurer<CorsConfigurer<H>, H> {
    // ......
}
```

AbstractHttpConfigurer是一个非常扯淡的抽象类，它又继承自SecurityConfigurerAdapter,SecurityConfigurerAdapter实现了接口SecurityConfigurer，我们可以看到，它们都和SecurityBuilder、HttpSecurityBuilder扯上了关系，通过上文可知SecurityBuilder是用来构建过滤器链的。而HttpSecurityBuilder构建了HttpSecurity。

```Java
public interface SecurityConfigurer<O, B extends SecurityBuilder<O>> {
    void init(B var1) throws Exception;

    void configure(B var1) throws Exception;
}

public abstract class AbstractHttpConfigurer<T extends AbstractHttpConfigurer<T, B>, B extends HttpSecurityBuilder<B>> extends SecurityConfigurerAdapter<DefaultSecurityFilterChain, B> {
    public AbstractHttpConfigurer() {
    }

    public B disable() {
        ((HttpSecurityBuilder)this.getBuilder()).removeConfigurer(this.getClass());
        return (HttpSecurityBuilder)this.getBuilder();
    }

    public T withObjectPostProcessor(ObjectPostProcessor<?> objectPostProcessor) {
        this.addObjectPostProcessor(objectPostProcessor);
        return this;
    }
}
```

SecurityConfigurer一目了然，两个函数init和configure分别用来初始化和配置SecurityBuilder。作为实现了SecurityConfigurer接口的抽象类SecurityConfigurerAdapter代码很长，init和configure仍然交给子类实现，

但其内部定义了一个CompositeObjectPostProcessor类，实现了ObjectPostProcessor接口，这个实现类在上文提到过，是一个复合类，里面维护了一个ObjectPostProcessor列表，其postProcess方法挨个调用列表中ObjectPostProcessor的postProcess方法。但是CompositeObjectPostProcessor的addObjectPostProcessor要注意，它在列表中添加了参数给予的ObjectPostProcessor对象后，根据注释顺序对ObjectPostProcessor进行了排序。

```Java
private boolean addObjectPostProcessor(ObjectPostProcessor<? extends Object> objectPostProcessor) {
    boolean result = this.postProcessors.add(objectPostProcessor);
    Collections.sort(this.postProcessors, AnnotationAwareOrderComparator.INSTANCE);
    return result;
}
```

除此之外，SecurityConfigurerAdapter定义了一个and方法，这个方法是不是很熟悉？AbstractHttpConfigurer在此基础上添加了注销配置的方法。

##### FilterComparator

首先了解一下FilterComparator维护的数据结构

```Java
final class FilterComparator implements Comparator<Filter>, Serializable {
    private static final int INITIAL_ORDER = 100;
    private static final int ORDER_STEP = 100;
    private final Map<String, Integer> filterToOrder = new HashMap();
    
    private Integer getOrder(Class<?> clazz) {
        while(clazz != null) {
            Integer result = (Integer)this.filterToOrder.get(clazz.getName());
            if (result != null) {
                return result;
            }

            clazz = clazz.getSuperclass();
        }

        return null;
    }

    // 私有静态内部类,辅助管理Filter顺序的过滤器
    private static class Step {
        private int value;
        private final int stepSize;

        Step(int initialValue, int stepSize) {
            this.value = initialValue;
            this.stepSize = stepSize;
        }

        int next() {
            int value = this.value;
            this.value += this.stepSize;
            return value;
        }
    }
    
    // 负责将filter的类名和顺序注册到filterToOrder中
    private void put(Class<? extends Filter> filter, int position) {
        String className = filter.getName();
        this.filterToOrder.put(className, position);
    }
    
    FilterComparator() {
        FilterComparator.Step order = new FilterComparator.Step(100, 100);
        this.put(ChannelProcessingFilter.class, order.next());
        this.put(ConcurrentSessionFilter.class, order.next());
        // ....
    }
    
    private Integer getOrder(Class<?> clazz) {
        while(clazz != null) {
            Integer result = (Integer)this.filterToOrder.get(clazz.getName());
            if (result != null) {
                return result;
            }

            clazz = clazz.getSuperclass();
        }

        return null;
    }
}
```

getOrder方法看了就知道根据类名获取对应顺序，如果没有该类就返回其父类的顺序。

```Java
    // 比较两个Filter的顺序
	public int compare(Filter lhs, Filter rhs) {
        Integer left = this.getOrder(lhs.getClass());
        Integer right = this.getOrder(rhs.getClass());
        return left - right;
    }
	// 返回一个Filter是否被注册
    public boolean isRegistered(Class<? extends Filter> filter) {
        return this.getOrder(filter) != null;
    }

    public void registerAfter(Class<? extends Filter> filter, Class<? extends Filter> afterFilter) {
        Integer position = this.getOrder(afterFilter);
        if (position == null) {
            throw new IllegalArgumentException("Cannot register after unregistered Filter " + afterFilter);
        } else {
            this.put(filter, position + 1);
        }
    }

    public void registerAt(Class<? extends Filter> filter, Class<? extends Filter> atFilter) {
        Integer position = this.getOrder(atFilter);
        if (position == null) {
            throw new IllegalArgumentException("Cannot register after unregistered Filter " + atFilter);
        } else {
            this.put(filter, position);
        }
    }

    public void registerBefore(Class<? extends Filter> filter, Class<? extends Filter> beforeFilter) {
        Integer position = this.getOrder(beforeFilter);
        if (position == null) {
            throw new IllegalArgumentException("Cannot register after unregistered Filter " + beforeFilter);
        } else {
            this.put(filter, position - 1);
        }
    }
```

通过源码阅读，我们不难理解FilterComparator只是实现给各个过滤器类设定顺序，然后实现了比较接口。。

所以我们可以回到HttpSecurity的代码，看addFilter方法：

```Java
    public HttpSecurity addFilterAfter(Filter filter, Class<? extends Filter> afterFilter) {
        this.comparator.registerAfter(filter.getClass(), afterFilter);
        return this.addFilter(filter);
    }

    public HttpSecurity addFilterBefore(Filter filter, Class<? extends Filter> beforeFilter) {
        //现在比较器中确定顺序
        this.comparator.registerBefore(filter.getClass(), beforeFilter);
        return this.addFilter(filter);
    }

    public HttpSecurity addFilter(Filter filter) {
        Class<? extends Filter> filterClass = filter.getClass();
        // 如果过滤器类没有在比较器中注册过，那就抛出异常
        if (!this.comparator.isRegistered(filterClass)) {
            throw new IllegalArgumentException("The Filter class " + filterClass.getName() + " does not have a registered order and cannot be added without a specified order. Consider using addFilterBefore or addFilterAfter instead.");
        } else {
            // 将过滤器加入过滤器列表中
            this.filters.add(filter);
            return this;
        }
    }
```

我们可以看到，对于没有在比较器中注册的过滤器，比较器无法安排过滤器的执行顺序，所以抛出异常比较好；对于已经注册的过滤器类，需要确定其执行顺序，这里使用addFilterAfter和addFilterBefore方法。

#### ExpressionUrlAuthorizationConfigurer

继承关系如下

![](F:\mycode\knowledgeArrangement\web\框架\expressurl.png)

(未完待续。。。)

### 整合SpringSecurity和JWT实现认证和授权

#### JWT

JWT是JSON WEB TOKEN的缩写，它是基于 RFC 7519 标准定义的一种可以安全传输的的JSON对象，由于使用了数字签名，所以是可信任和安全的。

##### JWT的组成

- JWT token的格式：header.payload.signature
- header中用于存放签名的生成算法

```json
{"alg": "HS512"}
```

- payload中用于存放用户名、token的生成时间和过期时间

```json
{"sub":"admin","created":1489079981393,"exp":1489684781}
```

- signature为以header和payload生成的签名，一旦header和payload被篡改，验证将失败

```json
//secret为加密算法的密钥	
String signature = HMACSHA512(base64UrlEncode(header) + "." +base64UrlEncode(payload),secret)
```

##### JWT实现认证和授权的原理

- 用户调用登录接口，登录成功后获取到JWT的token；
- 之后用户每次调用接口都在http的header中添加一个叫Authorization的头，值为JWT的token；
- 后台程序通过对Authorization头中信息的解码及数字签名校验来获取其中的用户信息，从而实现认证和授权。

#### 整合SpringSecurity及JWT

在pom.xml中添加项目依赖

```xml
<!--SpringSecurity依赖配置-->	
<dependency>	
    <groupId>org.springframework.boot</groupId>	
    <artifactId>spring-boot-starter-security</artifactId>	
</dependency>	
<!--Hutool Java工具包-->	
<dependency>	
    <groupId>cn.hutool</groupId>	
    <artifactId>hutool-all</artifactId>	
    <version>4.5.7</version>	
</dependency>	
<!--JWT(Json Web Token)登录支持-->	
<dependency>	
    <groupId>io.jsonwebtoken</groupId>	
    <artifactId>jjwt</artifactId>	
    <version>0.9.0</version>	
</dependency>
```

#### 实现认证和授权步骤

1. **添加SpringSecurity的配置类**

   配置类继承自WebSecurityConfigurerAdapter，需要实现configure方法，在configure(HttpSecurity httpSecurity)方法中完成认证策略的配置；在configure(AuthenticationManagerBuilder auth)方法中设置认证数据细节服务（userDetailsService）

2. **自定义userDetailsService**

   userDetailsService是为了实现根据用户名查出用户的方法。可以通过实现UserDetailsService接口，重写loadUserByUsername方法来实现。loadUserBuUsername返回UseDetails对象，这是一个接口，也需要实现对应的用户信息类。

3. **在HttpSecurity的配置中添加JWT认证过滤器。**

   实现方法不唯一，一般思路是在UsernamePasswordAuthenticationFilter前添加过滤器，过滤器继承OncePerRequestFilter，表明对每隔Request只过滤一次。当Request到来时，提取请求头中的tokenHeader（Authorization）。获取token后解析用户名密码并和数据库中对应用户名密码进行匹配，如果匹配则自信根据token信息登录。简单实现例子如下：

   ```Java
    	@Override
       protected void doFilterInternal(HttpServletRequest request,
                                       HttpServletResponse response,
                                       FilterChain chain) throws ServletException, IOException {
           String authHeader = request.getHeader(this.tokenHeader);
           if (authHeader != null && authHeader.startsWith(this.tokenHead)) {
               String authToken = authHeader.substring(this.tokenHead.length());// The part after "Bearer "
               String username = jwtTokenUtil.getUserNameFromToken(authToken);
               LOGGER.info("checking username:{}", username);
               if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                   UserDetails userDetails = this.userDetailsService.loadUserByUsername(username);
                   if (jwtTokenUtil.validateToken(authToken, userDetails)) {
                       UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                       authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                       LOGGER.info("authenticated user:{}", username);
                       SecurityContextHolder.getContext().setAuthentication(authentication);
                   }
               }
           }
           chain.doFilter(request, response);
       }
   ```

### SecurityContextHolder

保留系统当前的安全上下文细节，其中就包括当前使用系统的用户信息。

每个用户都会有它的上下文，上下文SecurityContext保存在SecurityContextHolder中。

多用户系统，比如典型的Web系统，整个生命周期可能同时有多个用户在使用。这时候应用需要保存多个SecurityContext（安全上下文），需要利用ThreadLocal进行保存，每个线程都可以利用ThreadLocal获取其自己的SecurityContext，及安全上下文。多用户系统，比如典型的Web系统，整个生命周期可能同时有多个用户在使用。这时候应用需要保存多个SecurityContext（安全上下文），需要利用ThreadLocal进行保存，每个线程都可以利用ThreadLocal获取其自己的SecurityContext，及安全上下文。

由源码可知，SecurityContextHolder利用了一个SecurityContextHolderStrategy（存储策略）进行上下文的存储。

SecurityContextHolderStrategy是一个接口：

```Java
package org.springframework.security.core.context;

public interface SecurityContextHolderStrategy {
    void clearContext();

    SecurityContext getContext();

    void setContext(SecurityContext var1);

    SecurityContext createEmptyContext();
}
```

SecurityContextHolder根据策略配置选择具体的实现类。SecurityContextHolder有三个实现类ThreadLocalSecurityContextHolderStrategy、InheritableThreadLocalSecurityContextHolderStrategy和GlobalSecurityContextHolderStrategy。当然也可以自定义。通过名字就可以知道前两个分别是通过ThreadLocal、InheritableThreadLocal实现的。

ThreadLocalSecurityContextHolderStrategy通过ThreadLocal实现为每个线程存取对应的SecuirtyContext。而SecurityContextHolder将SecurityContext的维护委托给了静态属性SecurityContextHolderStrategy，对SecurityContextHolder的使用为直接使用其静态方法实现SecurityContext的存取。

而SecuirtyContext是一个接口：

```Java
import java.io.Serializable;
import org.springframework.security.core.Authentication;

public interface SecurityContext extends Serializable {
    Authentication getAuthentication();

    void setAuthentication(Authentication var1);
}
```

而Authentication也是一个接口,GrantedAuthority就是权限对象嘛：

```Java
public interface Authentication extends Principal, Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();

    Object getCredentials();

    Object getDetails();

    Object getPrincipal();

    boolean isAuthenticated();

    void setAuthenticated(boolean var1) throws IllegalArgumentException;
}
```

常用的实现了Authentication接口的认证类有UsernamePasswordAuthenticationToken