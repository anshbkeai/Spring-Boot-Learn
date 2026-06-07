# Configuring Two JWT Filters in a Single Spring Boot Application

## The Problem

In my Spring Boot application, I needed to handle two different types of API access:

- **Internal API** — dashboard and user-level routes, secured with JWT Token V1
- **External API** — third-party client access via `/api/**` routes, secured with JWT Token V2 (OAuth2 client credentials style)

The goal was simple: two different JWT filters, each responsible for its own set of routes.

---

## What I Was Doing Before (Single JWT — The Easy Case)

In a standard monolith Spring Boot application with a single JWT, the setup is straightforward:

```java
@Component
public class JwtFilter extends OncePerRequestFilter {
    // filter logic
}
```

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
        .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
    return http.build();
}
```

This works perfectly for a single JWT because:

- One filter, one security chain
- Spring manages the filter as a bean
- No routing conflicts

You never have to think about it. Spring Boot handles everything.

---

## Where It Broke Down — Adding a Second JWT Filter

When I tried to add a second JWT filter for the external API routes, I used `@Order` on my security filter chains:

```java
@Bean
@Order(1)
public SecurityFilterChain externalApiChain(HttpSecurity http) throws Exception {
    http
        .securityMatcher("/api/**")
        .addFilterBefore(jwtFilterV1, UsernamePasswordAuthenticationFilter.class);
    return http.build();
}

@Bean
@Order(2)
public SecurityFilterChain internalChain(HttpSecurity http) throws Exception {
    http
        .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
    return http.build();
}
```

Both filters were declared as `@Component`. This is where the friction started.

---

## The Root Problem — `@Component` Registers Filters at Servlet Level

When you declare a filter as `@Component` in Spring Boot, **Spring Boot automatically registers it into the Servlet filter chain** — completely separate from Spring Security.

This means:

```
Servlet Container Filter Chain
        │
        ├── JwtFilter        ← auto-registered, runs on EVERY request
        ├── JwtFilterV1      ← auto-registered, runs on EVERY request
        └── SpringSecurityFilterChain
                ├── Chain 1 (/api/**)  → JwtFilterV1
                └── Chain 2 (rest)    → JwtFilter
```

Both filters were running on every single request — regardless of which route was being hit. The `securityMatcher` on the security chains scopes **authorization**, not the servlet-level filter execution.

This caused both filters to interfere with each other on every request.

---

## The Wrong Fix — `shouldNotFilter`

My first instinct was to add URL checks inside each filter:

```java
// In JwtFilterV1
@Override
protected boolean shouldNotFilter(HttpServletRequest request) {
    return !request.getRequestURI().startsWith("/api/");
}
```

This works technically, but it is **not a production solution** because:

- It duplicates your routing logic in two places
- It is brittle — change a route, remember to change the filter too
- It is a workaround for a problem that should not exist in the first place

---

## The Correct Fix — Don't Register Filters as Beans

The production solution is to **remove `@Component`** from both filter classes and let Spring Security own their lifecycle entirely.

```java
// ✅ No @Component — plain class
public class JwtFilterV1 extends OncePerRequestFilter {
    
    private final JWTServiceV1 jwtServiceV1;
    private final UserServiceImpl userServiceImpl;

    public JwtFilterV1(JWTServiceV1 jwtServiceV1, UserServiceImpl userServiceImpl) {
        this.jwtServiceV1 = jwtServiceV1;
        this.userServiceImpl = userServiceImpl;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        // filter logic
        filterChain.doFilter(request, response);
    }
}
```

Then instantiate filters directly inside the security configuration:

```java
@EnableWebSecurity
public class SecurityConfiguration {

    private final JWTServiceV1 jwtServiceV1;
    private final JWTService jwtService;
    private final UserServiceImpl userServiceImpl;
    private final AppConfiguration appConfiguration;

    public SecurityConfiguration(JWTServiceV1 jwtServiceV1,
                                  JWTService jwtService,
                                  UserServiceImpl userServiceImpl,
                                  AppConfiguration appConfiguration) {
        this.jwtServiceV1 = jwtServiceV1;
        this.jwtService = jwtService;
        this.userServiceImpl = userServiceImpl;
        this.appConfiguration = appConfiguration;
    }

    @Bean
    @Order(1)
    public SecurityFilterChain externalApiChain(HttpSecurity http) throws Exception {
        JwtFilterV1 jwtFilterV1 = new JwtFilterV1(jwtServiceV1, userServiceImpl);

        http
            .securityMatcher("/api/**")
            .csrf(csrf -> csrf.disable())
            .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
            .addFilterBefore(jwtFilterV1, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    @Order(2)
    public SecurityFilterChain internalChain(HttpSecurity http) throws Exception {
        JwtFilter jwtFilter = new JwtFilter(jwtService, userServiceImpl);

        http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> {
                auth.requestMatchers("/user/**", "/test/**", "/actuator/**").permitAll();
                auth.anyRequest().authenticated();
            })
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

Now the servlet container sees only one thing — the Spring Security filter chain. The individual JWT filters are internal to Spring Security, scoped exactly where they belong.

```
Servlet Container Filter Chain
        │
        └── SpringSecurityFilterChain only
                ├── Chain 1 (/api/**)  → JwtFilterV1 (only here)
                └── Chain 2 (rest)    → JwtFilter   (only here)
```

---

## The Memory Question

When I removed `@Component` and started instantiating filters manually, my first concern was:

> "Is a new filter object being created on every request? Won't that cause memory issues?"

The answer is **no**.

The filter instantiation happens inside the `@Bean` method of `SecurityFilterChain`. That bean is created **once at application startup**. The filter instance is held inside the chain and reused for every request that matches that chain.

Per-request cost = zero object creation. Same instance, every time.

---

## The Second Problem — 403 After Successful Validation

After fixing the filter registration, I hit a second issue. The logs showed:

```
JWT FILTER V1 IS RUNNING FOR /api/test
is valid for the route /api/*
Set SecurityContextHolder to anonymous SecurityContext   ← problem
Pre-authenticated entry point called. Rejecting access  ← 403
```

The token was valid. But Spring Security was still returning 403.

### Why This Happens

Spring Security asks exactly one question per request:

> "Is there an `Authentication` object set in the `SecurityContextHolder` for this thread?"

It does not know your filter ran. It does not know your token was valid. If nothing is set in the `SecurityContextHolder`, it treats the request as anonymous and rejects it.

### The Fix

After validating the token, you must set the authentication:

```java
if (token != null && jwtServiceV1.isValid(token)) {
    String clientId = jwtServiceV1.extractClientId(token);

    // For external API — no user, just a client principal
    UsernamePasswordAuthenticationToken authToken =
        new UsernamePasswordAuthenticationToken(
            clientId,
            null,
            List.of(new SimpleGrantedAuthority("ROLE_API_CLIENT"))
        );

    authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));

    // This is the line that tells Spring Security: "request is authenticated"
    SecurityContextHolder.getContext().setAuthentication(authToken);
}

filterChain.doFilter(request, response);
```

For external API routes, there is no user to load from the database. The principal is just the client ID string. This is correct for OAuth2 client credentials — lightweight, stateless, no user lookup.

---

## Key Takeaways

**1. `@Component` on a filter = servlet-level registration**
Spring Boot auto-registers any `@Component` filter into the servlet chain. This bypasses `securityMatcher` scoping entirely.

**2. Remove `@Component`, inject dependencies, instantiate inside the security config**
This hands full lifecycle control to Spring Security. The filter lives exactly where it is declared — inside one chain, not globally.

**3. `SecurityContextHolder` is not optional**
It is Spring Security's per-thread signal. Your filter must set it after validation. Without it, every request is anonymous regardless of token validity.

**4. `shouldNotFilter` is a smell, not a solution**
It is a symptom of a registration problem. Fix the registration, and you never need it.

**5. Manual instantiation does not mean per-request object creation**
`@Bean` methods run once at startup. The filter object lives inside the chain for the lifetime of the application.

---

## What Comes Next

This two-chain setup is the foundation for a proper OAuth2 client credentials implementation. The next problems worth thinking about:

- Token expiry handling for external clients
- Client secret rotation without downtime
- Rate limiting per client ID
- Token revocation when a client is compromised
