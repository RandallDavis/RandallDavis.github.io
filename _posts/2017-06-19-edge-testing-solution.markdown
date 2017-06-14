---
layout:     post
title:      "Microservice Edge Testing Solution"
date:       2017-06-19 11:00:00 -0400
categories: microservice testing edge
comments:   true
---
**_This post originally appeared on [Jet.com's blog][jet-blog-post]._**

I recently wrote a **[proposal on microservice edge testing][microservice-edge-testing]** to state a blind spot in current testing practices and propose a solution. There has also been some [further exploration of the subject][microservice-edge-ownership]. In the proposal, I dug deep into the problem and attempted to keep things agnostic to any specific implementation or architecture. In this post, I'll focus more on the approach we're experimenting with at Jet.com.

# Problem

There are some pretty common practices for testing microservices. These practices are largely used in non-microservice architectures, but microservices introduce some new complexities that push testing needs a little further.

| Testing Type | Coverage | Approach | Caveats |
| --- | --- | --- | --- |
| Unit testing | Individual microservice behavior | Should run in pure isolation, run with every build, be quick and parallelized. | Dependencies are mocked out and simplified. |
| Integration testing | Inter-microservice behavior | Generally focused on finding breaks due to code changes, expensive and complicated to create and run, have real or mocked dependencies depending on sophistication. | Usually geared toward testing vanilla system-wide flows due to complexity of permutations and length of runs. |
| Load testing | System-wide or individual microservice performance | Less frequent runs, usually leverage live dependencies. | Focused on vanilla success code paths. |
| QA | System-wide functionality | Usually focused on functionality visible to users and exercised manually. | Focused on vanilla success code paths. |

The problem in this is that **_non-vanilla interactions between microservices are not being adequately tested_**.

## Why Is This Happening?

The main reason this happens is because there is no natural home for these tests in the current approach:
* *Outside of unit testing, only simple behaviors are tested.* If you have more than one microservice, the focus quickly gravitates toward testing behaviors across many microservices. It's always about the big, important, and likely scenarios. Many scenarios are swept under the rug.
* *Unit tests are focused on code unique to the microservice being tested.* Unit tests are focused on internal behavior within a microservice. To accomplish this, external dependencies are mocked out. This results in the understanding of any dependencies (including external microservice edges) being simplified.
* *Tests are dumbed down to the understanding of a single team.* If we want to test the complex interactions between two microservices, experts on both microservices need to define the tests. Quite simply, this isn't happening. Even if it was happening, it would be costly to upkeep, as microservices evolve at different rates which would make things very complex operationally.

There's a definite gap in our testing, and it happens to be in a very dangerous place. This is mitigated in different ways in different environments. In each environment, there is either a decent amount of risk in Production or the process of getting code into Production is painful. I'd argue that many environments suffer both of these fates.

# Solution

**_The provider microservice edge should ship along with an intelligent [fake][fake-definition] that expresses the provider's behaviors. The consumer microservice should use that fake to exercise its non-vanilla interactions with the provider._** There are a few other requirements to make this work, but the idea here is that the expertise of both systems is represented in the consumer microservice's unit tests.

## Provider Fake

Since the fake of the provider microservice will be used in unit testing, it must meet the following criteria:
* It should be fast and light-weight.
* It should have no external dependencies.
* Its use can be fully parallelizable without shared state or competing for resources.
* It needs to cover both synchronous (i.e. RESTful endpoints) and asynchronous (i.e. queues & pub/sub) interactions.

## Provider Edge Testing

To make sure that the fake stays up to date, the provider microservice needs unit tests to validate it. Basically, we want to test the provider's real behavior and compare it to the fake's behavior. By making it a core part of the provider's code lifecycle, it's going to stay in great shape.

<p align="center"><img src="/_assets/img/EdgeTesting%20-%20providerEdgeUnitTest.png" height="300"></p>

## Consumer Edge Testing

In the consumer's unit tests, instead of mocking out interactions with the provider, the fake is used instead. We want to go beyond what we would normally do in consumer unit tests, and really exercise all the "what if" scenarios that might happen when interacting with the provider.

<p align="center"><img src="/_assets/img/EdgeTesting%20-%20consumerUnitTest.png" height="350"></p>

## Testing Commands

In addition to the standard interactions with the provider's edge, the fake will have additional features that are unique to testing scenarios. Here are a few examples:
* List behaviors that can be tested.
* Pretend you have a dependency outage.
* Pretend you're under heavy load.

# What's Next?

Well, at Jet, we're going to take a stab at building this out and see how it works in practice. Stay tuned!

[microservice-edge-testing]: https://randalldavis.github.io/microservice/testing/2017/06/05/microservice-edges.html
[microservice-edge-ownership]: https://randalldavis.github.io/microservice/testing/edge/2017/06/08/microservice-edge-ownership.html
[fake-definition]: https://stackoverflow.com/questions/346372/whats-the-difference-between-faking-mocking-and-stubbing
[jet-blog-post]: http://tech.jet.com
