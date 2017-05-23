# Service Discovery using Eureka

Service discovery allows us to register each service by name, and allow other services to address them, including launching more than one service instance and load balance it.

Past the Edge tier where geo-based routing is used \(AWS Route S3\), the Netflix architecture favours client load balancing where the client keeps track of all available instances of a given service, being able to make inteligent routing decisions based on what it is learning from the Service Discovery service \(Eureka\) and the instance/service health monitoring \(Hystrix\).

One thing at a time though, so let's start with Eureka first

### Creating an Eureka Service

In a cloud production environment you will want to [deploy Eureka as a cluster](https://github.com/Netflix/eureka/wiki/Deploying-Eureka-Servers-in-EC2), but for the purpose of getting up-and-running in your local environment lets use the pre-backed Spring recipe.



