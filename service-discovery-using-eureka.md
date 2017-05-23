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

