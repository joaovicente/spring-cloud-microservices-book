# The word-count service

We are going to follow very similar steps as in the previous chapter. First go up one folder `cd ..` and ensure you are in the `spring-cloud-microservices`  directory.

When you `ls -l` you should see

```
LICENSE        README.md    sentence-stats
```

Now let's create the word-count

```bash
spring init \
    -d=web,cloud-eureka,cloud-hystrix,cloud-feign \
    -groupId=com.apm4all \
    -artifactId=word-count \
    -name=wordCount \
    word-count
```

now `cd word-count`

edit`./src/main/resources/application.properties`as follows:

```
spring.application.name = word-count
server.port = 8082
```

edit`./src/main/java/com/apm4all/wordcount/WordCount.java`as follows:

```java
package com.apm4all.wordcount;

public class WordCount {
    String sentence;
    int numberOfWords;

    public WordCount(String sentence, int numberOfWords) {
        this.sentence = sentence;
        this.numberOfWords = numberOfWords;
    }

    public String getSentence() {
        return sentence;
    }

    public int getNumberOfWords() {
        return numberOfWords;
    }
}
```

edit`./src/main/java/com/apm4all/wordcount/WordCountController.java`as follows:

```java
package com.apm4all.wordcount;

import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class WordCountController {
    @RequestMapping(value="/word-count/{sentence}", method=RequestMethod.GET)
    public WordCount wordCount(@PathVariable String sentence) {
        return new WordCount(sentence , sentence.split(" ").length);
    }
}
```

At this point \(assuming you still have the sentence-stats service running\) run word-count

```
mvn spring-boot:run
```

And execute

```
curl http://localhost:8081/sentence-stats/hello%20world
```

You should now see the actual word count set to 2

```js
{"sentence":"hello world","numberOfWords":2}
```



