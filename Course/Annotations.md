# Java Application - XML Configuration and Annotations

## Component Scanning

### Auto Configuration
- The beans are added to the Bean Factory using annotations.
- **Powerful Annotation:** `@EnableAutoConfiguration`
  - In Auto Configuration, the **IOC (Inversion of Control)** is driven by the **Classpath dependencies** (JAR files included).

### Spring Boot Configuration
- Spring provides various **properties** that can be modified.
- Developers can define and configure properties such as:
  - **AuthenticationManager**
  - **AuthenticationProvider**

## Proxies Pattern
### Need more information on:
- How Beans in the `BeanFactory` are proxied?
- How annotations drive proxies?

---

## Additional Industry-Relevant Points for Review
1. **Difference Between XML and Annotation-Based Configuration**
   - XML configuration provides a **declarative** approach, whereas annotations allow for **concise and maintainable** code.
   - Industry prefers annotation-based configuration due to its **reduced boilerplate code** and ease of use.

2. **Understanding IOC and Dependency Injection**
   - **IOC (Inversion of Control):** Framework takes control of object creation and management.
   - **DI (Dependency Injection):** Objects are provided their dependencies instead of creating them manually.

3. **Spring Boot Auto Configuration Mechanism**
   - How `@EnableAutoConfiguration` detects dependencies and configures beans accordingly.
   - The role of `spring.factories` in auto-configuration.

4. **Proxy Pattern in Spring**
   - Used for **AOP (Aspect-Oriented Programming)** and **lazy loading**.
   - Spring creates a proxy for beans to **intercept method calls** (e.g., for transaction management, security, etc.).
   - Understanding **JDK Dynamic Proxies** vs. **CGLIB Proxies**.

5. **Security Configuration in Spring Boot**
   - Authentication mechanisms: Basic Auth, JWT, OAuth2.
   - Configuring `AuthenticationManager` and `AuthenticationProvider`.
   - Implementing **custom security filters**.

6. **Best Practices in Bean Management**
   - `@Component`, `@Service`, `@Repository`, and `@Controller` usage.
   - When to use **Prototype** vs. **Singleton** beans.

7. **Performance Considerations in Spring Applications**
   - Avoid unnecessary autowiring to improve startup time.
   - Use **lazy initialization** where applicable.
   - Optimize database queries with **Spring Data JPA**.

By reviewing these points, you will have a solid understanding of Spring Boot configurations and industry best practices.

