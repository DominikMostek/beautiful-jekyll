---
layout: post
title: Logger and Unit testing
permalink: 2017-06-19-logger-unit-testing
---
Have you ever considered using logger as a tool for unit testing your components or even testing logging itself? There is my take on this topic. TLDR version: don't do it.

#### Injection of a logger
Logger as an injected dependency? It even sounds wierd! There is no reason for logger to be injected. Logging is something that does not need to be switched at runtime, you dont need different logger for each instance, etc.
That is why logger initialization is always done statically:

```java
private static final Logger logger = LoggerFactory.getLogger(MyClass.class);
```
But how do you test logging in your unit test if it isn't injected? Simple answer: you don't use the logger in tests!

#### Do not test the logger
Logging is a concern that is not related to your business case. It is purely technical concern that we do in order to have enough data to monitor our software at runtime. Your unit test on the other hand should express use cases of your code. They are examples of what are valid inputs and expected outputs. A string logged to console or a file is not something I care about when I want to call a method on your class. 

Line of logging is something I expect that can be changed without breaking any builds. Its just some info string! 

I mentioned in my [in my previous post]({% post_url 2017-03-06-tdd-intro %}) that relation of inputs and outputs should be clear in unit tests. No magic values should appear wihout any context. How would an assert in test which uses logger to verify something look?

```java
verify(logger).warn(eq("Something happened wit class com.example.MyClass"));
```
There is no context to this string. Why this string? How is it related to the input? Is order of the words important? 


#### But how do I test ...
The best thing about unit tests and TDD is the feedback. One possible feedback you can get from your unit test is: **If it is hard to write the test, there is something wrong with your design**. How do you test the logger if it is not a injectable dependency? Some programmers will tend to do some ugly hacks such as

```java
ReflectionUtils.setField(target, "logger", loggerInstance);
```

This is screaming "You are doing it wrong" all over the place! If you must use reflection to setup your instances to be usable in unit test there is something really wrong with the design of the code. 

If you consider using the logger as a peephole to the code under the test, it is again the test giving you feedback. If there is no other way how to get the results from your code there is something wrong with it. 

Your unit tests are documentation for other programmers. What would you think about a component that is not able to give you results in some sane way, except hacking it through reflection? It would be [huge WTF](http://www.osnews.com/story/19266/WTFs_m), right? 

#### There is always an exception
If logging happens to be a important part of your class or unfortunately there is some third party system relying on logs of your app you should test your logger! So now it is OK to use reflection to set my logger? No, never do that! Simply add a logger as a dependency of your class. This communicates to everybody that logging is some important aspect of your code and should be changed carefuly. It also adds some flexibility to the future. The 3rd party system might change and now it expects JMS message instead of reading your log file. With logging as a dependency you don't have to change anything in your class.