# Spring Boot: Important Concepts and Commands

## 1. `@SpringBootApplication` Annotation
The `@SpringBootApplication` annotation is a convenience annotation that combines:
- `@Configuration` (Marks the class as a source of bean definitions)
- `@EnableAutoConfiguration` (Enables Spring Boot’s auto-configuration feature)
- `@ComponentScan` (Scans components in the package and sub-packages)

### **Main Method in a Spring Boot Application:**
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

## 2. Maven Commands
### **What does `mvn install` do?**
The `mvn install` command compiles the project, runs tests, and installs the package in the local Maven repository (`.m2/repository`), making it available for other projects.

### **Common Maven Commands:**
- `mvn clean` – Deletes `target` directory and cleans compiled files.
- `mvn package` – Compiles, runs tests, and packages the application into a JAR or WAR file.
- `mvn verify` – Runs integration tests.
- `mvn dependency:tree` – Displays the dependency hierarchy.

## 3. Setting Server Port and Other Configuration
In a real-world industry setup, server properties are configured in `application.properties` or `application.yml`.

### **Example (application.properties):**
```properties
server.port=8081
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret
```

### **Passing Parameters via Command Line:**
```sh
java -jar myapp.jar --server.port=9090
```

## 4. Multiple Instances and Tomcat Configuration
In modern Spring Boot applications, you can run multiple instances by configuring unique ports for each instance.

### **Example:**
```sh
java -jar myapp.jar --server.port=8081
java -jar myapp.jar --server.port=8082
```

In older environments using Tomcat, the setup required manual configuration in `server.xml`.

## 5. Converting JSON to Java Class in Spring Boot
If given a JSON file, we can use Jackson’s `ObjectMapper` to convert it into Java objects.

### **Example JSON:**
```json
{
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com"
}
```

### **Corresponding Java Class:**
```java
import com.fasterxml.jackson.databind.ObjectMapper;
import java.io.File;

public class JsonToJava {
    public static void main(String[] args) throws Exception {
        ObjectMapper objectMapper = new ObjectMapper();
        User user = objectMapper.readValue(new File("user.json"), User.class);
        System.out.println(user);
    }
}

class User {
    private int id;
    private String name;
    private String email;

    // Getters and Setters
}
```

## 6. `PropertyReferenceException`
This error occurs when Spring Data JPA cannot find a referenced property in an entity class.

### **Example Issue:**
If a repository method references a non-existent property:
```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByUserName(String name);  // If property is "username" but method says "userName"
}
```

### **Solution:**
Ensure the property name matches exactly as defined in the entity:
```java
@Column(name="username")
private String username;
```

---

### **Spring Boot Fresher Interview Questions & Answers**  

Here’s a structured list of interview questions, categorized based on difficulty and real-world relevance.

---

## **1. `@SpringBootApplication` Annotation**  
### **Basic Questions**  
**Q1: What is `@SpringBootApplication`, and why is it used?**  
📌 _Expected Answer:_ It is a convenience annotation that combines `@Configuration`, `@EnableAutoConfiguration`, and `@ComponentScan`, making Spring Boot configuration simpler.  

**Q2: Can we use `@SpringBootApplication` without `main()`?**  
📌 _Expected Answer:_ No, because the `main()` method is required to start the Spring Boot application using `SpringApplication.run()`.  

### **Advanced Questions**  
**Q3: How does `@SpringBootApplication` enable component scanning?**  
📌 _Expected Answer:_ It enables component scanning for the package it is declared in and all sub-packages, avoiding manual configuration of `@ComponentScan`.  

**Q4: How can we exclude auto-configuration in `@SpringBootApplication`?**  
📌 _Expected Answer:_ We can use `@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})` to prevent unnecessary auto-configurations.  

---

## **2. Maven Commands (`mvn install` and Others)**  
### **Basic Questions**  
**Q5: What does `mvn install` do in a Spring Boot project?**  
📌 _Expected Answer:_ It compiles, tests, and installs the built package (JAR/WAR) into the local Maven repository for use in other projects.  

**Q6: What is the difference between `mvn clean` and `mvn install`?**  
📌 _Expected Answer:_  
- `mvn clean` removes compiled files (`target/` directory).  
- `mvn install` compiles and installs the package in the local repository.  

### **Advanced Questions**  
**Q7: What is the difference between `mvn package` and `mvn install`?**  
📌 _Expected Answer:_  
- `mvn package` only creates the JAR/WAR file.  
- `mvn install` creates and installs it in the local repository for reuse.  

**Q8: How do we skip running tests when executing `mvn install`?**  
📌 _Expected Answer:_ Use `mvn install -DskipTests` to avoid running unit tests.  

---

## **3. Configuring Server Port in Spring Boot**  
### **Basic Questions**  
**Q9: How do you change the default server port in a Spring Boot application?**  
📌 _Expected Answer:_ Modify `application.properties` or `application.yml`:  
```properties
server.port=8081
```  

**Q10: How do you run a Spring Boot app with a different port using the command line?**  
📌 _Expected Answer:_ Use `java -jar app.jar --server.port=9090`  

### **Advanced Questions**  
**Q11: What happens if you configure multiple server ports in Spring Boot?**  
📌 _Expected Answer:_ If multiple values are set, Spring Boot uses the one with the highest precedence (Command-line > `application.yml` > Default 8080).  

**Q12: How can you run multiple instances of a Spring Boot app on different ports?**  
📌 _Expected Answer:_  
- Use different profiles (`application-dev.properties`, `application-test.properties`).  
- Set environment variables dynamically:  
  ```bash
  java -jar app.jar --server.port=8081
  java -jar app.jar --server.port=8082
  ```  

---

## **4. Running Multiple Instances & Tomcat Considerations**  
### **Basic Questions**  
**Q13: Can we run multiple instances of a Spring Boot application on the same server?**  
📌 _Expected Answer:_ Yes, by configuring different ports for each instance.  

**Q14: How does Spring Boot use an embedded Tomcat server?**  
📌 _Expected Answer:_ By default, Spring Boot runs with an embedded Tomcat server unless configured otherwise.  

### **Advanced Questions**  
**Q15: How do we disable the embedded Tomcat server in Spring Boot?**  
📌 _Expected Answer:_ Add the following in `application.properties`:  
```properties
spring.main.web-application-type=none
```  
Or exclude `spring-boot-starter-web` from dependencies.  

**Q16: How can you configure a Spring Boot app to run on an external Tomcat server instead of the embedded one?**  
📌 _Expected Answer:_  
- Package the app as a WAR (`mvn package`).  
- Deploy it to an external Tomcat server.  

---

## **5. Handling JSON Data in Spring Boot**  
### **Basic Questions**  
**Q17: How can you convert a JSON file into a Java object?**  
📌 _Expected Answer:_ Use Jackson’s `ObjectMapper`:  
```java
ObjectMapper objectMapper = new ObjectMapper();
User user = objectMapper.readValue(new File("user.json"), User.class);
```

**Q18: What is `@JsonProperty`, and why is it used?**  
📌 _Expected Answer:_ It maps JSON properties to Java fields when names don’t match.  

### **Advanced Questions**  
**Q19: How do you validate JSON input in a Spring Boot controller?**  
📌 _Expected Answer:_ Use `@Valid` and `@RequestBody` in the controller method:  
```java
@PostMapping("/users")
public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
    return ResponseEntity.ok(user);
}
```

**Q20: What is `@JsonIgnore`, and when should we use it?**  
📌 _Expected Answer:_ It prevents specific fields from being serialized in JSON.  

---

### **Bonus: Scenario-Based Questions**  
**Q21: How would you handle a scenario where a Spring Boot application fails to start due to a port conflict?**  
📌 _Expected Answer:_  
- Change the port in `application.properties`.  
- Kill the existing process using the port (`netstat -ano | findstr :8080`).  

**Q22: Suppose your Spring Boot app works fine locally but crashes on a production server. How do you debug the issue?**  
📌 _Expected Answer:_  
- Check logs (`logback.xml`, `application.log`).  
- Verify database connection settings.  
- Use `mvn clean install` to rebuild dependencies.  

---



---
1.  @SpringBootApplicaiotn  :  this  annotation make  about te Java  app to  tell me abou the  various  Setip in  the Project about  the  Main  Method (and  About the   QUestion tat   can   be  asksed ? )
2.  what is  the  Maven Commands  (mvn  intsall  waht  does  this  Do )
3.  How  to add he   things  in  the   command  about   that  in  the    Commands  about the   Server port and  all  are  done  n th e industry
4.  Main  part is  about  that   when  in the  pattern  we  can   have  the  Multiple instance  of that and   where  as in  the  pervious  env  using the   tomcat we need to  make about  the   things
5.   If i  give   you  the   Json  data n  the   spring  boot  or  he   Json  file  .  wreuet me     the  Java  Code  in  the   boot  to  compvert  that
6.    PropertyRefernceExpxtion
