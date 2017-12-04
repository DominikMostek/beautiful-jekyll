---
layout: post
title: Interface all the things
permalink: 2017-12-05-on-interface
---
Every code you write has its cost. It is in form of potential error, of time that is needed to read it, document it and change it. Every line of code needs to pay for its own cost and add some extra in order to be profitable. Is adding interfaces always profitable? 


### Readability
Interace can indeed make your code well readable. It hides all the implementation details and forces you to relate only on its contract. But lets have a look on typical interface in [mainstream spring application](http://dominikmostek.cz/2017-02-24-anemic-philippic).

```java
public interface CaseService {
    void touchCase(Case case);
    long countCases(Specification<Case> specification);
    Page<Case> getAvailableAllCases(String search, Specifications<Case> searchSpecifications, PageRequest pageRequest);
    Case getCase(long id);
    List<CaseFlag> getFlags(Long caseId);
    void saveFlag(Long caseId, caseFlagType type);
    List<Photo> getPhotos(long caseId);
    Photo getPhoto(long photoId);
    Photo savePhoto(Long caseId, String filename, String contentType, byte[] data);
    boolean photoExists(Long caseId, Long photoId);
    void saveAttachment(Long caseId, Attachment attachment);
    Attachment getAttachment(long attachmentId);
    void deleteAttachment(long caseId, long attachmentId);
    void signCase(Long caseId, String agreementCode);
    // 30 more
}
``` 

Has creating this interface improved readability of your code? 

**Note #1** if you can't design good class interfaces won't help you. Always stick with [Interface Segregation Principle](http://wiki.c2.com/?InterfaceSegregationPrinciple).

Designing interfaces such this one actually decreases readability for one simple reason. _Ok there are more than one, but one most obvious_. <br/>How do you call class that implements it? It should be something specific about the way how it is implemented. For example `class HttpResource implements Resource`. But what can be specific about implementation of `CaseService`? Perhaps `class IdontKnowWhatIAmDoingCaseService`. 

**Note #2** if you can't think of what is specific about your implementation maybe the interface is not designed well.

#### Testability
Previous interface dost not improve testability either. How do you write stub implementation of this monster? It is close to impossible. What you will end up with is mocking the interface with mockito or some other framework and create very unreadable and flaky tests. 
Not even mentioning that if there would be no interface, only the class, mocking would look the same. So this code has no real purpose. It is just 1:1 enumeration of public methods of some class. 

**Ok everything you have written is obvious if you have 50 methods in interface. Your services should be smaller and all problem go away**

Not really true. The size is problem, of course, but most importantly the problem is philosophy behind this kind of interfaces. The very first reason why they have been written, small or large. 

#### Premature abstraction
Do you automatically create interfaces? Why? I am sure you all of you heard that premature optimization is [root of evil](http://wiki.c2.com/?PrematureOptimization). The idea is that making decisions without enough data leads to wrong decisions. Same applies for abstractions. Without enough experience with modeled domain you can't design good abstractions. This also applies for technical domain not only business domain. When you design your interfaces beforehand and later you find out that they are not very suitable, you have a lot of code to change. And it leads to hacks and tricks in order to fight wrong abstractions.

**Note #3** resist the temptation of premature abstractions. Always create implementation first. Create more implementations first. Then abstractions will be much easier to spot and extract. 

Great technique to train this is [Test driven development](http://dominikmostek.cz/2017-03-06-tdd-tips). When writing code TDD way you write only as much code as needed to pass the test and refactor it later. Interfaces are not usually needed to pass any test so there is not need to write it.

**But what if** I need to write another implementation and there is no interface? Then create it! This is the time when interface will pay for its cost. Adding interface somewhere is easy. There is no reason to add it everywhere _"just in case"_. 


**Note #4** when writing an interface, ask yourself. How this interface will pay for its cost. Will there be several implementations in the near future? Do I need to introduce new domain concept in my app? 

### Cargo cults
It seems to me that in programming community there is a lot of [cargo cults](https://en.wikipedia.org/wiki/Cargo_cult). Making interfaces for everything is one of them. Programmers were taught that interface is good and now they don't even think about it and put it everywhere. But the context is the king here. Sometimes it brings you zero value, maybe even negative in form of extra code to maintain and wrong abstraction which will cause trouble in the future. 

Same principles applies for every piese of your software. Does introducing a layer adds some value **in this particular** case? I might be true in some other case but that does not imply it for any other. 

#### Tips
- Do not enforce interfaces/layers/... where they do not serve any real value. 
- Design interfaces with enough knowledge about the problem
- Design small interfaces
- Practice TDD without mocking frameworks
- Always name implementations about its specifics. 
- Make decisions in context not because of cargo cults