# Microservices-2

# 🛡️ Resiliency in Microservices
## ✅ Objective:

Ensure **system stability** and **fault tolerance** when one or more microservices fail or perform slowly.

---

## ⚠️ Problem Areas & Solutions

---

### 1️⃣ **Avoiding Cascading Failures**

> ❓ **Problem:**
> A failure in one microservice can cause a chain reaction, affecting other services.

> ✅ **Solution:**
> Use the **Circuit Breaker Pattern** to stop the ripple effect.

* **Circuit Breaker States:**

  * **Closed:** Requests flow normally.
  * **Open:** Requests fail instantly to avoid further load.
  * **Half-Open:** Test requests to check recovery.

> 🛠️ **Tool:**
> [Resilience4j](https://resilience4j.readme.io/) – A lightweight fault tolerance library for Java 8+ and functional programming.

---

### 2️⃣ **Handling Failures Gracefully with Fallbacks**

> ❓ **Problem:**
> What if a service fails during a chain of service-to-service communication?

> ✅ **Solution:**
> Implement **fallback mechanisms** like:

* Returning a **default value**
* Serving data from **cache**
* Calling an **alternative service/DB**

> 🎯 **Goal:**
> Ensure users still receive some response rather than a complete failure.

---

### 3️⃣ **Making Services Self-Healing Capable**

> ❓ **Problem:**
> How do we allow slow/failing services time to recover without crashing the system?

> ✅ **Solution:**
> Configure:

* **Timeouts** – Fail fast if service is too slow.
* **Retries with Backoff** – Try again after delays.
* **Rate Limiting** – Control traffic load.
* **Bulkheads** – Isolate service failures.



## 📌 Key Tools & Patterns

| Tool / Pattern                 | Purpose                                  |
| ------------------------------ | ---------------------------------------- |
| **Resilience4j**               | Circuit breaker, retry, rate limiting    |
| **Fallback Method**            | Default behavior on service failure      |
| **Timeouts**                   | Prevent indefinite waits                 |
| **Retries**                    | Try requests again on transient failures |
| **Bulkheads**                  | Isolate failures in specific services    |
| **Service Mesh (e.g., Istio)** | External resiliency management           |

---

## 🔄 Hystrix vs Resilience4j

| Feature                    | Hystrix (Netflix) | Resilience4j          |
| -------------------------- | ----------------- | --------------------- |
| Development Status         | Maintenance mode  | Actively maintained   |
| Java Compatibility         | Java 6+           | Java 8+ (functional)  |
| Integration                | Tightly coupled   | Modular & lightweight |
| Preferred for new projects | ❌                 | ✅                     |


## ✅ Summary

* **Circuit Breaker** → Avoid cascading failures
* **Fallbacks** → Gracefully handle failures
* **Timeouts & Retries** → Allow services to recover
* **Resilience4j** → Modern solution for resiliency in Java microservices

---


---
## 💻 What is the Circuit Breaker Pattern?

> A **resiliency design pattern** used in **distributed systems** to handle:

* **Transient faults**
* **Service timeouts**
* **Overloaded or unavailable resources**


## 🎯 Purpose

To **monitor remote service calls** and **fail fast** when failures exceed a threshold, avoiding system-wide failure.



## 🔄 How It Works

### Circuit Breaker States:

| State         | Description                                                                               |
| ------------- | ----------------------------------------------------------------------------------------- |
| **Closed**    | Normal state. All requests go through. Monitors failures.                                 |
| **Open**      | If failures exceed a threshold, circuit trips. All requests fail immediately (fail-fast). |
| **Half-Open** | After a cooldown, allows a few test requests. If successful, circuit closes again.        |



## 🧠 Behavior Overview

1. **Monitor** remote calls.
2. **Kill** calls that take too long (timeouts).
3. If enough calls fail, **“trip the breaker”**.
4. **Stop** sending requests to faulty service temporarily.
5. Periodically **test** if service is back (half-open).
6. If recovered, **restore** normal flow.


## ✅ Advantages of Circuit Breaker Pattern

| Benefit                | Description                                                     |
| ---------------------- | --------------------------------------------------------------- |
| **Fail Fast**          | Avoids wasting time on calls that are likely to fail.           |
| **Fail Gracefully**    | Returns fallback responses instead of crashing.                 |
| **Recover Seamlessly** | Automatically checks and resumes once service is healthy again. |


## 🛠️ Common Use Cases

* Microservice-to-microservice communication
* External API integrations
* Database or cache server connectivity



## 🧰 Tools for Implementation

| Tool/Library     | Language | Notes                                     |
| ---------------- | -------- | ----------------------------------------- |
| **Resilience4j** | Java     | Modern, lightweight, supports annotations |
| **Hystrix**      | Java     | Legacy Netflix library, now deprecated    |
| **Polly**        | .NET     | Resiliency framework for .NET             |

---



# 🚀 Circuit Breaker Pattern with Spring Cloud Gateway & Resilience4j
---

## 🧩 What is Circuit Breaker Pattern?

The **Circuit Breaker Pattern** prevents cascading failures in a distributed system by:

* Detecting when a service is down or slow.
* Stopping further calls to the failing service.
* Redirecting to a **fallback route or response**.
* Trying again after a cooling period to see if the service has recovered.

---

## 🟢 1. **CLOSED State**

> ✅ **Default/Initial State**
> The circuit breaker **starts in CLOSED state** and **allows all client requests** to pass through to the remote service.

### 🔍 What it does:

* Monitors the outcome of calls (success/failure).
* If failures start to increase, it checks if the **failure rate exceeds a configured threshold**.

### 🔁 Transition:

* If **failure rate exceeds threshold**, move to **OPEN**.

---

## 🔴 2. **OPEN State**

> ❌ **Fail-Fast Mode**
> The circuit **trips open** when too many failures are detected.

### 🔍 What it does:

* **Blocks all incoming requests** immediately.
* Prevents further load on the failing service.
* Returns fallback responses or error quickly.

### 🕒 Timeout:

* Remains in OPEN state for a **configured wait duration**.

### 🔁 Transition:

* After timeout, transitions to **HALF\_OPEN** to test if recovery has occurred.

---

## 🟡 3. **HALF\_OPEN State**

> 🔄 **Testing Phase**
> Allows a **limited number of test requests** to see if the remote service has recovered.

### 🔍 What it does:

* Monitors the success/failure of a few requests.
* Helps decide whether to close the circuit or re-open it.

### 🔁 Transition:

* If **requests succeed**: Move to **CLOSED** (recovery).
* If **requests fail** again: Move back to **OPEN** (still faulty).

---

## 🔁 Resilience4j Circuit Breaker State Transitions

```plaintext
[CLOSED] → (failure rate > threshold) → [OPEN]
[OPEN] → (after wait duration) → [HALF_OPEN]
[HALF_OPEN] → (failures) → [OPEN]
[HALF_OPEN] → (successes) → [CLOSED]
```

---

## 📊 Summary Table

| State          | Behavior                                    | Transition Condition             |
| -------------- | ------------------------------------------- | -------------------------------- |
| **CLOSED**     | All requests pass; monitors failures        | Failure rate > threshold → OPEN  |
| **OPEN**       | All requests fail-fast; no actual call made | After wait duration → HALF\_OPEN |
| **HALF\_OPEN** | Limited test calls allowed                  | Success → CLOSED; Failure → OPEN |


---
## 🔧 Step-by-Step Implementation

---

### ✅ **1. Add Required Dependency**

Add the following Maven dependency in your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
</dependency>
```

🔎 **Why?**
This dependency brings in support for **Resilience4j** (a popular fault-tolerance library) in **reactive Spring Cloud Gateway**.

---

### ✅ **2. Define Circuit Breaker in Route Configuration**

```java
@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route(p -> p.path("/eazybank/accounts/**")
            .filters(f -> f
                .rewritePath("/eazybank/accounts/(?<segment>.*)", "/${segment}") // rewrites incoming path
                .addResponseHeader("X-Response-Time", new Date().toString()) // adds a custom header to response
                .circuitBreaker(config -> config
                    .setName("accountsCircuitBreaker") // circuit breaker name
                    .setFallbackUri("forward:/contactSupport"))) // fallback route when service fails
            .uri("lb://ACCOUNTS")) // load-balanced service name (via Eureka/Service Discovery)
        .build();
}
```

🔎 **Explanation**

* `RouteLocator` is used to define dynamic routing rules.
* `circuitBreaker()` adds the Resilience4j circuit breaker.
* `setFallbackUri()` forwards the request to another internal route when failure occurs.
* `"lb://ACCOUNTS"` means it routes to the `ACCOUNTS` service via service discovery.

---

### ✅ **3. Create Fallback Controller**

```java
@RestController
public class FallbackController {

    @RequestMapping("/contactSupport")
    public Mono<String> contactSupport() {
        return Mono.just("An error occurred. Please try after some time or contact support team!!!");
    }

}
```

🔎 **Explanation**

* This REST controller handles fallback logic.
* `Mono<String>` is used for reactive programming.
* This message is shown when the `ACCOUNTS` service fails and fallback is triggered.

---

### ✅ **4. Configuration in `application.yml`**

```yaml
resilience4j.circuitbreaker:
  configs:
    default:
      slidingWindowSize: 10
      permittedNumberOfCallsInHalfOpenState: 2
      failureRateThreshold: 50
      waitDurationInOpenState: 10000 # in milliseconds (10 seconds)
```

🔎 **Explanation of Properties**:

| Property                                | Meaning                                                                |
| --------------------------------------- | ---------------------------------------------------------------------- |
| `slidingWindowSize`                     | No. of calls to evaluate for failure percentage                        |
| `permittedNumberOfCallsInHalfOpenState` | No. of test requests allowed when transitioning from OPEN → HALF\_OPEN |
| `failureRateThreshold`                  | % of failed calls to trip circuit (e.g. 50%)                           |
| `waitDurationInOpenState`               | How long to stay OPEN before going to HALF\_OPEN                       |

---

### ✅ **5. Customize Default Circuit Breaker (Optional)**

```java
@Bean
public Customizer<ReactiveResilience4JCircuitBreakerFactory> defaultCustomizer() {
    return factory -> factory.configureDefault(id -> new Resilience4JConfigBuilder(id)
        .circuitBreakerConfig(CircuitBreakerConfig.ofDefaults())
        .timeLimiterConfig(TimeLimiterConfig.custom()
            .timeoutDuration(Duration.ofSeconds(4)).build())
        .build());
}
```

🔎 **Explanation**

* Provides a **global default configuration**.
* Uses `CircuitBreakerConfig.ofDefaults()` for basic resilience settings.
* Adds a **timeout**: any call taking longer than 4 seconds will be considered a failure.

---

## 📊 Circuit Breaker State Transitions in Resilience4j

```plaintext
[CLOSED] → (50% failure rate) → [OPEN]
[OPEN] → (after 10 sec) → [HALF_OPEN]
[HALF_OPEN] → success → [CLOSED]
[HALF_OPEN] → fail → [OPEN]
```

---

## ✅ Summary

| Component            | Role                                                             |
| -------------------- | ---------------------------------------------------------------- |
| `pom.xml` dependency | Enables Spring Cloud Circuit Breaker with Resilience4j           |
| `RouteLocator` bean  | Adds circuit breaker and fallback filter                         |
| `FallbackController` | Returns a fallback message when service fails                    |
| `application.yml`    | Configures sliding window, thresholds, retry behavior            |
| `Customizer Bean`    | Globally configures timeout and circuit breaker for all services |

---

Let me know if you'd like:

* 🧪 A sample test scenario
* 🧭 Diagram of the flow
* 💡 Code for retry/timeout/bulkhead patterns as well.

Here’s a clean, detailed set of notes with code explanation for implementing the **Circuit Breaker Pattern using Spring Boot + Feign Client + Resilience4j**:

---

# 🛠️ Circuit Breaker Pattern in Spring Boot using Feign Client & Resilience4j

---

## 1. Add Maven Dependency

Add the following dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>
```

> **Purpose:** Enables Resilience4j circuit breaker integration with Spring Cloud OpenFeign.

---

## 2. Configure Feign Client with Circuit Breaker & Fallback

```java
@FeignClient(name = "cards", fallback = CardsFallback.class)
public interface CardsFeignClient {

    @GetMapping(value = "/api/fetch", consumes = "application/json")
    public ResponseEntity<CardsDto> fetchCardDetails(
        @RequestHeader("eazybank-correlation-id") String correlationId,
        @RequestParam String mobileNumber);
}
```

* `@FeignClient` declares this interface as a Feign client for service named **"cards"**.
* `fallback = CardsFallback.class` specifies the fallback class to handle failures.
* The method `fetchCardDetails` calls the remote service.

---

### Fallback Implementation

```java
@Component
public class CardsFallback implements CardsFeignClient {

    @Override
    public ResponseEntity<CardsDto> fetchCardDetails(String correlationId, String mobileNumber) {
        // Return null or default response when the 'cards' service fails
        return null;
    }
}
```

* The fallback class implements the Feign client interface.
* This is invoked automatically when the circuit breaker trips or the call fails.
* You can return a default response or cached data here.

---

## 3. Configure Circuit Breaker Properties (`application.yml`)

```yaml
spring:
  cloud:
    openfeign:
      circuitbreaker:
        enabled: true

resilience4j.circuitbreaker:
  configs:
    default:
      slidingWindowSize: 5                  # Number of calls to evaluate failure rate
      failureRateThreshold: 50              # % of failures to open the circuit
      waitDurationInOpenState: 10000        # Time (ms) to wait before attempting half-open state
      permittedNumberOfCallsInHalfOpenState: 2   # Number of calls allowed during half-open state
```

---

## ⚙️ How It Works

* When **5 calls** happen, if **50% or more fail**, the circuit breaker moves to **OPEN** state.
* While **OPEN**, calls fail immediately without calling remote service.
* After **10 seconds**, circuit breaker goes to **HALF\_OPEN**, allowing **2 test calls**.
* If test calls succeed, circuit breaker closes; otherwise, it reopens.
* Fallback class handles failed calls gracefully.

---

## ✅ Summary

| Step                       | Description                                      |
| -------------------------- | ------------------------------------------------ |
| Add Maven Dependency       | Include Resilience4j circuit breaker support     |
| Feign Client with Fallback | Define remote calls with fallback implementation |
| Configure Properties       | Set thresholds and timing in `application.yml`   |

---


# 🔄 Retry Pattern in Microservices

## ✅ What is the Retry Pattern?

* The **Retry Pattern** helps automatically reattempt failed operations **multiple times** before considering them as failed permanently.
* Commonly used in scenarios like **network glitches** or **temporary unavailability** of services, where retrying after a short delay may succeed.
* Helps improve system reliability and user experience by handling transient faults transparently.

The Retry Pattern allows a method to automatically retry a failed execution a specified number of times before finally failing. It’s most useful for **transient faults** such as:

* Temporary network issues
* Service timeouts
* Resource contention
  
## Key Components & Considerations

| Aspect                          | Description                                                                                                                                             |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Retry Logic**                 | Configure how many times and when to retry based on error types, exceptions, or response codes.                                                         |
| **Backoff Strategy**            | Use delays between retries to avoid overwhelming the service; often uses **exponential backoff** to increase wait times progressively.                  |
| **Circuit Breaker Integration** | Combine retries with circuit breakers to avoid wasting resources if the service is down for long. If retries fail consecutively, circuit breaker opens. |
| **Idempotent Operations**       | Ensure the retried operation is **idempotent** (safe to repeat) to avoid side effects like duplicate transactions.                                      |

---

## Example: Retry Pattern Using Spring Cloud Gateway Filter

```java
@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route(p -> p.path("/eazybank/loans/**")
            .filters(f -> f.rewritePath("/eazybank/loans/(?<segment>.*)", "/${segment}")
                .addResponseHeader("X-Response-Time", new Date().toString())
                .retry(retryConfig -> retryConfig
                    .setRetries(3)                          // Retry 3 times on failure
                    .setMethods(HttpMethod.GET)             // Apply retry only for GET requests
                    .setBackoff(
                        Duration.ofMillis(100),             // Initial backoff delay 100 ms
                        Duration.ofMillis(1000),            // Max backoff delay 1000 ms
                        2,                                 // Multiplier for exponential backoff
                        true))                             // Enable jitter to avoid thundering herd problem
            )
            .uri("lb://LOANS"))                         // Load-balanced URI of loan service
        .build();
}
```

---

## Explanation

* **`setRetries(3)`**: The gateway will retry up to 3 times if the request fails.
* **`setMethods(HttpMethod.GET)`**: Retry is applied only to GET HTTP method (safe/idempotent).
* **Backoff parameters:**

  * Start retry after 100ms,
  * Increase delay exponentially (multiplier 2),
  * Max delay 1000ms,
  * Jitter enabled to add randomness and avoid synchronized retries.
* The retry filter ensures transient errors don’t cause immediate failure and helps stabilize calls to the `LOANS` microservice.



## Summary

| Benefits                                        | Notes                                                           |
| ----------------------------------------------- | --------------------------------------------------------------- |
| Improves fault tolerance for transient failures | Helps recover from network glitches or temporary service issues |
| Reduces user-facing errors                      | Fewer failed requests reach the client                          |
| Needs careful configuration                     | Avoid excessive retries causing load spikes                     |
| Works best with idempotent APIs                 | Prevents side effects from repeated requests                    |

---


# 🔁 Retry Pattern in Spring Boot (Resilience4j)

## 🛠 Steps to Implement Retry Pattern

---

### 🔹 Step 1: Add Retry Annotation in Service

```java
import io.github.resilience4j.retry.annotation.Retry;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class BuildInfoController {

    @Retry(name = "getBuildInfo", fallbackMethod = "getBuildInfoFallBack")
    @GetMapping("/build-info")
    public ResponseEntity<String> getBuildInfo() {
        // Simulate external call or logic that may fail
        throw new RuntimeException("Service temporarily unavailable");
    }

    // Fallback method with same signature + Throwable at the end
    public ResponseEntity<String> getBuildInfoFallBack(Throwable t) {
        return ResponseEntity.ok("Fallback: Unable to fetch build info at the moment.");
    }
}
```

### 🔍 Explanation

| Annotation                                                               | Description                                                                                                  |
| ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------ |
| `@Retry(name = "getBuildInfo", fallbackMethod = "getBuildInfoFallBack")` | Tells Resilience4j to apply retry logic named `getBuildInfo` and use the fallback if all attempts fail.      |
| Fallback Method                                                          | Must have **same return type** and **same parameters**, with an additional `Throwable` parameter at the end. |

---

### 🔹 Step 2: Configure Retry in `application.yml`

```yaml
resilience4j:
  retry:
    configs:
      default:
        maxRetryAttempts: 3                     # Retry 3 times
        waitDuration: 500                       # Wait 500ms between retries
        enableExponentialBackoff: true          # Enable exponential backoff
        exponentialBackoffMultiplier: 2         # Wait time doubles on each retry
        retryExceptions:                        # Retry only for these exceptions
          - java.util.concurrent.TimeoutException
        ignoreExceptions:                       # Ignore these exceptions (no retry)
          - java.lang.NullPointerException
```

### 🔍 Explanation of Properties

| Property                       | Description                                              |
| ------------------------------ | -------------------------------------------------------- |
| `maxRetryAttempts`             | Maximum number of retries after initial failure.         |
| `waitDuration`                 | Base wait time (ms) between retries.                     |
| `enableExponentialBackoff`     | Increases wait duration exponentially (with multiplier). |
| `exponentialBackoffMultiplier` | How much to multiply the delay after each retry.         |
| `retryExceptions`              | Only retry if these exceptions occur.                    |
| `ignoreExceptions`             | Never retry for these exceptions.                        |

---

## 🧪 Use Case Example

Imagine your service calls a remote server that occasionally times out:

* First call fails due to a temporary timeout.
* Retry is triggered 3 times with increasing wait time: 500ms, 1000ms, 2000ms.
* If it still fails, fallback method is invoked.

## 📝 Summary

| Feature             | Benefit                                                     |
| ------------------- | ----------------------------------------------------------- |
| Retry + fallback    | Improve user experience without exposing errors             |
| Exponential backoff | Prevent retry storms that worsen the issue                  |
| Exception control   | Fine-grained retry behavior based on specific failure types |
| Fallback method     | Ensure graceful degradation when retries fail               |


---

## 🧠 What Is the Rate Limiter Pattern?

In microservices, the **Rate Limiter Pattern** is a critical **resilience and security mechanism** used to control how many requests a client (user, IP, system) can make to your service in a **given time window**.

If a client exceeds this limit:

* The service **rejects** the request.
* Returns a **`429 Too Many Requests`** response (standard HTTP error for rate limiting).

This pattern ensures **fair usage**, **prevents abuse**, and **protects backend services** from getting overwhelmed.

---

## 🎯 Why Use Rate Limiting?

| Scenario                                  | Benefit                                                                  |
| ----------------------------------------- | ------------------------------------------------------------------------ |
| 🧑‍💻 A free user sends 1000 requests/sec | Prevents service crash or resource exhaustion                            |
| 🛡️ DoS or brute force attack             | Throttles excess requests                                                |
| ⚖️ Tiered pricing model                   | Enforce usage limits per plan (e.g., Free: 10 req/min, Pro: 100 req/min) |
| 🔄 Bursty traffic (spikes)                | Smoothens load to ensure stability                                       |

---

# 🛠️ Implementing the Rate Limiter Pattern

## ✅ 1. Using **Spring Cloud Gateway**

Gateway is a great place to enforce rate limits because it sits at the edge of your system, before requests hit any internal service.

### ⚙️ Required Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

### 🧰 Redis Requirement

Spring Cloud Gateway's rate limiter uses **Redis** to store counters for:

* How many requests a specific client has made
* When the limit resets

> Redis allows this state to be **shared across multiple gateway instances** in a distributed system.

---

### 🧩 Gateway Code Example

```java
@Bean
public RouteLocator myRoutes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route(p -> p.path("/eazybank/cards/**")
            .filters(f -> f
                .rewritePath("/eazybank/cards/(?<segment>.*)", "/${segment}")
                .addResponseHeader("X-Response-Time", new Date().toString())
                .requestRateLimiter(config -> config
                    .setRateLimiter(redisRateLimiter())
                    .setKeyResolver(userKeyResolver())))
            .uri("lb://CARDS"))
        .build();
}
```

### 💡 What’s Happening?

* `rewritePath(...)`: Changes incoming URL to match the internal API path.
* `addResponseHeader(...)`: Adds diagnostic info.
* `requestRateLimiter(...)`: Core rate limiting logic.

---

### 🔑 RateLimiter Configuration

```java
@Bean
public RedisRateLimiter redisRateLimiter() {
    return new RedisRateLimiter(1, 1, 1); // Replenish 1 token per sec
}

@Bean
KeyResolver userKeyResolver() {
    return exchange -> Mono.justOrEmpty(exchange.getRequest().getHeaders().getFirst("user"))
        .defaultIfEmpty("anonymous");
}
```

**Explanation**:

* 1 request per second allowed
* KeyResolver uses a `user` header to identify users (e.g., `"user": "john123"`).
* Anonymous users default to one key—helps block unauthenticated abuse.

---

## ✅ 2. Using **Spring Boot with Resilience4j**

Ideal when **rate limiting per method or service-level** logic is needed.

### 🧱 Dependency (via Spring Boot Starter)

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot2</artifactId>
</dependency>
```

---

### ✍️ Sample Controller Code

```java
@RestController
public class JavaController {

    @RateLimiter(name = "getJavaVersion", fallbackMethod = "getJavaVersionFallback")
    @GetMapping("/java-version")
    public ResponseEntity<String> getJavaVersion() {
        return ResponseEntity.ok(System.getProperty("java.version"));
    }

    public ResponseEntity<String> getJavaVersionFallback(Throwable t) {
        return ResponseEntity.status(HttpStatus.TOO_MANY_REQUESTS)
                             .body("Rate limit exceeded. Try again later.");
    }
}
```

---

### ⚙️ Configuration (`application.yml`)

```yaml
resilience4j:
  ratelimiter:
    configs:
      default:
        timeoutDuration: 5000
        limitRefreshPeriod: 5000
        limitForPeriod: 1
```

**Explanation**:

* `limitForPeriod`: Only 1 request allowed every 5 seconds.
* `timeoutDuration`: Caller will wait 5 seconds for a permit before failing.
* `limitRefreshPeriod`: Resets the token every 5 seconds.

---

## 🧩 Spring Gateway vs. Spring Boot (Resilience4j)

| Feature       | Spring Cloud Gateway                                  | Spring Boot + Resilience4j            |
| ------------- | ----------------------------------------------------- | ------------------------------------- |
| Scope         | API Gateway / Edge Layer                              | Inside service (method-level)         |
| Distributed   | Yes (uses Redis)                                      | No (in-memory, per instance)          |
| Ideal for     | Enforcing global limits before reaching microservices | Per-user/service method-level control |
| Key Mechanism | Custom key via `KeyResolver` (IP/User)                | No key needed—annotation per method   |
| Use Case      | Protect backend from external users                   | Protect internal endpoints per method |

---

## 💼 Real-World Examples

### Example 1: **Public API Tier**

| Tier       | Limit             |
| ---------- | ----------------- |
| Free       | 10 requests/min   |
| Pro        | 100 requests/min  |
| Enterprise | 1000 requests/min |

* Use Gateway to apply limits based on API key or user ID header.
* Internally apply Resilience4j to sensitive methods (like payment, login).

---

### Example 2: **Chat Application**

* Limit `/send-message` to 5 messages per second per user.
* Gateway filters can apply rate limit using the user’s JWT claims or ID.
* Backend may also use Resilience4j rate limiter to prevent burst spam.



## ✅ Summary

| When to Use                              | Use Gateway Rate Limiter | Use Resilience4j Rate Limiter |
| ---------------------------------------- | ------------------------ | ----------------------------- |
| External traffic control                 | ✅                        | ❌                             |
| Internal method protection               | ❌                        | ✅                             |
| Distributed control (multiple instances) | ✅                        | ❌                             |
| Lightweight or non-Redis setup           | ❌                        | ✅                             |
| Tier-based API limits                    | ✅                        | ❌                             |



