**WEB-SOCKETS**
```
SOCKETS  ->  WEB  SOCKETS  (FULL  DUPLEX ,  Pub/sub  type ,  )
```
Client Subscibe  the  topic and if  any repsoen  on  he   topic  will be  Breoadcasted  to  the   Client  about  that   , that  is  Simple  way  that  it  works 
In  spring boot  ,  MessageBroker,  Stomp  endpoint  ,  
```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig  implements  WebSocketMessageBrokerConfigurer{

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws").withSockJS(); // WebSocket endpoint
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/topic"); // Enables in-memory broker
        registry.setApplicationDestinationPrefixes("/app");
    }

}
```
**The  Most  Loving part  is  to  just  to  visit  about  the  Annotation  and  leanr  about  the Class   that  thye  implement  and  made  about  the   things  ready  for  that  part** 
```
Lifecycle of WebSocket
Handshake => Starts with a regular HTTP request
If accepted, it upgrades to WebSocket (101 Switching Protocols)
Open Connection => Both sides can send/receive freely
Messaging => JSON/text/binary data
Close Connection => Either side can close it

Under the  hood  Still  usess abut  the   TCP  connection  
```
---
That  was  the   part about  understadning  of  the   web  sockets  
Now  improving  more  forward  to  the   security  part  about  the   Jwt  n the   simple  terms   
1.  Authication  =>  Normal about  the  Http  Auth we  have  in  here  . and   says  aboout  that 

So  in  the  web scoket  learnin  about  that  we  get  to  analysis  about that    
I am  adding ghe   Jwt Auth  and for  that  i  need  Signup  and   Login  part  that  i  will  Maintain  here  and more  over  i  have  to  Create about  that  we  page  and make  about  the   request  an  then  only we  can  be  added  to  that

```java
@Configuration
@EnableWebSocketSecurity
public class WebSocketSecurityConfig {

    @Bean
    public AuthorizationManager<Message<?>> messageAuthorizationManager() {
        return new AuthorizationManager<Message<?>>() {
            @Override
            public AuthorizationDecision check(Supplier<Authentication> authentication, Message<?> message) {
                // You can parse headers or destination to make decisions
                return new AuthorizationDecision(authentication.get().isAuthenticated());
            }
        };
    }
}
```
That  we  are  USING  TO  handle  the  Message baout  th e  STOMP  t hat  the   user is  the   Authcatied   
1.  What  we ahve to  do is  about that impletn te  HandShake  incpetor  as  that  is    the  important
   
```java
   @Configuration
@EnableWebSocketSecurity
public class WebSocketSecurityConfig {

    @Bean
    public SecurityMessageMatcherRegistry authorizationManager(MessageAuthorizationRegistry messages) {
        messages
            .nullDestMatcher().authenticated()     // CONNECT frame
            .simpDestMatchers("/topic/**").permitAll() // public topic
            .simpDestMatchers("/app/**").authenticated(); // only logged-in users
        return messages;
    }
}
```

```java
public class JwtHandshakeInterceptor implements HandshakeInterceptor {

    @Override
    public boolean beforeHandshake(
        ServerHttpRequest request,
        ServerHttpResponse response,
        WebSocketHandler wsHandler,
        Map<String, Object> attributes
    ) throws Exception {
        // Extract token from query param or header
        // Validate JWT and set user info in `attributes` if valid
        return true;
    }

    @Override
    public void afterHandshake(...) { }
}
```
```
registry.addEndpoint("/ws")
        .setAllowedOrigins("*")
        .addInterceptors(new JwtHandshakeInterceptor());
```


**MAIN  TABLE  DIFFERENCE**
| Concept                     | REST Security                        | WebSocket Security                                   |
|----------------------------|--------------------------------------|-----------------------------------------------------|
| Request                     | HTTP Request (GET/POST/etc.)        | STOMP Message over WebSocket                        |
| Authentication              | `UsernamePasswordAuthenticationToken`, JWT, etc. | WebSocket handshake authentication                 |
| Authorization               | Secured by `antMatchers()`          | Secured by `simpDestMatchers()` or message matchers |
| Filter Chain                | `SecurityFilterChain`               | `AuthorizationManager<Message<?>>` for messaging    |
| Endpoint access             | `/api/**`                           | `/app/**`, `/topic/**`, `/queue/**`, `/user/**`     |
| Stateless sessions (JWT)    | Common with `SessionCreationPolicy.STATELESS` | Often tied to WebSocket session                    |


----


## 🔐 WebSocket Security with JWT in Spring Boot – Industry-Aligned Summary

### 📌 Objective

Secure a WebSocket endpoint (`/ws`) using JWT, ensuring only authenticated users can connect and access specific messaging channels.

---

## ✅ Components Used

### 1. **HandshakeInterceptor (Authentication)**

* Intercepts the initial HTTP handshake (`/ws/info`, `/ws/123/xyz`) before WebSocket upgrade.
* Extracts JWT token from the **query parameter** (`?token=...`).
* Validates the token using `JwtTokenProvider`.
* Creates an `Authentication` object and saves it to the WebSocket session.

✅ **Why this is needed**: WebSockets do not support headers after the initial upgrade, so tokens must be passed here.

---

### 2. **ChannelInterceptor (STOMP Authentication Binding)**

* Runs on every STOMP frame (`CONNECT`, `SUBSCRIBE`, `SEND`).
* Retrieves the `Authentication` from session attributes.
* Injects it into `SimpMessageHeaderAccessor` using `setUser()`.

✅ **Why this is needed**: Without this, Spring won’t recognize the user’s identity during message sending/subscribing.

---

### 3. **AuthorizationManager (Message-Level Security)**

* Introduced in Spring Security 6.
* Used in `MessageSecurityMetadataSourceRegistry` to secure STOMP endpoints.

**Example**:

```java
authorizeDestination("/app/**").hasRole("USER")
authorizeDestination("/topic/admin/**").hasRole("ADMIN")
```

✅ **Why this is needed**: Enforces access control on who can send or subscribe to specific channels.

---

## ⚙️ Architecture Flow

1. **Client connects to `/ws?token=JWT`**
2. `HandshakeInterceptor` validates token → stores Authentication in session.
3. `ChannelInterceptor` injects the authenticated user into message context.
4. `AuthorizationManager` checks user roles on every message destination.
5. Messages are processed or rejected accordingly.

---

## 🧠 Key Concepts

| Term                   | Purpose                                                              |
| ---------------------- | -------------------------------------------------------------------- |
| `HandshakeInterceptor` | Authenticates JWT at initial WebSocket upgrade                       |
| `ChannelInterceptor`   | Binds user to STOMP message lifecycle (setUser)                      |
| `AuthorizationManager` | Authorizes access to STOMP endpoints by roles                        |
| Stateless REST         | JWT stored client-side, no session                                   |
| Stateful WS            | WebSocket maintains session per connection (uses STOMP user context) |

---

## 🔒 Industry Standards & Best Practices

| Practice                                                    | Industry Valid?    | Notes                                       |
| ----------------------------------------------------------- | ------------------ | ------------------------------------------- |
| JWT token in query param for `/ws`                          | ✅ Yes (with HTTPS) | Required workaround; secure via HTTPS       |
| Validating token in `HandshakeInterceptor`                  | ✅ Yes              | Equivalent to stateless filter chain for WS |
| Using `ChannelInterceptor` to inject principal              | ✅ Yes              | Enables message-level security              |
| Role-based message authorization via `AuthorizationManager` | ✅ Yes              | Recommended by Spring Security team         |

---

## ⚠️ Common Mistakes to Avoid

* ❌ Relying on header-only auth in WebSockets (not supported post-handshake)
* ❌ Skipping `ChannelInterceptor` → message user becomes `null`
* ❌ Not securing `/topic/**` or `/app/**` → allows open broadcast

---

## ✅ Summary

You implemented a production-grade WebSocket security model using:

* JWT + `HandshakeInterceptor` for stateless authentication
* `ChannelInterceptor` for session binding
* `AuthorizationManager` for fine-grained access control

This pattern is aligned with **modern Spring Boot and enterprise-grade security practices**.

---
