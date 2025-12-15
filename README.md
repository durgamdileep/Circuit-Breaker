## 🛑 Non-Transient Failures

- Non-transient failures are `persistent or serious failures` that are `unlikely to resolve themselves automatically`. They usually `require manual intervention or corrective action`. Examples:

- 🛢️ `Database failures or crashes` – the database is down or corrupted.
- ⚡ `Service overload` – sending a huge number of requests to a single service (e.g., DoS or excessive traffic).
- 💾 `Resource exhaustion` – memory, CPU, disk full, or network failure that is not temporary.

---

## 🌊 Cascading Failures

- When `one service fails` in a microservices architecture, it can `cause other dependent services to fail`. This is called a cascading failure.

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
