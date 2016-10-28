# Introduction

## Intend

Given a client and a supplier service, circuit breaker intend is:

- to avoid cascading failures
- to avoid waiting for supplier timeouts while supplier is down
- to provide a default response while supplier is down

## Implementation

Add a mediator, called circuit breaker, between client and supplier to monitor supplier availability.

When the client requests the supplier, the circuit breaker delegates to supplier if and only if commnunication with supplier is up (circuit closed), otherwise the circuit breaker returns a fallback response to client (circuit open).

Circuit breaker keeps returning the fallback until the communitation with supplier is re-established.

# Demo

## Services

- **Bookstore**: Supplier, provides recommended books: http://localhost/8090/recommended
- **To-Read**: Client, depends on Bookstore for retrieving recommended books: http://localhost/8080/to-read
- **Dashboard**: Circuit breaker monitor Dashboard: http://localhost:7979

## Flow

- Start Client service (To-Read)
- Request Client service with Supplier service (Bookstore) down, receive fallback response (circuit open)
- Start Supplier service
- Request Client service with Supplier up, receive response from Supplier (circuit closed)
- Stop Supplier service
- Request Client service with Supplier down, receive fallback response (circuit open)

## Steps

Open a terminal for running the supplier service.

```bash
cd supplier
mvn clean spring-boot:run
```

Get http://localhost/8090/recommended

You should get: *Spring in Action (Manning), Cloud Native Java (O'Reilly), Learning Spring Boot (Packt)*

Open a new terminal for running the client service.

```bash
cd client
mvn clean spring-boot:run
```

Get http://localhost/8080/to-read

You should get: *Spring in Action (Manning), Cloud Native Java (O'Reilly), Learning Spring Boot (Packt)*

Kill Supplier Service: hit **CTRL+C** in supplier terminal

Get http://localhost/8080/to-read

You should get fallback response from circuit breaker: *Cloud Native Java (O'Reilly)*

The circuit breaker continues returning the fallback response while the circuit is open (polling the supplier service doesn't get a response).

# Hystrix Dashboard

## Hystrix events stream

http://localhost/8080/hystrix.stream publishes events to be consumed by the Hystrix dashboard.

## Dashboard

In this demo, Hystrix Dashboards runs on http://localhost:7979

### Legends

**Short-Circuited** represents the requests that were sent to the fallback.

### Circuit

Shows a summary of the client-supplier collaboration.

### Thread Pools

Shows metrics of the thread pool defined by Hystrix on the client.

# Reference

http://martinfowler.com/bliki/CircuitBreaker.html

https://github.com/Netflix/Hystrix

https://spring.io/guides/gs/circuit-breaker/

https://github.com/spring-cloud-samples/hystrix-dashboard
