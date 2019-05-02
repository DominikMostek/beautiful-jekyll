---
layout: post
title: Tests are not a way out of misery
permalink: 2019-04-28-tests-no-way-from-misery
---

More test equals more quality code and product equation is false. There is one piece missing in that equation. 

#### My friend John
Meet John. He is an average Java developer with CV full of senior positions and list of three letter abbreviations so long that you don't even read it.
John is a good developer and his code is ok. He even writes tests! 
Typical code of Johns. 

```java
public class CalculationService {

    public CalculationService(WtfRepository wtfRepository) {
        this.wtfRepository = wtfRepository;
    }

    public BigDecimal calculateWtfScore(Long userId) {
        List<Wtf> wtfs = wtfRepository.loadForUser(userId);
        int score = calculateWtfScore(wtfs);
        return new BigDecimal(score).multiply(BigDecimal.valueOf(0.3465));
    }
    
    private int getScore(List<Wtf> wtfs) {
        // some very very complicated calculation!
    }
}
```

His algorithm is perfect. It is super-efficient and without any bugs! From my point of view its still not good. Because it is untestable and not reusable at all. 

> But I wrote a test so it is testable 

objects John. Here it is:

```java
@ExtendWith(MockitoExtension.class)
class LoanServiceImplTest {
	@Captor
    ArgumentCaptor<Long> userIdCaptor;
    @Mock
    private WtfRepository wtfRepository;
    @InjectMocks
    private CalculationService service;

    @Test
    public void testWtfCalculation() {
    	when(wtfRepository).thenReturn(…);
    	// and so on
    	verify(captor)…
    }
}
```

In this case I would argue that existence of a test is not a proof of testability. When I say testable I mean I can write a short piece of code in pure java without any library, reflection or any kinnd of hack, that will run my code and tests that with given input the output is correct.
What I mean by testable code is for example

```java
assertThat(new WtfScore(Arrays.asList(…)).forCoeficient(0.3465)).isEqualTo(xyz);
```

But where is the repository now? 
Its still there in the service 

```java
public class CalculationService {

    public CalculationService(WtfRepository wtfRepository) {
        this.wtfRepository = wtfRepository;
    }

    public BigDecimal calculateWtfScore(Long userId) {
        List<Wtf> wtfs = wtfRepository.loadForUser(userId);
        return new WtfScore(wtfs).forCoeficient(0.3465);
    }    
}
```

What I have done is I have moved the code to its own class. What is the difference between making the previous method `getScore` public or even static?
Making it public would still require construction of a service (and its dependencies) in order to test the calculation. 
Making it static is not composable in an object oriented way and also is not lazy-evaluable. 

That was just first step. Lets say that the computation is very complicated. The code is imperative with many for loops, ifs...
It can be tested now but in order to test that some weighted average inside that method works fine we need to create a list of some objects. That is again not testable.

#### Composition is the king

As a developer we have to solve complex problems by decomposition to a smaller easier problems. 
Our WtfScore class is just a composition of smaller computations put together in an imperative way. If we wrap those small computations in classes we get many benefits.
One of them is testability. I can test weighted average as a single unit and then just use it with confidence that it works. After some time in project I will get a nice library of small classes such as `WightedAverage`, `BatchIterator`, `LazyValue`, `ScaledBigDecimal`, `FormattedText`, and so on... 
All these classes will be well tested and documented. Your complex logic will now be just a composition of small classes.
This is a composition not calling five services from a controller.
It also comes up with the benefit of much more readable code. 

This is an example of some code I wrote

```java
Consumer<OrderQueueItem> consumer =
    new SwallowExceptionConsumer<>( 
            new CreateStalledOrderOnErrorConsumer(
                    new RemoveItemOnSuccessConsumer(
                            new LogExceptionConsumer(order -> processSingleOrder(order, currentDateTime)),
                            jobItemsQueue),
                    currentDateTime,
                    this.stalledOrderRepository));
```

I can see the whole chain of operations and if I am interested in implementation details I can read it in isolation from the problem I am trying to solve. For me that is so much easier than a imperative code that must be decoded as a whole. 

Yes it is a lot of classes. But eventualy you will reuse most of them when solving new problem. I love that feeling when I am coding and suddenly I realize that I can solve it just by correctly composing classes I already have. 

And yes it is not a coincidence that it is so similar to functional programming. FP and OOP have so much in common. In my eyes objects are just a set of partialy applied functions. 

#### You write shitty code

Now to the core of a problem I wanted to write. And its testing and writing tests. From time to time a bug in production occurs. Each time this happens someone will say it is because we do not have enough tests and we should write more tests. And we should do TDD! 

The problem is that writing more tests, as the Johns one at the top, will not help. You will just invoke crappy code more times. 
Teaching TDD also wont work. People will try to write the same code as before but with the test before. And it will be pain in the ass. So they will fallback to old way after some time anyway. I have seen this so many times in my professional life.

Programmers are not lazy to write tests, they also know that tests are good. They also know how to write tests. 
**The most crucial thing they don't know is how to write good code in general. They have never seen it.**

TDD is in my eyes much more about the good design of a code than about testing for correctness.
If a developer will start writing tests for correctness and will completely ignore the design feedback such tests are worth close to nothing. 

Writing good code and good tests requires ego-less aproach. Sometimes you will caught yourself totally unprepared for some design decisions, or will have to admit that you were doing something wrong for tha last ten years. Less ego leads to better code.

I write shitty code, everyone does. But I am trying to improve it step by step.

### Links

[No DB - The Clean Code Blog](https://blog.cleancoder.com/uncle-bob/2012/05/15/NODB.html)

[Architecture, the lost years](https://www.youtube.com/watch?v=HhNIttd87xs) 

**My other related posts**

[Anemic philippic]({% post_url 2017-02-24-anemic_model %})

[Tips for TDD and unit tests]({% post_url 2017-03-06-tdd-intro %})

[Domain objects rocks!]({% post_url 2017-07-30-16-use-domain-objects %})
