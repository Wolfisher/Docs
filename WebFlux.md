## Read Values from Mono and Flux

In Spring WebFlux, `Mono` and `Flux` are reactive types used for handling asynchronous operations and streams of data. To read values from a `Mono` or `Flux`, you have several options depending on your specific use case. Here are a few common approaches:

1. Subscribing to the `Mono` or `Flux`:
    - You can subscribe to the `Mono` or `Flux` and provide a consumer function to handle the emitted values.
    - Example with `Mono`:
        
        ```java
        Mono<String> mono = Mono.just("Hello");
        mono.subscribe(value -> System.out.println(value));
        
        ```
        
    - Example with `Flux`:
        
        ```java
        Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5);
        flux.subscribe(value -> System.out.println(value));
        
        ```
        

        In Spring WebFlux, calling `subscribe()` on a `Mono` or `Flux` does not block the execution of the code. However, the behavior of `subscribe()` depends on the context in which it is used.

        When call `subscribe()` on a `Mono` or `Flux`, it initiates the subscription process and allows the reactive pipeline to start executing asynchronously. The execution of the pipeline happens in a separate thread, and the main thread continues without blocking.

        Here's an example to illustrate the non-blocking behavior of `subscribe()`:

        ```java
        public void nonBlockingExample() {
            Mono<String> mono = Mono.just("Hello")
                .map(String::toUpperCase);

            mono.subscribe(value -> System.out.println("Received: " + value));

            System.out.println("Execution continues...");
        }
        ```

        In this example, when `subscribe()` is called on the `Mono`, it initiates the subscription process, and the reactive pipeline starts executing asynchronously. The main thread continues and prints "Execution continues..." immediately after the `subscribe()` call, without waiting for the pipeline to complete.

        The output of this code would be:
        ```
        Execution continues...
        Received: HELLO
        ```

        As you can see, the main thread continues execution without blocking, and the subscriber receives the value asynchronously.

        However, it's important to note that if you use blocking operators like `block()` or `toFuture().get()`, they will indeed block the execution until the `Mono` or `Flux` emits a value or completes. These blocking operators should be used cautiously and only when necessary, such as in testing or in certain scenarios where blocking is acceptable.

        In general, when working with reactive programming in Spring WebFlux, you should aim to maintain the non-blocking behavior by chaining operators, using `flatMap` or `flatMapMany` for asynchronous operations, and leveraging the reactive types throughout your application.




2. Using `block()` to get the value (**not recommended for non-blocking code**):
    - The `block()` method allows you to block the execution and wait for the `Mono` or `Flux` to emit a value.
    - Example with `Mono`:
        
        ```java
        Mono<String> mono = Mono.just("Hello");
        String value = mono.block();
        System.out.println(value);
        
        ```
        
    - Note: Using `block()` defeats the purpose of non-blocking reactive programming and should be used cautiously, mainly for testing or in certain scenarios where blocking is acceptable.
3. Chaining operators and transforming values:
    - You can chain various operators to transform and process the values emitted by a `Mono` or `Flux`.
    - Example with `Mono`:
        
        ```java
        Mono<String> mono = Mono.just("Hello")
            .map(String::toUpperCase)
            .filter(value -> value.length() > 3);
        mono.subscribe(value -> System.out.println(value));
        
        ```
        
    - Example with `Flux`:
        
        ```java
        Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5)
            .filter(value -> value % 2 == 0)
            .map(value -> value * 2);
        flux.subscribe(value -> System.out.println(value));
        
        ```
        
4. Returning `Mono` or `Flux` from a controller method:
    - In Spring WebFlux controllers, you can directly return `Mono` or `Flux` from your controller methods, and the framework will handle the subscription and emission of values.
    - Example with `Mono`:
        
        ```java
        @GetMapping("/hello")
        public Mono<String> hello() {
            return Mono.just("Hello");
        }
        
        ```
        
    - Example with `Flux`:
        
        ```java
        @GetMapping("/numbers")
        public Flux<Integer> getNumbers() {
            return Flux.just(1, 2, 3, 4, 5);
        }
        
        ```
        
5. Using `flatMap` or `flatMapMany` to process values:
    - The `flatMap` and `flatMapMany` operators allow you to transform a `Mono` or `Flux` into another `Mono` or `Flux` and perform operations on the emitted values.
    - Example with `Mono`:
        
        ```java
        Mono<String> mono = Mono.just("Hello")
            .flatMap(value -> Mono.just(value.toUpperCase()));
        mono.subscribe(value -> System.out.println(value));
        
        ```
        
    - Example with `Flux`:
        
        ```java
        Flux<Integer> flux = Flux.just(1, 2, 3)
            .flatMap(value -> Flux.range(1, value));
        flux.subscribe(value -> System.out.println(value));
        
        ```
        

6. Using `doOnNext()` operator:
   - The `doOnNext()` operator allows you to perform an action for each emitted value without modifying the value itself.
   - Example with `Mono`:
     ```java
     Mono<String> mono = Mono.just("Hello")
         .doOnNext(value -> System.out.println("Received: " + value));
     mono.subscribe();
     ```
   - Example with `Flux`:
     ```java
     Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5)
         .doOnNext(value -> System.out.println("Received: " + value));
     flux.subscribe();
     ```

7. Using `log()` operator for debugging:
   - The `log()` operator allows you to log the events and values emitted by a `Mono` or `Flux` for debugging purposes.
   - Example with `Mono`:
     ```java
     Mono<String> mono = Mono.just("Hello")
         .log();
     mono.subscribe();
     ```
   - Example with `Flux`:
     ```java
     Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5)
         .log();
     flux.subscribe();
     ```

8. Using `toIterable()` or `toStream()` for batch processing:
   - The `toIterable()` and `toStream()` methods allow you to convert a `Flux` into an `Iterable` or a `Stream`, respectively, for batch processing.
   - Example with `Flux` and `toIterable()`:
     ```java
     Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5);
     Iterable<Integer> iterable = flux.toIterable();
     for (Integer value : iterable) {
         System.out.println("Received: " + value);
     }
     ```
   - Example with `Flux` and `toStream()`:
     ```java
     Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5);
     Stream<Integer> stream = flux.toStream();
     stream.forEach(value -> System.out.println("Received: " + value));
     ```

9. Using `reduce()` or `collectList()` for accumulation:
   - The `reduce()` and `collectList()` operators allow you to accumulate the values emitted by a `Flux` into a single value or a list.
   - Example with `Flux` and `reduce()`:
     ```java
     Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5);
     Mono<Integer> sum = flux.reduce(0, Integer::sum);
     sum.subscribe(result -> System.out.println("Sum: " + result));
     ```
   - Example with `Flux` and `collectList()`:
     ```java
     Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5);
     Mono<List<Integer>> list = flux.collectList();
     list.subscribe(values -> System.out.println("Values: " + values));
     ```
## `@Async` Annotation 

The `@Async` annotation is typically used in Spring MVC with a **Servlet-based** application to perform method execution asynchronously using a separate thread pool. However, in the reactive programming model, where non-blocking operations are preferred, the use of @Async with thread pools can introduce unnecessary complexity and potentially block the event loop if not managed correctly. Reactive programming relies on non-blocking I/O operations and scheduling work on a small number of threads without dedicating threads to specific tasks for extended periods.

In a reactive application, instead of using `@Async` for asynchronous processing, leverage the native reactive types (`Mono`, `Flux`) and operators to manage asynchronous operations. This approach allows for better integration with the WebFlux framework and avoids potential issues with blocking the event loop.
