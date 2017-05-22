# The sentence-stats service

## Just an app

After having installed Spring Boot you can now create a template web project called sentencestats using the command line as follows.

```
spring init \
    -d=web,cloud-eureka,cloud-hystrix,cloud-feign \
    -groupId=com.apm4all \
    -artifactId=sentence-stats \
    -name=sentenceStats \
    sentence-stats
```

> We invoked spring init using `-d` option which defines comma-separated list of dependencies required. In this case since we want to create a web service, we picked the web dependency. We defined a few other dependencies which we will explore in later chapters when we explore service discovery and declarative REST clients.
>
> We also chose to define the groupId and artifactId, but we could have left it to defaults. Type `spring help init` to learn more about the defaults and available dependencies
>
> There is another way you can get a template, using Spring Initializr \([http://start.spring.io](http://start.spring.io)\), but I chose to use command line in this book as it is easier to follow step-by-step.

This application does not do much yet, but you can build it.

```
cd sentence-stats
mvn clean package
```

an execute it

```
mvn spring-boot:run
```

All going well you will see output as follows

    [INFO] Scanning for projects...
    [INFO]
    [INFO] ------------------------------------------------------------------------
    [INFO] Building sentenceStats 0.0.1-SNAPSHOT
    [INFO] ------------------------------------------------------------------------

    ...

      .   ____          _            __ _ _
     /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
    ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
     \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
      '  |____| .__|_| |_|_| |_\__, | / / / /
     =========|_|==============|___/=/_/_/_/
     :: Spring Boot ::        (v1.5.3.RELEASE)

     ...

    2017-05-22 21:33:05.369  INFO 89819 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8081 (http)
    2017-05-22 21:33:05.374  INFO 89819 --- [           main] c.a.s.SentenceStatsApplication           : Started SentenceStatsApplication in 2.44 seconds (JVM running for 4.664)Adding the REST endpoint

So the application runs but is does not do anything useful, so lets stop the app now with Ctrl-C and let's create a `SentenceStatsController`by editing `./src/main/java/com/apm4all/sentencestats/SentenceStatsController.java`

Now create the `SentenceStatsController.java` class and define the class as a `@RestController` and add the `@RequestMapping` handler method as shown below

```java
package com.apm4all.sentencestats;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SentenceStatsController {
    @RequestMapping(value="/sentence-stats/{sentence}", method=RequestMethod.GET)
    public String sentenceStats()   {
        return "No stats yet!\n";
    }
}
```

Let's also name the service `sentence-stats` and make it listen on port `8081` by editing `./src/main/resources/application.properties` as follows

```
spring.application.name = sentence-stats
server.port = 8081
```

Run the application again

```
mvn spring-boot:run
```

Now, that you have exposed the _word-count_ REST resource, you can access it using curl.

```
curl http://localhost:8080/sentence-stats
```

And get \(somewhat disapointing\) output

```
No stats yet!
```

## Handling input variable and producting output

Let's enhance `SentenceStatsController.java` a little more to take in a sentence and return the sentence back

```java
package com.apm4all.sentencestats;

import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SentenceStatsController {
    @RequestMapping(value="/sentence-stats/{sentence}", method=RequestMethod.GET)
    public String sentenceStats(@PathVariable String sentence) {
        return sentence + "\n";
    }
}
```

After re-build and re-run, when we pass in a sentence

```
curl http://localhost:8080/sentence-stats/hello%20world
```

the service returns the sentence back

```
hello world
```

## Structuring the output

So far we are only returning an unstructured string, so we can do better than that. Let's instead return some JSON containing:

* The `sentence` requested
* The `numberOfWords`

We are not going to compute the wordCount and charCount in this service as we want to illustrate `sentencestats` service interacting with `wordcount` and `charcount` microservices, so we are just going to define a Java Bean `./src/main/java/com/apm4all/sentencestats/SentenceStats.java` to encapsulate the response object.

```java
package com.apm4all.sentencestats;

public class SentenceStats {

    String sentence;
    int numberOfWords;

    public SentenceStats(String sentence, int numberOfWords) {
        this.sentence = sentence;
        this.numberOfWords = numberOfWords;
    }

    public int getNumberOfWords() {
        return numberOfWords;
    }

    public String getSentence() {
        return sentence;
    }
}
```

So we now modify `./src/main/java/com/apm4all/sentencestats/SentenceStatsController.java` slightly to constuct `SentenceStats` and return it, so that Spring MVC can convert it into JSON.

```java
package com.apm4all.sentencestats;

import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SentenceStatsController {
    @RequestMapping(value="/sentence-stats/{sentence}", method=RequestMethod.GET)
    public SentenceStats sentenceStats(@PathVariable String sentence) {
        return new SentenceStats(sentence, 0);
    }
}
```

After re-build and re-run, when we pass in a sentence

```
curl http://localhost:8080/sentence-stats/hello%20world
```

the service now returns the JSON object

```js
{"sentence":"hello world","numberOfWords":0}
```

So, we now have the requested `sentence` being returned but we will defer the calculation of the `numberOfWords` and `numberOfChars`, as we are going to delegate these calculations to other microservices, so we can illustrate microservice interoperability.

## Implementing a Client to word-count

Even though we don't yet have a word-count service, let's create a the REST client logic in sentence-stats.

For now we are going to use a pure RestTemplate hard-wired to the word-count which is going to be listening on `localhost:8082/word-count/{sentence}`

So, we'll create a new class to represent the response from word-count `./src/main/java/com/apm4all/sentencestats/WordCount.java`

```java
package com.apm4all.sentencestats;

public class WordCount {

    String sentence;
    int numberOfWords;

    public int getNumberOfWords() {
        return numberOfWords;
    }

    public String getSentence() {
        return sentence;
    }
}
```

And add the RestTemplate usage in `./src/main/java/com/apm4all/sentencestats/SentenceStatsController.java`

```java
package com.apm4all.sentencestats;

import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class SentenceStatsController {
    @RequestMapping(value = "/sentence-stats/{sentence}", method = RequestMethod.GET)
    public SentenceStats sentenceStats(@PathVariable String sentence) {

        // Make RestTemplate invoke the hard-wired URL and map response to WordCount Bean
        RestTemplate restTemplate = new RestTemplate();
        WordCount wordCount =
                restTemplate.getForObject("http://localhost:8082/word-count/"+sentence, WordCount.class);

        // Use WordCount returned from the word-count service to set sentenceStats.numberOfWords
        SentenceStats sentenceStats = new SentenceStats(sentence, wordCount.getNumberOfWords());
        return sentenceStats;
    }
}
```

Now given we have not implemented word-count, after re-build and re-run, when we pass in a sentence

```
curl http://localhost:8080/sentence-stats/hello%20world
```

the service now tries to call `http://localhost:8082/word-count/hello%20world`  but it fails because the word-count does not exist yet.

```js
{
  "timestamp": 1495491436318,
  "status": 500,
  "error": "Internal Server Error",
  "exception": "org.springframework.web.client.ResourceAccessException",
  "message": "I/O error on GET request for \"http://localhost:8082/word-count/hello%20world\": Connection refused; nested exception is java.net.ConnectException: Connection refused",
  "path": "/sentence-stats/hello%20world"
}
```

In the next chapter we'll create the `word-count`service.

