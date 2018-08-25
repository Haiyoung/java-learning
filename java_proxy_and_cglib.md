
Java 静态代理、动态代理JDK实现与CGLIB实现
<!-- TOC -->

- [设计模式之代理模式](#设计模式之代理模式)
    - [代理模式类图](#代理模式类图)
    - [类图解析](#类图解析)
- [java 静态代理](#java-静态代理)
    - [静态代理定义](#静态代理定义)
    - [静态代理的优缺点](#静态代理的优缺点)
    - [静态代理demo](#静态代理demo)
- [java 动态代理](#java-动态代理)
    - [动态代理定义](#动态代理定义)
    - [动态代理的类图](#动态代理的类图)
    - [类图解析](#类图解析-1)
    - [动态代理的优缺点](#动态代理的优缺点)
    - [动态代理demo](#动态代理demo)
- [cglib](#cglib)
    - [cglib是什么？](#cglib是什么)
    - [cglib入口类Enhancer](#cglib入口类enhancer)
    - [Enhancer类图解析](#enhancer类图解析)
    - [cglib实现动态代理类图](#cglib实现动态代理类图)
    - [cglib实现动态代理demo](#cglib实现动态代理demo)
    - [cglib实现动态代理的优缺点](#cglib实现动态代理的优缺点)
- [reference](#reference)

<!-- /TOC -->
### 设计模式之代理模式
- 代理模式为另一个对象提供一个替身或占位符以控制对这个对象的访问
- 代理模式不仅仅可以解耦客户端和委托类；我们还可以借助代理来增加一些功能，而不需要修改原有代码，符合开闭原则

代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务。就是这样，真正的业务功能还是有委托类来实现，但是在实现业务类之前的一些公共服务。例如在项目开发中我们没有加入缓冲，日志这些功能，后期想加入，我们就可以使用代理来实现，而没有必要打开已经封装好的委托类。
#### 代理模式类图
![代理模式类图](/imgs/javaProxy/1.png)
#### 类图解析
- Subject 为RealSubject和Proxy提供了接口，通过实现同一接口，Proxy在RealSubject出现的地方取代它
- RealSubject 是真正做事的对象，它是被Proxy代理和控制访问的对象
- Proxy 持有RealSubject的引用；客户和RealSubject的交互都必须通过Proxy,因为Proxy和RealSubject实现了同样的接口，所以任何用到RealSubject的地方，都可以用Proxy来取代；Proxy也控制了对RealSubject的访问，在某些情况下，这种访问控制可能是我们需要的，这些情况包括RealSubject是一个远程对象，RealSubject创建开销大，或者需要为RealSubject添加其他操作

### java 静态代理
#### 静态代理定义
所谓静态代理也就是在程序运行前就已经存在代理类的字节码文件，代理类和委托类的关系在运行前就已经确定了。
#### 静态代理的优缺点
- 优点
    - 静态代理比较容易理解和实现，由于代理类和委托类的关系是在编译期静态决定的，所以执行的时候也没有任何额外开销
- 缺点
    - 代理对象的一个接口只服务于一种类型的对象，如果要代理的方法很多，势必要为每一种方法都进行代理，静态代理在程序规模稍大时就无法胜任了。
    - 如果接口增加一个方法，除了所有实现类需要实现这个方法外，所有代理类也需要实现此方法。增加了代码维护的复杂度。
#### 静态代理demo
- Subject 定义接口
```java
public interface Subject {

    public void hello();

    public void hello(String arg);
}
```
- RealSubject 定义委托类
```java
public class RealSubject implements Subject{
    @Override
    public void hello() {
        System.out.println("hello world");
    }

    @Override
    public void hello(String arg) {
        System.out.println("hello "+arg);
    }
}
```
- Proxy 代理类
```java
public class Proxy extends RealSubject implements Subject {

    private Subject subject;

    public Proxy(Subject subject){
        this.subject = subject;
    }

    @Override
    public void hello() {
        System.out.println("before ... ... ...");
        subject.hello();
        System.out.println("after ... ... ...");
    }

    @Override
    public void hello(String arg) {
        System.out.println("before ... ... ...");
        subject.hello(arg);
        System.out.println("after ... ... ...");
    }
}
```
- Client 客户端
```java
public class ProxyTest {
    public static void main(String[] args){
        Subject realSubject = new RealSubject();
        Proxy staticProxy = new Proxy(realSubject);
        staticProxy.hello();
        staticProxy.hello("Haiyoung");
    }
}
/*before ... ... ...
hello world
after ... ... ...
before ... ... ...
hello Haiyoung
after ... ... ...*/
```
### java 动态代理
#### 动态代理定义
所谓动态代理，就是代理类的源码是在程序运行期间由JVM根据反射等机制动态的生成，所以不存在代理类的字节码文件。代理类和委托类的关系是在程序运行时确定。
#### 动态代理的类图
![代理模式类图](/imgs/javaProxy/2.png)
#### 类图解析
- Subject 为RealSubject和Proxy提供了接口，通过实现同一接口，Proxy在RealSubject出现的地方取代它
- RealSubject 是真正做事的对象，它是被Proxy代理和控制访问的对象
- InvocationHandler 是一个JDK提供的标准接口,通过这个handler类来调用委托类的方法
```java
public interface InvocationHandler {
    /**
    * 
    * @param proxy 代理对象
    * @param method 委托类的方法
    * @param args 委托类方法参数
    * @return 委托类方法返回值
    */
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```
- DynamicProxy 实现 InvocationHandler 接口，绑定委托类生成代理类实例
#### 动态代理的优缺点
- 优点
    - 动态代理与静态代理相比较，最大的好处是接口中声明的所有方法都被转移到调用处理器一个集中的方法中处理（InvocationHandler.invoke）。这样，在接口方法数量比较多的时候，我们可以进行灵活处理，而不需要像静态代理那样为每一个方法进行中转。
- 缺点
    - java 动态代理只支持对接口的代理，对没有实现接口的类，无法通过InvocationHandler动态代理的方式去代理它的行为
#### 动态代理demo
- DynamicProxy
```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class DynamicProxy implements InvocationHandler {

    private Object obj;

    public Object bind(Object obj){
        this.obj = obj;
        // 返回代理类实例
        return Proxy.newProxyInstance(obj.getClass().getClassLoader(),
                obj.getClass().getInterfaces(), this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //　　在代理真实对象前我们可以添加一些自己的操作
        System.out.println("before ... ... ...");
        System.out.println("Method:" + method);
        //    当代理对象调用真实对象的方法时，其会自动的跳转到代理对象关联的handler对象的invoke方法来进行调用
        Object res = method.invoke(obj, args);
        //　　在代理真实对象后我们也可以添加一些自己的操作
        System.out.println("after ... ... ...");
        return res;

    }
}
```
- Client 客户端
```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;

public class DyProxyTest {
    public static void main(String[] args){

        Subject subject = (Subject) new DynamicProxy().bind(new RealSubject());
        subject.hello();
        subject.hello("DynamicProxy");
    }
}
/*
before ... ... ...
Method:public abstract void com.haiyoung.ProxyTest.Subject.hello()
hello world
after ... ... ...
before ... ... ...
Method:public abstract void com.haiyoung.ProxyTest.Subject.hello(java.lang.String)
hello DynamicProxy
after ... ... ...*/
```

### cglib
#### cglib是什么？
- CGLIB是一个强大的高性能的代码生成包。它广泛的被许多AOP的框架使用，例如Spring AOP和dynaop，为他们提供方法的interception（拦截）。最流行的OR Mapping工具hibernate也使用CGLIB来代理单端single-ended(多对一和一对一)关联（对集合的延迟抓取，是采用其他机制实现的）。EasyMock和jMock是通过使用模仿（mock）对象来测试java代码的包。它们都通过使用CGLIB来为那些没有接口的类创建模仿（mock）对象。
- CGLIB包的底层是通过使用一个小而快的字节码处理框架ASM，来转换字节码并生成新的类。除了CGLIB包，脚本语言例如Groovy和BeanShell，也是使用ASM来生成java的字节码。当然不鼓励直接使用ASM，因为它要求你必须对JVM内部结构包括class文件的格式和指令集都很熟悉。
#### cglib入口类Enhancer
![Enhancer类图](/imgs/javaProxy/3.png)
#### Enhancer类图解析
- cglib入口类是Enhancer，它继承自 AbstractClassGenerator ，而 AbstractClassGenerator 实现了 ClassGenerator 接口，ClassLoaderData 又是 AbstractClassGenerator 的内部类
- Enhancer中有几个常用的方法, setSuperClass和setCallback, 设置好了SuperClass后, 可以使用create生成字节码对象，最后生成代理类实例，setCallback设置拦截接口MethodInterceptor的实现
- 在调用目标方法时，CGLib会回调MethodInterceptor接口方法拦截，来实现你自己的代理逻辑，类似于JDK中的InvocationHandler接口
#### cglib实现动态代理类图
![cglib实现动态代理类图](/imgs/javaProxy/4.png)

其中，代理类继承目标类，并为所有方法生成一个MethodProxy用于代理目标方法，在调用代理类的方法时，会委托MethodInterceptor调用intercept方法。

#### cglib实现动态代理demo
- CglibProxy
```java
import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class CglibProxy {

    private Object obj;

    public Object bind(final Object target){
        this.obj = target;
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(obj.getClass());
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                System.out.println("before... this is cglib proxy");
                Object res = method.invoke(target, objects);
                System.out.println("after... this is cglib proxy");
                return res;
            }
        });
        return enhancer.create();
    }
}
```
- Client 客户端
```java
public class CglibProxyTest {
    public static void main(String[] args){
        Subject subject = (Subject) new CglibProxy().bind(new RealSubject());
        subject.hello();
        subject.hello("CglibProxy");
    }
}
/*
before... this is cglib proxy
hello world
after... this is cglib proxy
before... this is cglib proxy
hello CglibProxy
after... this is cglib proxy*/
```
#### cglib实现动态代理的优缺点
- 优点
    - 它为没有实现接口的类提供代理，为JDK的动态代理提供了很好的补充。通常可以使用Java的动态代理创建代理，但当要代理的类没有实现接口或者为了更好的性能，CGLIB是一个好的选择
    - 动态生成一个要代理类的子类，子类重写要代理的类的所有非final的方法。在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑。它比使用java反射的JDK动态代理要快。
- 缺点
    - 对于final类和方法，无法进行代理
### reference
- [为什么使用代理模式](https://blog.csdn.net/wangyongxia921/article/details/46124197)
- [Proxy.newProxyInstance的秘密](https://blog.csdn.net/lovejj1994/article/details/78080124)
- [cglib百度百科](https://baike.baidu.com/item/cglib/9178356)
- [cglib github](https://github.com/cglib/cglib)
- [cglib动态代理实现解析](https://blog.csdn.net/chinrui/article/details/79905074)