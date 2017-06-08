---
layout:     post
title:      "Microservice Edge Ownership"
date:       2017-06-08 11:00:00 -0400
categories: microservice testing
comments:   true
---
I recently wrote a **[proposal on microservice edge testing][microservice-edge-testing-post]**, which elicited a bit of direct interest and feedback. This is an ongoing and active exploration, but one question from [Daniel Bergey][daniel-bergey] really got me thinking.

To quote Daniel:
> How does this compare to contract testing?  I think they are dual solutions to the same problem - host team provides a fake to the client teams, vs. client teams provide test cases to the host team.

I wasn't previously familiar with [contract testing][contract-testing], and am delighted to learn about its existence. It does, in fact, seek to address the problem that my proposal also seeks to address, but there are a few fundamental differences in the approach. Note that I have no practical experience or expertise (or even a deep knowledge) of contract testing - this post reflects my gut response exploration of the subject.

## Contract Testing Pros

If you dig deeper into the contract testing and the recommendations around it, you'll quickly find [consumer driven contracts][consumer-driven-contracts] and [self initializing fakes][self-initializing-fakes]. Piecing all of this together, we're led into a paradigm where the consumer microservice is able to:
* Express its needs to the provider microservice.
* Be the more dominant owner of the contract that the provider exposes to the consumer.
* Build its own fake of provider responses by recording real provider responses.

This ultimately leads into a relationship microservices where the consumer is king. On the surface, this feels very attractive, leading to the following benefits:
* The consumer can adopt this pattern for any dependency without cooperation. This can be used for both provider microservices that don't provide intelligent fakes, as well as external services (such as 3rd party APIs).
* The consumer is able to express its needs which serve as requirements for creating provider capabilities.
* Any provider capability that is dropped from the consumer's contract indicates to the provider that it can be deprecated (at least for that one consumer).

## Contract Testing Cons

There are some benefits there, and I like the fact that this can be done for immature provider microservices that aren't providing good fakes. However, from a testing perspective, it seems pretty flimsy.

My first issue with this is that **the "tested behavior" is going to reflect what the consumer wants, rather than the provider's actual behavior**. I love that this approach clarifies the consumer needs, so that the provider can focus on building useful functionality that will actually be used, but the greatest benefit here is in clarifying what features get built into the provider. Our intent in testing is not to defend "what should have been built", but to defend "what might actually happen in Production". Our tested behavior needs to focus on the actual provider behaviors and how they relate to the consumer. The provider knows what can go wrong in Production, and needs to drive these concerns forward in testing.

My second concern is that **snapshots of provider responses are insanely naive**. When I asked you what you had for breakfast today, you said "cereal", therefore you will always answer that way perpetually. Really? This makes huge assumptions about the criteria and processes that go into creating that response. This seems like a good way to defend against changing contracts / behaviors, but there are more mature ways to handle change management.

If consumers are driving contract definitions, it means that we are going to have **a lot of contracts floating around that are largely redundant**. I actually have mixed feelings about this one, because I'm not against the idea of having contracts that are explicitly catered to individual consumers. The provider is ideally going to be capable of deduping things behind the scenes to aim toward simplicity and reuse. The danger here is if each contract is so custom that the provider has a hard time keeping things simple internally. Let's not consider this one a negative, but just semi-dangerous territory unless properly managed.

The primary gripe that my proposal focuses on isn't addressed by this consumer-centric approach. The consumer doesn't understand the depth of what can go wrong with the provider, so traditional consumer-driven testing is **only going to exercise vanilla interactions between the consumer and the provider**. This leaves us with a false sense of security. I'm not going to dig deeply into this one, because my proposal covers this in depth.

## Contract Ownership

Let's now put on our product development hats.

While I don't like this approach from a testing perspective, for the reasons mentioned above, this gets into something much bigger. Who owns the contract between the consumer and provider?


[microservice-edge-testing-post]: https://randalldavis.github.io/microservice/testing/2017/06/05/microservice-edges.html
[daniel-bergey]: http://github.com/bergey
[contract-testing]: https://martinfowler.com/bliki/IntegrationContractTest.html
[consumer-driven-contracts]: https://martinfowler.com/articles/consumerDrivenContracts.html
[self-initializing-fakes]: https://martinfowler.com/bliki/SelfInitializingFake.html
