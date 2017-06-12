---
layout:     post
title:      "Microservice Edge Testing Solution"
date:       2017-06-014 11:00:00 -0400
categories: microservice testing edge
comments:   true
---
I recently wrote a **[proposal on microservice edge testing][microservice-edge-testing]** to state a blind spot in current testing practices and propose a solution. There has also been some [further exploration of the subject][microservice-edge-ownership]. In the proposal, I dig deep into the problem and attempt to keep things agnostic to any specific implementation or architecture. In this post, I'll focus more on the approach we're experimenting with at Jet.com.


## Problem

There are some pretty common practices for testing microservices. These practices are largely common to non-microservice architectures, but microservices introduce some new complexities that push testing needs a little further.

| Testing Type | Coverage | Expectations | Realities |
| --- | --- | --- | --- |
| Unit testing | Localized microservice behavior | Should run in pure isolation, run with every build, be quick and parallelized. | Dependencies are mocked out and simplified. |
| Integration testing | Inter-microservice behavior | Generally focused on finding broken behavior due to code changes, expensive and complicated to create and run, have real or mocked dependencies depending on sophistication. | Usually geared toward testing system-wide flows, generally focusing on vanilla success and failure code paths due to complexity of permutations and length of runs. |
| Load testing | System-wide or individual microservice performance | Usually exercising live dependencies. | Focused on vanilla success code paths. |
| QA | System-wide functionality | Usually focused on functionality visible to users and exercised manually. | Focused on vanilla success code paths. |

The problem in this is that **_non-vanilla interactions between microservices are not being adequately tested_**.

What makes edge testing difficult is that the way that we operate, there's no natural place to handle them:
* *Outside of unit testing, only simple behaviors are tested.* If you have more than one microservice, the focus quickly becomes on testing behaviors across many microservices. The focus is always on the big, important, and likely scenarios. Many scenarios are swept under the rug.
* *Unit tests are focused on code unique to the microservice being tested.* Unit tests are focused on internal beahvior within a microservice. To accomplish this, external dependencies are mocked out. This results in the understanding of any dependencies (including external microservice edges) being simplified.
* *Tests are dumbed down to the understanding of a single team.* If we wanted to test the complex interactions between two microservices, experts on both microservices would need to define the tests. Quite simply, this isn't happening. Even if it was happening, it would be costly to upkeep, as microservices evolve at different rates - that expertise needs to address permutations of behavior across differing versions.

There's a definite gap in our testing, and it happens to be in a very dangerous place. This is mitigated in different ways in different environments, but in each environment there is either a decent amount of risk in Production or the process of getting code into Production is painful. I'd argue that many environments suffer both of these fates.


## Solution

My proposal dives into a solution in more depth, but let me cover the basics here.

What we're trying to accomplish is that there is thorough testing of the behaviors between a "consumer microservice" interacting with a "provider microservice". In this, we want the expertise of both teams to affect our testing. We also want to create maturity in our process such that this practice is ingrained in the way we work - we want to be able to depend on the accuracy of these tests.

How do we accomplish it? **The provider microservice edge should ship along with an intelligent fake that expresses that edge's behaviors. The consumer microservice then has unit tests to exercise its non-vanilla interactions with the provider's edge.** There are a few other requirements to make this work, but the idea here is that the expertise of both systems is represented in ther consumer microservice's unit tests.

### Provider Edge Fake

Since the fake of the provider microservice's edge will be used in unit testing, it must meet the following criteria:
* It should be fast and light-weight.
* It should have no external dependencies.
* Its use can be fully parallelizable without shared state or competing for resources.

### Provider Edge Testing




### Consumer Edge Testing







## Architectural Basics
### different types of microservice architectures
### different types of microservice edge communications
### microservice edge best practices

## Solution Design

[microservice-edge-testing]: https://randalldavis.github.io/microservice/testing/2017/06/05/microservice-edges.html
[microservice-edge-ownership]: https://randalldavis.github.io/microservice/testing/edge/2017/06/08/microservice-edge-ownership.html
