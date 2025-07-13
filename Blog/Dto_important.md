
Here’s a ready-to-use Markdown (`.md`) blog post that explains your use case in a clear and developer-friendly way:

---

# ✅ Handling Optional Fields in DTOs Using `@JsonInclude` in Spring Boot

When designing APIs with Spring Boot, a common challenge arises:
**"How do I handle optional fields in a request body without breaking the API?"**

Let’s take a real-world example — say you’re designing a system for organizations, and you want to allow **optional custom roles and permissions** **only when** a flag (`isCustomAllowed`) is `true`.
If it’s `false`, the system should still accept the request — even **without** these fields.

### 💡 The Problem

Traditionally, when you create a Data Transfer Object (DTO), all fields are expected to be passed. If they’re missing, it can lead to deserialization issues or unnecessary null checks.

You might face challenges like:

* Having to write boilerplate validation logic
* Handling `null` values unnecessarily
* Keeping the request body flexible and clean

---

## 🧱 Solution: Using `@JsonInclude(JsonInclude.Include.NON_NULL)`

Spring Boot uses **Jackson** for JSON deserialization. By using `@JsonInclude(JsonInclude.Include.NON_NULL)`, we tell Jackson:

> “If this field is `null`, just ignore it while serializing or deserializing.”

Here’s how we apply it:

```java
@Data
public class OrganisationDto {

    private String name;
    private boolean isCustomAllowed;

    @JsonInclude(JsonInclude.Include.NON_NULL)
    private List<String> customRoles;

    @JsonInclude(JsonInclude.Include.NON_NULL)
    private List<String> customPermissions;

}
```

### ✅ Benefits:

* Cleaner request bodies
* Optional logic handled at the backend
* No need to send empty arrays like `[]` if not required

---

## 📥 Example Request Body

This request is **totally valid** — even though we’re **not passing `customRoles` or `customPermissions`**.

```json
{
  "name": "Team Beta",
  "isCustomAllowed": false
}
```

And if the client **wants** to send them, they can:

```json
{
  "name": "Team Alpha",
  "isCustomAllowed": true,
  "customRoles": ["ADMIN", "EDITOR"],
  "customPermissions": ["CREATE", "DELETE"]
}
```

---

## 🧠 Backend Handling Logic

In your service/controller, you can simply check if these fields are null or not:

```java
if (dto.isCustomAllowed()) {
    if (dto.getCustomRoles() != null) {
        // Process custom roles
    }

    if (dto.getCustomPermissions() != null) {
        // Process custom permissions
    }
}
```

If `isCustomAllowed` is false, you can even **ignore** or **validate that these fields are not present**, depending on your business rules.

---

## 🚀 Conclusion

Using `@JsonInclude(JsonInclude.Include.NON_NULL)` is a clean, declarative way to make your DTOs flexible. It avoids forcing clients to send unnecessary or invalid data and makes your API **more robust and user-friendly**.

> 🧑‍💻 As a Spring Boot developer, embrace the power of Jackson annotations to keep your code elegant and API contracts clean.

---

Let me know if you'd like this formatted for Jekyll, Medium, or another blogging platform.

---

So I'm basically what we learned about our age like I was creating a data transfer object a D2 object yeah
So what happens in the spring boot when the traditionally which I was creating applications, the data transfer object
Fine with the API the data the post request the party request that we have in the object so which made it a very hard for me to in order to
Like for example, if we want some data to not to pass, maybe it's an exceptional way if it is it's OK it does not pass. It is also.
Based on a custom custom understanding of the field, the property that is required

So this is the issue you should have I was handling actually in the particular project where I have to pass the custom rules according to the if they're allowed or not
``` java
  @Data
public class OrgainsationDto {

    // therse will 
    private String name;
    private boolean isCustomAllowed;

    // if the then we. can ahve. them. else. not ways 
    @JsonInclude(JsonInclude.Include.NON_NULL) // this will make sure about that. if they are not passes still work for. the enviromnet
    private List<String> customRoles;
    
     @JsonInclude(JsonInclude.Include.NON_NULL)
    private List<String> customPermissions;

    
}
```
So far above that we have included, the Jason include the important fact that we have included there is help us to allow that
In the back and logic, we need to handle if they are provided then how to handle it if they are not provide provided than how to handle
So in my body request, I can also give a request. That is a successful request without passing this parameters.
```

{
  "name": "Team Beta",
  "isCustomAllowed": false
}

```
