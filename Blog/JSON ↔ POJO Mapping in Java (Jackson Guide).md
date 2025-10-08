

# 🧩 JSON ↔ POJO Mapping in Java (Jackson Guide)

### 📘 Overview

In the Java ecosystem, **JSON serialization and deserialization** is a core skill for every backend developer.
Unlike JavaScript (where JSON is native), Java enforces **strict typing**, so you must define **POJOs (Plain Old Java Objects)** that represent your JSON data structure.

This document covers:

* Conceptual understanding
* How mapping works
* Common annotations
* Real-world examples
* Best practices for maintainable and robust models

---

## 🚀 1. What Is JSON Mapping?

**Serialization:**
Convert a Java object → JSON string

```java
String json = new ObjectMapper().writeValueAsString(object);
```

**Deserialization:**
Convert a JSON string → Java object

```java
TransactionEvent event = new ObjectMapper().readValue(json, TransactionEvent.class);
```

Jackson automatically matches **JSON keys** to **Java fields** by name.
If names differ, annotations help bridge the gap.

---

## 🧱 2. Example JSON → Java Class Structure

### Input JSON:

```json
{
  "paymentInfo": {
    "instructionId": "TXN-ID-ABC-123",
    "instructedAmount": {
      "currency": "INR",
      "amount": 140000.00
    }
  },
  "debtor": {
    "name": "Ramesh Kumar",
    "identification": "CUST12345"
  },
  "creditor": {
    "name": "Pune Institute of Technology",
    "account": {
      "id": "pune.institute@icicibank"
    }
  },
  "remittanceInfo": {
    "unstructured": "Payment for Semester 5 Fees, Student ID 9876"
  }
}
```

### Java Model:

```java
public class TransactionEvent {
    private PaymentInfo paymentInfo;
    private Debtor debtor;
    private Creditor creditor;
    private RemittanceInfo remittanceInfo;
}

public class PaymentInfo {
    private String instructionId;
    private InstructedAmount instructedAmount;
}

public class InstructedAmount {
    private String currency;
    private double amount;
}

public class Debtor {
    private String name;
    private String identification;
}

public class Creditor {
    private String name;
    private Account account;
}

public class Account {
    private String id;
}

public class RemittanceInfo {
    private String unstructured;
}
```

---

## ⚙️ 3. Core Jackson Annotations

| Annotation                                     | Description                                                    | Example                                               |
| ---------------------------------------------- | -------------------------------------------------------------- | ----------------------------------------------------- |
| `@JsonProperty("json_name")`                   | Maps JSON field to Java field                                  | `@JsonProperty("full_name") private String fullName;` |
| `@JsonIgnoreProperties(ignoreUnknown = true)`  | Ignores extra JSON fields not present in your class            | `@JsonIgnoreProperties(ignoreUnknown = true)`         |
| `@JsonInclude(JsonInclude.Include.NON_NULL)`   | Excludes null fields from serialized JSON                      | `@JsonInclude(Include.NON_NULL)`                      |
| `@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")` | Defines how date/time fields are formatted                     | `@JsonFormat(pattern = "yyyy-MM-dd")`                 |
| `@JsonIgnore`                                  | Excludes a field completely from serialization/deserialization | `@JsonIgnore private String password;`                |
| `@JsonRootName("rootName")`                    | Handles wrapped root JSON objects                              | `@JsonRootName("transactionEvent")`                   |
| `@JsonCreator` / `@JsonValue`                  | Custom control for enum and value-based mapping                | Used for enums like currency or status                |

---

## 🧠 4. Mental Model — How to Think Like a Senior

When reading JSON, train your mind to **visualize the object hierarchy**:

* Every `{}` → new class
* Every `[]` → `List<T>`
* Every key → field
* Every nested object → nested POJO

Then ask yourself:

> “If this JSON changes tomorrow, how can I make my code resilient?”

That mindset helps you design flexible, future-proof POJOs.

---

## 🧩 5. Example: Using Jackson in a Kafka Consumer

```java
@KafkaListener(topics = "transaction-events")
public void consume(String message) throws JsonProcessingException {
    ObjectMapper mapper = new ObjectMapper();
    TransactionEvent event = mapper.readValue(message, TransactionEvent.class);

    log.info("Received payment: {}", event.getPaymentInfo().getInstructionId());
}
```

---

## 🧰 6. Common Pitfalls & Fixes

| Problem                    | Cause                          | Solution                                          |
| -------------------------- | ------------------------------ | ------------------------------------------------- |
| `Unrecognized field`       | JSON has new/extra field       | Add `@JsonIgnoreProperties(ignoreUnknown = true)` |
| `Cannot deserialize value` | Field type mismatch            | Check field type (e.g., number vs string)         |
| Missing data after mapping | Field names differ             | Use `@JsonProperty("actual_json_key")`            |
| Null pointer after mapping | Optional nested object missing | Use `@JsonInclude(Include.NON_NULL)`              |

---

## 🧠 7. Practice Exercises

1. Convert 5 different real-world JSON payloads (GitHub API, Stripe API, Weather API, etc.) into POJOs.
2. Change field names and fix mapping using annotations.
3. Add missing fields and handle versioning using `@JsonIgnoreProperties`.
4. Serialize objects back into JSON — verify format with Postman or JSONLint.

---

## 🧱 8. Design Principles (Senior-Level)

* ✅ Keep POJOs **flat and modular** — avoid giant single classes.
* ✅ Use clear class names (`Debtor`, `PaymentInfo`) to reflect domain intent.
* ✅ Always mark external-facing models with `@JsonIgnoreProperties(ignoreUnknown = true)`.
* ✅ Separate **internal models** (for persistence) from **DTOs** (for API I/O).
* ✅ Add unit tests to verify deserialization works when schema evolves.

---

## 🧭 9. Why This Skill Matters

| Reason            | Impact                                       |
| ----------------- | -------------------------------------------- |
| Strong typing     | Catch errors at compile-time, not at runtime |
| Stability         | Schema changes won’t break consumers         |
| Maintainability   | Easier debugging, clearer contracts          |
| Industry standard | Jackson is used in 90%+ Java microservices   |
| Integration ready | Essential for Spring Boot, Kafka, REST APIs  |

---

## 🏁 10. Recommended Learning Path

| Step | Resource                                                                                                    |
| ---- | ----------------------------------------------------------------------------------------------------------- |
| 1️⃣  | Read: [Jackson Annotations Wiki](https://github.com/FasterXML/jackson-annotations/wiki/Jackson-Annotations) |
| 2️⃣  | Try: [jsonschema2pojo.org](https://www.jsonschema2pojo.org/) — generate and study output                    |
| 3️⃣  | Practice mapping sample JSON APIs (GitHub, Stripe, OpenWeather)                                             |
| 4️⃣  | Learn advanced topics — custom deserializers, modules, mixins                                               |
| 5️⃣  | Use it in a real project — Spring Boot + Kafka + REST endpoints                                             |

