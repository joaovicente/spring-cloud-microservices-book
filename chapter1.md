# The first web app

## The bare bones

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

So the application runs but is does not do anything useful, so lets stop the app now with Ctrl-C and let's create a `SentenceStatsController`

```
touch ./src/main/java/com/apm4all/sentencestats/SentenceStatsController.java
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

Now, that you have exposed the _word-count_ REST resource, you can access it using curl, as follows

```
curl http://localhost:8080/sentence-stats
No stats yet!
```

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

After re-build and re-run, when we pass in a sentence, the service returns the sentence back

```
curl "http://localhost:8080/sentence-stats/hello%20world"
hello world
```



