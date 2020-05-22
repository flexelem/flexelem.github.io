---
title: Spring Schedule Tutorial
author: buraktas
layout: post
permalink: /spring-schedule-tutorial/
dsq_needs_sync:
  - 1
categories:
  - Spring Framework
tags:
  - spring-framework
  - java
comments: true
---
In this tutorial we will walk through how to implement and define crob jobs with a schedule behind of it in Spring Framework. Generally, in software architectures we want to have some background jobs to do some processing, aggregating or even just fetching some data
and streaming them. Instead of implementing separate cron jobs and managing them as a separate application we can create and maintain cron jobs in same spring application with using `@Scheduled` annotation. Generally, all the scheduled tasks will run on the same thread which means when current task is being processed others will get blocked and has to wait current thread to finish. However, this can be configured by implementing our own `ThreadPoolTaskExecutor` to assign a thread pool. So that each task will run asyncrohonously.

<!--more-->

<br>
<h2>Dependencies</h2>

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
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.4.RELEASE</version>
        </dependency>
    </dependencies>
```

<br>
<h2>Enabling Scheduling</h2>
First, we have to enable scheduling globally from Spring context to our <code>@Configuration</code> class by adding <code>@EnableScheduling</code> annotation.

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

Now before diving in there are some properties belongs to `@Scheduled` annotation that we need to know.

* initialDelay
* fixedDelay
* fixedRate
* cron

<h3>initialDelay</h3>
This property is for making a task to wait for its first execution after application has started. For example; if initialDelay is set to 5000 then the task will wait 5 seconds after the spring application started.

<h3>fixedDelay</h3>
This property is for making the task to wait N milliseconds after the previous task has finished. For example; if fixedDelay is set to 5000 then Thread\_2 will start 5 seconds after the end of the execution of Thread\_1.

<h3>fixedRate</h3>
This property is for making the task to run at every N millisecond without wating the previous task to finish. For example; if fixedRate is set to 5000 then Thread\_2 will start 5 seconds after Thread\_1's start time without waiting it to be finished. One thing to note here is since the current running thread won't wait other ones which will cause all threads allocated in the thread pool. And you might also get StackOverflowException if you run out of memory while running all the threads.

<h3>cron</h3>
This is the cron expression to define running interval for a task.

<br>

<h2>1. Schedule a Task with FixedDelay</h2>
The first the task will start 5 seconds after spring application started and each of the following task will start 3 seconds after the previous one finished.

```java
package com.buraktas.jobs;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class JobHandler {
    Logger logger = LoggerFactory.getLogger(JobHandler.class);

    @Scheduled(initialDelay = 5000L, fixedDelay = 3000L)
    public void jobWithFixedDelay() throws InterruptedException {
        logger.info("Hi from jobWithFixedDelay");
        logger.info("Sleeping started.");
        Thread.sleep(1000L);
        logger.info("Sleeping finished.");
    }
}
```

```
2019-09-03 10:55:55.954  INFO 89447 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2019-09-03 10:55:56.057  INFO 89447 --- [           main] o.s.s.c.ThreadPoolTaskScheduler          : Initializing ExecutorService 'taskScheduler'
2019-09-03 10:55:56.109  INFO 89447 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-09-03 10:55:56.112  INFO 89447 --- [           main] com.buraktas.Application                 : Started Application in 16.3 seconds (JVM running for 21.671)
2019-09-03 10:56:01.085  INFO 89447 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Hi from jobWithFixedDelay
2019-09-03 10:56:01.085  INFO 89447 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Sleeping started.
2019-09-03 10:56:02.089  INFO 89447 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Sleeping finished.
2019-09-03 10:56:05.094  INFO 89447 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Hi from jobWithFixedDelay
2019-09-03 10:56:05.094  INFO 89447 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Sleeping started.
2019-09-03 10:56:06.097  INFO 89447 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Sleeping finished.
2019-09-03 10:56:09.099  INFO 89447 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Hi from jobWithFixedDelay
2019-09-03 10:56:09.099  INFO 89447 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Sleeping started.
2019-09-03 10:56:10.099  INFO 89447 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Sleeping finished.
```

<br>

<h2>2. Schedule a Task with FixedRate</h2>
The first the task will start 5 seconds after spring application started and each of the following task will start right after the previous one has finished.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class JobHandler {
    Logger logger = LoggerFactory.getLogger(JobHandler.class);

    @Scheduled(initialDelay = 5000L, fixedRate = 3000L)
    public void jobWithFixedRate() throws InterruptedException {
        logger.info("Hi from jobWithFixedRate");
        logger.info("Sleeping started.");
        Thread.sleep(1000L);
        logger.info("Sleeping finished.");
    }
}
```

```
2019-09-03 10:54:14.202  INFO 89443 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2019-09-03 10:54:14.323  INFO 89443 --- [           main] o.s.s.c.ThreadPoolTaskScheduler          : Initializing ExecutorService 'taskScheduler'
2019-09-03 10:54:14.378  INFO 89443 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-09-03 10:54:14.381  INFO 89443 --- [           main] com.buraktas.Application                 : Started Application in 16.432 seconds (JVM running for 21.853)
2019-09-03 10:54:19.352  INFO 89443 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Hi from jobWithFixedRate
2019-09-03 10:54:19.352  INFO 89443 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Sleeping started.
2019-09-03 10:54:20.356  INFO 89443 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Sleeping finished.
2019-09-03 10:54:22.351  INFO 89443 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Hi from jobWithFixedRate
2019-09-03 10:54:22.351  INFO 89443 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Sleeping started.
2019-09-03 10:54:23.353  INFO 89443 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Sleeping finished.
2019-09-03 10:54:25.350  INFO 89443 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Hi from jobWithFixedRate
2019-09-03 10:54:25.351  INFO 89443 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Sleeping started.
2019-09-03 10:54:26.352  INFO 89443 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Sleeping finished.
```

<br>

<h2>3. Schedule a Task with Cron Expression</h2>
First thing to note here is Spring Cron is not same as [Unix Cron](https://en.wikipedia.org/wiki/Cron) format which consists of 5 fields to define an expression. Where a Spring Cron expression has 6 fields including time unit of `second` in addition to other 5 fields.
Here is the description from [CronSequenceGenerator](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/support/CronSequenceGenerator.html) in Spring;

`The pattern is a list of six single space-separated fields: representing second, minute, hour, day, month, weekday. Month and weekday names can be given as the first three letters of the English names.`

**Unix Cron**

```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │
# │ │ │ │ │
# * * * * * command to execute
```

**Spring Cron**

```
# ┌────────────── second (0 - 59)
# | ┌───────────── minute (0 - 59)
# | │ ┌───────────── hour (0 - 23)
# | │ │ ┌───────────── day of the month (1 - 31)
# | │ │ │ ┌───────────── month (1 - 12)
# | │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# | │ │ │ │ │                                   7 is also Sunday on some systems)
# | │ │ │ │ │
# | │ │ │ │ │
# * * * * * * command to execute
```

<br>

As an example we will define a cron job which will run at 5th minute at each hour from Monday to Friday. So, the time of running in an hours will be `05, 15, 25, 35, 45, 55` and so on.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class JobHandler {
    Logger logger = LoggerFactory.getLogger(JobHandler.class);

    @Scheduled(cron = "0 5 * * * Mon-Fri")
    public void jobWithCronexpression() throws InterruptedException {
        logger.info("Hi from jobWithCronexpression");
        logger.info("Sleeping started.");
        Thread.sleep(1000L);
        logger.info("Sleeping finished.");
    }
}
```

```
2019-09-03 13:01:42.116  INFO 90578 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2019-09-03 13:01:42.218  INFO 90578 --- [           main] o.s.s.c.ThreadPoolTaskScheduler          : Initializing ExecutorService 'taskScheduler'
2019-09-03 13:01:42.270  INFO 90578 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-09-03 13:01:42.273  INFO 90578 --- [           main] com.buraktas.Application                 : Started Application in 16.406 seconds (JVM running for 21.814)
2019-09-03 13:05:00.003  INFO 90578 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Hi from jobWithCronexpression
2019-09-03 13:05:00.004  INFO 90578 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Sleeping started.
2019-09-03 13:05:01.008  INFO 90578 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Sleeping finished.
```

<br>

Before finalizing this tutorial there is one thing to mention which is about configuring a thread pool to make these tasks running on multiple threads concurrently. You might notice that all the tasks we ran was running on one thread with name `scheduling-1`. Generally, in sofware systems we had more than one job to make them running without blocking each other. So, to make threads running concunrently we will create our own `ThreadPoolTaskScheduler`. Here is a quote from the documentation of [@EnableScheduling](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/annotation/EnableScheduling.html)

```
By default, will be searching for an associated scheduler definition: either a unique TaskScheduler bean in the context, or a TaskScheduler bean named "taskScheduler" otherwise; the same lookup will also be performed for a ScheduledExecutorService bean. If neither of the two is resolvable, a local single-threaded default scheduler will be created and used within the registrar.

When more control is desired, a @Configuration class may implement SchedulingConfigurer. This allows access to the underlying ScheduledTaskRegistrar instance. For example, the following example demonstrates how to customize the Executor used to execute scheduled tasks:
```

Now we will create a task executor with a thread pool size of 5 inside a configuration class

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.SchedulingConfigurer;
import org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler;
import org.springframework.scheduling.config.ScheduledTaskRegistrar;

import java.util.concurrent.Executor;

@EnableScheduling
@Configuration
public class SchedulerConfig implements SchedulingConfigurer {

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.setScheduler(taskExecutor());
    }

    @Bean(destroyMethod="shutdown")
    public Executor taskExecutor() {
        ThreadPoolTaskScheduler taskScheduler = new ThreadPoolTaskScheduler();
        taskScheduler.setPoolSize(5);
        taskScheduler.setThreadNamePrefix("task-scheduler-");
        taskScheduler.initialize();
        return taskScheduler;
    }
}
```

```
2019-09-03 14:30:32.767  INFO 91108 --- [           main] o.s.s.c.ThreadPoolTaskScheduler          : Initializing ExecutorService 'taskScheduler'
2019-09-03 14:30:33.039  INFO 91108 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-09-03 14:30:33.042  INFO 91108 --- [           main] com.buraktas.Application                 : Started Application in 16.433 seconds (JVM running for 21.874)
2019-09-03 14:30:34.009  INFO 91108 --- [ask-scheduler-1] com.buraktas.jobs.JobHandler             : Hi from jobWithFixedRate_2
2019-09-03 14:30:34.311  INFO 91108 --- [ask-scheduler-3] com.buraktas.jobs.JobHandler             : Hi from jobWithFixedRate_2
2019-09-03 14:30:34.612  INFO 91108 --- [ask-scheduler-1] com.buraktas.jobs.JobHandler             : Hi from jobWithFixedRate_2
2019-09-03 14:30:34.912  INFO 91108 --- [ask-scheduler-1] com.buraktas.jobs.JobHandler             : Hi from jobWithFixedRate_2
2019-09-03 14:30:35.010  INFO 91108 --- [ask-scheduler-4] com.buraktas.jobs.JobHandler             : Hi from jobWithFixedRate_3
2019-09-03 14:30:35.212  INFO 91108 --- [ask-scheduler-3] com.buraktas.jobs.JobHandler             : Hi from jobWithFixedRate_2
2019-09-03 14:30:35.510  INFO 91108 --- [ask-scheduler-3] com.buraktas.jobs.JobHandler             : Hi from jobWithFixedRate_2
2019-09-03 14:30:35.812  INFO 91108 --- [ask-scheduler-3] com.buraktas.jobs.JobHandler             : Hi from jobWithFixedRate_2
2019-09-03 14:30:36.010  INFO 91108 --- [ask-scheduler-1] com.buraktas.jobs.JobHandler             : Hi from jobWithFixedRate
```

We can clearly see that all the tasks are distributed and scheduled on different threads from the thread pool we initialized.

The final project structure can be found [here](https://github.com/flexelem/spring-tutorials/tree/master/spring-scheduled-job)