---
layout: post
title: Layers of hell
permalink: 2018-06-03-layers-of-hell
---

Before reading this post, please read these two articles [Your coding conventions are hurting you](http://www.carlopescio.com/2011/04/your-coding-conventions-are-hurting-you.html) and [The madness of layered architecture](http://johannesbrodwall.com/2014/07/10/the-madness-of-layered-architecture/). If you understood them and agree with the point, stop reading now. I have nothing to add. Those articles are perfect. The reason I am writing another one is most of the people I work with are still convinced that layers and conventions solves problems better than code that fits the solution best. I have written twice about writing code that does not add any real value, it's just there because programmers are so used to do things one way even if it makes no sense. One is about [anemic model]({% post_url 2017-02-24-anemic_model %}) the second about [interfaces]({% post_url 2017-12-05-on-interface %}).

#### TLDR; version
Conventions and established rules do not lead to better code. The decision process which preceded the creation of those conventions and the context of that decision does.

### Original problem
What I want to describe is very general, but the discussion erupted around specific code. So I will describe it first.
The component I am writing right now is a translation component of API. There are lots of services on one side (with ugly api sometimes) but the goal is to have single API and the complexity of the services and ugliness of old APIs is hidden behind a facade. 
We call it an API Adapter module. It is written in [spring boot](https://projects.spring.io/spring-boot/), [map struct](http://mapstruct.org/) and [feign](https://github.com/OpenFeign/feign). Typical task is: translate one DTO of new API into one or more DTOs of old API (using mapstruct mostly) and call the corresponding services (feign clients).

### Naming again?
Naming, the hardest problem in computer science. How would you call the class where this all happens? Since it is also exposing the endpoints of new API we called it `*Controler`. We got also Mappers and Clients (both generated classes) which are dependencies of those controllers.  
Is it ok to have only `Controller` that does all the work without any additional layer? 

##### Intermezzo
> Adding `@RestController` and its mappings does not add another responsibility to the class. The class is completely usable without this functionality. This responsibility is added by runtime environment  (spring context). The code without applying any annotation processor (aspects, proxies, etc..) does only what is inside its methods not a bit more.

### The layer Objection 
What my colleague sees as a problem is that now we are mixing responsibilities in controller and the logic should live in another layer.
But is there some mix of responsibilities? As I said in intermenzo there is no controller responsibility. What is the component responsible then for? Calling mappers, doing some non-trivial mappings and calling some service using client component. Do you see a layer in there? 
I don't. The nontrivial mappings? 

```java
if (loanDetail.getInvestments() != null) {
    detailMO.setInvestorCount(loanDetail.getInvestments().size());
    detailMO.setAverageInvestment(getAvg(loanDetail.getInvestments()));
}
```
mostly. Exception handling sometimes. Collection filtering etc...

Does such simple code justify another layer? In my view not at all. Adding more code must always have a non-negative value. It must solve important business or technological problem. Is desire of having conventions everywhere so important that it pays for the cost of extra code? 

Maybe all the misunderstanding is based on a simple naming. It is named a `Controller`, but is it a controller? Or is it an adapter with some controller annotations applied? 

Have a look at another example. Very simple example

```java
@RestController
public class SomeController {
	private final SomeService service; // somehow injected
	@RequestMapping("/some-resource")
	public SomeDTO getMeSome(Long id) {
		return service.getSome(id);
	}
}
// and the service 
@Service
public class SomeService {
	@Transactional
	public SomeDTO getSome(Long id) {
		return ....; 
	}
}
```
We have layers. That means that we do have clean and highly maintainable code. Right? But what is the difference with code like this

```java
@RestController
public class SomeService {
	@Transactional
	@RequestMapping("/some-resource")
	public SomeDTO getSome(Long id) {
		return ....; 
	}
}
```
It does the same thing with less code and less layers. The class name is wrong now, but apart from that it is completely ok. 

Wait. Am I suggesting to get rid of controllers? Not at all. But if your app does something so simple and most of your controller methods contain only simple service method calls that there is no single reason for them. And one of your 30 controllers that does need some extensive logic? Yeah, then find solution inside the layer you have. It does not have to be ugly. **Do not make rules based on rare use cases**. When the situation changes and now most of the components on the layer does "controller-related" work, then its time to move on and create a layer. 

<center>
	<font size="+4">
		<strong>e = mc<sup>2</sup></strong>
	</font>
	<br />
	<font size="+2">
		<strong>Errors = More Code<sup>2</sup></strong>
	</font>
</center>

The beauty of so called technical debt is that it will let you postpone the decision into the future where you have much more experience and you are able to make much better decisions than you are now. Making such rules and conventions upfront means you have no data to base your decision on. Its a pure guess that you will need the extra layer. Maybe new framework or technology will be born in the meantime that will solve the problem better than your layers. Or you will learn better pattern. But making it upfront, you lost a lot of money and time building something that you are now prone to [sunk cost fallacy](https://en.wikipedia.org/wiki/Sunk_cost#Loss_aversion_and_the_sunk_cost_fallacy).

### Add value not code
What pays the extra code of extracting this code into separate component is ease of testing or code reuse not conventions. 
But what about code readability and clean code rules? Does having layers add readability "per se"? Again no. Adding a layer does not  automatically means readable code. Wrapping ugly method in a layer or a component does not make it beautiful. 

Another point my colleague was making is that people are used to layers and they expect them and are confused when they do not find them. That is actually a valid point. We have a saying in Czechia "stinky but warm". It means that being comfortable with something does not make it smell good. You should make choices based on context, which fits the current problem not based on conformity. 

**Context based choices over [cargo cults](https://en.wikipedia.org/wiki/Cargo_cult)!**

Every line of code, every class, every layer and component have its cost. When creating it, be prepared to defend the cost with arguments and examples where it will pay for itself. 

Show how simple tests can be now written. Show some past change that would be now much easier with your extra layer. Is there a lot of code that can be removed with your layer? Show the value or don't write that code.

Few quotes on topic

> A change in perspective is worth 80 IQ points. 
> ~ Alan Key

> It seems that perfection is attained not when there is nothing more to add, but when there is nothing more to remove
> ~ Antoine de Saint Exupéry

Also note the rule four from book [4 Rules of Simple Design](https://martinfowler.com/bliki/BeckDesignRules.html): "Soultion has the fewest possible elements (classes and methods)"

After publishing this post I came across a post from Yegor256 [Speed vs Quality](http://www.yegor256.com/2018/03/06/speed-vs-quality.html). Which basicly says - if some standard quality is met proceed as fast as possible because that is what makes you money. Adding a layer just to fulfill some stupid convention is not adding quality and keeps you from delivering. 

The [layers of hell reference](https://en.wikipedia.org/wiki/Inferno_(Dante)#Nine_circles_of_Hell)