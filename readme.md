# Introduction

## Intent

Given a client and a supplier service, circuit breaker intent is:

- to avoid cascading failures
- to avoid waiting for supplier timeouts while supplier is down
- to provide a default response while supplier is down

## Participants

A **Client**, a **Supplier** and a mediator between them called **Circuit Breaker**.

## Collaboration

The Circuit Breaker monitors Supplier availability.

When the Client requests the Supplier, the Circuit Breaker delegates to Supplier if and only if commnunication with supplier is up (circuit closed), otherwise the circuit breaker returns a fallback response to client (circuit open).

Circuit breaker keeps returning the fallback until the communitation with supplier is re-established.


# Demo

## Services

- **Bookstore**: Supplier, provides recommended books: <http://localhost:8090/recommended>
- **To-Read**: Client, depends on Bookstore for retrieving recommended books: <http://localhost:8080/to-read>
- **Dashboard**: Circuit breaker monitor Dashboard: <http://localhost:7979>

## Flow

- Start Supplier service
- Start Client service
- Request Client service with Supplier **up**, receive response from Supplier (circuit **closed**)
- Stop Supplier service
- Request Client service with Supplier **down**, receive fallback response (circuit **open**)

## Steps

### Startup

If you use **```gnome-terminal```**, open a terminal and execute: ```./run-demo```

The script will open 3 ```gnome-terminal``` tabs and execute ```Supplier```, ```Client``` and ```Dashboard``` applications.

If you don't use ```gnome-terminal``` you will have to execute the following commands, each on a different terminal:

Tab 1: ```cd supplier && mvn clean spring-boot:run```

Tab 2: ```cd client && mvn clean spring-boot:run```

Tab 3: ```cd hystrix-dashboard && mvn clean spring-boot:run```


### Requests

Get <http://localhost:8090/recommended>

You should get: *Spring in Action (Manning), Cloud Native Java (O'Reilly), Learning Spring Boot (Packt)*

Get <http://localhost:8080/to-read>

You should get: *Spring in Action (Manning), Cloud Native Java (O'Reilly), Learning Spring Boot (Packt)*

Kill Supplier Service: hit **CTRL+C** in Supplier terminal

Get <http://localhost:8080/to-read>

You should get fallback response from circuit breaker: *Cloud Native Java (O'Reilly)*

The Circuit Breaker continues returning the fallback response while the circuit is open (pinging the Supplier service doesn't get a response).

# Hystrix

## Code

We are going to use ```spring-cloud-starter-hystrix``` as the dependency for adding Circuit Breaker support to our Client app.

We add fault tolerance to a service method. The service class must be annotated with ```@Service```:

```Java
@Service
public class BookService {
    ...
```
Then the service method is annotated with ```@HystrixCommand```.

```Java
@HystrixCommand(fallbackMethod = "reliable")
public String readingList() {
    ...
```

The ```fallbackMethod``` attribute of ```@HystrixCommand``` indicates the name of the method that provides the fallback, in this example ```reliable()```.

```Java
public String reliable() {
    return "Cloud Native Java (O'Reilly)";
}
```

**NOTE**: the fallback method ```reliable()``` must be declared in the same class than the intercepted method ```readingList()```.

For the complete example see [BookService.java](client/src/main/java/ar/com/kamikazesoftware/BookService.java).

## Hystrix Dashboard

In this demo, Hystrix Dashboards runs on <http://localhost:7979>

In the home page of the Dashboard, connect to the event stream <http://localhost:8080/hystrix.stream>

### Legends

**Short-Circuited** represents the requests that were sent to the fallback.

### Sections

#### Circuit

Shows a summary of the client-supplier collaboration.

#### Thread Pools

Shows metrics of the thread pool defined by Hystrix on the Client.


# Reference

https://spring.io/blog/2016/10/27/spring-tips-circuit-breakers

https://spring.io/guides/gs/circuit-breaker/

https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-javanica

https://github.com/spring-cloud-samples/hystrix-dashboard

https://github.com/Netflix/Hystrix

http://martinfowler.com/bliki/CircuitBreaker.html
