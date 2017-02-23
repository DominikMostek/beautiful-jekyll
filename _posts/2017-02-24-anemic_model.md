---
layout: post
title: Anemic philippic
permalink: 2017-02-24-anemic-philippic
---
Sometimes we spend so much time thinking how to do something that we stop thinking if we even [should do it](https://www.youtube.com/watch?v=0Nz8YrCC9X8&t=1m54s). The thing in this article is [Anemic domain model](https://martinfowler.com/bliki/AnemicDomainModel.html).

To my surprise a lot of my peers dont know the term Anemic model. I think the reason for that is that it became defacto standard of writing java&spring code that you dont need a name for it - it is just normal way of doing things. 


**What do I mean when I talk about standard java code?**
I am sure everyone must have seen this. There is a `user` table in database, `User` entity class, `UserDao` or `UserRepository`, `UserService`. Almost every class name ends with Impl and service/dao interfaces have at least two pages of methods each. Every `....Impl` class has at least twenty dependent classes autowired by [field injection](http://dominikmostek.cz/2016-02-07-dependency-misinjection). 


### What exactly is wrong with that?
Order of the points is random and does not reflect any priorities.

#### Database shapes your business code
Your domain model consist of classes that reflects database structure. There is not a single reason why a change in database structure should have any impact to your business code. Habbit og having one to one business model to domain class relationship is very restrictive and ties your hands in many decisions.

#### It is the oposite of OO
In object oriented design we want to bind our data with the code that gives them meaning. In this case we are doing complete oposite. We tear apart code to services and data to entity classes. 

#### Mutable by default
All entity objects are by default mutable. I see mutability as [premature optimization](http://wiki.c2.com/?PrematureOptimization) and not something we should base our code on. With entity class mutability comes another issue. You cannot guarantee that the entity is in valid state. Just a simple example. 
```
User admin = new User();
```
Given that user has a mandatory username and email we just constructed completely invalid object. It becomes valid once we set these properties. But once passed to other method, the method cannot know if the object is valid or not. Can our second method call `user.getName().length()` and be sure that it will not produce NPE? Of course not. This leads to two situations. We have possible NPE's everywhere in the code or we have code bloated with null checks. Object should **always** be in valid state. 

#### Nesting of entities 
I will begin with an example
```
user.getMembership().getProduct().getCategory();
```
This is so called [Train wreck](http://wiki.c2.com/?TrainWreck). Also it is serious violation of [Law of Demeter](http://wiki.c2.com/?LawOfDemeter).
It leads to yet another NPEs. But what I dislike most about it: It almost completely disable testing abilities. The test then looks like

```
max = new User();
max.setMemberShips(new ArrayList());
membership = new Membership();
product = new Product();
product.setCategory(category);
membership.setProduct(product);
max.getMemberships().add(memberships)
...
```

If setup even for a simple test is nightmare, no wonder that programmers do not want to write unit test. 
What it also leads to is that your code is highly coupled with structures of the data and any change to the structure results of changes of business code. 

#### Code reuse in service layer
As your requirements grow you will find yourself in the situation that you need to share some code in several services. There are several options. The most used one is also the ugliest one. Having an abstract class with common code and anyone who wants to use it inherits the class. Inheritance was invented as a tool for extension of existing code not for sharing common code. We have composition for that. If you want to do math calculations you also don't subclass `java.lang.Math`

#### Testing of service layers
Record of autowired dependencies I have ever seen is 82. That is impossible to test. You never know which dependency will be called from the method under the test so you [mock them all](http://dominikmostek.cz/2016-02-07-dependency-misinjection) just in case. Those service layers also often violates [SRP](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod) because the feature is not a composition of small testable pieces but a single huge method. This is on of the reasons there are lot of questions ["how do I test private methods"](http://stackoverflow.com/search?q=test+private+methods) on stack overflow, because everybody feels that testing those public monsters is impossible. With violation of SRP comes not only testing problem but also lot of other issues. 

#### Those bloody interfaces
Because you cannot share code easily you stick any new method to the existing service. This service have, for some mysterious reason, an interface. So you add your new method to that interface. This iface now have tens of methods. Another violation, this time [interface segraegation](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod). Your interface has usualy only one implementation so why is there an interface in the first place? Because you are violating SRP there is no single word that could describe your implementation so you just stick `Impl` to the end of the class name instead of naming it properly.  

#### Ok, show me the alternative
Ok I told you how not to do it. Now you expect that I give you list of frameworks, tools and architectures you should use instead. But there actually is no need for new alternatives. [As history of programming shown](http://homepages.cwi.nl/~storm/teaching/reader/Dijkstra68.pdf), sometimes it is simply enough to stop doing stupid things in order to move forward. 
In this case alternative is simply stick to [good ol' SOLID](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod) principles. Practice TDD and pay atention to clean code.


**Disclaimer:** Anemic model is tighlty linked with ORM and J2EE standards, but I wanted to keep this article purely a rant about anemic model and the way how we design our apps. Rant about ORM would add another pages to this post so I will leave to the future. I realize that some points are not critique of anemic model directly but simply a bad code, but I also think that anemic model way of programming stimulate our inner bad-coders to write that kind of code.