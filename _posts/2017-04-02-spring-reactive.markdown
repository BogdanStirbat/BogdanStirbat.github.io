---
layout: post
comments: true
title:  "Reactive Programming in Spring Framework"
date:   2017-04-02 12:25:18 +0300
categories: jekyll update
---
Reactive programming is, according to [Wikipedia](https://en.wikipedia.org/wiki/Reactive_programming), an asynchronous programming paradigm oriented around data streams and the propagation of change. In recent years, multi core CPU systems become the norm, and microservice architectures becomes more and more common. In this context, programmers have to deal with multithreading programming challenges more often. Multithreading programming is also notorious hard, and Reactive Programming makes it simpler.

In this context, Reactive Programming, with is not something new, caught attention. Many libraries, frameworks, were developed to support this paradigm. A notable initiative is [Reactive Streams](http://www.reactive-streams.org/). Quoting the project's authors,
> The scope of Reactive Streams is to find a minimal set of interfaces, methods and protocols that will describe the necessary operations and entities to achieve the goal—asynchronous streams of data with non-blocking back pressure.

We see a new term, back pressure. When a server sends data to a client, with a rare high enough that the client is overwhelmed and can't handle this load (this pressure), we want the server to slow or even stop emitting new events. This is what back pressure is.
So, Reactive Streams is a specification (a [interface](https://github.com/reactive-streams/reactive-streams-jvm)). Thus, if a client and server communicate in a Reactive way, and we want to have back pressure, the minimum requirement is that both client and server use a reactive library compatible with Reactive Streams specification - but not the same!

In Java world, and in many other JVM and not-JVM specific language, a lot of Reactive libraries were developed. Reactive Streams specification made such a great impact, that libraries implementing and not implementing this specification were in a different classes. David Karnok, a contributor of Reactive Streams and other project, made a classification for these libraries, a classification that becomed an de-facto standard. The classification can be found in this [blog post](https://akarnokd.blogspot.ro/2016/03/operator-fusion-part-1.html), referred in many places across the internet. For understanding the current Reactive landscape, I strongly recommend reading this [blog post](https://akarnokd.blogspot.ro/2016/03/operator-fusion-part-1.html).

[Reactive Streams](http://www.reactive-streams.org/) made such a big impact, that Reactive Streams interfaces were incorporated even in [Java 9](http://download.java.net/java/jdk9/docs/api/java/util/concurrent/Flow.html)! This is a strong sign, that Reactive is here to stay. Spring authors also reacted to this change; from Spring 5, [Project Reactor](https://projectreactor.io/), a library compatible with Reactive Streams specification, was incorporated into [Spring core framework](https://spring.io/blog/2016/07/28/reactive-programming-with-spring-5-0-m1). In an [DEVOXX presentation](https://www.youtube.com/watch?v=Cj4foJzPF80), Brian Clozel states that adding a dependency to Spring Framework happens not very often.

Spring team made an excellent job in documenting the new Reactive features. Spring's new WebFlux framework framework is documented not only in [official documentation](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#web-reactive). Also, on Spring's blog, [here (Part 1)](https://spring.io/blog/2016/06/07/notes-on-reactive-programming-part-i-the-reactive-landscape), [here (Part 2)](https://spring.io/blog/2016/06/13/notes-on-reactive-programming-part-ii-writing-some-code) and [here (Part 3)](https://spring.io/blog/2016/07/20/notes-on-reactive-programming-part-iii-a-simple-http-server-application) Reactive Programming, and Spring adaption of Reactive Programming, are clear, buzzword-free, with advantages, disadvantages, and use cases, explained. I strongly recommend reading this series. Apart from crystal-clear explanations, many general misconceptions, confusions, regarding Reactive Programming are clarified. They even made an [API Hands On workshop](https://github.com/reactor/lite-rx-api-hands-on).

I expect the visitors to read the specified articles. It would be impractical to repeat the same information here.

In the next [blog post]({% link _posts/2017-04-04-spring-reactive_part_II.markdown %}), I will describe some experiments with Spring WebFlux framework.