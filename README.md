## 🛑 Non-Transient Failures

- Non-transient failures are `persistent or serious failures` that are `unlikely to resolve themselves automatically`. They usually `require manual intervention or corrective action`. Examples:

- 🛢️ `Database failures or crashes` – the database is down or corrupted.
- ⚡ `Service overload` – sending a huge number of requests to a single service (e.g., DoS or excessive traffic).
- 💾 `Resource exhaustion` – memory, CPU, disk full, or network failure that is not temporary.

---

## 🌊 Cascading Failures

- When `one service fails due non-transient failure` in a microservices architecture, it can `cause other dependent services to fail`. This is called a cascading failure.

### 🔍 Example scenario:

1. Service A depends on Service B, and Service B depends on Service C.
2. Service C crashes (non-transient failure).
3. Service B keeps trying to call Service C → it starts failing or slowing down.
4. Service A calls Service B → it starts failing as well.

**⚠️ Result: Multiple services are affected → cascading failure.**

---

## 🛠️ How to Prevent Cascading Failures

- 🔌 `Circuit Breaker Pattern` – stops calls to failing services temporarily.
- ⏱️ `Timeouts` – avoid waiting indefinitely for a response.
- 🔄 `Retries with backoff` – retry intelligently without overwhelming the service.
- 🛡️ `Bulkheads` – isolate parts of the system so one failure doesn’t spread.

---

# 🔧 Circuit Breaker

- A circuit breaker is a design pattern used in microservices to `prevent cascading failures by stopping calls to a failing service temporarily` and `improve system resilience`. 
- 🌐 Microservices often communicate over networks (like HTTP or gRPC), and sometimes a service might be slow, unavailable, or failing. If one service keeps trying to call a failing service, it can overload it and cause system-wide outages. The circuit breaker helps prevent that.

---

## ⚡ States of a Circuit Breaker

A circuit breaker typically has three states:

- ✅ **Closed (default)**  
   - Normal operation. Calls to the service are allowed.  
   - The circuit breaker monitors for failures.

- ⛔ **Open**  
   - Triggered when failures exceed a threshold (e.g., 5 failed calls in a row).  
   - Calls are immediately rejected without trying the service.  
   - Prevents overloading the failing service.

- 🟡 **Half-Open**  
   - After a timeout, the circuit breaker allows a limited number of test requests.  
   - If they succeed, the circuit closes again.  
   - If they fail, the circuit opens again.

---

## 💡 Why Use a Circuit Breaker in Microservices?

- 🚫 `Prevent cascading failures`: One failing service doesn’t crash others.
- 🛠️ `Improve system resilience`: Keeps the system responsive even if some parts fail.
- ⚡ `Fail fast`: Clients get quick failure responses instead of waiting for timeouts.
- 📊 `Monitor health`: Can alert when a service is consistently failing.

---


## ⏱️ TimeLimiter

- ➡️ Used to limit the maximum execution time of a service call.

### 🎯 Purpose:
- ⛔ Stops slow responses  
- 🧵 Prevents threads from waiting too long  

- 👉 Commonly used with async methods (**CompletableFuture**).

``` java
 @TimeLimiter(name = "<name>")

```
---

## 🔁 Retry

- ➡️ Used to retry a failed request before marking it as a failure.
- `@Retry is activated` when `a method call fails with an exception`, before the Circuit Breaker decides to open (depending on order).
  - Retry retries the same request
  - It is mainly for `transient failures`

### 🎯 Purpose:
- 🩹 Handles temporary failures  
- 🔄 Retries failed calls automatically (with delay)  

``` java
   @Retry(name = "<name>")
```
---

## 🔄 Execution Flow with Circuit Breaker & Retry

### 1️⃣ Method executes first  
📨 The client request calls the method.  
🔒 Circuit Breaker is in **CLOSED** state → request is allowed.


### 2️⃣ Method fails  
❌ Failure is recorded by the Circuit Breaker.  
📈 Failure count increases.


### 3️⃣ 🔁 @Retry is triggered  
🔄 Retry automatically calls the same method again.

Each retry attempt:  
- 🆕 Is treated as a new call  
- 📈 Increases failure count if it fails again  


### 4️⃣ 🚨 Failure threshold exceeded  
📊 When failure rate crosses the configured threshold:  
🔓 Circuit Breaker moves to **OPEN** state


### 5️⃣ ⛔ Circuit Breaker in OPEN state  
❌ No more calls are allowed  
❌ Remaining retry attempts are **NOT executed**  
⚡ Calls fail immediately (fail-fast)


### 6️⃣ ⏳ After wait duration  
🔄 Circuit Breaker moves to **HALF-OPEN** state


### 7️⃣ 🧪 HALF-OPEN state  
🔢 Limited number of test calls are allowed  
🔁 Retry **IS allowed** for these test calls  

- ✅ If calls succeed → circuit closes  
- ❌ If calls fail → circuit opens again 

---

``` java

resilience4j:
  circuitbreaker:
    instances:

      userService:                         # Circuit breaker name
        registerHealthIndicator: true       # Exposes health status for this circuit breaker

        slidingWindowType: COUNT_BASED      # Failure calculation based on number of calls
        # slidingWindowType: TIME_BASED     # (Alternative) Failure calculation based on time

        slidingWindowSize: 10               # Number of calls (or time window size)
        minimumNumberOfCalls: 5             # Minimum calls before circuit breaker activates

        failureRateThreshold: 50            # % of failures to open the circuit

        waitDurationInOpenState: 10s         # Time circuit stays OPEN before retrying

        permittedNumberOfCallsInHalfOpenState: 3
        # Allowed test calls when circuit is HALF-OPEN

        automaticTransitionFromOpenToHalfOpenEnabled: true
        # Automatically moves from OPEN to HALF-OPEN

        event-consumer-buffer-size: 10       # Stores last 10 circuit breaker events

@CircuitBreaker(name = "userService", fallbackMethod = "fallbackMethod") // here userService is my circui breaker name we need add separate properties for each circuit breaker name
@Retry(name = "userService")
@TimeLimiter(name = "userService")

```

