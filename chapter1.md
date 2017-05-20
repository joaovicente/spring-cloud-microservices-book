# A simple web app

After having installed Spring Boot you can now create a template web project called wordcount using the command line as follows.

```
spring init -d=web -groupId=com.apm4all -artifactId=wordcount wordcount
```

This application does not do much yet, but you can build it

```
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

So the application runs but is does not do anything useful, so lets stop the app now with Ctrl-C and let's create a WordCountController

```
touch ./src/main/java/com/apm4all/wordcount/WordCountController.java
```

Not create the class and define the class as a @RestController and add the @RequestMapping handler method as shown below

```
package com.apm4all.wordcount;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class WordCountController {
    @RequestMapping("/word-count")
    public String wordcount()   {
        return "Counting nothing yet!";
    }
}
```

When you run 

