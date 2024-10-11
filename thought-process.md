# Hi Welcome!
This document outlines the development process of this Java application that processes HTTP requests with a focus on handling unique requests efficiently. Below are the step by step processs and the challenges that I faced and different approaches tried.

## Approaches Tried
    1. Using ConcurrentHashMap in local(Deprecated)
    2. Using Redis as distributed in-mem cache store.(Deprecated)
    3. Integrated Spring Webflux with Redis(Currently In-Use)
    4. Using Cassandra inplace of Redis(IN progress)

### Initial setup

* Started by creating a simple Spring Boot application with a REST endpoint `api/verve/accept` that accepts an integer ID as a mandatory query parameter. The response returns "ok" or "failed" based on processing.

* To find out the unique integers processed in a minute I decided to keep the operations as minimum as possible to reduce the operational overhead. So initally started with **ConcurrentHashMap** as it's thread-safe and ideal for concurrent environments. Simpleted performed a put and using a Scheduler, fetched the number of keys in the hashmap then later clearing all the contents in it after each minute. 

* To increase the default pool size of spring boot which uses tomcat, I increasd it to 500 `server.tomcat.threads.max=500`

* It worked, with the basic set-up I tired to find the max throughput of application. So I looked into some of the load testing softwares and pickedup **Apache Jmeter** as it is simple and easy to use.

* In the initial Testing with 200k records the application can able to process almost on an avg of **8~9k requests per second** and quite more latency of **222.7ms**. But the drawback is since the HashMap is in local memory and when the application is deployed at multi-instance level it'll fail to identify duplicate id's. So moved on to next approach.

![Using ConcurrentHashMap](http://url/to/img.png)


### Using Redis Cache

* To Achieve the multi-instace deduplication, I have decided to use distributed cache or db, since redis is in-memory I choose redis. When we use traditional get/set approach it involves operational overhead, so I decided to go with `RedisSet` which will perfrom it on the server side. 

* The functionality simple includes pushing the id's into a set for the given minute as a key and for each minute getting the count of the elements in it.

* I tested using the same Jmeter with same 200k requests but the throughput decreased significatly to **8~9 requests per second** and also with very low **avg latency of 12ms**. I'm using in a very tradidtional way of programming and synchronous usage of redis and rest api's. So I decided to try something that runs non-blocking and asynchronous way. 
![Using Redis](http://url/to/img.png)


### Integrating Redis with Spring web Flux

* WebFlux is non-blocking and uses reactive programming paradigm, which allows the application to handle more concurrent requests efficiently. 

Made a little changes to the application to make it asynchronous and non-blocking. 

    1. Replaced HttpClient with WebClient which allows to send asynchronous requests to the endpoint.

    2. Changed return type to Mono<String> to enable async handling of requests.

    3. Used ReactiveRedisTemplate instead of Jedis to interact with Redis asynchronously.

* by these changes I could able to effectively reduce the latency and throughput of the applicaiton. When tested with the same volume of 200k requests from 1000 clients each sending 200 req/sec, the application was able to process **10~11k requests per second** with low latency(10ms).

![Reactive Programming with Redis](asdfa)

* I futher dig down and found that the my JVM with web Flux has reached the max concurrent thread pool limit so `reduced the client size to 50 and each of them sending 4000 requests` the latency drop was huge and came down to **3.89ms** and acheiving upto more then **12k req/sec**. So my single personal laptop with 16 gb of memory, 4 cores of processor can able to handle this so the multiple severs with load balancers should handle more I guess.

```bash
The results indicate that the application will likely scale better as you increase the request volume, especially with load balancing or additional resources and optimizations like auto-scaling and more code optimizations.
```

#### Since I'm familiar with Cassandra I had an Idea to use cassandra as unique value store insted of redis.

* In Cassandra the combination of primary keys are always unique if we try to insert a record with same primary keys it'll replace the existing record. So By setting minute value as parititon key and the actual Id as clustering key we could just perform insert and fetch the record count of each minute. Since cassandra is write-heavy database it could withstand more workload specially if done using spark or other processing engines.

It is not completed, just tried it.

----
## This is one of the most enjoyed task I have done. Thanks team!.