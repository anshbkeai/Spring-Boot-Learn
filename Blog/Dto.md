## One of the most learing about that i leant i

in Dto never go isActive -> isIsActive in java comes about to that 
Here's a Markdown (`.md`) file that documents your issue and the resolution — **in a developer-level revision/learning format** — so you can revise, learn, and avoid such bugs in the future:

---

### 🧾 `organisation-creation-bug.md`


---

##  Context

While sending a `POST` request to create an organisation, the payload was:

```json
{
  "name": "Braclays",
  "isCustomAllowed": true,
  "customRoles": ["VIEWER", "EDITOR", "SUPERVISOR"],
  "customPermissions": ["READ_DATA", "WRITE_DATA", "EXPORT_REPORT"]
}
```

But in logs, this DTO showed:

```
isCustomAllowed=false
```

This caused the system to load **default roles/permissions**, even though custom values were passed.

---

## 🔍 Root Cause

### ✅ DTO Field Name

```java
private boolean isCustomAllowed;
```

> This is **valid**, but causes issues in serialization/deserialization.

###  What went wrong?

Jackson (used by Spring Boot for JSON mapping) follows **JavaBeans getter/setter conventions**.

For a primitive `boolean` field:

* It expects `isXyz()` for getter.
* But when the field is named `isXyz`, it **thinks** the actual field is `xyz`, not `isXyz`.

####  Problem:

* Field: `isCustomAllowed`
* Getter: `isCustomAllowed()`
* Jackson thought the property was: `customAllowed`
* So `isCustomAllowed` in the JSON was **not mapped**.

---

##  Fix

**Change field name** to drop the `is` prefix:

```java
private boolean customAllowed;
```

Let Lombok auto-generate:

* Getter: `isCustomAllowed()` ✅
* Setter: `setCustomAllowed(boolean)` ✅

This now aligns correctly with Jackson.

---

## 🔧 Fixed DTO

```java
@Data
public class OrganisationDto {

    private String name;
    private boolean customAllowed;  // ✅ Renamed

    @JsonInclude(JsonInclude.Include.NON_NULL)
    private List<String> customRoles;

    @JsonInclude(JsonInclude.Include.NON_NULL)
    private List<String> customPermissions;
}
```

---

## 📦 Tips for Production-Ready DTOs

1. ✅ Avoid starting boolean field names with `is...`.

   * Instead: use `enabled`, `allowed`, `active`, etc.

2. ✅ Always **log DTO values** if logic depends on them.

3. ✅ Use `@JsonProperty` if you must keep awkward names:

   ```java
   @JsonProperty("isCustomAllowed")
   private boolean isCustomAllowed;
   ```

4. ✅ Prefer wrapper `Boolean` over primitive `boolean` when `null` might be valid.

---

## 🧪 Testing Advice

* Add a unit test that deserializes JSON → DTO and logs fields.
* Validate:

  ```json
  { "customAllowed": true } → dto.isCustomAllowed() == true
  ```

---

## 📘 Related Learning

* [Jackson Boolean Mapping Pitfalls](https://stackoverflow.com/questions/51689081/jackson-does-not-set-boolean-field-if-name-starts-with-is)
* [Lombok and Jackson Integration Guide](https://projectlombok.org/features/GetterSetter)
* [Effective Java – Naming Conventions for Booleans](https://www.oracle.com/technical-resources/articles/java/namingconventions.html)


