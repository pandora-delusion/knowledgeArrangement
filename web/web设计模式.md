# Java Web开发中需要掌握的设计模式和方法

### Java反射技术

#### 通过反射构建对象

```java
public class ReflectServiceImpl {
    public void sayHello(String name) {
        System.err.println("hello, " + name);
    }
}

// 通过反射构建上面的类实例
public ReflectServiceImpl getInstance() {
    ReflectServiceImpl object = null;
    try {
        //给类加载器注册了一个类的全限定名，然后通过newInstance方法初始化了一个类对象
        object = (ReflectServiceImpl)
            Class.forName("com.xxx.xxx.xxx.ReflectServiceImpl").newInstance();
    } catch(ClassNotFoundException | InstantiationException | IllegalAccessException ex) {
        ex.printStackTrace();
    }
    return object;
}

public class ReflectServiceImpl2 {
    private String name;
    public ReflectServiceImpl2(String name) {
        this.name = name;
    }
    
    public void sayHello() {
        System.err.println("hello, " + this.name);
    }
} 

// 通过反射生成带有参数的构建方法
public ReflectServiceImpl2 getInstance() {
    ReflectServiceImpl2 object = null;
    try {
        //使用getConstructor方法，对重名方法进行排除，在用newInstance方法生成对象。
        object = (ReflectServiceImpl2)Class.forName("com.xxx.xxx.ReflectServiceImpl2").
            getConstructor(String.Class).newInstance("成");
    } catch(ClassNotFoundException | InstantiationException | IllegalAccessException 
           | NoSuchMethodException | SecurityException | IllegalArgumentException |
           InvocationTargetException ex) {
        ex.printStackTrace();
    }
    return object;
}
```

反射的优点是只要配置就可以生成对象，可以接触程序的耦合度，比较灵活。反射的缺点是运行比较缓慢。Spring IoC容器就在广泛地使用反射机制。

### 动态代理模式

动态代理的意义就是在于生成一个占位符（又称代理对象），来代理真实对象，从而控制真实对象的访问。代理的作用是在真实对象访问前或者之后加入对应的逻辑，或者根据其他规则控制是否使用真实对象。

代理必须分为两个步骤：

1. 代理对象和真实对象建立代理关系
2. 实现代理对象的代理逻辑方法

Java中有多种动态代理技术，比如JDK、CGLIB、Javassist、ASM，其中最常用的是JDK动态代理和CGLIB，MyBatis还使用了Javassist

#### JDK动态代理

jdk动态代理是java.lang.reflect.*包提供的方式，它必须借助一个接口才能产生代理对象，所以先定义接口。

```Java
public interface HelloWorld {

    void sayHelloWorld(String name);
}

// 提供实现类来实现接口
public class HelloWorldImpl implements HelloWorld {

    @Override
    public void sayHelloWorld(String name) {
        System.out.println("Hello world, " + name);
    }
}

//实现代理逻辑类必须要实现java.lang.reflect.InvocationHandler接口
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class DynamicProxyTest implements InvocationHandler {

    private Object target;
    private String interceptorClass;

    public DynamicProxyTest(Object target, String str) {
        this.target = target;
        this.interceptorClass = str;
    }

    // 代理方法逻辑，第一个参数是代理对象，就是bind方法生成的对象，method是当前调度的方法，args是调度方法的参数。
    // 当我们使用了代理对象的调度方法后，它就是进入到invoke方法里面
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (interceptorClass == null) {
            return method.invoke(target, args);
        }

        Object res = null;
        Interceptor interceptor = (Interceptor) Class.forName(interceptorClass).newInstance();
        if (interceptor.before(proxy, target, method, args)) {
            res = method.invoke(target, args);
        } else {
            interceptor.around(proxy, target, method, args);
        }
        interceptor.after(proxy, target, method, args);
        return res;
    }

    // 建立代理对象和真实对象的关系
    public static Object bind(Object object, String interceptorClass) {
        // newProxyInstance方法包含三个参数：第一个是类加载器，第二个是把生成的动态代理对象下挂到哪些接口下，
        // 第三个是实现方法逻辑的代理类。
        return Proxy.newProxyInstance(object.getClass().getClassLoader(),
                object.getClass().getInterfaces(), new DynamicProxyTest(object, interceptorClass));
    }

}
```



#### CGLIB动态代理

JDK动态代理必须提供接口才能使用，在一些不能提供接口的环境中，只能采用其他第三方技术，比如CGLIB动态代理。它的优势在于不需要提供接口，只要一个非抽象类就可以实现动态代理。

```Java
import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class CglibProxyExample implements MethodInterceptor {

    // 生成CGLIB代理对象
    public Object getProxy(Class cls) {
        //·GLIB enmhancer增强类对象
        Enhancer enhancer = new Enhancer();
        // 设置增强类型
        enhancer.setSuperclass(cls);
        // 定义代理逻辑对象为当前对象，要求当前对象实现MethodInterceptor方法
        enhancer.setCallback(this);
        // 生成并返回代理对象
        return enhancer.create();
    }

    /** 
    代理逻辑方法
    proxy：代理对象
    method：方法
    args：方法参数
    methodProxy： 方法代理
    return：代理逻辑返回
    */
    @Override
    public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("调用真实对象前");
        Object res = methodProxy.invokeSuper(proxy, args);
        System.out.println("调用真实对象后");
        return res;
    }

}
```

#### 拦截器

由于动态代理一般都比较难以理解，程序设计者会设计一个拦截器接口供开发者使用，开发者只需要知道拦截器接口的方法、含义和作用即可，无需知道动态代理是怎么实现的。

定义拦截器接口如下：

```Java
import java.lang.reflect.Method;

public interface Interceptor {
    /**
    proxy代理对象、target真实对象、method方法、args运行方法参数
    */
    boolean before(Object proxy, Object target, Method method, Object[] args);
    void around(Object proxy, Object target, Method method, Object[] args);
    void after(Object proxy, Object target, Method method, Object[] args);
}

public class MyInterceptor implements Interceptor {
    @Override
    public boolean before(Object proxy, Object target, Method method, Object[] args) {
        System.err.println("反射方法前逻辑");
        // 不反射被代理对象原有方法
        return false;
    }

    @Override
    public void around(Object proxy, Object target, Method method, Object[] args) {
        System.err.println("取代了被代理对象的方法");
    }

    @Override
    public void after(Object proxy, Object target, Method method, Object[] args) {
        System.err.println("反射方法后逻辑");
    }
}
```

使用JDK动态代理，就可以区实现这些方法在适当时的调用逻辑了。使用上面JDK动态代理的代码，

```Java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class DynamicProxyTest implements InvocationHandler {

    // 真实对象
    private Object target;
    // 拦截器全限定名
    private String interceptorClass;

    public DynamicProxyTest(Object target, String str) {
        this.target = target;
        this.interceptorClass = str;
    }

    // 代理方法逻辑，第一个参数是代理对象，就是bind方法生成的对象，method是当前调度的方法，args是调度方法的参数。
    // 当我们使用了代理对象的调度方法后，它就是进入到invoke方法里面
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (interceptorClass == null) {
            // 没有设置拦截器则直接反射原有方法
            return method.invoke(target, args);
        }

        Object res = null;
        // 通过反射生成拦截器
        Interceptor interceptor = (Interceptor) Class.forName(interceptorClass).newInstance();
        // 调用前置方法
        if (interceptor.before(proxy, target, method, args)) {
            // 反射原有对象方法
            res = method.invoke(target, args);
        } else {
            // 返回false执行around方法
            interceptor.around(proxy, target, method, args);
        }
        // 调用后置方法
        interceptor.after(proxy, target, method, args);
        return res;
    }

    // 绑定委托对象并返回一个代理占位
    public static Object bind(Object object, String interceptorClass) {
        // newProxyInstance方法包含三个参数：第一个是类加载器，第二个是把生成的动态代理对象下挂到哪些接口下，
        // 第三个是实现方法逻辑的代理类。
        return Proxy.newProxyInstance(object.getClass().getClassLoader(),
                object.getClass().getInterfaces(), new DynamicProxyTest(object, interceptorClass));
    }

}
```

开发者只要知道拦截器的作用就可以编写拦截器了，编写完后就可以设置拦截器，这样就完成了开发任务。

测试拦截器代码如下：

```Java
public class Main {

    public static void main(String... args) {
        HelloWorld helloWorld = (HelloWorld) DynamicProxyTest.bind(new HelloWorldImpl(),
                "com.delusion.ch4.dynamicproxy.MyInterceptor");
        helloWorld.sayHelloWorld("cheng");
    }
}
```