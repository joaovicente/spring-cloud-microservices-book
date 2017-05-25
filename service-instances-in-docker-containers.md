# Service instances in Docker containers

We have seen the power of service discovery in the previous chapter, and the flexibility to just launch service instances and have them easily discoverable via Eureka clients.

However, when you have a larger amount of services and instances, and we have to find a new port for each one of them, things become a bit messy.

So in this chapter we introduce Docker, with which we should be able compose any number of service instances more elegantly. Each Docker container will be a lighweight Linux VM, and each Docker containers will hold a Spring Boot application. The beauty of Docker is that each container can listen on the same port internally \(e.g. 8080 at the guest\) but that port will automatically be assigned to a random port on the Host OS.

So, if we can get Eureka client to register the Host port instead of the guest port, we should be able to automagically address each and every service, without having to worry about port collisions ... fingers crossed!...

## Running the first container



