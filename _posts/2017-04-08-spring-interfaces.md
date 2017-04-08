---
layout: post
title: Spring proxies, interfaces and final methods
permalink: spring-proxy-interfaces
--- 
Bean loading and proxying is topic so basic that I am almost ashamed that it took me so long to solve the problem caused by a simple spring mechanism.
Simple bean with one dependency was throwing NullPointerException when using this dependency event if it was injected correctly. How is it possible?

## Beans with an interface
The purpose of interface is clear. It decouples callers from specific implementations. But it is no surprise that in spring world there is something more.
When declaring component in spring with some additional features using annotations

```java
@Component
@Cacheable(/* caching configuration */)
public class UserServiceImpl implements UserService { ... }
```

spring will wrap your class in proxy that will do the job, in this example caching the return values. This proxy will implement same interface as the target class.  
This is the reason why this code

```java
System.out.println(context.getBean(UserService.class).getClass());
```
outputs `class com.sun.proxy.$Proxy20` not a `UserService`. Now what output will this code yield?

```java
System.out.println(context.getBean(UserServiceImpl.class).getClass());
```
Correct answer is none. It will fail due to exception

```
NoSuchBeanDefinitionException: No qualifying bean of type 'UserServiceImpl' available
```

How is this possible when the bean clearly exist in the context? It sure exists, but it is the proxy of type `UserService` not a `UserServiceImpl`. 
This proxy is a standard [JDK Proxy](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Proxy.html). This proxy can be used only for interfaces not concrete classes. 

This can be solved by using another proxy implementation. Annotate the component with

```java
@Scope(proxyMode = ScopedProxyMode.TARGET_CLASS)
```
And spring will use [CglibAopProxy](https://github.com/cglib/cglib/wiki/Tutorial) instead JDK Proxy. Now the previous code snippet outputs.

```
class UserServiceImpl$$EnhancerBySpringCGLIB$$82ee3d31
```

If your component declares no interfaces spring will automatically use cglib proxy.

## Final methods
Usage cglib proxy has another consequence. It can't proxy final methods! 
Consider this simple example

```java
@Component
public class MyComponent {

    private final Dependency dependency;

    public MyComponent(final Dependency dependency) {
        this.dependency = dependency;
    }

    public final void doSomethingFinal(int key) {
        dependency.getNumber();
    }
}
```
There is nothing wrong with this code and it will execute fine. But now somebody adds caching to it. 

```java
@Cacheable(value = "test", key = "#key")
public final void doSomethingFinal(int key) { ...
```

And now it fails because of `NullPointerException`. The reason is that spring will inject the bean to the instance of cglib proxy. But the call to `doSomethingFinal` method is not proxied but invoked directly. 

Spring developers are trying to help you catch that problem early by logging it 

```
INFO: Unable to proxy method [public final void MyComponent.doSomethingFinal(int)] because it is final: All calls to this method via a proxy will NOT be routed to the target instance.
```

But when starting spring application this can easily go unnoticed. 

**What about final classes?**  In this case you are all covered because cglib will fail even to instantiate the class with an exception
```
Could not generate CGLIB subclass of class [class MyComponent]: Common causes of this problem include using a final class or a non-visible class
```

