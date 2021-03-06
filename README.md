# Introduction

I have been fascinated and inspired by the generosity of open source projects such as [Spring](https://spring.io/) and Netflix. When I found out that Spring was adopting [Netflix OSS](https://netflix.github.io/) projects into [Spring Cloud Netflix](http://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/1.3.0.RELEASE/), I could not wait to try them.

I find that the best way to learn is by trying something out yourself, so my goal is to try to build-your-own microservices architecture that will work both on your local environment as well as in the Cloud.

[Pivotal Cloud Foundry](http://docs.pivotal.io/pivotalcf/1-10/installing/pcf-docs.html) offers an [out-of-the-box solution](https://spring.io/blog/2015/01/20/microservice-registration-and-discovery-with-spring-cloud-and-netflix-s-eureka) for [multi-cloud](http://docs.pivotal.io/pivotalcf/1-10/refarch/index.html), which seems like a mitigation strategy for Cloud lock-in, while still utilising pure-open-source \(Spring and Netflix\) building blocks.

In this book, I'll be exploring the roll-your-own approach though, to learn how the Spring Cloud building blocks come together, primarily targeting a local dev deployment, but ensuring the architecture/approach can be deployed to a Cloud environment with little additional effort.

We will write a sentence-stats service which given a sentence, will call other services to derive specialised statistics such as word-count and char-count. We'll be keeping the application logic to a minimum so we focus primarily on the service interoperability.

So the plan is to start with a simple set of services which will be tightly bound to pre-defined ports, and then we will evolve to introduce the ability to:

* Loosely couple services using [Eureka](https://github.com/Netflix/eureka) Auto Discover service
* Package and compose all microservices using [Docker](https://www.docker.com/)
* Introduce an API gateway which can both do routing using [Zuul](https://github.com/netflix/zuul)
* Illustrate how to protect and control your services health using [Hystrix](https://github.com/netflix/hystrix)

Hopefully at the end of this journey we'll be in a position to confidently build microservices that can both be deployed in your local environment as well as in a cloud environment.

---

Written by [Joao Vicente](https://github.com/joaovicente)

[![](/assets/twitter.png)](https://twitter.com/TheJoaoVicente)[![](/assets/linkedin.png)](http://www.linkedin.com/in/joaodiogovicente)

This work is licensed under a [Creative Commons Attribution-NonCommercial-NoDerivs 3.0 Unported License](http://creativecommons.org/licenses/by-nc-nd/3.0/)

[![](https://i.creativecommons.org/l/by-nc-nd/3.0/88x31.png)](http://creativecommons.org/licenses/by-nc-nd/3.0/)

