# The first web service

## Just an app

After having installed Spring Boot you can now create a template web project called sentencestats using the command line as follows.

```
spring init -d=web -groupId=com.apm4all -artifactId=sentencestats sentencestats
```

> We invoked spring init using `-d` option which defines comma-separated list of dependencies required. In this case since we want to create a web service, we picked the web dependency.
>
> We also chose to define the groupId and artifactId, but we could have left it to defaults. Type `spring help init` to learn more about the defaults and available dependencies
>
> There is another way you can get a template, using Spring Initializr \([http://start.spring.io](http://start.spring.io)\), but I chose to use command line in this book as it is easier to follow step-by-step.

This application does not do much yet, but you can build it.

```
cd sentencestats
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
    [INFO] Building demo 0.0.1-SNAPSHOT
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

    2017-05-20 13:35:05.908  INFO 84795 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
    2017-05-20 13:35:05.912  INFO 84795 --- [           main] com.apm4all.wordcount.DemoApplication    : Started DemoApplication in 2.014 seconds (JVM running for 4.3)

## Adding the REST endpoint

So the application runs but is does not do anything useful, so lets stop the app now with Ctrl-C and let's create a `SentenceStatsController`

```
./src/main/java/com/apm4all/sentencestats/SentenceStatsController.java
```

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
* The `numberOfChars`

We are not going to compute the wordCount and charCount in this service as we want to illustrate `sentencestats` service interacting with `wordcount` and `charcount` microservices, so we are just going to define a Java Bean `./src/main/java/com/apm4all/sentencestats/SentenceStats.java` to encapsulate the response object.

```java
package com.apm4all.sentencestats;

public class SentenceStats {

    String sentence;
    int numberOfWords;
    int numberOfChars;

    public SentenceStats(String sentence) {
        this.sentence = sentence;
    }

    public int getNumberOfWords() {
        return numberOfWords;
    }

    public int getNumberOfChars() {
        return numberOfChars;
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
        return new SentenceStats(sentence);
    }
}
```

After re-build and re-run, when we pass in a sentence

```
curl http://localhost:8080/sentence-stats/hello%20world
```

the service now returns the JSON object

```js
{"sentence":"hello world","numberOfWords":0,"numberOfChars":0}
```

So, we now have the requested sentence being returned but we will defer the calculation of the numberOfWords and numberOfChars, as we are going to delegate these calculations to other microservices, so we can illustrate microservice orchestration.

