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

The problem in this is that **_non-vanilla interactions between adjacent microservices are not being adequately tested_**. There are a few opportunities to do this (the best of which would be in integration testing), but in practice this rarely happens.

What makes edge testing difficult is that the way that we operate, there's no natural place to handle them:
* *Outside of unit testing, only simple behaviors are tested.* If you have more than one microservice, the focus quickly becomes on testing behaviors across many microservices. The focus is always on the big, important, and likely scenarios. Many scenarios are swept under the rug.
* *Unit tests are focused on code unique to the microservice being tested.* Unit tests are focused on internal beahvior within a microservice. To accomplish this,b external dependencies are intentionally mocked out. This results in the understanding of any dependencies (including external microservice edges) being simplified.
* *Tests are dumbed down to the understanding of a single team.* If we wanted to test the complex interactions between two microservices, experts on both microservices would need to define the tests. Quite simply, this isn't happening. Even if it was happening, it would be costly to upkeep, as microservices evolve at different rates - that expertise needs to address permutations of behavior across differing versions.

There's a definite gap in our testing, and it happens to be in a very dangerous place. This is mitigated in different ways in different environments, but in each environment, either there is a decent amount of risk in Production or the process in getting code into Production is painful. I'd argue that many environments suffer both of these fates.


## Solution Requirements

## Architectural Basics
### different types of microservice architectures
### different types of microservice edge communications
### microservice edge best practices

## Solution Design

[microservice-edge-testing]: https://randalldavis.github.io/microservice/testing/2017/06/05/microservice-edges.html
[microservice-edge-ownership]: https://randalldavis.github.io/microservice/testing/edge/2017/06/08/microservice-edge-ownership.html
