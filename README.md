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

Here’s a **well-structured and professional note** based on the topic and explanation you provided:

---

## 🛡️ Microservices Security :

### Securing Microservices from Unauthorized Access

### 🔍 Problem Statement

Currently, our microservices are **exposed without any security**:

* Any client or end user can invoke our services and access **sensitive data**.
* There is **no authentication or authorization mechanism** in place.

### 🚨 Risks

* **Data breaches** due to unauthorized access.
* **Lack of accountability** (anyone can access APIs without identification).
* **Violation of compliance standards** (e.g., GDPR, HIPAA).

---

### ✅ Objectives

To secure microservices, we need to ensure:

1. **Authentication** – Who is making the request?
2. **Authorization** – What is the requestor allowed to do?
3. **Identity Management** – Where and how are users and permissions stored and managed?

---

### 🔐 Authentication & Authorization

Each microservice must:

* **Authenticate** the identity of the client (user or service).
* **Authorize** access based on roles, permissions, or scopes.

#### ✨ Best Practices:

* Use **JWT (JSON Web Tokens)** for stateless authentication.
* Use **Role-Based Access Control (RBAC)** or **Attribute-Based Access Control (ABAC)** for fine-grained authorization.

---

### 🏢 Centralized Identity and Access Management (IAM)

To avoid duplicating auth logic in each service, use a **central IAM system**:

#### 🔧 Tools:

* **OAuth2 / OpenID Connect** – For secure, token-based authentication.
* **Keycloak** – Open-source Identity and Access Management server.
* **Spring Security** – For enforcing security policies in each service.

#### 📌 Benefits of Central IAM:

* Centralized **user credentials storage**.
* **Single Sign-On (SSO)** capabilities.
* **Centralized policy management** (roles, permissions, etc.).
* Easier **auditing** and **compliance**.


### 📚 Summary

| Feature              | Tool / Approach           |
| -------------------- | ------------------------- |
| Authentication       | OAuth2 / OpenID Connect   |
| Authorization        | Roles, Scopes (RBAC/ABAC) |
| Central IAM          | Keycloak                  |
| Security in services | Spring Security           |
| Token Format         | JWT                       |


Here’s a **well-organized note** for the topic:
**"Why should we use OAuth2 instead of Basic Authentication for securing microservices?"**

---

## 🔐 Why Use OAuth2 Instead of Basic Authentication in Microservices?

---

### 🔑 1. Understanding Basic Authentication

In **Basic Authentication**:

* User credentials (username and password) are sent in every request (often Base64-encoded).
* Server verifies credentials and typically starts a session.
* Session details are stored (in memory or cookie) to keep the user logged in.

#### 🧱 Typical Flow:

```
Client ➝ Sends username & password ➝ Server ➝ Verifies ➝ Creates session ➝ Stores session in cookie
```

---

### ⚠️ Drawbacks of Basic Authentication

| Drawback                      | Explanation                                                                                                      |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Tightly coupled**           | Authentication and authorization logic reside in the same application — hard to scale or manage.                 |
| **Not REST/API friendly**     | Relies on sessions and cookies — not suitable for stateless APIs or mobile apps.                                 |
| **Poor for 3rd-party access** | Cannot securely delegate access to other clients (e.g., allowing an app to access your data on another service). |
| **Scalability issues**        | Each service must handle user credentials and manage sessions — not ideal for distributed systems.               |

---

### ❓ Problem That OAuth2 Solves

> **How does Google allow me to use one account across Gmail, Drive, YouTube, and many third-party apps?**

✅ **OAuth2 is the answer.**

OAuth2 provides:

* A way to **delegate access** to third-party apps without sharing passwords.
* A **centralized authentication server** that issues tokens to access services securely.
* Support for **Single Sign-On (SSO)** across multiple apps and platforms.

---

### 🚀 Benefits of Using OAuth2 in Microservices

| Feature                     | Why It Matters                                                       |
| --------------------------- | -------------------------------------------------------------------- |
| **Token-based Auth (JWT)**  | Secure, stateless, and scalable.                                     |
| **Centralized Auth Server** | Decouples authentication logic from microservices.                   |
| **Supports SSO**            | One login for multiple services.                                     |
| **Third-party Access**      | Safe delegation of access using authorization grants.                |
| **Standard Protocol**       | Widely adopted and supported in web, mobile, and enterprise systems. |

---

### 🏗️ OAuth2 High-Level Architecture in Microservices

```
+---------+        +-----------------+        +----------------+
|  Client | --->   |  Auth Server     | --->  | Microservice A |
+---------+        | (e.g., Keycloak) |       +----------------+
                       | Issues JWT |
                       +------------+
```

Each microservice:

* Validates the **access token (JWT)** sent by clients.
* Extracts **user roles/permissions** from the token.
* Authorizes access based on those claims.


### 📚 Summary: OAuth2 vs Basic Auth

| Feature                  | Basic Auth                 | OAuth2                     |
| ------------------------ | -------------------------- | -------------------------- |
| Credential Handling      | Sent with each request     | Exchanged once for token   |
| Token Support            | ❌ No                       | ✅ Yes (JWT, Bearer tokens) |
| Mobile/API Friendly      | ❌ Session/cookie dependent | ✅ Stateless and RESTful    |
| 3rd Party Integration    | ❌ Difficult                | ✅ Built-in                 |
| Centralized Auth Support | ❌ No                       | ✅ Yes                      |

## 🔐 Introduction to OAuth2

### 📖 What is OAuth2?

**OAuth2** (Open Authorization 2.0) is a **free, open security protocol** that allows one application to access data or perform actions in another application **on behalf of a user**, **without sharing user credentials**.

* It is based on **IETF standards** and governed by the **Open Web Foundation**.
* **OAuth 2.1** is the latest version, focusing on simplifying flows and improving security.


### 🎯 Key Concept: **Delegated Authorization**

> OAuth2 allows **delegated access** — one application (client) can access user data or functionality in another system (resource server) **without needing the user's password**.

This is commonly seen in:

* "Sign in with Google/Facebook"
* Third-party apps accessing your calendar, email, drive, etc.

---

### 🌟 Advantages of OAuth2

| Feature                                | Description                                                                                                                                                                           |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Supports All Types of Apps**         | OAuth2 supports web apps, mobile apps, native apps, IoT devices, smart TVs, etc.                                                                                                      |
| **Multiple Grant Types**               | Different **authorization flows** are available to suit the use case:<br> - Authorization Code<br> - Client Credentials<br> - Password Grant (legacy)<br> - Device Code               |
| **Centralized Security (Auth Server)** | - A **dedicated Authorization Server** handles all login/auth logic.<br> - Applications delegate authentication to this server.<br> - Simplifies user management and security audits. |
| **Credential Safety**                  | Third-party apps can **access resources without knowing the user's credentials**.<br> Only a **token** is shared — like a limited-access keycard.                                     |
| **Scalable & Decoupled Architecture**  | Since security logic is centralized, new services/apps can be added easily without rewriting auth code.                                                                               |
| **Token-Based Access**                 | Access Tokens (e.g., JWTs) are used for resource access — stateless and verifiable.                                                                                                   |

---

### 🏢 Analogy: OAuth2 Token as a Hotel Keycard

Think of an OAuth2 access token like a hotel room keycard:

* The **front desk (auth server)** gives you a card (token).
* The card lets you open your room (limited access).
* You don’t have the master key (your actual credentials).
* If needed, the card can be **revoked** or **expired**.

Here is a well-organized note for your topic:
**"OAuth2 Terminology"**

---

## 📘 OAuth2 Terminology

Understanding the core components of the OAuth2 ecosystem is essential for implementing secure authorization in modern applications.

---

### 🧑‍💼 1. **Resource Owner**

* The **end user** who owns the protected resources (like profile, email, calendar, etc.).
* Grants permission to a third-party application to access their data.

🟩 **Example**:
In a scenario where StackOverflow wants to access a user's GitHub profile,
👉 the **GitHub user** is the **Resource Owner**.

---

### 🧩 2. **Client**

* The **application requesting access** to the resource owner's data.
* Can be a **web app**, **mobile app**, or **API**.
* Acts **on behalf of the resource owner**.

🟩 **Example**:
👉 The **StackOverflow website** acts as the **Client** when it tries to fetch GitHub details.

---

### 🛡️ 3. **Authorization Server**

* The server responsible for **authenticating the resource owner** and **issuing access tokens** to the client.
* Knows and verifies the user's credentials.

🟩 **Example**:
👉 The **GitHub login system** acts as the **Authorization Server** — it authenticates users and issues tokens.

---

### 🗄️ 4. **Resource Server**

* The server that **hosts protected resources** (like user profile data, emails, files).
* Verifies the **access token** before serving data to the client.

🟩 **Example**:
👉 **GitHub’s API** server acts as the **Resource Server** providing access to user email, profile, etc.

---

### 🎯 5. **Scopes**

* **Granular permissions** requested by the client.
* Defines **what actions** the client can perform or **which resources** it can access.
* Example scopes:

  * `email`
  * `profile`
  * `read:messages`
  * `write:repo`

🟩 **Example**:
👉 StackOverflow might request the **`email` scope** to access the user’s GitHub email address.


### 📌 Summary Table

| Term                     | Description                                     | Example (StackOverflow + GitHub) |
| ------------------------ | ----------------------------------------------- | -------------------------------- |
| **Resource Owner**       | The user who owns the data                      | GitHub user                      |
| **Client**               | Application requesting access                   | StackOverflow website            |
| **Authorization Server** | Server that authenticates users & issues tokens | GitHub login system              |
| **Resource Server**      | Hosts and serves the protected data             | GitHub API                       |
| **Scopes**               | Defines what data/actions the client can access | `email`, `read:user`, etc.       |



## 🔐 What is OpenID Connect & Why is it Important?

---

### 📌 **What is OpenID Connect (OIDC)?**

**OpenID Connect** is an **authentication protocol** built **on top of OAuth 2.0**.
While **OAuth 2.0** provides **authorization**, OpenID Connect adds **authentication**, making it a **complete Identity and Access Management (IAM)** solution.

It introduces a new concept:
➡️ **ID Token** — a token that contains user identity information in the form of **claims** (e.g., name, email, etc.), typically in **JWT** format.

> ✅ **OAuth 2.0** → Delegated **authorization**
> ✅ **OIDC** = OAuth 2.0 + **Authentication**

---

### 📊 **OIDC vs OAuth2 Flow**

| Step | OAuth 2.0                     | OpenID Connect                                             |
| ---- | ----------------------------- | ---------------------------------------------------------- |
| 1️⃣  | Access Token                  | Access Token + **ID Token**                                |
| 2️⃣  | Scopes: e.g., `read`, `write` | **Scopes include `openid`, `profile`, `email`, `address`** |
| 3️⃣  | For accessing resources       | For accessing resources **+ authenticating the user**      |
| 4️⃣  | No user info standard         | **Standard `/userinfo` endpoint** to fetch identity        |

---

### 🧩 **Why is OpenID Connect Important?**

1. 🔑 **Adds Authentication to OAuth2**
   OAuth2 lacks built-in authentication; OIDC fills this gap by verifying the identity of the user.

2. 📇 **Provides Standardized Identity Data**

   * Via the **ID Token** (JWT format)
   * Claims include: user ID, name, email, etc.

3. 🌍 **Enables Secure Identity Sharing Across Applications**
   In today's world of interconnected services, identity federation is essential. OIDC makes this secure and standardized.

4. 🧠 **Completes IAM Strategy**
   Together with OAuth2 (for authorization), OIDC (for authentication) delivers full **Identity and Access Management** (IAM).

5. 🔁 **Improves Interoperability**

   * Applications (e.g., mobile, SPA, API clients) can securely share identity information using a unified standard.
   * Widely supported by providers like Google, Microsoft, Keycloak, Okta, etc.

---

### 🧾 Key Additions in OpenID Connect

| Feature                | Description                                              |
| ---------------------- | -------------------------------------------------------- |
| **Scopes**             | Standard scopes: `openid`, `profile`, `email`, `address` |
| **ID Token**           | JWT token carrying identity claims                       |
| **/userinfo Endpoint** | Standard endpoint to fetch additional user info          |

---

### 🧠 Summary

| Component          | Role                                    |
| ------------------ | --------------------------------------- |
| OAuth 2.0          | Handles authorization (what app can do) |
| OpenID Connect     | Adds authentication (who the user is)   |
| ID Token           | Identity claims in JWT format           |
| Access Token       | Grants access to protected resources    |
| /userinfo Endpoint | Provides user details in standard way   |


Here's a structured and clear note for:

---

## 🔐 **Authorization Code Grant Type Flow in OAuth2**

---

### 🚦 **Overview**

The **Authorization Code Grant Type** is the **most secure OAuth2 flow**, designed for apps running on **web servers** where the **client secret can be safely stored**.

It separates **authentication of the user** and **token issuance to the client**, making it resistant to attacks like token leakage.

---

### 📑 **Step-by-Step Flow**

#### 👤 **1. User Initiates Access**

> **User → Client**
> "I want to access my resources!"

---

#### 🌐 **2. Client Requests Authorization from Auth Server**

> **Client → Authorization Server**
> "Hey Auth Server, here's the user's identity and my app info. Please ask the user to authorize me."

🔽 The client sends a **request with these parameters**:

* `client_id`: Unique ID of the client app.
* `redirect_uri`: Where the Auth Server should redirect after login.
* `scope`: Permissions requested (e.g., `read`, `email`).
* `state`: CSRF protection token.
* `response_type=code`: Indicates we want an **authorization code**.

---

#### 🔐 **3. Authorization Server Authenticates User**

> **Auth Server → User**
> "Please log in and approve this app to access your resources."

* If successful, the user is redirected to the **redirect URI** with an **authorization code**.

---

#### 📥 **4. Client Exchanges Authorization Code for Access Token**

> **Client → Authorization Server**
> "Here's my **authorization code** and **client credentials**, please give me an access token."

🔽 Client sends:

* `code`: The received authorization code.
* `client_id` and `client_secret`: Credentials to authenticate the client.
* `redirect_uri`: Must match the original redirect URI.
* `grant_type=authorization_code`: Identifies this flow.

✅ **Auth Server returns → Access Token**

---

#### 📡 **5. Client Uses Access Token to Access Resource Server**

> **Client → Resource Server**
> "Here is the **access token**, please give me the user's resources."

✅ If token is valid, the **resource server returns the protected data**.

---

### 🔍 **Why 2 Steps?**

You might wonder: **Why not give the access token directly in Step 3?**

> ✅ The 2-step approach ensures that:

* The **user interacts directly** with the Auth Server (no token sent through browser).
* The **client must authenticate** to prove it’s legit before getting a token.

⚠️ **Single-step flows** like **Implicit Grant** were used earlier — but are now **deprecated** due to security risks (e.g., token leakage in browser URLs).

---

### 📘 **Summary Diagram**
```

Step 1:
[User] ──→ wants to access app ──→ [Client Application (e.g., Web App)]

Step 2:
[Client App] ──→ Redirects user to Authorization Server
               with:
               - client_id
               - redirect_uri
               - scope
               - response_type=code
               - state
               ──→ [Authorization Server (e.g., Keycloak, Google)]

Step 3:
[Authorization Server] ──→ Authenticates user (login)
                        ──→ Asks for user consent
                        ←─ If approved, redirects back to client
                           with **Authorization Code**

Step 4:
[Client App] ──→ Sends:
               - authorization code
               - client_id & client_secret
               - redirect_uri
               - grant_type=authorization_code
               ──→ [Authorization Server]

Step 5:
[Authorization Server] ←─ Validates and returns:
                        ←─ **Access Token** (and optionally a Refresh Token)

Step 6:
[Client App] ──→ Sends access token
               ──→ [Resource Server (API that holds user data)]

Step 7:
[Resource Server] ←─ Validates token
                  ←─ Returns **protected resource/data**

```

## 🔐 Securing Gateway Using Authorization Code Grant Type Flow in OAuth2

---

### 🖥️ **Flow Overview**

When an end user accesses your system through a **web or mobile app**, and these apps call APIs behind a **Spring Cloud Gateway**, the **Authorization Code Grant Type** flow is used to secure authentication and authorization.

---

### 🔄 **Step-by-Step Flow**

1. **User Tries to Access Secure Page**

   * End user uses a **UI Web or Mobile App**
   * User tries to access a secure page that calls APIs hosted behind the **Gateway Server**

2. **User Login via Authorization Server (Keycloak)**

   * Client redirects user to **Keycloak (Auth Server)** login page
   * User enters credentials and authenticates successfully

3. **Client Receives Access Token**

   * Upon successful login, the client app gets an **Access Token** from Keycloak (Auth Server)
   * This token represents user's authorization to access protected APIs

4. **Client Sends Requests to Gateway (Edge Server)**

   * Client invokes API paths on **Spring Cloud Gateway** (edge server), attaching the **access token** in the request headers

5. **Gateway Validates Access Token**

   * Gateway validates the token with the Auth Server (Keycloak)
   * If valid, gateway forwards the request to the respective backend microservices

6. **Backend Microservices Serve Requests**

   * Protected services like **Accounts**, **Loans**, **Cards** microservices receive the request
   * They respond with the requested secure data

---

### 🔑 **Key Points**

* **OAuth2 Authorization Code Grant flow** is required whenever an end user interacts with the gateway through UI apps.
* The **Gateway (Spring Cloud Gateway)** acts as an edge server that **validates the access token** before forwarding requests.
* Backend microservices are **unsecured internally** but protected by the Gateway and network firewalls (Docker or Kubernetes network).
* This approach **centralizes security** at the gateway and offloads authorization checks from individual microservices.

---

### 🖼️ **Simple Diagram**

```
[User] → (access secure UI) → [Client App] → (redirect to login) → [Auth Server (Keycloak)]
                   ← (user logs in) ←
                   → (access token) →
[Client App] → (API call with token) → [Gateway/Edge Server] → (validate token) → [Microservices]
```



### ✅ **Code with Explanation Comments**

```java
package com.eazybytes.gatewayserver.config;

import org.springframework.core.convert.converter.Converter;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.oauth2.jwt.Jwt;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

/**
 * Custom converter to extract roles from Keycloak JWT and convert them into Spring Security authorities.
 */
public class KeycloakRoleConverter implements Converter<Jwt, Collection<GrantedAuthority>> {

    @Override
    public Collection<GrantedAuthority> convert(Jwt source) {
        // Extract the 'realm_access' claim from the JWT
        Map<String, Object> realmAccess = (Map<String, Object>) source.getClaims().get("realm_access");

        // If 'realm_access' is missing or empty, return an empty list (no roles)
        if (realmAccess == null || realmAccess.isEmpty()) {
            return new ArrayList<>();
        }

        // Extract the list of roles inside the 'realm_access' claim
        // Example: "roles": ["ACCOUNTS", "CARDS", "LOANS"]
        Collection<GrantedAuthority> returnValue = ((List<String>) realmAccess.get("roles"))
                .stream()
                // Prefix each role with "ROLE_" to match Spring Security's role convention
                .map(roleName -> "ROLE_" + roleName)
                // Convert each role string into a SimpleGrantedAuthority
                .map(SimpleGrantedAuthority::new)
                // Collect into a list of GrantedAuthority
                .collect(Collectors.toList());

        // Return the list of authorities to be used by Spring Security
        return returnValue;
    }
}
```

---

### 📌 **Summary**

| **Aspect**          | **Description**                                                                                       |
| ------------------- | ----------------------------------------------------------------------------------------------------- |
| **Class Purpose**   | Converts `realm_access.roles` from a Keycloak JWT into Spring Security `GrantedAuthority` list        |
| **Key Input**       | JWT token from Keycloak                                                                               |
| **Key Output**      | `Collection<GrantedAuthority>` usable by Spring Security                                              |
| **Role Extraction** | Reads roles from `realm_access.roles` claim                                                           |
| **Prefixing**       | Adds `"ROLE_"` prefix to match Spring Security expectations (`hasRole("XYZ")` looks for `"ROLE_XYZ"`) |
| **Return Behavior** | Returns empty list if no roles are present                                                            |

---

### 🔐 Example JWT Payload

```json
{
  "realm_access": {
    "roles": ["ACCOUNTS", "CARDS"]
  }
}
```

### 🔄 Output from Converter

```java
[
  new SimpleGrantedAuthority("ROLE_ACCOUNTS"),
  new SimpleGrantedAuthority("ROLE_CARDS")
]
```

package com.eazybytes.gatewayserver.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.convert.converter.Converter;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AbstractAuthenticationToken;
import org.springframework.security.config.annotation.web.reactive.EnableWebFluxSecurity;
import org.springframework.security.config.web.server.ServerHttpSecurity;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.security.oauth2.server.resource.authentication.JwtAuthenticationConverter;
import org.springframework.security.oauth2.server.resource.authentication.ReactiveJwtAuthenticationConverterAdapter;
import org.springframework.security.web.server.SecurityWebFilterChain;
import reactor.core.publisher.Mono;

@Configuration
@EnableWebFluxSecurity // Enables Spring Security for reactive WebFlux apps (like Spring Cloud Gateway)
public class SecurityConfig {

    @Bean
    public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity serverHttpSecurity) {
        serverHttpSecurity
            .authorizeExchange(exchanges -> exchanges
                .pathMatchers(HttpMethod.GET).permitAll() // Allow all GET requests (public access)
                .pathMatchers("/eazybank/accounts/**").hasRole("ACCOUNTS") // Needs ROLE_ACCOUNTS
                .pathMatchers("/eazybank/cards/**").hasRole("CARDS")       // Needs ROLE_CARDS
                .pathMatchers("/eazybank/loans/**").hasRole("LOANS")       // Needs ROLE_LOANS
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt.jwtAuthenticationConverter(grantedAuthoritiesExtractor())) // Convert roles from JWT
            );

        serverHttpSecurity.csrf(csrf -> csrf.disable()); // Disable CSRF (not needed for stateless APIs)
        return serverHttpSecurity.build();
    }

    /**
     * This method returns a reactive JWT authentication converter.
     * It uses KeycloakRoleConverter to extract roles from the JWT.
     */
    private Converter<Jwt, Mono<AbstractAuthenticationToken>> grantedAuthoritiesExtractor() {
        JwtAuthenticationConverter jwtAuthenticationConverter = new JwtAuthenticationConverter();
        jwtAuthenticationConverter.setJwtGrantedAuthoritiesConverter(new KeycloakRoleConverter());

        // Adapt to reactive Mono-based model (required for WebFlux)
        return new ReactiveJwtAuthenticationConverterAdapter(jwtAuthenticationConverter);
    }
}
```
[User] → Logs in via Keycloak login page → Gets redirected with Auth Code
[Gateway Client] → Exchanges Auth Code for Access Token (JWT)
[JWT] → Contains realm_access.roles: ["ACCOUNTS", "CARDS", "LOANS"]

[Gateway SecurityConfig]
→ Uses KeycloakRoleConverter
→ Converts roles to: ROLE_ACCOUNTS, ROLE_CARDS, etc.
→ Allows/denies access to microservices based on these roles
```


Here are the **complete setup steps to configure Keycloak** to work with a Spring Cloud Gateway (or any OAuth2-secured microservices architecture), including **Realm, Client, Roles, and Users**:

---

## 🧰 Step-by-Step Keycloak Setup

### ✅ 1. **Start Keycloak Server**

If using Docker:

```bash
docker run -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:24.0.1 start-dev
```

Then access:
👉 [http://localhost:8080](http://localhost:8080)
Login: `admin` / `admin`

---

### 🏢 2. **Create a Realm**

A **Realm** is a space where all your users, roles, and clients live.

1. Go to **Realm Selector** → click **Create Realm**.
2. Enter:

   * **Realm Name**: `eazybank`
   * Click **Create**

---

### 👤 3. **Create a User**

1. In the left menu: **Users** → click **Create New User**
2. Enter:

   * **Username**: `john`
   * **Email**: `john@example.com` *(optional)*
   * **First Name**: `John`
   * **Enabled**: ✅ Yes
   * Click **Create**

#### 🔐 Set Password:

1. Go to **Credentials** tab
2. Enter:

   * Password: `12345`
   * Confirm Password: `12345`
3. Turn off "Temporary" → Save

---

### 🎭 4. **Create Roles**

1. Go to **Roles** → Click **Add Role**
2. Create roles:

   * `ACCOUNTS`
   * `CARDS`
   * `LOANS`

Repeat for each role.

---

### 👥 5. **Assign Roles to User**

1. Go to **Users** → click on `john`
2. Go to **Role Mappings**
3. In **Available Roles**, select `ACCOUNTS`, `CARDS`, `LOANS`
4. Click **Add Selected**

---

### 📦 6. **Create a Client (App)**

1. Go to **Clients** → click **Create Client**
2. Fill in:

   * **Client ID**: `gateway-client`
   * **Client Protocol**: `openid-connect`
   * Click **Next**
3. Set:

   * **Root URL**: `http://localhost:8080`
   * **Valid Redirect URIs**: `http://localhost:8080/login/oauth2/code/*`
   * **Web Origins**: `+`
   * Click **Save**

#### 🔑 Enable Authorization Code Flow:

1. In the client settings:

   * **Access Type**: `confidential`
   * Enable:

     * ✅ Standard Flow (Authorization Code)
     * ✅ Direct Access Grants (for password flow testing)
   * **Client Authentication**: Enabled (you’ll get `client_secret`)
2. Save

---

### 🔑 7. **Get Client Secret**

1. Go to **Clients** → click `gateway-client`
2. Click on **Credentials** tab
3. Copy the `Client Secret` for use in your Spring configuration

---

### 🌍 8. **Get OpenID Configuration URL**

This is needed by Spring Security to validate tokens automatically.

```http
http://localhost:8080/realms/eazybank/.well-known/openid-configuration
```

Spring Boot reads this to get:

* Token issuer
* Public key URL
* Auth endpoints, etc.

---

## ✅ Now You're Ready to Secure Spring Cloud Gateway

You'll use these details in `application.yml` or `application.properties` for your **gateway**:

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8080/realms/eazybank
      client:
        registration:
          keycloak:
            client-id: gateway-client
            client-secret: YOUR_CLIENT_SECRET
            scope: openid
            redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"
        provider:
          keycloak:
            issuer-uri: http://localhost:8080/realms/eazybank
```

---

## ✅ Summary

| Item       | Value / Example                         |
| ---------- | --------------------------------------- |
| Realm      | `eazybank`                              |
| Client ID  | `gateway-client`                        |
| Roles      | `ACCOUNTS`, `CARDS`, `LOANS`            |
| User       | `john` / `12345`                        |
| Issuer URI | `http://localhost:8080/realms/eazybank` |

---
