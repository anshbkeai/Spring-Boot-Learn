Mediun  Link  = https://medium.com/javarevisited/java-completablefuture-c47ca8c885af
---


# Future vs CompletableFuture in Java

## Key Differences

| Feature              | Future                          | CompletableFuture                      |
|----------------------|----------------------------------|----------------------------------------|
| **Blocking**         | Blocking (uses `get()`)         | Non-blocking (uses `then*` methods)    |
| **Composition**      | Limited                          | Powerful chaining with `thenCompose()`, `thenCombine()`, etc. |
| **Exception Handling** | Manual handling                 | Declarative with `exceptionally()`, `handle()` |
| **Completion Control** | No manual completion            | Manual completion with `complete()`, `completeExceptionally()`, `cancel()` |

## Summary

`CompletableFuture` is a more powerful, flexible, and modern tool for handling asynchronous programming compared to `Future`. It enables non-blocking operations, better composition of tasks, robust exception handling, and more control over task completion.

---

## Async Methods of CompletableFuture

### 1. `thenApplyAsync()`

Used to process the result asynchronously and transform it.

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "Hello");

CompletableFuture<Integer> transformedFuture = future.thenApplyAsync(s -> {
    System.out.println("Thread: " + Thread.currentThread().getName());
    return s.length();
});

transformedFuture.thenAccept(length -> {
    System.out.println("Thread: " + Thread.currentThread().getName());
    System.out.println("Length of Hello: " + length);
});
````

**Sample Output:**

```
Thread: ForkJoinPool.commonPool-worker-1
Thread: ForkJoinPool.commonPool-worker-1
Length of Hello: 5
```

---

### 2. `thenAcceptAsync()`

Consumes the result asynchronously without returning a value.

```java
CompletableFuture.supplyAsync(() -> "Hello")
    .thenAcceptAsync(s -> {
        System.out.println("Thread: " + Thread.currentThread().getName());
        System.out.println(s.toUpperCase());
    });
```

**Sample Output:**

```
Thread: ForkJoinPool.commonPool-worker-1
HELLO
```

---

### 3. `runAsync()`

Runs a task asynchronously without returning or consuming any value.

```java
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    System.out.println("Thread: " + Thread.currentThread().getName());
    System.out.println("Hello from async task");
});
```

**Sample Output:**

```
Thread: ForkJoinPool.commonPool-worker-1
Hello from async task
```

---

### 4. `thenComposeAsync()`

Chains multiple asynchronous tasks where the second depends on the result of the first.

```java
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");

CompletableFuture<String> future2 = future1.thenComposeAsync(s ->
    CompletableFuture.supplyAsync(() -> s + " World")
);

future2.thenAccept(System.out::println);
```

**Output:**

```
Hello World
```

---

## Real-World Example: Aggregating Async REST Calls

We want to fetch responses from `/users`, `/products`, and `/orders` asynchronously and combine them.

### Service Class

```java
@Service
public class AggregatorService {
    @Autowired
    private AsyncRestTemplate restTemplate;

    public CompletableFuture<AggregatedResponse> getAggregatedResponse() {
        CompletableFuture<User[]> usersFuture = CompletableFuture.supplyAsync(() ->
            restTemplate.getForObject("http://localhost:8080/users", User[].class)
        );

        CompletableFuture<Product[]> productsFuture = CompletableFuture.supplyAsync(() ->
            restTemplate.getForObject("http://localhost:8080/products", Product[].class)
        );

        CompletableFuture<Order[]> ordersFuture = CompletableFuture.supplyAsync(() ->
            restTemplate.getForObject("http://localhost:8080/orders", Order[].class)
        );

        return CompletableFuture.allOf(usersFuture, productsFuture, ordersFuture)
            .thenApply(v -> new AggregatedResponse(
                usersFuture.join(),
                productsFuture.join(),
                ordersFuture.join()
            ));
    }
}
```

---

### Controller Class

```java
@RestController
public class AggregatorController {
    @Autowired
    private AggregatorService aggregatorService;

    @GetMapping("/aggregate")
    public CompletableFuture<AggregatedResponse> getAggregatedResponse() {
        return aggregatorService.getAggregatedResponse();
    }
}
```

---

### AggregatedResponse Class

```java
public class AggregatedResponse {
    private User[] users;
    private Product[] products;
    private Order[] orders;

    // Getters and Setters
}
```

---

**Note:** `AsyncRestTemplate` is deprecated in modern Spring applications. It's recommended to use `WebClient` from Spring WebFlux for future development.

---
 like this saved into a downloadable `.md` file or want it converted into PDF or HTML format too?
```
