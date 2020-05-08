---
title: Spring Security with JWT Tutorial
author: buraktas
layout: post
permalink: /spring-security-with-jwt-tutorial/
dsq_needs_sync:
  - 1
categories:
  - Spring Framework
tags:
  - java
  - spring-framework
  - spring-security
comments: true
---

In this tutorial we will learn how to implement secured APIs in Spring Boot with using JWT tokens. First of all, we have to understand what is a JSON Web Token (JWT).

JWT is an open standard ([RFC7519](https://tools.ietf.org/html/rfc7519)) to share information between entities in a JSON format. It is digitally signed thus it can be verified by entities. It is not encrypted but encoded which means if the man in the middle gets the token then it can be used for accessing secured resources. Because of this it shouldn't contain any critical information like password, credit card number etc. This can be prevented by encryption with TLS or you can basically choose to encrypt JWT as well which is optional.

<!--more-->

The most common usage of JWT is for accessing authorized resources. Here is an example illustration of JWT usage; 

![jwt_workflow]({{ site.url }}/public/images/2019/11/jwt_workflow.png)

1. User sends a request to authenticate with his credentials.
2. Application server authenticates the user with given credentials and generates a JWT.
3. Return back the generated JWT to the user.
4. User will send this JWT each time to access resources on application server.
5. Application server will validate the JWT token and check if user is authorized for the resource.
6. Return data from the resource.

JWT consists of 3 parts;

* Header
* Payload
* Signature

Each JWT part is a base64 encoded string concatenated with with dot (`.`), so a compact JWT structure will look like;

> header.payload.signature

and here is an real jwt value as an example;

> eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJidXJha3RhcyIsImV4cCI6MTU3NDAyMjEyNSwiaWF0IjoxNTc0MDA0MTI1fQ.M8TuJjpA-cERfokcCt0VBfUGTKMkNsdHPpfMnctKaG1ZLkDurUtPAA5Oz9i3Ht1T3MoPEi1BO8DTi5J0IGsVDQ

<h3>Header</h3>
This is the first part which contains two fields; the type (typ) of the token (which is always JWT) and the name of signature algorithm (alg) to create JWT signature. Here is the JSON format for header;

```
{
 "typ": "JWT",
 "alg": "HS512"
}
```

`HS512` stands for `HMAC + SHA512`. List of the supported algorithms can be found [RFC7518](https://tools.ietf.org/html/rfc7518#section-3)

<h3>Payload</h3>
Payload is the part where it can contain predefined fields (or `claims` in other words) or custom fields for sharing information between entities. Here is an example JSON format for Payload section;

```
{
	"sub": "buraktas",
	"iat": 1574080496,
	"exp": 1574102096,
	"userId": 12345
}
```

* `sub` - Subject (or Id/Name) of the principal for the JWT.
* `iat` - (Issued At) - Numeric time value which represents the time this JWT is issued.
* `exp` - (Expiration Time) - Numeric time value which represents expiration time of this JWT.
* `userId` - Custom field added by us.

Hence the fact that, JWT size will become larger when the number of claims increased.

<h3>Signature</h3>
Signature section is created by joining base64 encoded `Header` and `Payload` sections with `.` and applying the algorithm we defined in `alg` section along with a defined `secret` to sign it. Basically, the formula will be like;

```
signature = HMACSHA512(base64Encoded(Header) + "." + base64Encoded(Payload), secret)
```

This is the section our application checks if JWT is tampered.

<br>

<h2>Spring Application</h2>
We will implement a Spring Boot application containing 3 endpoints to illustrate authentication and authorization for a user by using JWT. The 3 endpoints will be;

* `POST /register` - Registering a user with username and password. We will keep this information in a PostgreSQL database.
* `POST /authenticate` - Authenticating a user with given username and password.
* `GET /hello-world` - Protected resource which will be accessible with a valid JWT.

<br>

<h3>Database Schema</h3>
We will use a very basic users table including only a few fields along with a sequence counter for id generation.

```sql
create table users (
  id serial primary key,
  username varchar(255) not null,
  password varchar(255) not null
);

create sequence users_seq;
```

<br>

<h3>Dependencies</h3>

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
            <version>2.1.2.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.4.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.postgresql/postgresql -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.2.8</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.10</version>
            <scope>provided</scope>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.9</version>
        </dependency>

    </dependencies>
```

<br>

<h3>DAO and Repository Classes</h3>
Here are the simple UserDAO and Repository implementations we will use in this application.

```java
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.SequenceGenerator;
import javax.persistence.Table;

@Entity
@Table(name = "users")
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
public class UserDAO {
    @Id
    @SequenceGenerator(name = "users_seq", sequenceName = "users_seq", allocationSize = 1)
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "users_seq")
    private Long id;

    @Column
    private String username;

    @Column
    private String password;
}
```

<br>

As I mentioned above we will use a sequencer named `users_seq` to assign ids for each newly registered user. Our repository will only have `findByUsername` to get UserDAO object from database.

<br>

```java
import com.buraktas.entity.UserDAO;
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface UserRepository extends CrudRepository<UserDAO, Long> {

    UserDAO findByUsername(String username);
}
```

<br>

<h3>JWT And Security Components</h3>
Our first goal is going to implement a utility class for JWT operations like; generating and validating a jwt, parsing claims (fields of Payload section) from a jwt. We will give 6 hours for expiration time in milliseconds and we will use HS512 algorighm (HMAC + SHA512) for signing the jwt along with a `secret` which is `springjwt` in this case (it is coming from application.properties). Additionally, we will use subject, expiration and issuedAt claims for this example. Note that `claims` is basically a Map object which lets us to add custom claims.

```java
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;

@Component
public class JWTProvider {

    private static final long JWT_VALIDITY_IN_MILLISECONDS = 6 * 60 * 60 * 1000;

    @Value("${jwt.secret}")
    private String secret;

    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        return Jwts.builder()
                .setClaims(claims)
                .setSubject(userDetails.getUsername())
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + JWT_VALIDITY_IN_MILLISECONDS))
                .signWith(SignatureAlgorithm.HS512, secret)
                .compact();
    }

    public boolean validateToken(String token, UserDetails userDetails) {
        return StringUtils.equalsIgnoreCase(getSubject(token), userDetails.getUsername()) && !isTokenExpired(token);
    }

    public String getSubject(String token) {
        return getAllClaims(token).getSubject();
    }

    private Date getExpirationDate(String token) {
        return getAllClaims(token).getExpiration();
    }

    private Claims getAllClaims(String token) {
        return Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody();
    }

    private boolean isTokenExpired(String token) {
        return getExpirationDate(token).before(new Date());
    }
}
```

<br>

Now we will create an implementation of `UserDetailsService` which will get a user from `UserRepository`.

```java
import com.buraktas.entity.UserDAO;
import com.buraktas.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.ArrayList;

@Service
public class JWTUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        UserDAO user = userRepository.findByUsername(username);
        if (user == null) {
            throw new UsernameNotFoundException("User not found with username: " + username);
        }

        return new User(user.getUsername(), user.getPassword(), new ArrayList<>());
    }
}
```

<br>

As the next step, we will create an implementation of `OncePerRequest` to be defined in Spring Security Filter Chain. `doFilterInternal` is the only method we need to override. This is the core place where we will resolve jwt and check if a valid username is provided.

```java
import com.buraktas.service.JWTUserDetailsService;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Component
public class JWTRequestFilter extends OncePerRequestFilter {

    private static String AUTHORIZATION_HEADER = "Authorization";

    @Autowired
    private JWTProvider jwtProvider;

    @Autowired
    private JWTUserDetailsService jwtUserDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        String jwt = resolveToken(request);

        if (StringUtils.isNotBlank(jwt)) {
            String username = jwtProvider.getSubject(jwt);

            if (StringUtils.isNotBlank(username) && SecurityContextHolder.getContext().getAuthentication() == null) {
                UserDetails userDetails = jwtUserDetailsService.loadUserByUsername(username);
                if (jwtProvider.validateToken(jwt, userDetails)) {
                    UsernamePasswordAuthenticationToken token = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                    SecurityContextHolder.getContext().setAuthentication(token);
                }
            }
        }

        filterChain.doFilter(request, response);
    }

    private String resolveToken(HttpServletRequest request) {
        String authToken = request.getHeader(AUTHORIZATION_HEADER);
        if (StringUtils.isNotBlank(authToken) && authToken.startsWith("Bearer ")) {
            return authToken.substring(7);
        }

        return null;
    }
}
```

<br>

Our final step is configuring security for our spring application. We will only have two endpoints which doesn't need any authentication.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserDetailsService jwtUserDetailsService;

    @Autowired
    private JWTRequestFilter jwtRequestFilter;

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(jwtUserDetailsService).passwordEncoder(passwordEncoder());
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        httpSecurity
                .csrf().disable()
                .addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class)
                .authorizeRequests()
                .antMatchers("/authenticate", "/register").permitAll()
                .anyRequest().authenticated()
                .and()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
    }
}
```

<br>

`@EnableWebSecurity` annotation enables us to override/configure HttpSecurity object. Furthermore, there are a few parts to focus in `configure` method. We are adding `JWTRequestFilter` into Spring Security Filter Chain to make it run before `UsernamePasswordAuthenticationFilter`. We are marking `/register` and `/authenticate` endpoints to not require any authentication. Any other endpoint will require an authentication which is a valid jwt. Also, we are telling our Spring application to not create any sessions because our application is stateless.

<br>

<h2>REST Controllers and Services</h2>
This is the final part of our Spring application. We will now add the 3 endpoints we mentioned earlier in this tutorial. First one is for registering a user into our database, second one is for authentication which will return the jwt we spoke about and finally the third api is an authenticated one which requires a valid jwt and will return hello-world response.

`/register` endpoint accepts a JSON object (UserDTO in this case) which has username and password fields. Then it checks if the given username has been already taken. If it is an existing one it will throw `UsernameAlreadyExistException` which extends from RunTimeException. If it is not an existing one we will add new user into our database and return its id.

```java
import com.buraktas.entity.UserDAO;
import com.buraktas.entity.UserDTO;
import com.buraktas.entity.exception.UsernameAlreadyExistException;
import com.buraktas.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping(value = "/register", method = RequestMethod.POST)
    public ResponseEntity<?> register(@RequestBody UserDTO userDTO) {
        UserDAO user = userService.findOneByUsername(userDTO.getUsername());

        if (user != null) {
            throw new UsernameAlreadyExistException();
        }

        return ResponseEntity.ok(userService.save(userDTO));
    }
}
```

<br>

`/authenticate` endpoint also accepts a JSON object contains username and password and will return a valid jwt if input is valid. Otherwise, it will return `403 Forbidden` response which is the default one.

```java
import com.buraktas.entity.AuthRequest;
import com.buraktas.entity.AuthResponse;
import com.buraktas.security.JWTProvider;
import com.buraktas.service.JWTUserDetailsService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AuthController {

    Logger logger = LoggerFactory.getLogger(AuthController.class);

    @Autowired
    private JWTProvider jwtTokenUtil;

    @Autowired
    private JWTUserDetailsService jwtUserDetailsService;

    @Autowired
    private AuthenticationManager authenticationManager;

    @RequestMapping(value = "/authenticate", method = RequestMethod.POST)
    public ResponseEntity<?> createAuthenticationToken(@RequestBody AuthRequest authenticationRequest) {
        UsernamePasswordAuthenticationToken token =
                new UsernamePasswordAuthenticationToken(authenticationRequest.getUsername(), authenticationRequest.getPassword());
        try {
            authenticationManager.authenticate(token);
        } catch (BadCredentialsException e) {
            logger.error("Invalid credentials", e);
            throw e;
        }

        UserDetails userDetails = jwtUserDetailsService.loadUserByUsername(authenticationRequest.getUsername());

        return ResponseEntity.ok(new AuthResponse(jwtTokenUtil.generateToken(userDetails)));
    }
}
```

<br>

Our last controller will contain `hello-world` endpoint which requires authentication.

```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloWorldController {

    @RequestMapping(value = "/hello-world", method = RequestMethod.GET)
    public String helloWorld() {
        return "Hello World";
    }
}
```

<br>

<h2>Testing</h2>
Lets first register a user;

> curl -X POST localhost:8080/register --header "Content-Type: application/json" --data '{"username": "buraktas","password": "123456"}'

which will return a new id of the newly added user. Now before calling `/authenticate` with correct credentials we will make a call with invalid credentials to verify our authentication works as expected.

> curl -X POST localhost:8080/authenticate --header "Content-Type: application/json" --data '{"username": "buraktas","password": "111111"}'

```
{
	"timestamp": "2019-11-20T12:22:42.536+0000",
	"status": 403,
	"error": "Forbidden",
	"message": "Access Denied",
	"path": "/authenticate"
}
```

<br>

We can also verify we logged BadCredentialsException from our spring application;

```
2019-11-20 14:59:34.816 ERROR 40445 --- [nio-8080-exec-5] com.buraktas.controller.AuthController   : Invalid credentials

org.springframework.security.authentication.BadCredentialsException: Bad credentials
	at org.springframework.security.authentication.dao.DaoAuthenticationProvider.additionalAuthenticationChecks(DaoAuthenticationProvider.java:93) ~[spring-security-core-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.springframework.security.authentication.dao.AbstractUserDetailsAuthenticationProvider.authenticate(AbstractUserDetailsAuthenticationProvider.java:166) ~[spring-security-core-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.springframework.security.authentication.ProviderManager.authenticate(ProviderManager.java:175) ~[spring-security-core-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.springframework.security.authentication.ProviderManager.authenticate(ProviderManager.java:200) ~[spring-security-core-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter$AuthenticationManagerDelegator.authenticate(WebSecurityConfigurerAdapter.java:503) ~[spring-security-config-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at com.buraktas.controller.AuthController.createAuthenticationToken(AuthController.java:39) ~[classes/:na]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_191]
```

<br>

Great we got `403 Forbidden` as mentioned earlier in this tutorial. Now time to get a valid jwt token;

> curl -X POST localhost:8080/authenticate --header "Content-Type: application/json" --data '{"username": "buraktas","password": "123456"}'

```
{
	"token": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJidXJha3RhcyIsImV4cCI6MTU3NDI3NDI0OCwiaWF0IjoxNTc0MjUyNjQ4fQ.Y__b_QdlJq_MeF5mwy2UhAq-cX6QbikS4a_oCDkGwpb0UrYDV5Vtw0KBJWHQcp_qX3yrnDMjsM4eIXtp03KHBA"
}
```

<br>

Finally, it is time to make a call to our only authenticated method `hello-world` endpoint;

> curl -X GET localhost:8080/hello-world --header "Content-Type: application/json" --header "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJidXJha3RhcyIsImV4cCI6MTU3NDI3NDI0OCwiaWF0IjoxNTc0MjUyNjQ4fQ.Y__b_QdlJq_MeF5mwy2UhAq-cX6QbikS4a_oCDkGwpb0UrYDV5Vtw0KBJWHQcp_qX3yrnDMjsM4eIXtp03KHBA" --data '{"username": "buraktas","password": "123456"}'

And our response will be `Hello World` as expected.

Thanks a lot for reading my tutorial about using JWT in a Spring Application. You can also have the full codebase from tutorial side in [github](https://github.com/flexelem/spring-tutorials/tree/master/spring-security)