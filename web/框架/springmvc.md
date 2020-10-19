### MVC模式

Model-View-Controller（模型-视图-控制器）模式，这种模式用于应用程序的分层开发

- Model）：模型代表一个存取数据的对象或者Java POJO。它也可以带有逻辑，在数据变化时更新控制器
- View：视图代表模型包含的数据的可视化
- Controller：作用于模型和视图上。他控制数据流向模型对象，并在数据变化时更新视图。它使视图于模型分开。

Spring MVC架构如下：

![](F:\mycode\knowledgeArrangement\web\springmvc.png)

### Spring MVC组件与流程

​	SpringMVC框架是围绕着DispatcherServlet工作的。在Servlet初始化（调用init方法）时，SpringMVC会根据配置，获取配置信息，从而得到统一资源描述符（URI）和处理器（Handler）之间的映射关系，为了更加灵活和增强功能，SpringMVC还给处理器增加拦截器，所以还可以在处理器执行前后加入自己的代码，这样就构成了一个处理器的执行链，并且根据上下文初始化视图解析器等内容，当处理器返回的时候就可以通过视图解析器定位视图，然后将数据模型渲染到视图中，用来响应用户的请求。

​	当一个请求到来时，DispatcherServlet首先通过请求和事先解析好的HandlerMapping配置，找到对应的处理器（Handler）这样就准备开始运行处理器和拦截器组成的执行链，而运行处理器需要有一个对应的环境，这样它就有了一个处理器的适配器（HandlerAdapter），通过这个适配器就能运行对应的处理器和拦截器，这里的处理器包含了控制器的内容和其他的增强功能，在处理器返回模型和视图给DispatcherServlet后，DispatcherServlet就会把对应的视图信息传递给视图解析器。最后由视图解析器把模型渲染到视图中去。

![](F:\mycode\knowledgeArrangement\web\liucheng.png)

### web.xml

web应用的根目录下，必须有一个WEB-INF目录，WEB-INF目录下有一个web.xml，同时还可以有classes和lib目录

classes存放web应用需要的class文件

lib目录存放web引用需要的其他类库

JSP文件和其他资源文件放在当前web应用根目录及其子目录下

WEB-INF目录是用户无法访问的目录

web.xml中的所有元素出现的先后顺序和次数等规则都下载XML Schema文件中

```xml
 <?xml version="1.0" encoding="UTF-8"?>
 <web-app xmlns="http://java.sun.com/xml/ns/javaee"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                              http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
          version="2.5">
     <!--配置Spring IoC配置文件路径-->
     <context-param>
         <param-name>contextConfigLocation</param-name>
         <param-value>/WEB-INF/applicationContext.xml</param-value>
     </context-param>
     <!--配置ContextLoaderListener用以初始化Spring IoC容器-->
     <listener>
         <listener-class>org.springframework.web.Context.ContextLoaderListener</listener-class>
     </listener>
     <!--配置DispacherServlet-->
     <servlet>
         <!--注意：Spring MVC框架会根据servlet-name配置，找到/WEB-INF/dispatcher-servlet.xml作为配置文件载入web工程中-->
         <servlet-name>dispatcher</servlet-name>
         <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
         <!--使得Dispatcher在服务器启动的时候就初始化-->
         <load-on-startup>2</load-on-startup>
     </servlet>
     <servlet-mapping>
         <selvlet-name>dispatcher</selvlet-name>
         <url-pattern>*.do</url-pattern>
     </servlet-mapping>
 </web-app>
```

**web.xml中各个标记的出现顺序**：

- web-app：根元素
- context-param：Servlet中的ServletContext对象读取的初始参数，以及JSP中的application内置对象读取的初始参数
- filter：过滤器
- filter-mapping：过滤器需要过滤的资源，下设的url-pattern的数目不受限制，可以使用*通配符
- listener：监听器
- servlet：配置selvlet
- servlet-mapping：配置servlet访问路径
- session-config：配置session、cookie等的相关信息
- mime-mapping：mime类型映射到扩展名，用于规定下载格式

contextConfigLocation配置会告诉Spring MVC其Spring IoC的配置文件在哪里，这样Spring会找到并配置，如果是多个配置文件，就可以用逗号将他们分隔开来，默认值为/WEB-INF/applicationContext.xml

ContextLoaderListener实现了接口ServletContextListener，ServletContextListener在整个web工程前后加入自定义代码，所以可以在Web工程初始化之前，它先完成对Spring IoC容器的初始化，也可以在Web工程关闭之时释放Spring IoC容器的资源。

配置DispatcherServlet拦截以后以后缀“do”结束的请求

根据上文的web.xml文件，它还会加载一个/WEB-INF/dispatcher-servlet.xml文件，他是与SpringMVC配置相关的内容，实例如下：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
                           http://www.springframework.org/schema/tx
                           http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context-4.0.xsd
                           http://www.springframework.org/schema/mvc
                           http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">
    <!--使用注解驱动-->
    <mvc:annotation-driven />
    <!--定义扫描装载的包-->
    <context:component-scan bease-package="com.*" />
    <!--定义视图解析器-->
    <!--找到web工程/WEB-INF/JSP文件夹，且文件结尾为jsp的文件作为映射-->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver" p:prefix="/WEB-INF/jsp/" p:suffix=".jsp" />
    <!--如果有配置数据库事务，需要开启注解事务的，需要开启这段代码-->
    <tx:annotation-driven transaction-manager="transactionManager" />
</beans>
```

mvc:annotation-driven表示使用注解驱动SpringMVC

component-scan定义扫描的包，加载对应的控制器和其他一些组件

viewResolver定义视图解析器，

@Controller是一个控制器，SpringMVC扫描的时候就会把它作为控制器加载进来。@RequestMapping制定了对应的请求的URI，SpringMVC在初始化后就会将这些信息解析，存放起来，于是便有了HandlerMapping。当发生请求时，SpringMVC就回去使用这些信息去找到对应的控制器提供服务。

![实例组件和运行流程](F:\mycode\knowledgeArrangement\web\yunxinliucheng.png)

### ServletContext

ServletContext官方叫servlet上下文。服务器会为每一个工程创建一个对象，这个对象就是ServletContext对象。这个对象全局唯一，而且工程内部的所有servlet都共享这个对象。所以叫全局应用程序共享对象。

作用：

1. 是一个域对象
2. 可以读取全局配置参数
3. 可以搜索当前工作目录下面的资源文件
4. 可以获取当前工程名字

### Spring MVC初始化

#### 初始化Spring IoC上下文

Java Web容器为其生命周期提供ServletContextListener接口，这个接口可以在Web容器初始化和结束期中执行一定的逻辑，只要实现ServletContextListener接口的方法就可以了。源码如下：

```Java
package org.springframework.web.context;

import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;

public class ContextLoaderListener extends ContextLoader implements ServletContextListener {
    public ContextLoaderListener() {
    }

    public ContextLoaderListener(WebApplicationContext context) {
        super(context);
    }

    public void contextInitialized(ServletContextEvent event) {
        // 初始化Spring IoC容器，使用的是满足ApplicationContext接口的Spring Web IoC容器
        this.initWebApplicationContext(event.getServletContext());
    }

    public void contextDestroyed(ServletContextEvent event) {
        // 关闭Web IoC容器
        this.closeWebApplicationContext(event.getServletContext());
        // 清除相关参数
        ContextCleanupListener.cleanupAttributes(event.getServletContext());
    }
}
```

​	在Servlet API中有一个javax.servlet.ServletContextListener接口，它能够监听ServletContext对象的生命周期，实际上就是监听Web应用的生命周期。

​	当Servlet容器启动或终止Web应用时，会触发ServletContextEvent事件，该事件由 ServletContextListener 来处理。在 ServletContextListener 接口中定义了处理ServletContextEvent事件的两个方法：

- contextInitialized(ServletContextEvent sce)：当Servlet容器启动Web应用时调用该方法。在调用完该方法之后，容器再对Filter初始化，并且对那些在Web应用启动时就需要被初始化的Servlet进行初始化
- contextDestroyed(ServletContextEvent sce)：当Servlet容器终止Web应用时调用该方法。在调用该方法之前，容器会先销毁所有的Servlet和Filter过滤器

这样通过ContextLoaderListener在DispatcherServlet初始化前王城Spring IoC容器的初始化，在结束期间完成对Spring IoC容器的销毁。

#### 初始化映射请求上下文

​	通过DispatcherServlet初始化，可以根据需要配置它在启动时初始化，或者等待用户第一次请求的时候初始化。如果在你的Web工程中没有注册ContextLoaderListener，这个时候DispatcherServlet就会在其初始化的时候进行对Spring IoC容器的初始化。

​	大部分场景下，都应该让DispatcherServlet在服务器启动期间就完成Spring IoC容器的初始化，因为不仅Spring IoC容器初始化是一个耗时的操作，而且在整个web的初始化中，不只是DispatcherServlet需要使用到Spring IoC资源，其他组件也可能需要。

DispatcherServlet的设计图

![](F:\mycode\knowledgeArrangement\web\dispatcher.png)

从上图可以看出DispatcherServlet是一个可以载入Web容器中Servlet

Web容器对于Servlet的初始化，首先是调用init方法，这个方法在HttpServletBean里

```Java
	// 该方法在HttpServletBean里
	public final void init() throws ServletException {
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Initializing servlet '" + this.getServletName() + "'");
        }
		
        // 根据参数初始化bean的属性
        try {
            PropertyValues pvs = new HttpServletBean.ServletConfigPropertyValues(this.getServletConfig(), this.requiredProperties);
            BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
            ResourceLoader resourceLoader = new ServletContextResourceLoader(this.getServletContext());
            bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, this.getEnvironment()));
            this.initBeanWrapper(bw);
            bw.setPropertyValues(pvs, true);
        } catch (BeansException var4) {
            if (this.logger.isErrorEnabled()) {
                this.logger.error("Failed to set bean properties on servlet '" + this.getServletName() + "'", var4);
            }

            throw var4;
        }
		// 此方法交给子类实现
        this.initServletBean();
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Servlet '" + this.getServletName() + "' configured successfully");
        }
    }
	
	// 在FrameworkServlet里
	protected final void initServletBean() throws ServletException {
        this.getServletContext().log("Initializing Spring FrameworkServlet '" + this.getServletName() + "'");
        if (this.logger.isInfoEnabled()) {
            this.logger.info("FrameworkServlet '" + this.getServletName() + "': initialization started");
        }

        long startTime = System.currentTimeMillis();

        try {
            // 初始化Spring IoC容器
            this.webApplicationContext = this.initWebApplicationContext();
            this.initFrameworkServlet();
        } catch (ServletException var5) {
            this.logger.error("Context initialization failed", var5);
            throw var5;
        } catch (RuntimeException var6) {
            this.logger.error("Context initialization failed", var6);
            throw var6;
        }

        if (this.logger.isInfoEnabled()) {
            long elapsedTime = System.currentTimeMillis() - startTime;
            this.logger.info("FrameworkServlet '" + this.getServletName() + "': initialization completed in " + elapsedTime + " ms");
        }

    }

	protected WebApplicationContext initWebApplicationContext() {
        // ServletContext是Servlet上下文，该对象全局唯一，所有servlet共享
        WebApplicationContext rootContext = WebApplicationContextUtils.getWebApplicationContext(this.getServletContext());
        WebApplicationContext wac = null;
        // 判断上下文是否已经被初始化
        if (this.webApplicationContext != null) {
            wac = this.webApplicationContext;
            // 如果Web IoC容器已经在启动的时候初始化，那么就沿用它
            if (wac instanceof ConfigurableWebApplicationContext) {
                ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext)wac;
                if (!cwac.isActive()) {
                    // 如果父容器为空，将web Ioc容器下挂在根应用环境下
                    if (cwac.getParent() == null) {
                        cwac.setParent(rootContext);
                    }

                    this.configureAndRefreshWebApplicationContext(cwac);
                }
            }
        }
		// 没有初始化，则查找是否由存在的Spring IoC容器
        if (wac == null) {
            wac = this.findWebApplicationContext();
        }
		// 如果没有初始化，也没有查到，则DispatcherServlet自己创建它
        if (wac == null) {
            wac = this.createWebApplicationContext(rootContext);
        }
		// 当onRefresh方法没有被调用过，执行onRefresh方法
        if (!this.refreshEventReceived) {
            // 这是给子类实现的
            this.onRefresh(wac);
        }
		// 作为Servlet的上下文属性发布IoC容器
        if (this.publishContext) {
            String attrName = this.getServletContextAttributeName();
            this.getServletContext().setAttribute(attrName, wac);
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Published WebApplicationContext of servlet '" + this.getServletName() + "' as ServletContext attribute with name [" + attrName + "]");
            }
        }

        return wac;
    }

	protected void onRefresh(ApplicationContext context) {
        this.initStrategies(context);
    }

    protected void initStrategies(ApplicationContext context) {
        // 初始化文件解析器
        this.initMultipartResolver(context);
        // 本地解析化
        this.initLocaleResolver(context);
        // 主题解析
        this.initThemeResolver(context);
        // 处理器映射
        this.initHandlerMappings(context);
        // 处理器的适配器
        this.initHandlerAdapters(context);
        // Handler的异常处理解析器
        this.initHandlerExceptionResolvers(context);
        // 当处理器没有返回逻辑视图名等相关信息时，自动将请求URL映射为逻辑视图名
        this.initRequestToViewNameTranslator(context);
        // 视图逻辑名称转化器，允许返回逻辑视图名称，它会找到真实的视图
        this.initViewResolvers(context);
        // 关注Flash开发的Map管理器
        this.initFlashMapManager(context);
    }
```

- MultipartResolver：文件解析器，用于支持服务器的文件上传
- LocaleResolver：国际化解析器，可以提供国际化的功能
- ThemeResolver：主题解析器，类似于软件皮肤的转换功能
- HandlerResolver：它会包装用户提供一个控制器的方法和对它的一些拦截器
- HandlerAdapter：处理器适配器，因为处理器会在不同的上下文中运行，所以Spring MVC会先找到合适的适配器，然后运行处理器服务方法，比如对于控制器的SimpleControllerHandlerAdapter、对于普通请求的HttpRequestHandlerAdapter。
- HandlerExceptionResolver：处理器异常解析器
- RequestToViewNameTranslator：视图逻辑名称转换器，在控制器中返回一个视图的名称，通过它可以找到实际的视图
- ViewResolver：视图解析器，当控制器返回后，通过视图解析器会把逻辑视图名称进行解析，然后定位实际视图

对于这些组件DispatcherServlet会根据其配置文件DispatcherServlet.properties进行初始化

在启动期间DispatcherServlet会加载这些配置的组件进行初始化。

#### 使用注解配置方式初始化

​	在Servlet3.0后得规范允许取消web.xml配置，只是用注解方式就可以了。需要继承AbstractAnnotationConfigDispatcherServletInitializer，然后实现它所定义的方法

```java
package com.ssm.chapter14.config;

import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    @Override
    protected Class<?>[] getRootConfigClasses() {
        // 返回Spring Web IoC的Java配置类数组
        return new Class<?>[] {};
    }
    
    @Override 
    protected Class<?>[] getServletConfigClasses() {
        // 返回DispatcherServlet的URI映射关系配置类数组
        return new Class<?>[] {WebConfig.class};
    }
    
    // DispatcherServlet拦截内容
    @Override
    protected String[] getServletMappings() {
        return new String[] {"*.do"};
    }
}

// WebConfig.java
package com.ssm.chapter14.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

@Configuration
@ComponentScan("com.*")
// 启用Spring Web MVC
@EnableWebMvc
public class WebConfig {
    
    @Bean(name="viewResolver")
    public ViewResolver initViewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/jsp/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }
}
```

SpringMVC会使用实现了ServletContainerInitializer接口的SpringServletContainerInitializer来加载实现了WebApplicationInitializer接口的初始化器了（AbstractAnnotationConfigDispatcherServletInitializer继承了WebApplicationInitializer）。其实现如下：

```Java
// 定义初始化器的类型，只要实现WebApplicationInitializer接口则为初始化器
@HandlesTypes({WebApplicationInitializer.class})
public class SpringServletContainerInitializer implements ServletContainerInitializer {
    public SpringServletContainerInitializer() {
    }

    public void onStartup(Set<Class<?>> webAppInitializerClasses, ServletContext servletContext) throws ServletException {
        // 允许多个初始化器
        List<WebApplicationInitializer> initializers = new LinkedList();
        Iterator var4;
        // 从各个包中加载对应的初始化器
        if (webAppInitializerClasses != null) {
            var4 = webAppInitializerClasses.iterator();
            while(var4.hasNext()) {
                Class<?> waiClass = (Class)var4.next();
                if (!waiClass.isInterface() && !Modifier.isAbstract(waiClass.getModifiers()) && WebApplicationInitializer.class.isAssignableFrom(waiClass)) {
                    try {
                        // 创建初始化器实例
                        initializers.add((WebApplicationInitializer)waiClass.newInstance());
                    } catch (Throwable var7) {
                        throw new ServletException("Failed to instantiate WebApplicationInitializer class", var7);
                    }
                }
            }
        }

        if (initializers.isEmpty()) {
            servletContext.log("No Spring WebApplicationInitializer types detected on classpath");
        } else {
            servletContext.log(initializers.size() + " Spring WebApplicationInitializers detected on classpath");
            AnnotationAwareOrderComparator.sort(initializers);
            var4 = initializers.iterator();
			// 调用各个初始化器的onStartup方法
            while(var4.hasNext()) {
                WebApplicationInitializer initializer = (WebApplicationInitializer)var4.next();
                initializer.onStartup(servletContext);
            }

        }
    }
}
```

只要继承AbstractAnnotationConfigDispatcherServletInitializer类就完成了DispatcherServlet映射关系和Spring IoC容器的初始化工作的原因。

- getRootConfigClasses获取SpringIoC容器的Java配置类，用以装载各类Spring Bean
- getServletConfigClasses获取各类Spring MVC的URI和控制器的配置关系类，用以生成Web请求上下文
- getServletMappings定义DispatcherServlet拦截的请求。

### Spring MVC开发流程详解

主要是以一个注解@Controller标注，通过扫描配置就能将其扫描出来，还要结合注解@RequestMapping去配置它。@RequestMapping可以配置在类或者方法上，它的作用是指定URI和哪个类（或者方法）作为一个处理请求的处理器。为了更加灵活，SpringMVC还定义了处理器的拦截器，当启动Spring MVC的时候，Spring MVC还定义了处理器的拦截器，当启动Spring MVC时，Spring MVC就回去解析@Controller中的@RequestMapping的配置，再结合所配置的拦截器，这样它就会组成多个拦截器和一个控制器的形式，存放到HandlerMapping中去。当请求来到服务器，首先是通过请求信息找到对应的HandlerMapping，进而找到对应的拦截器和处理器，这样就能够运行对应的控制器和拦截器。

@RequestMapping源码如下：

```Java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {
    // 请求路径
    String name() default "";

    // 请求路径，可以是数组
    @AliasFor("path")
    String[] value() default {};

    @AliasFor("value")
    String[] path() default {};
	
    // 请求类型，GET、POST等
    RequestMethod[] method() default {};

    // 请求参数，当请求带有配置的参数时，才匹配处理器
    String[] params() default {};

    // 请求头，当http请求头为配置项时，才匹配处理器
    String[] headers() default {};
	
    // 请求类型为配置类型才匹配处理器
    String[] consumes() default {};
	
    // 处理器之后的响应用户的结果类型，比如{"application/json; charset-UYF-8", "text/plain", "application/*"}
    String[] produces() default {};
}
```

最常用的时请求路径和请求类型，其他大部分作为限定项，根据需要配置。

```Java
@RequestMapping(value="/index2", method=RequestMethod.GET)
public ModelAndView index2() {
    ModelAndView mv = new ModelAndView();
    mv.setViewName("index");
    return mv;
}
```

#### 控制器开发

控制器开发是Spring MVC的核心内容，其步骤一般分为3步：

- 获取请求参数
- 处理业务逻辑
- 绑定模型和视图

##### 获取请求参数

建议不要使用Servlet容器所给予的API，因为这样控制器将会依赖于Servlet容器。

可以使用注解@RequestParam来获取Http请求的参数

```Java
@RequestMapping(value="/index2", method=RequestMethod.GET)
public ModelAndView index2(@RequestParam("id") Long id) {
    System.out.println("params[id] = " + id);
    ModelAndView mv = new ModelAndView();
    mv.setViewName("index");
    return mv;
}
```

默认情况下@RequestParam要求参数不为空，但可以使用配置项设置：

- required是boolean值，默认为true，不允许参数为空
- defaultValue

获取Session可以使用@SessionAttribute

```Java
@RequestMapping(value="/index3", method=RequestMethod.GET)
public ModelAndView index2(@SessionAttribute("userName") String userName) {
    System.out.println("session[userName] = " + userName);
    ModelAndView mv = new ModelAndView();
    mv.setViewName("index");
    return mv;
}
```

##### 实现逻辑和绑定视图

```Java
package com.ssm.chapter14.controller;
@Controller
@RequestMapping("/role")
public class RoleController {
    // 注入角色服务类
    @Autowired
    private RoleService roleService = null;
    
    @RequestMapping(value="/getRole", method=RequestMethod.GET)
    public ModelAndView getRole(@RequestParam("id") Long id) {
        Role role = roleService.getRole(id);
        ModeAndView mv = new ModelAndView();
        mv.setViewName("roleDetails");
        mv.addObject("role", role);
        return mv;
    }
}
```

#### 视图渲染

Spring MVC默认使用JstlView进行渲染，将查询出来的模型绑定到JSTL（JSP标准库标签）模型中，通过JSTL就可以把数据模型在JSP中读出展示数据了。

可以返回JSON数据给前端使用

```Java
// 获取角色
@RequestMapping(value="/getRole", method=RequestMethod.GET)
public ModelAndView getRole(@RequestParam("id") Long id) {
    Role role = roleService.getRole("id");
    ModelAndView mv = new ModelAndView();
    mv.addObject("role", role);
    // 指定视图类型
    mv.setView(new MappingJackson2JsonView());
    return mv;
}
```

### 深入Spring MVC组件开发

#### 控制器接收各类请求参数

SpringMVC提供了许多的注解来解析参数，其目的在于把控制器从复杂的Servlet API中剥离，这样就可以在非Web容器环境中重用控制器，也方便测试人员对其进行有效测试。

##### 接受普通请求参数

```Java
@RequestMapping("/commonParams")
public ModelAndView commonParams(String roleName, String note) {
    // ....
    ModelAndView mv = new ModelAndView();
    mv.setViewName("index");
    return mv;
}
```

通过参数名称和HTTP请求参数的名称一致来获取参数，如果不一致就无法获取，这样的方式允许参数为空。

```Java
public class RoleParams {
    private String roleName;
    private String note;
    
    /**** setter and getter ****/
}

@RequestMapping("/commonParamPojo")
public ModelAndView commonParamPojo(RoleParams roleParams) {
    System.out.println("roleName =>" + roleParams.getRoleName());
    System.out.println("note =>" + roleParams.getNote());
    ModelAndView mv = new ModelAndView();
    mv.setViewName("index");
    return mv;
}
```

在灭有任何注解的情况下，SpringMVC也有映射POJO的能力。

##### 使用RequestParam注解获取参数

```Java
@RequestMapping("/requestParam")
public ModelAndView requestParam(@RequestParam("role_name") String roleName, String note) {
    // ....
    
   	ModelAndView mv = new ModelAndView();
    mv.setViewName("index");
    return mv;
}

// 如果允许参数为空，可是设置配置项required
@RequestParam(value="role_name", required=false)
```

##### 使用URL传递参数

```Java
@Autowired
RoleService roleService;

@RequestMapping("/getRole/{id}")
public ModelAndView pathVariavle(@PathVariable("id") Long id) {
    Role role = roleService.getRole(id);
    ModelAndView mv = new ModelAndView();
    mv.addObject(role);
    // 设置为JSON视图
    mv.setView(new MappingJackson2JsonView());
    return mv;
}
```

##### 传递JSON参数

```Java
// 分页参数
public class PageParams {
    private int start;
    private int limit;
    // setter and getter
}

public class RoleParams {
    private String roleName;
    private String note;
    private PageParams pageParams = null;
    // setter and getter
}

// 使用@RequestBody接收JSON参数,@RequestBody处理post请求
/*
接收的JSON文件
{
	roleName: 'role',
	note: 'note',
	pageParams: {
		start: 1,
		limit: 20
	}
}
*/
@RequestMapping("/findRoles")
public ModelAndView findRoles(@RequestBody RoleParams roleParams) {
    List<Role> roleList = roleService.findRoles(roleParams);
    ModelAndView mv = new ModelAndView();
    mv.addObject(roleList);
    // 设置为JSON视图
    mv.setView(new MappingJackson2JsonView());
    return mv;
}
```

##### 接收列表数据和表单序列化

```Java
// 使用JSON格式传递数组数据
// [1,2,3]
@RequestMapping("/deleteRoles")
// long[] 也行
public ModelAndView deleteRoles(@RequestBody List<Long> idList) {
    ModelAndView mv = new ModelAndView();
    int total = roleService.deleteRoles(idList);
    mv.addObject("total", total);
    mv.setView(new MappingJackson2JsonView());
    return mv;
}

/* [{roleName: 'role_name_1', note: 'note_1'},
	{roleName: 'role_name_2', notr: 'note_2'}] */

public ModelAndView addRoles(@RequestBody List<Role>roleList) {
    //....
}

// 表单序列化 roleName=xxx&&note=xxx传递
@RequestMapping("/commonParamPojo2")
public ModelAndView commonParamPojo2(String roleName, String note) {
    // ....
}
```

#### 重定向

```Java
@RequestMapping("/addRole")
public String addRole(Model model, String roleName, String note) {
    Role role = new Role();
    role.setRoleName(roleName);
    role.setNote(note);
    //回填id
    roleService.insertRole(role);
    model.addAttribute("roleName", roleName);
    model.addAttribute("note", note);
    model.addAttribute("id", role.getId());
    return "redirect:./showRoleJsonInfo.do";
}

@RequestMapping("addRole2")
public ModelAndView addRole2(ModelAndView mv, String roleName, String note) {
    Role role = new Role();
    role.setRoleName(roleName);
    role.setNote(note);
    //回填id
    roleService.insertRole(role);
    
    mv.addObject("roleName", roleName);
    mv.addObject("note", note);
    mv.addObject("id", role.getId());
    mv.setViewName("redirect:./showRoleJsonInfo.do");
}
```

在URL重定向的过程中，并不能有效传递对象，因为HTTP的重定向参数是以字符串传递的，所以如果要传递对象参数，需要使用RedirectAttributes

```Java
@RequestMapping("/addRole3")
// RedirectAttribute对象Spring MVC会自动初始化它
public String addRole3(RedirectAttributes ra, Role role) {
    roleService.insertRole(role);
    ra.addFlashAttribute("role", role);
    return "redirect:./showRoleJsonInfo2.do";
}
```

使用addFlashAttribute方法后，SpringMVC会将数据保存到Session中，重定向后就将其清除。这样就能够传递数据给下一个地址了。

#### 保存并获取属性参数

##### 注解@RequestAttribute

主要作用使从HTTP的request对象中取出请求属性，它的范围周期是在一次请求中存在。

```Java
@Controller
@RequestMapping("/attribute")
public class AttributeController {
    @Autowired
    private RoleService roleService = null;
    
    @RequestMapping("/requestAttribute")
    public ModelAndView reqAttr(@RequestAttribute("id") Long id) {
        ModelAndView mv = new ModelAndView();
        Role role = roleService.getRole(id);
        mv.addObject("role", role);
        mv.setView(new MappingJackson2JsonView());
        return mv;
    }
}
```

@RequestParam：处理GET和POST传的参数

@RequestBody：处理JSON数据

@RequestAttribute：处理HttpAttribute数据

##### 注解@SessionAttribute和注解@SessionAttributes

@SessionAttributes的作用是当这个类被注解后，SpringMVC执行完控制器的逻辑后，将数据模型中对应的属性名称或者属性类型保存到HTTP的Session对象中。

```Java
@Controller
@RequestMapping("/attribute")
@SessionAttributes(name={"id"}, types={Role.class})
public class AttributeController {
    @Autowired
    private RoleService roleService = null;
    
    @RequestMapping
    public ModelAndView sessionAttrs(Long id) {
        ModelAndView mv = new ModelAndView();
        Role role = roleService.getRole(id);
        // Session会保存di和role
        mv.addObject("role", role);
        mv.addObject("id", id);
        mv.setViewName("sessionAttribute");
        return mv;
    }
}
```

获取Session属性

```Java
@RequestMapping("/sessionAttribute")
public ModelAndView sessionAttr(@SessionAttribute("id") Long id) {
    ModelAndView mv = new ModelAndView();
    Role role = roleService.getRole(id);
    mv.addObject("role", role);
    mv.setView(new MappingJackson2JsonView());
    return mv;
}
```

##### 注解@CookieValue和注解@RequestHeader

```Java
@RequestHeader(value="User-Agent", required=false, defaultValue="attribute")
@CookieValue(value="JSESSIONID", required=false, defaultValue="MyJSessionId")
```

#### 拦截器

可以在进入处理器之前做一些工作，或者在处理器完成后进行操作，甚至是在渲染视图后进行操作。

##### 拦截器的定义

实现接口org.springframework.web.servlet.HandlerInterceptor

```Java
package org.springframework.web.servlet;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public interface HandlerInterceptor {
    // 在处理器之前执行，返回boolean值决定手机否执行处理器和后面的视图解析和渲染方法
    boolean preHandle(HttpServletRequest var1, HttpServletResponse var2, Object var3) throws Exception;
	
    // 在处理器之后执行
    void postHandle(HttpServletRequest var1, HttpServletResponse var2, Object var3, ModelAndView var4) throws Exception;
	
    // 无论是否由异常都会在渲染视图后执行
    void afterCompletion(HttpServletRequest var1, HttpServletResponse var2, Object var3, Exception var4) throws Exception;
}
```

##### 拦截器的执行流程

![](F:\mycode\knowledgeArrangement\web\interceptor.png)

##### 开发拦截器

当XML配置文件加入元素<mvc:annotation-driven>或者手机用Java配置使用注解@EnableWebMvc时，系统就会初始化拦截器ConversionServiceExposingInterceptor，它是一开始就被SpringMVC默认加载的拦截器，作用时根据配置在控制器上的注解来完成响应的功能。还可以通过继承公共拦截器HandlerInterceptorAdapter来实现拦截器。

```Java
package org.springframework.web.servlet.handler;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.servlet.AsyncHandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

public abstract class HandlerInterceptorAdapter implements AsyncHandlerInterceptor {
    public HandlerInterceptorAdapter() {
    }

    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return true;
    }

    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    }

    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    }

    public void afterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    }
}
```

在SpringMVC配置文件中加入代码

```xml
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/role/*.do" />
        <bean class="com.ssm.chapter15.interceptor.RoleInterceptor" />
    </mvc:interceptor>
</mvc:interceptors>
```

多个拦截器的执行轨迹

```
preHandle1
preHandle2
preHandle3
....控制逻辑....
postHandle3
postHandle2
postHandle1
......
afterCompletion3
afterCompletion2
afterCompletion1
......
```

#### 验证器

```Java
package com.ssm.chapter15.controller;

@Controller
@RequestMapping("/validate")
public class ValidateController {
    @RequestMapping("/annotation")
    public ModelAndView annotationValidate(@Valid Transaction trans, Errors errors) {
        if (errors.hasErrors()) {
            List<FieldError> errorList = errors.getFieldErrors();
            //.....
        }
    }
}
```

Spring提供了Validator接口来实现检验，它将在进入控制器逻辑之前对参数的合法性进行检验

```Java
package org.springframework.validation;

public interface Validator {
    // 判断当前验证其是否用户检验var1类型的POJO
    boolean supports(Class<?> var1);

    // 检验POJO的合法性
    void validate(Object var1, Errors var2);
}
```

使用@InitBinder将验证器和控制器捆绑在一起

```Java
@InitBinder
public void initBinder(DataBinder binder) {
    binder.setValidator(new TransactionValidator());
}

@RequestMapping("/validator")
public ModelAndView validator(@Valid Transaction trans, Errors errors) {
    
    if (errors.hasErrors()) {
        List<FieldError> errorList = errors.getFieldErrors();
        //.....
    }
}
```

#### 数据模型

从控制机器获取数据后，会装载数据到ModelAndView中，然后将视图名称转发到视图解析器中，通过解析器解析后得到最终视图，最后将数据模型渲染到视图中，展示最终的结果给用户。

ModelAndView有一个类型为ModelMap的属性Model，而ModelMap继承了LinkedHashMap<String, Object>，为了进一步定义数据模型功能，Spring还创建了类ExtendedModelMap，这个类实现了数据模型定义的Model接口，并且再此基础上还派生了关于数据绑定的类——BindingAwareModelMap。

在控制器的方法中，可以将ModelAndView、Model、ModelMap作为参数。在SpringMVC运行的时候，会自动初始化它们，因此可以选择ModelMap或者Model作为数据模型。实际上SpringMVC创建的是一个BindingAwareModelMap实例。ModelAndView初始化后，Model属性为空，当调用它增加数据模型的方法后，会自动创建一个ModelMap实例，用以保存数据模型。

```java
@RequestMapping(value="/getRoleByModelMap", method=RequestMethod.GET)
public ModelAndView getRoleByModelMap(@RequestParam("id") Long id, ModelMap modelMap) {
    Role role = roleService.getRole(id);
    ModelAndView mv = new ModelAndView();
    mv.setViewName("roleDetails");
    modelMap.addAttribute("role", role);
    return mv;
}

@RequestMapping(value="/getRoleByModel" method=RequestMethod.GET)
public ModelAndView getRoleByModel(@RequestParam("id") Long id, Model model) {
    Role role = roleService.getRole(id);
    ModelAndView mv = new ModelAndView();
    mv.setViewName("roleDetails");
    model.addAttribute("role", role);
    return mv;
}

@RequestMapping(value="/getRoleByMv" method=RequestMethod.GET)
public ModelAndView getRoleByMv(@RequestParam("id") Long id, ModelAndView mv) {
    Role role = roleService.getRole(id);
    mv.setViewName("roleDetails");
    mv.addObject("role", role);
    return mv;
}
```

无论是使用Model还是ModelMap作为参数，SpringMVC都是创建的BindingAwareModelMap实例。它是一个继承了ModelMap、实现了Model接口的类。在控制器方法中不需要将数据模型绑定给ModelAndView，这一步是SpringMVC在完成控制器模型后，自动绑定的。

#### 视图和视图解析器

如果是非逻辑视图，则不会通过视图解析器定位视图，直接将数据模型渲染就结束了；而逻辑视图则要对其进行进一步解析，以便定义真实视图，这就是视图解析器的作用了。

视图接口View

```Java
package org.springframework.web.servlet;

import java.util.Map;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public interface View {
    // 响应状态属性
    String RESPONSE_STATUS_ATTRIBUTE = View.class.getName() + ".responseStatus";
    // 定义数据模型下取出变量路径
    String PATH_VARIABLES = View.class.getName() + ".pathVariables";
    // 选择响应内容类型
    String SELECTED_CONTENT_TYPE = View.class.getName() + ".selectedContentType";
	// 响应客户端的类型
    String getContentType();
	// 渲染方法，model是数据模型
    void render(Map<String, ?> var1, HttpServletRequest var2, HttpServletResponse var3) throws Exception;
}
```

当控制器返回ModelAndView的时候，视图解析器就会解析它，然后将数据模型传递给render方法，这样就可以渲染视图了。

对于逻辑视图而言需要一个视图解析器，常见的配置如下：

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver" p:prefix="/WEB-INF/jsp/" p:suffix=".jsp" />
```

或者Java配置

```Java
@Bean(name="viewResolver")
public ViewResolver initViewResolver() {
    InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
    viewResolver.setPrefix("/WEB-INF/jsp/");
    viewResolver.setSuffix(".jsp");
    return viewResolver;
}
```

##### 视图解析器

把视图名称转换为逻辑视图是一个必备过程。当配置完视图解析器后，它就会加载到SpringMVC的视图解析器列表中去，当返回ModelAndView的时候，SpringMVC就会在 视图解析器列表中遍历，找到对应的视图解析器去解析视图。