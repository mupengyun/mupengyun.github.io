---
layout: post
title: "JAVA代理模式详解"
date: 2014-05-02 13:47:00 
description: "JAVA 静态代理，动态代理"
tag: JAVA
---
    前言：代理，什么是代理，生活中处处可见，代缴公积金，代办进京证，代销商等。编程语言中的解释为，不直接调运目标对象，直接调运代理对象，让代理对象去调运目标对象。

### 简介：
* **代理解释：** 代理模式是常用的java设计模式，他的特征是代理类与委托类有同样的接口，代理类主要负责为委托类预处 理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。 代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，代理类的 对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提 供特定的服务。

* **代理使用场景：** 当我们不想做某件事(买个手机，可以去体验店，不一定非去厂家)，或者因为某种情况不能做某件事（喝酒之后，找代驾），这时侯，我们需要用到代理。

* **代理分类：** 静态代理和动态代理

## JAVA代理详解
**涉及对象**

`抽象角色`：声明真实对象和代理对象的共同接口。

`代理角色`：代理对象角色内部含有对真实对象的引用，从而可以操作真实对象，同时代理对象提供与真实对象相同的接口以便在任何时刻都能够代替真实对象。同时，代理对象可以在执行真实对象操作时，附加其他的操作，相当于对真实对象进行封装。

`真实角色`：代理角色所代表的真实对象，是我们最终要引用的对象。
    
以下以《Java与模式》中的示例为例：
### 静态代理

所谓静态也就是在程序运行前就已经存在代理类的字节码文件，代理类和委托类的关系在运行前就确定了。     

#####  抽象角色类，可以定义一个接口或者抽象类：　

```
/**
* 真实对象和代理对象的共同接口
* 
*/
public abstract class Subject
{
    public abstract void request();
}
```

##### 真实角色的类，实现接口，并完成实际工作:
```
/**
* 真实角色
*/
public class RealSubject extends Subject
{
    @Override
    public void request()
    {
        System.out.println("From Real Subject!");
    }
}
```

##### 代理角色，内部包含对真实角色对象的引用，并且封装功能：
```
/**
* 代理角色
* 
*/
public class ProxySubject extends Subject
{
    //代理角色对象内部含有对真实对象的引用
    private RealSubject realSubject;

    @Override
    public void request()
    {
        //在真实角色操作之前所附加的操作
        preRequest();
        if(null == realSubject)
        {
            realSubject = new RealSubject();
        }
        
        //真实角色所完成的事情
        realSubject.request();
        //在真实角色操作之后所附加的操作
        postRequest();
    }
    
    private void preRequest()
    {
        System.out.println("Pre Request.");
    }
    private void postRequest()
    {
        System.out.println("Post Request");
    }
}
```

##### 生成静态代理类工厂 

```
public class SubjectStaticFactory 
{  
    //客户类调用此工厂方法获得代理对象。  
    //对客户类来说，其并不知道返回的是代理类对象还是委托类对象。  
    public static Subject getInstance(){   
    return new ProxySubject(new RealSubject());  
    }  
} 
``` 

##### 在客户端进行调用时：

```
/**
* 客户类
*  
*/
public class Client
{
    public static void main(String[] args)
    {
        Subject subject = SubjectStaticFactory.getInstance;
        subject.request();        
    }    
}    
```

**静态代理缺点**

1)  代理对象的一个接口只服务于一种类型的对象，如果要代理的方法很多，势必要为每一种方法都进行代理，静态代理在程序规模稍大时就无法胜任了。 

2)  如果接口增加一个方法，除了所有实现类需要实现这个方法外，所有代理类也需要实现此方法。增加了代码维护的复杂度。 
    
### jdk动态代理
* jdk动态代理是由Java内部的反射机制来实现的，cglib动态代理底层则是借助asm来实现的。

* 总的来说，反射机制在生成类的过程中比较高效，而asm在生成类之后的相关执行过程中比较高效（可以通过将asm生成的类进行缓存，这样解决asm生成类过程低效问题）。

`jdk动态代理`的应用前提，必须是目标类基于统一的接口。如果没有上述前提，jdk动态代理不能应用。由此可以看出，jdk动态代理有一定的局限性，cglib这种第三方类库实现的动态代理应用泛，且在效率上更有优势。

`jdk动态代理`
在java的动态代理机制中，有两个重要的类或接口，一个是 `InvocationHandler(Interface)`、另一个则是 `Proxy(Class)`，这一个类和接口是实现我们动态代理所必须用到的。首先我们先来看看java的API帮助文档是怎么样对这两个类进行描述的：

`InvocationHandle`:
每一个动态代理类都必须要实现InvocationHandler这个接口，并且每个代理类的实例都关联到了一个handler，当我们通过代理对象调用一个方法的时候，这个方法的调用就会被转发为由InvocationHandler这个接口的 invoke 方法来进行调用。我们来看看InvocationHandler这个接口的唯一一个方法 invoke 方法：

Object `invoke`(Object proxy, Method method, Object[] args) throws Throwable

`proxy`:　　指代我们所代理的那个真实对象

`method`:　　指代的是我们所要调用真实对象的某个方法的Method对象

`args`:　　指代的是调用真实对象某个方法时接受的参数

---

#####  接下来我们来看看`Proxy`这个类：

`Proxy`这个类的作用就是用来动态创建一个代理对象的类，它提供了许多的方法，但是我们用的最多的就是 newProxyInstance 这个方法：

`public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,  InvocationHandler h)  throws IllegalArgumentException`
这个方法的作用就是得到一个动态的代理对象，其接收三个参数，我们来看看这三个参数所代表的含义：

`loader`:　　一个ClassLoader对象，定义了由哪个ClassLoader对象来对生成的代理对象进行加载

`interfaces`:　　一个Interface对象的数组，表示的是我将要给我需要代理的对象提供一组什么接口，如果我提供了一组接口给它，那么这个代理对象就宣称实现了该接口(多态)，这样我就能调用这组接口中的方法了

`h`:   一个InvocationHandler对象，表示的是当我这个动态代理对象在调用方法的时候，会关联到哪一个InvocationHandler对象上

#### DEMO 示范
```
public interface Subject   
{   
    public void doSomething();   
} 
```  
```
public class RealSubject implements Subject   
{   
  public void doSomething()   
  {   
    System.out.println( "call doSomething()" );   
  }   
}  
``` 
``` 
public class ProxyHandler implements InvocationHandler   
{   
private Object proxied;   
    
public ProxyHandler( Object proxied )   
{   
this.proxied = proxied;   
}   
 ``` 
 ```     
public Object invoke( Object proxy, Method method, Object[] args ) throws Throwable   
    {   
    //在转调具体目标对象之前，可以执行一些功能处理

    //转调具体目标对象的方法
    return method.invoke( proxied, args);  

    //在转调具体目标对象之后，可以执行一些功能处理
    }    
} 
``` 
``` 
import java.lang.reflect.InvocationHandler;   
import java.lang.reflect.Method;   
import java.lang.reflect.Proxy;   
import sun.misc.ProxyGenerator;   
import java.io.*;   
public class DynamicProxy   
{   
    public static void main( String args[] )   
    {   
        RealSubject real = new RealSubject();   
        Subject proxySubject = (Subject)Proxy.newProxyInstance(Subject.class.getClassLoader(), 
        new Class[]{Subject.class}, 
        new ProxyHandler(real));
            
        proxySubject.doSomething();

        //write proxySubject class binary data to file   
        createProxyClassFile();   
    }   
        
    public static void createProxyClassFile()   
    {   
        String name = "ProxySubject";   
        byte[] data = ProxyGenerator.generateProxyClass( name, new Class[] { Subject.class } );   
        try  
        {   
        FileOutputStream out = new FileOutputStream( name + ".class" );   
        out.write( data );   
        out.close();   
        }   
        catch( Exception e )   
        {   
        e.printStackTrace();   
        }   
    }   
} 
``` 

[**动态代理 代码原文**](http://www.cnblogs.com/flyoung2008/archive/2013/08/11/3251148.html)

### cglib动态代理 

>  `Cglib`是一个优秀的动态代理框架，它的底层使用ASM在内存中动态的生成被代理类的子类，使用CGLIB即使代理类没有实现任何接口也可以实现动态代理功能。<br/>
CGLIB具有简单易用，它的运行速度要远远快于JDK的Proxy动态代理：

`CGLIB`的核心类：<br/>

* net.sf.cglib.proxy.Enhancer – 主要的增强类

* net.sf.cglib.proxy.MethodInterceptor – 主要的方法拦截类，它是Callback接口的子接口，需要用户实现

* net.sf.cglib.proxy.MethodProxy – JDK的java.lang.reflect.Method类的代理类，可以方便的实现对源对象方法的调用,如使用：

* Object o = methodProxy.invokeSuper(proxy, args); – 虽然第一个参数是被代理对象，也不会出现死循环的问题。

* net.sf.cglib.proxy.MethodInterceptor; – 接口是最通用的回调（callback）类型，它经常被基于代理的AOP用来实现拦截（intercept）方法的调用。这个接口只定义了一个方法
   
* public Object intercept(Object object, java.lang.reflect.Method method,Object[] args, MethodProxy proxy) throws Throwable; – 第一个参数是代理对像，第二和第三个参数分别是拦截的方法和方法的参数。原来的方法可能通过使用java.lang.reflect.Method对象的一般反射调用，或者使用 net.sf.cglib.proxy.MethodProxy对象调用。net.sf.cglib.proxy.MethodProxy通常被首选使用，因为它更快。
 
```
import net.sf.cglib.proxy.MethodInterceptor;  
import net.sf.cglib.proxy.MethodProxy;  
  
import java.lang.reflect.Method;  
  
  
public class CglibProxy implements MethodInterceptor {  
    @Override  
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {  
        System.out.println("++++++before " + methodProxy.getSuperName() + "++++++");  
        System.out.println(method.getName());  
        Object o1 = methodProxy.invokeSuper(o, args);  
        System.out.println("++++++before " + methodProxy.getSuperName() + "++++++");  
        return o1;  
    }  
}  
```
 
```  
import com.meituan.hyt.test3.service.UserService;  
import com.meituan.hyt.test3.service.impl.UserServiceImpl;  
import net.sf.cglib.proxy.Enhancer;  
  
   
public class Main {  
    public static void main(String[] args) {  
        CglibProxy cglibProxy = new CglibProxy();  
  
        Enhancer enhancer = new Enhancer();  
        enhancer.setSuperclass(UserServiceImpl.class);  
        enhancer.setCallback(cglibProxy);  
  
        UserService o = (UserService)enhancer.create();  
        o.getName(1);  
        o.getAge(1);  
    }  
}
```  

### 总结

一个典型的动态代理创建对象过程可分为以下四个步骤：

* 1、通过实现InvocationHandler接口创建自己的调用处理器 IvocationHandler handler = new InvocationHandlerImpl(...);
* 2、通过为Proxy类指定ClassLoader对象和一组interface创建动态代理类
    Class clazz = Proxy.getProxyClass(classLoader,new Class[]{...});
* 3、通过反射机制获取动态代理类的构造函数，其参数类型是调用处理器接口类型
Constructor constructor = clazz.getConstructor(new Class[]{InvocationHandler.class});
* 4、通过构造函数创建代理类实例，此时需将调用处理器对象作为参数被传入
Interface Proxy = (Interface)constructor.newInstance(new Object[] (handler));
* 5、为了简化对象创建过程，Proxy类中的newInstance方法封装了2~4，只需两步即可完成代理对象的创建。
生成的ProxySubject继承Proxy类实现Subject接口，实现的Subject的方法实际调用处理器的invoke方法，而invoke方法利用反射调用的是被代理对象的的方法（Object result=method.invoke(proxied,args)）

### 美中不足

Proxy已经设计得非常优美，但是还是有一点点小小的遗憾之处，那就是它始终无法摆脱仅支持interface代理的桎梏，因为它的设计注定了这个遗憾。
回想一下那些动态生成的代理类的继承关系图，它们已经注定有一个共同的父类叫Proxy。Java的继承机制注定了这些动态代理类们无法实现对class的动态代理，原因是多继承在Java中本质上就行不通。有很多条理由，人们可以否定对 class代理的必要性，但是同样有一些理由，相信支持class动态代理会更美好。接口和类的划分，本就不是很明显，只是到了Java中才变得如此的细化。如果只从方法的声明及是否被定义来考量，有一种两者的混合体，它的名字叫抽象类。实现对抽象类的动态代理，相信也有其内在的价值。此外，还有一些历史遗留的类，它们将因为没有实现任何接口而从此与动态代理永世无缘。如此种种，不得不说是一个小小的遗憾。但是，不完美并不等于不伟大，伟大是一种本质，Java动态代理就是佐例。