---
layout: post
title: Handling Hystrix timeouts
permalink: hystrix-timeouts
--- 
[Hystrix](https://github.com/Netflix/Hystrix) is a perfect tool for handling communication with remote systems. One of the topics it covers for you is timeouts. But it might not be as straightforward as it may seem.

Before jumping to the main issue I want to discuss a little theory about how Hystrix works.

### On which thread my code runs?
Hystrix will run your main method in two isolation modes. [Semaphore or thread](https://github.com/Netflix/Hystrix/wiki/How-it-Works#isolation).
If you select semaphore isolation the main method will run on the same thread as the caller. If you choose thread isolation it will run in separate thread. This is pretty straightforward. But on which thread the fallback method runs depends on the reason why the fallback was invoked. 
If your main method threw an exception it will run on the same thread. But if the main method failed because of the timeout it will run on the timer thread. This is important to bare in mind. In consequence anything that is thread bound is not guaranteed to be available in fallback method.

Default value [is Thread isolation](https://github.com/Netflix/Hystrix/wiki/Configuration#execution.isolation.strategy)

### How timeouts work?
Hystrix supports [timeouts](https://github.com/Netflix/Hystrix/wiki/Configuration#execution.isolation.thread.timeoutInMilliseconds). 
They are implemented by [timeout listener](https://github.com/Netflix/Hystrix/blob/master/hystrix-core/src/main/java/com/netflix/hystrix/util/HystrixTimer.java#L110) which are triggered after time set by clients in timeout properties. One of these listeners is registered in [HystrixObservableTimeoutOperator](https://github.com/Netflix/Hystrix/blob/master/hystrix-core/src/main/java/com/netflix/hystrix/AbstractCommand.java#L1173). This operator emits `HystrixTimeoutException` when this timer is called before the actual method returns. [Interrupt is called](https://github.com/Netflix/Hystrix/blob/master/hystrix-core/src/main/java/com/netflix/hystrix/HystrixCommand.java#L404) on the thread which executed the main method only if isolation level is Thread. If isolation level is Semaphore interrupting is not an option because the method does not run on separate thread.

### Interrupting the thread
And here comes the problem. Thread interruption in Java does not actualy interrupt the execution of the thread! It sets the [interrupted flag](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.2.3) on that thread. It is up to the implementation to handle this flag. Unfortunately most implementations in `java.net` package does not respond to it. Any long running computation won't probably respond to it. 

### Surprise
Without interruption handling your main method will continue after the blocking call is completed even if fallback method was executed because of timeout!

Basic example can be found in [this gist](https://gist.github.com/DominikMostek/ec9c72052a645d8f73e24055142a6177).
Output of that example is

```
Main method runs in hystrix-Demo-1
Fallback method runs in HystrixTimer-1
Fallback because of HystrixTimeoutException
Supplier returned on hystrix-Demo-1
Thread hystrix-Demo-1 is interrupted
```

This behavior might lead to unexpected bugs especialy when there is some state changing code in the main method after the blocking call.

### How to fix it
Fixes can vary depending on type of the code that is being executed. But if canceling the operation immediately is not needed you can use `com.netflix.hystrix.AbstractCommand#isResponseFromFallback` method to query if fallback was executed and throw an exception from the main method in that case. 
Note that `Thread.currentThread.isInterrupted()` will be true only in case of thread isolation thus cannot be used both in Semaphore and Thread isolated commands.
In case of apache http client you can call `org.apache.http.client.methods.AbstractExecutionAwareRequest#abort` which will cause the `org.apache.http.client.HttpClient#execute(org.apache.http.client.methods.HttpUriRequest)` method to throw an `SocketException` and to close the current connection. In other situations you have to handle the situations yourself.


If you are using `hystrix-javanica` project be aware of [this bug](https://github.com/Netflix/Hystrix/issues/974). `Null` is passed instead of timeout exception to the fallback method.

### Links

- [Understanding thread interruption in java](https://praveer09.github.io/technology/2015/12/06/understanding-thread-interruption-in-java/)
- [SO: Interrupt/stop thread with socket I/O blocking operation](http://stackoverflow.com/questions/12315149/interrupt-stop-thread-with-socket-i-o-blocking-operation)
- [Faster detection of interrupted connections during PUT operation with Apache 'HttpClient'](http://stackoverflow.com/questions/25168586/faster-detection-of-interrupted-connections-during-put-operation-with-apache-ht)
- [Interrupting a connecting HTTP request thread incorrectly becomes a timeout exception](https://issues.apache.org/jira/browse/HTTPCLIENT-731)

