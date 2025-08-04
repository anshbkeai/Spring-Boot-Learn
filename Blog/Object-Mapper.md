
# 📄 **Partial Updates with ObjectMapper in Java (Revision Sheet)**

---

##  WHAT — Core Concept

> Use `ObjectMapper.readerForUpdating(existingObject)` to **safely merge incoming JSON into an existing Java object** during a `PATCH` operation.

* Preserves existing values
* Updates only the fields provided in the JSON
* Prevents data loss from `null` overwrites

---

##  WHY — Why Is This Needed?

| Problem                                | Without `readerForUpdating`            |
| -------------------------------------- | -------------------------------------- |
| Frontend sends only part of the object | You override missing fields with null  |
| Nested objects get updated partially   | Entire nested object gets reset        |
| You want to avoid over-fetching data   | You're forced to resend whole payloads |
| Need better control in PATCH           | Limited by Spring’s default binding    |

---

##  WHO — When Should *You* Use It?

 Use this if:

* Your app supports `PATCH` APIs
* You allow **selective updates**
* You work with **nested data** (`address`, `profile`)
* You get JSON from **external clients** or **mobile apps**

 Skip if:

* You only use full `PUT` updates
* All fields are required always
* Your data model is flat/simple
* You own both frontend + backend and enforce structure

---

## ⚙️ HOW — Integration Steps

### 1. Replace `@RequestBody Entity` with `@RequestBody JsonNode`

```java
@PatchMapping("/users/{id}")
public ResponseEntity<User> patchUser(@PathVariable Long id, @RequestBody JsonNode patch) {
    User existing = repo.findById(id).orElseThrow();
    objectMapper.readerForUpdating(existing).readValue(patch);
    repo.save(existing);
}
```

### 2. Optional: Use a service layer for merging

```java
@Component
public class JsonMergeService {
    public <T> T merge(JsonNode json, T existing) {
        return objectMapper.readerForUpdating(existing).readValue(json);
    }
}
```

### 3. Handle nested objects and optional validation manually

---

## 🚀 BENEFITS — How It Improves Your System

| Improvement           | Description                                        |
| --------------------- | -------------------------------------------------- |
| 🔐 Safer APIs         | No accidental null fields or data loss             |
| 📦 Smaller payloads   | Send only what's needed — ideal for mobile clients |
| 🔁 Flexible patching  | Support nested/optional fields gracefully          |
| ⚙️ Cleaner logic      | Keep existing values; just update the changes      |
| 📈 Easier debugging   | Merge logic is controlled and traceable            |
| 🧪 Better testability | You can test merge logic independently             |

---

## 🧠 KEY TECH POINTS TO REMEMBER

* ✅ `ObjectMapper.readerForUpdating(existingObject).readValue(json)`
* ⚠️ Works best with mutable objects (setters present)
* ❌ Does not trigger `@Valid` or bean validation by default
* ✅ Can handle nested objects properly
* 💥 Fails if JSON structure is mismatched (add tests!)
* 🔍 Ideal for PATCH, not for POST/PUT

---

## 🔧 BONUS: Common Enhancements

| Enhancement                                   | Use Case                                                |
| --------------------------------------------- | ------------------------------------------------------- |
| `@JsonIgnoreProperties(ignoreUnknown = true)` | Ignore unexpected fields                                |
| `@JsonFormat(pattern = "dd-MM-yyyy")`         | Handle custom date formats                              |
| `@JsonAnySetter`                              | Collect extra unknown fields                            |
| `Custom Deserializers`                        | Handle renamed/legacy fields (`zipcode` → `postalCode`) |
| `Audit Log`                                   | Compare `before` and `after` using `JsonNode`           |

---

## ✅ QUICK CHECKLIST FOR PATCH API

| ✅                                                         | Task |
| --------------------------------------------------------- | ---- |
| 🔲 Do you support PATCH and partial updates?              |      |
| 🔲 Are nested objects merged, not replaced?               |      |
| 🔲 Is null overwriting prevented?                         |      |
| 🔲 Are edge cases tested?                                 |      |
| 🔲 Are updates validated post-merge?                      |      |
| 🔲 Is merge logic shared (service/utility class)?         |      |
| 🔲 Do you log merged data or field diffs for audit/debug? |      |

---

## 📚 SAVE & REVISE QUICKLY

| Section         | Use When                                 |
| --------------- | ---------------------------------------- |
| **WHAT**        | Remember the purpose                     |
| **WHY**         | Explain in interviews/designs            |
| **WHO**         | Decide if your API needs it              |
| **HOW**         | Copy-paste for real implementation       |
| **BENEFITS**    | Justify to stakeholders/product managers |
| **CHECKLIST**   | Code review / PR guide                   |
| **TECH POINTS** | Interview + deep-dive prep               |

---
