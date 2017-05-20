# Introduction

I have been fascinated and inspired by the generosity of open source projects such as Spring and Netflix. When I found out that Spring was adopting Netflix cloud OSS projects into [Spring Cloud](http://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/1.3.0.RELEASE/) which have heavily influenced core services built inside AWS, I could not wait to try them.

I find that the best way to learn is by trying something out yourself, so my goal is to try to build-your-own microservices architecture that will work both on your local environment as well as in the Cloud.

[Pivotal Cloud Foundry](http://docs.pivotal.io/pivotalcf/1-10/installing/pcf-docs.html) offers an out-of-the-box solution for multi-cloud, which seems like a good option. In this book, I'll be looking at more basic role-your-own, for the purpose of learning about the Spring Cloud building blocks.

Throughout this book we will demonstrate how to develop Spring microservices using Spring Boot and Spring Cloud.

We will write a sentence-stats service which given a sentence, will call other services to derive specialised statistics such as word-count and char-count.

We will start with a simple set of services which will be tightly bound to pre-defined ports, and then we will evolve to introduce the ability to:

Loosely couple services using Eureka Auto Discover service

Leverage the client load balancing capabilities using Ribbon

Show how you can protect your services using Circuit Breakers

Package and compose all microservices using Docker containers

Introduce an API gateway which can both do routing using Zuul

Illustrate how to  Hystrix



