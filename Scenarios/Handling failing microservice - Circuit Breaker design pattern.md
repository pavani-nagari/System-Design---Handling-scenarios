# Circuit Breaker Design Pattern  

The **Circuit Breaker Pattern** is used in distributed systems and microservices architectures to **handle failures gracefully** and **prevent cascading system crashes**. It acts like an electrical circuit breaker that stops the flow when there are too many failures.  

---

## 🔹 States of a Circuit Breaker  

1. **Closed State**  
   - Requests flow normally to the service.  
   - The Circuit Breaker monitors performance metrics such as error rate, response time, and timeouts.  
   - If the failure threshold is reached, it transitions to **Open**.  

2. **Open State**  
   - Requests are **not forwarded** to the failing service.  
   - Instead, an error message or a **fallback response** is returned immediately.  
   - This prevents overloading a service that is already failing.  
   - The circuit remains open for a **cool-off period** to allow recovery.  

3. **Half-Open State**  
   - After the cool-off period, the breaker transitions to **Half-Open**.  
   - A limited number of requests are sent to test if the service has recovered.  
   - If the requests succeed, the circuit transitions back to **Closed**.  
   - If failures continue, it moves back to **Open**.  

---

## 🔹 Example  

Imagine an **e-commerce application** calling a **Payment Service**.  

- **Closed State:**  
  The payment service is healthy, and requests (e.g., processing credit card transactions) succeed normally.  

- **Open State:**  
  Suddenly, the payment service goes down.  
  Instead of continuing to send requests that will fail and slow down the whole checkout process, the circuit breaker opens.  
  The application now returns a **“Payment service unavailable, please try again later”** message or falls back to an **alternative payment method**.  

- **Half-Open State:**  
  After some time, the system allows a few test requests to the payment service.  
  - If those succeed, the breaker closes, and the payment flow resumes.  
  - If they fail, it returns to open, giving more time for recovery.  

---

## 🔹 Benefits  

- Prevents cascading failures across microservices.  
- Provides a better user experience with graceful fallback responses.  
- Allows failing systems time to recover.  

---

✅ This pattern is commonly implemented in frameworks like **Netflix Hystrix**, **Resilience4j**, or **Spring Cloud Circuit Breaker** in Java ecosystems, and libraries like **Polly** in .NET.  
## References: https://www.baeldung.com/cs/microservices-circuit-breaker-pattern
