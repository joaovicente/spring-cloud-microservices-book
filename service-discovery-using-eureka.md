# Service Discovery using Eureka

Service discovery allows us to register each service by name, and allow other services to address them, including launching more than one service instance and load balance it.

Past the Edge tier where geo-based routing is used \(AWS Route S3\), the Netflix architecture favours client load balancing where the client keeps track of all available instances of a given service, being able to make inteligent routing decisions based on what it is learning from the Service Discovery service \(Eureka\) and the instance/service health monitoring \(Hystrix\).

One thing at a time though, so let's start with Eureka first

### Creating an Eureka Service

In a cloud production environment you will want to [deploy Eureka as a cluster](https://github.com/Netflix/eureka/wiki/Deploying-Eureka-Servers-in-EC2), but for the purpose of getting up-and-running in your local environment lets use the pre-backed Spring recipe.

To create a Spring Boot packaged Eureka server all we need is the `cloud-eureka-server` dependency as shown below

```
spring init \
    -d=cloud-eureka-server \
    -groupId=com.apm4all \
    -artifactId=eureka-service \
    -name=eureka-service \
    eureka-service
```

Go into eureka-service folder

```
cd eureka-service
```

Now add `@EnableEurekaServer` to `./src/main/java/com/apm4all/eurekaservice/EurekaServiceApplication.java` as shown below

```java
package com.apm4all.eurekaservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableEurekaServer
@SpringBootApplication
public class EurekaServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServiceApplication.class, args);
    }
}
```

And add the following to ./src/main/resources/application.properties

```
server.port = 8761
eureka.client.register-with-eureka = false
eureka.client.fetch-registry = false
```

Now, let's start the Eureka server

```
mvn spring-boot:run
```

When you go to [http://localhost:8761](http://localhost:8761) in your browser you will should see the Eureka UI

![](/assets/eureka-server-idle.png)

Notice however that there are no instances registered with Eureka server yet, but not for long...

## Register word-count with Eureka

If you remember we created word-count we added `cloud-eureka` as a dependency. This dependency is required to use an Eureka client.

Let's go back to word-count

`cd ../word-count`

And add `import org.springframework.cloud.client.discovery.EnableDiscoveryClient;` and `@EnableDiscoveryClient`to `./src/main/java/com/apm4all/wordcount/WordCountApplication.java`

```java
package com.apm4all.wordcount;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@EnableDiscoveryClient
@SpringBootApplication
public class WordCountApplication {
    public static void main(String[] args) {
        SpringApplication.run(WordCountApplication.class, args);
    }
}
```

Now, after you re-run word-count

```
mvn spring-boot:run
```

And you go back to [http://localhost:8761](http://localhost:8761) in your browser you will see that word-count service instance has registered with Eureka.![](/assets/eureka-server-word-count-registered.png)

Notice that we had previously defined `spring.application.name=word-count` in `./src/main/resources/application.properties` which  is what defines the name of the service we see in Eureka server.

Also notice that we did not have to tell word-count service where to find Eureka server, simply because the default is [http://localhost:8761](http://localhost:8761) . To explicitly configure eureka server location add  `eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka` to the `application.properties`

## Get sentence-stats client to discover word-count service

To do service discovery we are going to use [Eureka client](http://cloud.spring.io/spring-cloud-static/Dalston.SR1/#_service_discovery_eureka_clients) and [Feign](http://cloud.spring.io/spring-cloud-static/Dalston.SR1/#spring-cloud-feign) \(declarative REST client\). For a deeper dive read spring-cloud-netflix documentation.

So add `@EnableFeignClients` and `@EnableEurekaClient`

to `./src/main/java/com/apm4all/sentencestats/SentenceStatsApplication.java` as follows

```java
package com.apm4all.sentencestats;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.feign.EnableFeignClients;

@EnableFeignClients
@EnableEurekaClient
@SpringBootApplication
public class SentenceStatsApplication {
    public static void main(String[] args) {
        SpringApplication.run(SentenceStatsApplication.class, args);
    }
}
```

Create a word-clount Feign client `./src/main/java/com/apm4all/sentencestats/WordCountClient.java`

```java
package com.apm4all.sentencestats;
import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@FeignClient("word-count")
interface WordCountClient{
    @RequestMapping(method = RequestMethod.GET, value = "/word-count/{sentence}")
    WordCount call(@PathVariable("sentence") String sentence);
}
```

On the `SentenceStatsController.java` we drop the old RestTemplate implementation

```java
    @RequestMapping(value = "/sentence-stats/{sentence}", method = RequestMethod.GET)
    public SentenceStats sentenceStats(@PathVariable String sentence) {
        RestTemplate restTemplate = new RestTemplate();
        WordCount wordCount =
                restTemplate.getForObject("http://localhost:8082/word-count/"+ sentence, WordCount.class);

        SentenceStats sentenceStats = new SentenceStats(wordCount.getSentence(), wordCount.getNumberOfWords());
        return sentenceStats;
    }
```

Autowire the Feign `WordCloudClient`

```java
@RestController
public class SentenceStatsController {
    private final WordCountClient wordCountClient;

    @Autowired
    public SentenceStatsController(WordCountClient client) {
        this.wordCountClient = client;
    }
...
```

And update the `sentence-stats` handler to use the Feign client

```java
    @RequestMapping(value = "/sentence-stats-feign/{sentence}", method = RequestMethod.GET)
    public SentenceStats sentenceStatsFeign(@PathVariable String sentence) {
        WordCount wordCount = wordCountClient.call(sentence);
        SentenceStats sentenceStats = new SentenceStats(wordCount.getSentence(), wordCount.getNumberOfWords());
        return sentenceStats;
    }
```

The full listing of `./src/main/java/com/apm4all/sentencestats/SentenceStatsController.java`becomes

```java
package com.apm4all.sentencestats;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class SentenceStatsController {
    private final WordCountClient wordCountClient;

    @Autowired
    public SentenceStatsController(WordCountClient client) {
        this.wordCountClient = client;
    }

    @RequestMapping(value = "/sentence-stats/{sentence}", method = RequestMethod.GET)
    public SentenceStats sentenceStatsFeign(@PathVariable String sentence) {
        WordCount wordCount = wordCountClient.call(sentence);
        SentenceStats sentenceStats = new SentenceStats(wordCount.getSentence(), wordCount.getNumberOfWords());
        return sentenceStats;
    }
}
```



After running the sentence-stats

```
mvn spring-boot:run
```

When we execute

```
curl http://localhost:8081/sentence-stats/hello%20world
```

You should now see the actual word count set to 2

```js
{"sentence":"hello world","numberOfWords":2}
```

So, what happened was that when we made the curl request sentence-stats Eureka client requested the list of Servers \(host:port\)  providing the word-count service from Eureka server, as shown in the console log below

```
2017-05-25 00:23:21.876  INFO 98095 --- [nio-8081-exec-1] c.n.l.DynamicServerListLoadBalancer      : DynamicServerListLoadBalancer for client word-count initialized: DynamicServerListLoadBalancer:{NFLoadBalancer:name=word-count,current list of Servers=[192.168.1.12:8082],Load balancer stats=Zone stats: {defaultzone=[Zone:defaultzone;    Instance count:1;    Active connections count: 0;    Circuit breaker tripped count: 0;    Active connections per server: 0.0;]
},Server stats: [[Server:192.168.1.12:8082;    Zone:defaultZone;    Total Requests:0;    Successive connection failure:0;    Total blackout seconds:0;    Last connection made:Thu Jan 01 01:00:00 GMT 1970;    First connection made: Thu Jan 01 01:00:00 GMT 1970;    Active Connections:0;    total failure count in last (1000) msecs:0;    average resp time:0.0;    90 percentile resp time:0.0;    95 percentile resp time:0.0;    min resp time:0.0;    max resp time:0.0;    stddev resp time:0.0]
```

Once Eureka client gets the list of servers, the Feign/Eureka client will distribute the calls across the word-count registered servers in a load balanced fashion.

In the example above we only have one server, but you get the idea that if we were to launch another word-count service \(on a different port\), once the new word-count service instance registers with Eureka server, then the load is automagically distributed across the two service instances.

There you, service discovery delivered to your doorstep!

