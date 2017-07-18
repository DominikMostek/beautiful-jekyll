---
layout: post
title: Domain objects rocks!
permalink: 2017-07-30-16-domain-objects
---
Today I am going to tell you a little secret. It must be a secret since almost no programmers seems to know about it. Modeling your code based on your domain is the key to maintainable codebase! 

It actually is not secret, it is a well known anti-pattern to use [primitive types to model your domain](http://wiki.c2.com/?PrimitiveObsession). I would like to advocate in favor of domain objects.

### Abstraction
When coding we should always aim at creating domain specific language based on the software use cases (aka domain). 
This is called abstraction. Do not confuse abstraction with indirection. Abstraction is process of hiding complexity and providing tightly targeted objects (functions). Indirection on the other hand is a way to achieve low coupling of components. Those two terms are often tangled together, usually loosing benefits of both in the process. In java/spring world word domain object often refers to ORM entities in [anemic architecture]({% post_url 2017-02-24-anemic_model %}). This is obviously not correct since entities represent structure of the database not your business models and use cases.  

### Domain objects
What are domain objects? They are small objects describing your business. An example from finance software would be classes `InteresetRate`, `LoanAmount`, `Anuity`, `DaysPastDue`, etc. This seems like a boilerplate code since we can represent these concepts by a language primitives like `double`, `int`, `BigDecimal` and others. I would like to explain how using non-primitive objects even for the simplest concepts is beneficial.

<img src="https://i.redd.it/rj8raf1riyny.png"/>

### Maintainability
Most of our programming time is [spent in already existing code](https://blogs.msdn.microsoft.com/oldnewthing/20070406-00/?p=27343/). We have to fix bugs, change behavior or just reuse older code in new components. 

> Most common anti-pattern among Developers: making choices based on speed of initial development rather than ease of debugging.
> — Mikeal Rogers (@mikeal) [24 Jun 2017](https://twitter.com/mikeal/status/878380425529393152)

By spending a bit more time using domain object instead of a primitive will save you many times more in the future.

#### Code readability
Naming variables [is hard](https://martinfowler.com/bliki/TwoHardThings.html). If you do not use domain object you feel the tendency to use the object name as variable name or at least to include it to the name. This is then repeated throughout the codebase again and again. Lets omit that and encode that name directly to the variable type!

```java
double offeredInterestRate ....;
// vs 
InterestRate offered = ...;
```
I would be even tempted to create `OfferedInterestRate` in this case :)
Ok, shorter variable names are not a big deal. But whole method using domain concepts instead of primitives, that is something! Such code could be read by non-programmers too. Try describing `double` and `InterestRate` to your business team, which one will be easier?

#### Apples and oranges
This is the most important consequence of using domain objects in typed systems.
Imagine a simple method `double calcualteInterest(...);` Does it return a percents in `4.99 %` or `0.0499` form? We can only hope that the author of the code included that in the javadoc of that method. Even if he did, you have to remember that every time when you call it and that it will not change unexpectedly in newer versions. 
Use a domain object instead and encapsulate that fact inside that object `InterestRate calcualteInterest(...);`. Now I don't care which form it does have. And there is more!

```java
// consider a method signature
double calcualteInterest(double amount, double annuity);
// and its usage
double amount = ...;
double anuity = ...; 
double interest = calcualteInterest(annuity, amount);
```
Can you spot a bug? Of course you do, its three lines of code! But can you spot that in a large codebase and long pull requests? I know a guy who can spot such bugs with ease. It's in fact his job! He is called type system and together with a compiler he will yell at you every time you make such a mistake. He has one condition though: you have to use types for your domain. 

#### Valid states
Lets take the same method as in previous example `calcualteInterest(double amount, double annuity)` again. Is negative amount a valid loan amount or even negative annuity? This method looks like they are because it will accept them as parameters. We do not have [Dependent types](https://en.wikipedia.org/wiki/Dependent_type) in java and I think we wont have them for a very long time so we should explain our constraints with ... surprise ... domain objects. Objects maintain their own always valid state (OOP lesson number one). Is complete work of Shakespeare a valid email address? No, it is not, so why you have a `String email;` Represent your domain with always-valid objects and you can get rid off validity checks that spread across all your code base. Another point for better maintenance!

#### Documentation
You cannot put javadoc on top of primitive objects and library classes but you can do that for your domain objects! Thank you captain obvious, right? :) This is such a trifle, but it makes a huge difference. When I see `int daysPastDue = ...;` what does it mean? I have to search some internal wiki or ask someone. Instead I could just navigate to that class and read 

```java
/**
 * Represents number of days that user is due with a payment with an annuity. It is calculated as ....
 * Other very useful info.
 * Domain objects rulez!
 */
public class DaysPastDue { ... }
```
<br />

#### Tests
[I mentioned before]({% post_url 2017-03-06-tdd-intro %}) that when TDDing you should not invent the implementation in the test. How can you achieve that without encapsulating the domain in domain objects? By using a `BigDecimal` in a test you are forcing yourself to use that type in your code and thus tieing your test tightly to an implementation.

#### Refactoring
Find usage feature now becomes very usefull since you can actualy search where an InterestRate is used. Searching by variable names, such as `double interestRate`, might not be that successful since it depends on well named variables without typos.

#### Performance
Are you worried about performance of a code that uses custom objects over primitives? Yes it might cause some problems in software in which you are saving milliseconds. That is not the case for most apps. And there is even a chance that your code will perform better. With well structured and designed code, [JVM can perform better analysis](https://wiki.openjdk.java.net/display/HotSpot/PerformanceTechniques) which in turn boosts your performance. And remember to benchmark carefully with [JMH](http://openjdk.java.net/projects/code-tools/jmh/).


<br />
Not everything needs to be a domain object but the more the better.


#### Further reading 
- [Domain Driven Design](https://martinfowler.com/tags/domain%20driven%20design.html)
- [Primitive obsession](http://wiki.c2.com/?PrimitiveObsession)
- [ORM Is an Offensive Anti-Pattern](http://www.yegor256.com/2014/12/01/orm-offensive-anti-pattern.html)