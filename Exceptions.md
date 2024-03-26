# Exception Handling:
1. Using `onErrorResume()` or `onErrorReturn()`:
   - The `onErrorResume()` operator allows you to provide a fallback `Mono` or `Flux` in case of an error.
   - The `onErrorReturn()` operator allows you to return a default value in case of an error.
   - Example with `onErrorResume()`:
     ```java
     Mono<String> mono = Mono.error(new RuntimeException("Error occurred"))
         .onErrorResume(e -> Mono.just("Fallback value"));
     ```
   - Example with `onErrorReturn()`:
     ```java
     Mono<String> mono = Mono.error(new RuntimeException("Error occurred"))
         .onErrorReturn("Default value");
     ```

2. Using `doOnError()` for error handling:
   - The `doOnError()` operator allows you to perform an action when an error occurs, such as logging the error or updating metrics.
   - Example:
     ```java
     Mono<String> mono = Mono.error(new RuntimeException("Error occurred"))
         .doOnError(e -> {
             // Log the error
             log.error("An error occurred: ", e);
             // Update error metrics
             errorCounter.increment();
         });
     ```

3. Handling exceptions in controller methods:
   - You can handle exceptions in controller methods by returning a `Mono` or `Flux` that represents an error response.
   - Example:
     ```java
     @GetMapping("/data")
     public Mono<String> getData() {
         return dataService.getData()
             .onErrorResume(e -> {
                 // Log the error
                 log.error("An error occurred: ", e);
                 // Return an error response
                 return Mono.just("Error occurred");
             });
     }
     ```

# Metric Logging:
1. Using `Micrometer` library with Spring Boot Actuator:
   - Spring Boot Actuator integrates with the Micrometer library for capturing metrics.
   - You can use Micrometer to record metrics such as request count, response time, and error count.
   - Example:
     ```java
     @RestController
     public class DataController {
         private final Counter requestCounter;
         private final Timer requestTimer;

         public DataController(MeterRegistry meterRegistry) {
             this.requestCounter = meterRegistry.counter("data.requests.count");
             this.requestTimer = meterRegistry.timer("data.requests.timer");
         }

         @GetMapping("/data")
         public Mono<String> getData() {
             return Mono.just("Data")
                 .doOnSubscribe(s -> requestCounter.increment())
                 .doOnSuccess(data -> requestTimer.record(Duration.ofMillis(100)));
         }
     }
     ```

2. Using `log()` operator for logging:
   - The `log()` operator allows you to log the events and values emitted by a `Mono` or `Flux`.
   - Example:
     ```java
     Mono<String> mono = Mono.just("Data")
         .log("DataMono");
     ```

3. Using `doOnNext()`, `doOnSuccess()`, or `doOnError()` for custom logging:
   - The `doOnNext()`, `doOnSuccess()`, and `doOnError()` operators allow you to perform actions for each emitted value, on successful completion, or on error, respectively.
   - Example:
     ```java
     Mono<String> mono = Mono.just("Data")
         .doOnNext(data -> log.info("Received data: {}", data))
         .doOnSuccess(data -> log.info("Data processing completed"))
         .doOnError(e -> log.error("Error occurred: ", e));
     ```

