---
title: Spring Retry Tutorial
author: buraktas
layout: post
permalink: /spring-retry-tutorial/
dsq_needs_sync:
  - 1
categories:
  - Spring Framework
tags:
  - spring-framework
  - java
comments: true
---
In this tutorial I will talk about some retry examples in Spring and how to do it. Retry logic or in other words retry strategies are being used
when an operation is failed and we want to basically retry that operation again by a logic. Most of the software systems are doing external calls, operations etc.
which are dependent on other systems or resources. For example, charging a client by sending a charge request to Stripe, sending a message to Amazon SQS queue which might failed for a short time due to a transitive issue. Instead of, failing the process at the very first try we can basically have a retry strategy to try the current operation multiple times to continue current process without any issue. In Spring, there are multiple ways to define how to retry on some operation.

<!--more-->

</br>
<h2>Dependencies</h2>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.buraktas</groupId>
    <artifactId>retry-tutorial</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.retry</groupId>
            <artifactId>spring-retry</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
    </dependencies>

</project>
```

</br>
<h2>Enabling Retry</h2>
First, we have to enable retry globally from Spring context to our <code>@Configuration</code> class by adding <code>@EnableRetry</code> annotation.

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.retry.annotation.EnableRetry;

@SpringBootApplication
@EnableRetry
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

</br>
<h2>1. Retry with Retryable Annotation</h2>
To define a retry strategy for a method we have to add <code>@Retryable</code> annotation on it. Additionally, we can define the behaviour of retry by providing some set of inputs. We will take a closer look for the most general ones which are;

* maxAttemps - Maximum number of attemps to retry.
* value - Exception types to enable retry.
* backoff - Backoff property to define backoff logic.
    * delay - The duration of delay given milliseconds
    * multiplier - Multiplier value for delay

A retry with fixed delay example;

```java
@Retryable(
    maxAttempts = 5,
    backoff = @Backoff (delay = 1000L),
    value = {
        RuntimeException.class
    }
)
public void processRetryWithFixedDelay(String requestId) {
    LOGGER.info("Processing request: {}", requestId);
    throw new RuntimeException("Failed operation");
}
```

In this case we have a retry strategy having maximum attemps by 5 where each of them will attempt run with 1 second of delay. Later on, if all attempts failed there is no any error handling logic for the last thrown exception. To cover this, we will define a recovery action by implementing a method annotated with <code>@Recover</code>. This recovery method should have a method signature including the exception thrown from retried method along with other method parameters from method to recover. With this way spring will resolve the correct recover method. So, our recovery method will be like;

```java
@Recover
public void recoverFromFixedRetry(RuntimeException ex, String requestId) {
    LOGGER.info("Recovering request {}", requestId);
}
```

An example output will be like;

```
21:38:06.271 [main] DEBUG org.springframework.retry.support.RetryTemplate - Retry: count=0
21:38:06.282 [main] INFO com.buraktas.service.MainService - Processing request foobar
21:38:07.289 [main] DEBUG org.springframework.retry.support.RetryTemplate - Checking for rethrow: count=1
21:38:07.289 [main] DEBUG org.springframework.retry.support.RetryTemplate - Retry: count=1
21:38:07.289 [main] INFO com.buraktas.service.MainService - Processing request foobar
21:38:08.291 [main] DEBUG org.springframework.retry.support.RetryTemplate - Checking for rethrow: count=2
21:38:08.291 [main] DEBUG org.springframework.retry.support.RetryTemplate - Retry: count=2
21:38:08.291 [main] INFO com.buraktas.service.MainService - Processing request foobar
21:38:09.296 [main] DEBUG org.springframework.retry.support.RetryTemplate - Checking for rethrow: count=3
21:38:09.296 [main] DEBUG org.springframework.retry.support.RetryTemplate - Retry: count=3
21:38:09.296 [main] INFO com.buraktas.service.MainService - Processing request foobar
21:38:10.297 [main] DEBUG org.springframework.retry.support.RetryTemplate - Checking for rethrow: count=4
21:38:10.297 [main] DEBUG org.springframework.retry.support.RetryTemplate - Retry: count=4
21:38:10.297 [main] INFO com.buraktas.service.MainService - Processing request foobar
21:38:10.298 [main] DEBUG org.springframework.retry.support.RetryTemplate - Checking for rethrow: count=5
21:38:10.300 [main] DEBUG org.springframework.retry.support.RetryTemplate - Retry failed last attempt: count=5
21:38:10.301 [main] INFO com.buraktas.service.MainService - Recovering request foobar
```


Now, if we want to have an exponential retry strategy then we have to set <code>multiplier</code> in <code>backoff</code> property. Here is an example;

```java
@Retryable(
        maxAttempts = 4,
        backoff = @Backoff (delay = 1000L, multiplier = 2),
        value = {
                RuntimeException.class
        }
)
public void processRetryWithExponentialDelay(String requestId) {
    LOGGER.info("Processing request {}", requestId);
    throw new RuntimeException("Failed operation");
}
```

This means, retry each attempt by having with delay of 1000 * \\( 2^{n} \\) where `n` is the current number of retry. An example output from a test would be like;

```java
21:39:31.340 [main] DEBUG org.springframework.retry.support.RetryTemplate - Retry: count=0
21:39:31.348 [main] INFO com.buraktas.service.MainService - Processing request foobar
21:39:31.350 [main] DEBUG org.springframework.retry.backoff.ExponentialBackOffPolicy - Sleeping for 1000
21:39:32.350 [main] DEBUG org.springframework.retry.support.RetryTemplate - Checking for rethrow: count=1
21:39:32.350 [main] DEBUG org.springframework.retry.support.RetryTemplate - Retry: count=1
21:39:32.350 [main] INFO com.buraktas.service.MainService - Processing request foobar
21:39:32.351 [main] DEBUG org.springframework.retry.backoff.ExponentialBackOffPolicy - Sleeping for 2000
21:39:34.354 [main] DEBUG org.springframework.retry.support.RetryTemplate - Checking for rethrow: count=2
21:39:34.354 [main] DEBUG org.springframework.retry.support.RetryTemplate - Retry: count=2
21:39:34.354 [main] INFO com.buraktas.service.MainService - Processing request foobar
21:39:34.354 [main] DEBUG org.springframework.retry.backoff.ExponentialBackOffPolicy - Sleeping for 4000
21:39:38.358 [main] DEBUG org.springframework.retry.support.RetryTemplate - Checking for rethrow: count=3
21:39:38.358 [main] DEBUG org.springframework.retry.support.RetryTemplate - Retry: count=3
21:39:38.358 [main] INFO com.buraktas.service.MainService - Processing request foobar
21:39:38.358 [main] DEBUG org.springframework.retry.support.RetryTemplate - Checking for rethrow: count=4
21:39:38.361 [main] DEBUG org.springframework.retry.support.RetryTemplate - Retry failed last attempt: count=4
21:39:38.361 [main] INFO com.buraktas.service.MainService - Recovering request foobar
```

<h2>2. Retry with RetryTemplate</h2>
Another solution to provide a retry strategy is by using <code>RetryTemplate</code>. Here I will use <code>FixedBackOffPolicy</code> implementation
as an example. First we implement a bean method in a configuration class for `RetryTemplate`;

```java
@Bean
public RetryTemplate retryTemplate() {
    RetryTemplate retryTemplate = new RetryTemplate();

    FixedBackOffPolicy fixedBackOffPolicy = new FixedBackOffPolicy();
    fixedBackOffPolicy.setBackOffPeriod(1000L);
    retryTemplate.setBackOffPolicy(fixedBackOffPolicy);

    SimpleRetryPolicy retryPolicy = new SimpleRetryPolicy();
    retryPolicy.setMaxAttempts(5);
    retryTemplate.setRetryPolicy(retryPolicy);
    return retryTemplate;
}
```

Now we can use this `RestTemplate` anywhere in spring to apply a retry logic by its `execute` method. There are 3 implementations of `execute` where I will show the common one by giving `RetryCallback` and `RecoveryCallback` functions. 

```java
public void processRetryWithRestTemplate(String requestId) {
    LOGGER.info("Processing request {}", requestId);
    retryTemplate.execute(context -> {
        LOGGER.info("Processing request...");
        throw new RuntimeException("Failed operation");
    }, context -> {
        LOGGER.info("Recovering request...");
        return true;
    });
}
```

In this example we defined a function which will be retried by number of attemps defined for `RestTemplate` and a recovery method which will be called when all retries exhausted. In other words, we can implement these callback classes and pass it in `execute` method. Something like;

```java
public void processRetryWithRestTemplate(String requestId) {
    LOGGER.info("Processing request {}", requestId);
    retryTemplate.execute(new ProcessCallback(), new ProcessRecoveryCallback());
}

private class ProcessCallback implements RetryCallback<Boolean, RuntimeException> {

    @Override
    public Boolean doWithRetry(RetryContext context) throws RuntimeException {
        LOGGER.info("Processing request...");
        throw new RuntimeException("Failed operation");
    }
}

private class ProcessRecoveryCallback implements RecoveryCallback<Boolean> {

    @Override
    public Boolean recover(RetryContext context) {
        LOGGER.info("Recovering request...");
        return true;
    }
}
```

And the output would be like;

```
23:20:18.889 [main] INFO com.buraktas.service.MainService - Processing request foobar
23:20:18.892 [main] DEBUG org.springframework.retry.support.RetryTemplate - Retry: count=0
23:20:18.892 [main] INFO com.buraktas.service.MainService - Processing request...
23:20:19.893 [main] DEBUG org.springframework.retry.support.RetryTemplate - Checking for rethrow: count=1
23:20:19.893 [main] DEBUG org.springframework.retry.support.RetryTemplate - Retry: count=1
23:20:19.893 [main] INFO com.buraktas.service.MainService - Processing request...
23:20:20.898 [main] DEBUG org.springframework.retry.support.RetryTemplate - Checking for rethrow: count=2
23:20:20.898 [main] DEBUG org.springframework.retry.support.RetryTemplate - Retry: count=2
23:20:20.898 [main] INFO com.buraktas.service.MainService - Processing request...
23:20:20.898 [main] DEBUG org.springframework.retry.support.RetryTemplate - Checking for rethrow: count=3
23:20:20.898 [main] DEBUG org.springframework.retry.support.RetryTemplate - Retry failed last attempt: count=3
23:20:20.898 [main] INFO com.buraktas.service.MainService - Recovering request...
23:20:20.900 [main] DEBUG org.springframework.test.context.support.AbstractDirtie
```

You can find the whole project from my [github][1]

[1]: https://github.com/flexelem/spring-retry-example