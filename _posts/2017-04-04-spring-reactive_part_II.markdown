---
layout: post
comments: true
title:  "Reactive Programming in Spring Framework (part II)"
date:   2017-04-04 18:25:18 +0300
categories: jekyll update
---
In the previous [blog post]({% link _posts/2017-04-02-spring-reactive.markdown %}), some general facts about Reactive Programming and Spring support for this paradigm.

Here, I will present some experiments with this framework. In this [GitHub repository](https://github.com/BogdanStirbat/reactive-demo) there are 4 projects:
* demo-server
* reactiveweb-app
* servlet-app
* benchmark-runner

In this setup, we have a database, consisting of 100 messages (each with it's own id, generated from 1 to 100). demo-server exposes an API that accepts an message id and returns the associated message.
For this server, there are 2 other servers, each consuming the demo-server's API. One server is running in an "Reactive container" (reactiveweb-app), the other one is running in an traditional servlet container (servlet-app). Each of these 2 servers accept an message id, call demo-server to retrieve the corresponding message, and returns the message back to client.

The purpose of this experiment was to test 2 different web applications, one Reactive and one non-Reactive. So, an client is also included - benchmark-runner. This client runs in the command line. Opens a number of requests, to one of the 2 servers. Both parameters - number of requests, with server - are given as command line arguments.

In the main thread, benchmark-runner creates an ExecutorService, and for each request, supplies an async task using Java 8's CompletableFuture.
Code:

{% highlight java linenos %}
    ExecutorService es = Executors.newFixedThreadPool(numberOfRequests);
    List<CompletableFuture> completableFutures = new ArrayList<>();

    for(int i = 1; i <= numberOfRequests; i++) {
        final int messageId = i;
        CompletableFuture<Message> completableFuture = CompletableFuture.supplyAsync(() -> loadMessage(strings[0], selectedUri, messageId));
        completableFutures.add(completableFuture);
    }

    CompletableFuture<Void> combined = CompletableFuture.allOf(completableFutures.toArray(new CompletableFuture[completableFutures.size()]));
    combined.get();
{% endhighlight %}

I wanted to observe how each of the 4 components behave under test. For this, logs were added, in all 4 projects. Each controller's method execution (demo-server, reactiveweb-app, servlet-app) is surrounded in following code block:

{% highlight java linenos %}
LOGGER.info("Received request for message id: {}", id);
...
LOGGER.info("Finished request for message id: {}", id);
{% endhighlight %}

To simulate a long processing by demo-server, 15 seconds are waited before returning the result from DB.

First, I have tested servlet-app component.
{% highlight bash %}
spring-reactive/benchmark-runner$ java -jar target/benchmark-runner-0.0.1-SNAPSHOT.jar servlet 10
{% endhighlight %}

Console outputs:
benchmark-runner
{% highlight bash %}
Prepare to send 10 requests to servlet
Supply async: 1
Supply async: 2
Supply async: 3
Supply async: 4
Supply async: 5
Supply async: 6
Supply async: 7
Supply async: 8
Supply async: 9
Supply async: 10
2017-04-03 20:53:56.135  INFO 16442 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 3
2017-04-03 20:53:56.134  INFO 16442 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 1
2017-04-03 20:53:56.135  INFO 16442 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 2
2017-04-03 20:54:12.736  INFO 16442 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 3
2017-04-03 20:54:12.736  INFO 16442 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 4
2017-04-03 20:54:12.737  INFO 16442 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 1
2017-04-03 20:54:12.737  INFO 16442 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 5
2017-04-03 20:54:12.736  INFO 16442 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 2
2017-04-03 20:54:12.743  INFO 16442 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 6
2017-04-03 20:54:27.780  INFO 16442 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 4
2017-04-03 20:54:27.780  INFO 16442 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 7
2017-04-03 20:54:27.797  INFO 16442 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 6
2017-04-03 20:54:27.797  INFO 16442 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 8
2017-04-03 20:54:27.798  INFO 16442 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 5
2017-04-03 20:54:27.798  INFO 16442 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 9
2017-04-03 20:54:42.824  INFO 16442 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 7
2017-04-03 20:54:42.824  INFO 16442 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 10
2017-04-03 20:54:42.840  INFO 16442 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 8
2017-04-03 20:54:42.848  INFO 16442 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 9
2017-04-03 20:54:57.852  INFO 16442 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 10
{% endhighlight %}

servlet-app
{% highlight bash %}
2017-04-03 20:53:56.757  INFO 15669 --- [nio-8082-exec-1] c.b.r.d.s.controller.MessageController   : Received request for message id: 2
2017-04-03 20:53:56.760  INFO 15669 --- [nio-8082-exec-2] c.b.r.d.s.controller.MessageController   : Received request for message id: 1
2017-04-03 20:53:56.761  INFO 15669 --- [nio-8082-exec-3] c.b.r.d.s.controller.MessageController   : Received request for message id: 3
2017-04-03 20:54:12.653  INFO 15669 --- [nio-8082-exec-1] c.b.r.d.s.controller.MessageController   : Finished request for message id: 2
2017-04-03 20:54:12.653  INFO 15669 --- [nio-8082-exec-2] c.b.r.d.s.controller.MessageController   : Finished request for message id: 1
2017-04-03 20:54:12.658  INFO 15669 --- [nio-8082-exec-3] c.b.r.d.s.controller.MessageController   : Finished request for message id: 3
2017-04-03 20:54:12.739  INFO 15669 --- [nio-8082-exec-4] c.b.r.d.s.controller.MessageController   : Received request for message id: 4
2017-04-03 20:54:12.743  INFO 15669 --- [nio-8082-exec-5] c.b.r.d.s.controller.MessageController   : Received request for message id: 5
2017-04-03 20:54:12.751  INFO 15669 --- [nio-8082-exec-6] c.b.r.d.s.controller.MessageController   : Received request for message id: 6
2017-04-03 20:54:27.778  INFO 15669 --- [nio-8082-exec-4] c.b.r.d.s.controller.MessageController   : Finished request for message id: 4
2017-04-03 20:54:27.784  INFO 15669 --- [nio-8082-exec-5] c.b.r.d.s.controller.MessageController   : Finished request for message id: 5
2017-04-03 20:54:27.790  INFO 15669 --- [nio-8082-exec-6] c.b.r.d.s.controller.MessageController   : Finished request for message id: 6
2017-04-03 20:54:27.795  INFO 15669 --- [nio-8082-exec-7] c.b.r.d.s.controller.MessageController   : Received request for message id: 7
2017-04-03 20:54:27.802  INFO 15669 --- [nio-8082-exec-8] c.b.r.d.s.controller.MessageController   : Received request for message id: 8
2017-04-03 20:54:27.803  INFO 15669 --- [nio-8082-exec-9] c.b.r.d.s.controller.MessageController   : Received request for message id: 9
2017-04-03 20:54:42.819  INFO 15669 --- [nio-8082-exec-7] c.b.r.d.s.controller.MessageController   : Finished request for message id: 7
2017-04-03 20:54:42.831  INFO 15669 --- [nio-8082-exec-8] c.b.r.d.s.controller.MessageController   : Finished request for message id: 8
2017-04-03 20:54:42.835  INFO 15669 --- [io-8082-exec-10] c.b.r.d.s.controller.MessageController   : Received request for message id: 10
2017-04-03 20:54:42.846  INFO 15669 --- [nio-8082-exec-9] c.b.r.d.s.controller.MessageController   : Finished request for message id: 9
2017-04-03 20:54:57.850  INFO 15669 --- [io-8082-exec-10] c.b.r.d.s.controller.MessageController   : Finished request for message id: 10
{% endhighlight %}

demo-server
{% highlight bash %}
2017-04-03 20:53:57.128  INFO 15801 --- [nio-8080-exec-3] c.b.r.d.s.controller.MessageController   : Received request for message id: 3
2017-04-03 20:53:57.128  INFO 15801 --- [nio-8080-exec-1] c.b.r.d.s.controller.MessageController   : Received request for message id: 1
2017-04-03 20:53:57.128  INFO 15801 --- [nio-8080-exec-2] c.b.r.d.s.controller.MessageController   : Received request for message id: 2
2017-04-03 20:54:12.128  INFO 15801 --- [nio-8080-exec-3] c.b.r.d.s.controller.MessageController   : Finished request for message id: 3
2017-04-03 20:54:12.129  INFO 15801 --- [nio-8080-exec-1] c.b.r.d.s.controller.MessageController   : Finished request for message id: 1
2017-04-03 20:54:12.131  INFO 15801 --- [nio-8080-exec-2] c.b.r.d.s.controller.MessageController   : Finished request for message id: 2
2017-04-03 20:54:12.750  INFO 15801 --- [nio-8080-exec-4] c.b.r.d.s.controller.MessageController   : Received request for message id: 5
2017-04-03 20:54:12.750  INFO 15801 --- [nio-8080-exec-5] c.b.r.d.s.controller.MessageController   : Received request for message id: 4
2017-04-03 20:54:12.757  INFO 15801 --- [nio-8080-exec-6] c.b.r.d.s.controller.MessageController   : Received request for message id: 6
2017-04-03 20:54:27.751  INFO 15801 --- [nio-8080-exec-4] c.b.r.d.s.controller.MessageController   : Finished request for message id: 5
2017-04-03 20:54:27.755  INFO 15801 --- [nio-8080-exec-5] c.b.r.d.s.controller.MessageController   : Finished request for message id: 4
2017-04-03 20:54:27.759  INFO 15801 --- [nio-8080-exec-6] c.b.r.d.s.controller.MessageController   : Finished request for message id: 6
2017-04-03 20:54:27.799  INFO 15801 --- [nio-8080-exec-7] c.b.r.d.s.controller.MessageController   : Received request for message id: 7
2017-04-03 20:54:27.806  INFO 15801 --- [nio-8080-exec-8] c.b.r.d.s.controller.MessageController   : Received request for message id: 8
2017-04-03 20:54:27.809  INFO 15801 --- [nio-8080-exec-9] c.b.r.d.s.controller.MessageController   : Received request for message id: 9
2017-04-03 20:54:42.799  INFO 15801 --- [nio-8080-exec-7] c.b.r.d.s.controller.MessageController   : Finished request for message id: 7
2017-04-03 20:54:42.806  INFO 15801 --- [nio-8080-exec-8] c.b.r.d.s.controller.MessageController   : Finished request for message id: 8
2017-04-03 20:54:42.810  INFO 15801 --- [nio-8080-exec-9] c.b.r.d.s.controller.MessageController   : Finished request for message id: 9
2017-04-03 20:54:42.841  INFO 15801 --- [io-8080-exec-10] c.b.r.d.s.controller.MessageController   : Received request for message id: 10
2017-04-03 20:54:57.842  INFO 15801 --- [io-8080-exec-10] c.b.r.d.s.controller.MessageController   : Finished request for message id: 10
{% endhighlight %}

As we can see, when we run our benchmark, the main thread supplies 10 async tasks, the collects them. The benchmark application, even if creates 10 application, seems to have only 3 running at any time. First 3 threads are scheduled, are blocked, and when they receive answers the other 3 are run, and so on.

The servlet-app seems to be doing a similar thing - for each concurrent request, allocates a thread. The thread (and client) is blocked until the external long-running service (demo-server, in our case) returns.

Unsurprisingly, the demo-server (running in an traditional Servlet context) is doing a similar thing.


Interesting things are observed when testing the reactiveweb-app.

Code:
{% highlight bash %}
spring-reactive/benchmark-runner$ java -jar target/benchmark-runner-0.0.1-SNAPSHOT.jar reactive 10
{% endhighlight %}

Console outputs:
benchmark-runner
{% highlight bash %}
Supply async: 1
Supply async: 2
Supply async: 3
Supply async: 4
Supply async: 5
Supply async: 6
Supply async: 7
Supply async: 8
Supply async: 9
Supply async: 10
2017-04-03 20:58:22.927  INFO 16554 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 1
2017-04-03 20:58:22.930  INFO 16554 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 3
2017-04-03 20:58:22.928  INFO 16554 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 2
2017-04-03 20:58:39.049  INFO 16554 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 1
2017-04-03 20:58:39.049  INFO 16554 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 2
2017-04-03 20:58:39.049  INFO 16554 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 4
2017-04-03 20:58:39.049  INFO 16554 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 3
2017-04-03 20:58:39.049  INFO 16554 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 6
2017-04-03 20:58:39.049  INFO 16554 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 5
2017-04-03 20:58:54.095  INFO 16554 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 6
2017-04-03 20:58:54.095  INFO 16554 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 7
2017-04-03 20:58:54.100  INFO 16554 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 4
2017-04-03 20:58:54.100  INFO 16554 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 5
2017-04-03 20:58:54.100  INFO 16554 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 8
2017-04-03 20:58:54.100  INFO 16554 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 9
2017-04-03 20:59:09.147  INFO 16554 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 9
2017-04-03 20:59:09.148  INFO 16554 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 10
2017-04-03 20:59:09.156  INFO 16554 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 8
2017-04-03 20:59:09.160  INFO 16554 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 7
2017-04-03 20:59:24.187  INFO 16554 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 10
{% endhighlight %}

We can see, the client behaved in a similar way as in our previous experiment: main thread allocates a new thread for each request, runs them in an ExecutorService. The ExecutorService has 10 threads, but only 3 are executing actual work (calling the server, waiting for result) at any given time.

An interesting thing can be observed from reactiveweb-app console:
{% highlight bash %}
2017-04-03 20:58:23.439  INFO 15631 --- [-server-epoll-6] c.b.r.d.r.controller.MessageController   : Received request for message id: 1
2017-04-03 20:58:23.439  INFO 15631 --- [-server-epoll-7] c.b.r.d.r.controller.MessageController   : Received request for message id: 3
2017-04-03 20:58:23.439  INFO 15631 --- [-server-epoll-8] c.b.r.d.r.controller.MessageController   : Received request for message id: 2
2017-04-03 20:58:23.482  INFO 15631 --- [-server-epoll-6] c.b.r.d.r.controller.MessageController   : Finished request for message id: 1
2017-04-03 20:58:23.484  INFO 15631 --- [-server-epoll-8] c.b.r.d.r.controller.MessageController   : Finished request for message id: 2
2017-04-03 20:58:23.484  INFO 15631 --- [-server-epoll-7] c.b.r.d.r.controller.MessageController   : Finished request for message id: 3
2017-04-03 20:58:39.051  INFO 15631 --- [-server-epoll-7] c.b.r.d.r.controller.MessageController   : Received request for message id: 6
2017-04-03 20:58:39.051  INFO 15631 --- [-server-epoll-8] c.b.r.d.r.controller.MessageController   : Received request for message id: 4
2017-04-03 20:58:39.052  INFO 15631 --- [-server-epoll-6] c.b.r.d.r.controller.MessageController   : Received request for message id: 5
2017-04-03 20:58:39.052  INFO 15631 --- [-server-epoll-6] c.b.r.d.r.controller.MessageController   : Finished request for message id: 5
2017-04-03 20:58:39.052  INFO 15631 --- [-server-epoll-7] c.b.r.d.r.controller.MessageController   : Finished request for message id: 6
2017-04-03 20:58:39.052  INFO 15631 --- [-server-epoll-8] c.b.r.d.r.controller.MessageController   : Finished request for message id: 4
2017-04-03 20:58:54.101  INFO 15631 --- [-server-epoll-7] c.b.r.d.r.controller.MessageController   : Received request for message id: 7
2017-04-03 20:58:54.101  INFO 15631 --- [-server-epoll-7] c.b.r.d.r.controller.MessageController   : Finished request for message id: 7
2017-04-03 20:58:54.102  INFO 15631 --- [-server-epoll-8] c.b.r.d.r.controller.MessageController   : Received request for message id: 8
2017-04-03 20:58:54.106  INFO 15631 --- [-server-epoll-5] c.b.r.d.r.controller.MessageController   : Received request for message id: 9
2017-04-03 20:58:54.107  INFO 15631 --- [-server-epoll-5] c.b.r.d.r.controller.MessageController   : Finished request for message id: 9
2017-04-03 20:58:54.108  INFO 15631 --- [-server-epoll-8] c.b.r.d.r.controller.MessageController   : Finished request for message id: 8
2017-04-03 20:59:09.164  INFO 15631 --- [-server-epoll-6] c.b.r.d.r.controller.MessageController   : Received request for message id: 10
2017-04-03 20:59:09.165  INFO 15631 --- [-server-epoll-6] c.b.r.d.r.controller.MessageController   : Finished request for message id: 10
{% endhighlight %}

We can see, the request arrives, and immediately returns (observe times for 'Received request' and 'Finished request'; compare the times with previous example). The message is retrieved via a Mono, using a WebClient. Thus, the server does not has to wait. Behind the scenes, framework will subscribe to this Mono. So, the server handles requests fast, is always prepared to handle more requests. But the client is blocked. Logs for the demo-server are similar as before, so adding them here would be pointless.
This leads us straight to conclusions.


### Conclusions
1. The server no longer handles each client request in a new thread. Client requests are immediately processed, and server is always ready to handle more requests.
2. Behind the scenes, the framework subscribes to the Mono encapsulating the response. Thus, the client is still blocked.