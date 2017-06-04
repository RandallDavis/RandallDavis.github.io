---
layout:     post
title:      "Microservice Edge Testing"
date:       2017-06-05 11:00:00 -0400
categories: microservice testing
comments:   true
---
# Introduction

This is a proposal on how microservice interconnectivity layers (AKA edges) should be tested. The main focus is to present a glaring blind spot that currently exists in microservice edge testing and propose a solution, but this leads into other implications / considerations. Some of this relates to what I've seen and directly put into practice, but the purpose of this work is to push beyond my experiences and delve into exploratory territory. Ideally, this will surface some better practices that will improve the landscape of microservice architectures.

# Background

A few years ago, I was on a project to create a microservice framework. We wanted to enable engineers to move their code off of a monolithic architecture and into microservices without losing momentum, hitting heavy learning curves, or having to migrate everything at once. We had a big emphasis on testing capability, so there was some magic done to make for easy mocking of external microservice interactions - we didn't want to introduce any barriers in unit testing microservices in isolation. The focus of the project was to build the framework and uphold the status quo as much as possible. It was the right focus, but in the midst of this, something struck me as fundamentally wrong. I had an idea on how to address it, but it wasn't worth bringing up at the time because the project was already very ambitious.

Everywhere I look I see that same fundamental flaw, and I really think it's just the way that things are done. Microservice architectures raise it to the surface and make it more evident and painful, but I think it's been there all along. The solution I envisioned a few years ago still seems to be the answer, but now that I've gained more experience and a handful of new grey hairs, I think that the exact implementation needs to be innovated on to really flesh it out.

# The Problem

Let's say that one microservice interacts with another. For sanity, we'll say that a "client microservice" interacts with a "host microservice". They're both peers in an overall microservice ecosystem, but in this case, the client has a dependency on the host. Let's say for now that it makes synchronous calls to an API layer. The microservices are owned by different teams (let's call them Team Client and Team Host), and both teams get big pats on the back because they take testing seriously. What we'd expect to see is that both microservices are unit tested in isolation and then their interactions are tested in a live integration testing environment. Everybody's happy, everybody feels safe.

Digging a little deeper into this, we'll find that the client's unit tests are focused exclusively on testing the client's behavior. That makes sense. In reality, the client's behavior depends on the host's behavior, but that's largely out of scope for the client's unit tests. Any interactions with the host are likely to be mocked out within the client's unit tests to gain the necessary isolation. Those interactions will be exercised in integration tests anyway, so Team Client really wants to focus on the intricacies of the client's code in their unit testing. The host's failure conditions and edge cases get punted on, so that we're not distracted from covering the nuances of the client's behavior.

Okay - so far, so good. So now we get to integration testing. In an ideal world, we'll first focus on the specific interactions between the client and host and then separately test out larger scale flows involving many microservices. Usually there's more of a focus on the larger interactions because they inherently exercise the smaller scale interactions. It's a trade-off, but it's a common one and usually it's not wrong. Integration tests are heavy - there's a lot to do, and we might want to test conditions under load. Sacrifices have to be made, so we are more likely to only exercise the vanilla interactions between the client and host. Hopefully we'll hit some failure scenarios too, but most likely we're going to focus on successful code paths. Chaos engineering can get some of those failure scenarios tested for us by picking machines off in a QA environment, but we don't have a regular, proactive focus on failures.

So - we feel relatively safe, but there's a sore point in our testing. We aren't really exercising the non-vanilla interactions between the client and the host. We acknowledge this and are going to feel compelled to do two very good things to derisk this. First of all, we're going to keep the host's edge really simple and intuitive - that's a huge win, and we need continual pressure to keep that practice in place. Second, we're going to invest in some additional unit tests in the client to cover some of those scenarios we're concerned with. Gold stars all around.

So, we're safe right? In my opinion, we're not. Here's the issue I have with this. Team Client is solely responsible for testing interactions with the host. This means that the knowledge behind that coverage is going to be dumbed down to Team Client's understanding of the host's behavior. Those tests will end up being the lowest common denominator between what Team Host bothered to communicate, what Team Client was capable of understanding, and Team Client's prioritization of working on those tests (which is only driven by fear stemming from recent Production problems). Even worse, as the host's behavior changes, it will be really hard for Team Client to understand those changes and align test creation with the host's release cycle. This is an unnatural paradigm that is set up to fail.

I want to be clear about that failure potential. This is one area of microservice architectures that scares me a lot. I'll get into this in a little more depth later, but coordinating behavioral changes across microservices is really hard to do, and that danger is most prominently expressed in microservice edges. One of the most brittle parts of our architecture has the least disciplined testing practice. Do we still feel safe?

Theoretically, the answer to the problem is simple:
* Team Host, who best knows the behavior of the host microservice, needs to have the responsibility of expressing the host's edge behavior.
* Team Client needs visibility into the depth of their testing coverage of the client's interactions with the host's edge.

That's all easier said than done. Word of mouth fails. Edge specifications aren't kept up, get ignored, or are misunderstood. Team Host can't write tests for Team Client without suffering the same knowledge gaps (and who would want to do that anyway?). We need a more formal solution.

# Proposed Solution

So here's what I think we need to do. The host's contract needs to ship with an intelligent fake of the host that Team Host maintains. To be clear, a fake is a lightweight alternate implementation of the host that is intended for testing purposes - [this might help with terminology][fake-answer]. The client's unit tests should be able to leverage the host's fake in isolation to test real-world behavior. The fake is still only going to be as good as Team Host conceives of being necessary (although the next paragraph helps with this), but that's already a weakness in any unit testing. What we've gained here is that the host's fake evolves alongside the host itself and is owned by Team Host. It's more likely to be true and complete. There are other benefits and considerations, but this is the big win.

So far this sounds like a separate codebase that's built alongside the host, but doesn't have any direct correlation to it - the fake is likely to be incomplete unless we do something about that. The solution is that Team Host should dogfood their fake for testing their edge in their unit tests. The fake becomes the behavioral definition of what the host's edge is supposed to do. The host's unit tests should include assertions that the fake behaves the same way as the host itself - if this practice is embraced, the fake is going to be kept in great shape and becomes a powerful tool for both teams. It even paves a way toward Behavioral Driven Development on the edge, which is where such a practice has the most value. The fake truly describes the host microservice's behavior as a black box.

By the way, remember how I said that it was a good thing to feel pressure to keep contracts simple? Now Team Host feels that pressure directly from its own system and is therefore even more likely to do the right thing. Dogfooding is wonderful.

To deal with the fact that Team Client still isn't going to know which exact behaviors to exercise, the fake should be able to keep track of which behaviors have been tested. At the very least, it should provide or ship with an inventory of testable behaviors.

There's way more that I'll cover, but this is the critical stuff. I've never seen this done in practice, and have never witnessed this being the intention. The status quo on this makes me sad. In searching the web for anyone else adopting this practice, I was able to find a bright shining beacon of hope in [this blog post][vertical-slice-testing] by Sebastian Lambla - huge props to him. Unfortunately, further evidence of this practice appears to be scarce, and his blog post is from 2013.

# Versioning

As I said earlier, managing change across microservices is really hard. It's wonderful and powerful to let each microservice evolve at its own cadence, but it's also dreadful when you consider the diverse versioning landscape you end up with as a whole and the unknowns that brings into play. I've witnessed many approaches to managing this, and they all have huge tradeoffs in developer freedom, build times, testing cycles, Production risk, cost, etc. That is a gigantic and exciting topic that I have opinions on, but I'm not going to preach a single answer here - both because it would distract from my overall point and because I don't have a single answer to offer.

I will however, make a firm assertion. Regardless of the approach, explicit versioning is critical for microservice edges. There should be absolutely no ambiguity in what capabilities the host is providing, and there should equally be no ambiguity in what the client is expecting. If we think of edges as being the mechanism by which two microservices talk to each other, it's fair to think of the edge version as being the dialect that they're speaking. Languages change subtly, and without knowing the exact dialect, you may not know the true intent. We can strive for backward compatibility in contracts, but the plain fact is that if you don't know the contract version, you're missing a critical piece of information.

There are obvious operational consequences attached to not tracking this. Say that the host has a method that should be deprecated, but Team Host just isn't sure if any teams might still need it - better play it safe and keep that method around. I've seen that one a lot. If Team Host knew which contract versions were being used and knew which version dropped the method, it's pretty obvious whether or not they can stop supporting it. Even better, if there's a straggler that needs to upgrade their contract, Team Host knows exactly who to hunt down.

I personally think that we should go as far with this as decorating all of our data with versions. I don't just care about knowing the versions of live system interactions - I want explicit information on data in flight and potentially even data that's being permanently stored. Knowledge is power.

There's a particular benefit to having explicitly versioned contracts coupled with the intelligent fakes that I'm pitching. When the host's behavior changes and has an impact on the client, it's suddenly very easy to reason about. Even better, because the fake can describe the behavior of any given version, we can see problems coming ahead of time rather than being reactive.

Without this, if the host rolls out a new behavior, and the client is a couple weeks behind in adopting it, it's a huge pain point if there's any incompatibility. Either Team Client has to catch up under duress or Team Host has to figure out how to roll back (which might cause problems for other microservices that are already relying on the new behavior). A simple lack of coordination just caused a real problem. Discipline and transparency in upcoming behavior changes are a huge help in heading these situations off.

# Synchronous vs. Asynchronous Interactions

It's easier to envision testing against the fake for synchronous interactions, but that's not going to be enough. Asynchronous interactions are powerful tools in microservice scalability, and at all times we should encourage their use. Our edge behaviors are likely to include one-way queues, pub/sub patterns, etc., and those behaviors have to be included in our testing. In some cases the client is going to have a causal effect on the host doing work, and at other times it will simply receive data that it had no hand in creating. All of these things need to be represented in our fake.

We want to uphold isolation in unit testing, so ideally we can have streaming data interactions with the host's edge without having dependencies on specific infrastructures. For example, if in Production the client subscribes to a Kafka topic to consume data that the host produces, the fake shouldn't be forced to have any dependencies on Kafka. The fake should be able to produce a data stream without hitting any global infrastructure or being bogged down with its own infrastructure in containers. This requires a bit of elegance around structuring the interactions with the host's production streams so that the same code can instead leverage the fake. Infrastructure abstractions are our friends here.

# Error Outputs

Since I'm already on my soapbox, I'm going to throw in another piece of advice. Errors are really useful. If a microservice is failing to provide successful behavior, it's really important that it provides information to state that fact. No news is bad news - the client needs to be able to differentiate failures from pending successes. If the host is failing for some reason, the client needs to know that so it can act accordingly. I consider errors to be outputs, the same as any output resulting from successful behavior.

However, we want to keep our contracts simple, and errors are a part of that contract. The error information that the host provides to the client should have a decent amount of abstraction. Should the client be aware of which part of the infrastructure is giving the host trouble? Probably not. Team Host needs detailed error information, but the response from the host is a different story. In my opinion, error outputs should be sculpted to the needs of the client's perspective - usually that results in things being very simple. Since Team Client is getting a simple contract for error conditions, there should be no barrier in incorporating handling logic and writing unit tests to defend that logic. We all just got happier.

# Failure Simulations

One thing that our intelligent fake is not going to provide is anything relating to load. The fake is going to be lightweight, ideally operating entirely in memory with no external dependencies, and the data it provides will be pre-determined rather than being generated through painstaking processes. Its job is to simulate behavior, but some behaviors just can't be simulated. Make no mistake - our unit tests are not going to take the place of a live environment that can experience load.

However, the whole point of our fake is to let the client experience the host's behaviors in isolation. Some of those behaviors will only be triggered if the host is experiencing duress or a weakness in a dependency, whether that dependency be another microservice or an infrastructure resource. In some cases, the host might time out or provide garbled responses. Known behaviors are worth expressing in the fake, but we might have to get a little clever about how or when to simulate them.

The real trick here is that the client will need a way to ask the host's fake to express these behaviors. Since that part of the client's communication to the host's edge would never exist in Production, it's not going to fit naturally into the host's contract. That's okay. Let's just know that up front and keep things organized. The fake's contract is going to end up being an extension of the host's contract that provides additional testing-related capabilities.

# Conclusion

Well, I've said a lot here, and there's no way someone can write a technical document this long without angering the masses. To me, a lot of these little pieces of advice fit together into a larger machine that works pretty well. My main concern, however, is the core argument that I'm trying to make.

I propose that intelligent fakes should be shipped along with microservice contracts to simulate microservice edge behavior, that the proper practices should be put in place to ensure those fakes' validity, and that microservices interacting with that edge should have adequate edge testing coverage. The glue between microservice interactions is potentially the flimsiest part of microservice architectures, and this directly addresses the ambiguities that currently seem to be prevalent.

I've stated a bunch of scattered requirements and suggestions that should be considered in any implementation of these fakes, but have intentionally stopped short of suggesting a specific design. I don't think there is a single true design to handling this. The design needs to fit naturally into the microservice ecosystem that this is meant to serve - those ecosystems vary a lot and are still evolving. My point is that we've been missing something along the way, and it needs our attention. We need to start building these intelligent fakes and working them into our testing cycles. We're behind on this, and need to start innovating. The next time a document like this emerges, it should be stating best practices from experience, rather than expressing a need for this to exist at all.

[fake-answer]: https://stackoverflow.com/questions/346372/whats-the-difference-between-faking-mocking-and-stubbing
[vertical-slice-testing]: https://serialseb.com/blog/2013/07/11/unit-testing-is-out-vertical-slice-testing-is-in/
