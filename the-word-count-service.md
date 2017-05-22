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
    int wordCount;

    public WordCount(String sentence, int wordCount) {
        this.sentence = sentence;
        this.wordCount = wordCount;
    }

    public String getSentence() {
        return sentence;
    }

    public int getWordCount() {
        return wordCount;
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
        return new WordCount(sentence, 0);
    }
}
```

Now let's run it

