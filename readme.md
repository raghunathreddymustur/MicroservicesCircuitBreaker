# Microservices Resilience

Implementing Circuit Breaker using Gateway server
------------------------------------------------
1. Add the resilience4j Circuit breaker dependency
2. ```xml
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
		</dependency>
   ```
3. Add the filter to configure the circuit breaker
   `.circuitBreaker(config -> config.setName("accountsCircuitBreaker")`
4. add configuration in properties
5. ```yaml
    resilience4j.circuitbreaker:
    configs:
    accountsCircuitBreaker:
    slidingWindowSize: 10
    permittedNumberOfCallsInHalfOpenState: 2
    failureRateThreshold: 50
    waitDurationInOpenState: 10000
   ```
6. Add the fallback mechanism
   ```java
		return routeLocatorBuilder.routes()
						.route(p -> p
								.path("/eazybank/accounts/**")
								.filters( f -> f.rewritePath("/eazybank/accounts/(?<segment>.*)","/${segment}")
										.addResponseHeader("X-Response-Time", LocalDateTime.now().toString())
										.circuitBreaker(config -> config.setName("accountsCircuitBreaker").setFallbackUri("forward:/contactSupport")))
								.uri("lb://ACCOUNTS"))
   ```
7. Invoke the api through gateway server and monitor the metrics via actuator
   ![img.png](img.png)

   
Circuit Breaker Using feign client
------------------------------
1. add the following dependency in respective microservice
2. ```xml
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
		</dependency>

   ```
3. Modify the Feign clients to work for feedback
   ```java
   @FeignClient(name="cards", fallback = CardsFallback.class)
   public interface CardsFeignClient {
   
       @GetMapping(value = "/api/fetch",consumes = "application/json")
       public ResponseEntity<CardsDto> fetchCardDetails(@RequestHeader("eazybank-correlation-id")
                                                            String correlationId, @RequestParam String mobileNumber);
   
   }
   ```
4. Add the fallback class
   ```java
   package com.eazybytes.accounts.service.client;
   
   import com.eazybytes.accounts.dto.CardsDto;
   import org.springframework.http.ResponseEntity;
   import org.springframework.stereotype.Component;
   
   @Component
   public class CardsFallback implements CardsFeignClient{
   
       @Override
       public ResponseEntity<CardsDto> fetchCardDetails(String correlationId, String mobileNumber) {
           return null;
       }
   
   }
   ```
