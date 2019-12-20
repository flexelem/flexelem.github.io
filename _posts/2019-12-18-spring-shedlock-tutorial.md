---
title: Spring Shedlock Tutorial
author: buraktas
layout: post
permalink: /spring-shedlock-tutorial/
dsq_needs_sync:
  - 1
categories:
  - Spring Framework
tags:
  - spring-framework
  - java
  - shedlock
comments: true
---

I talked about how to create a scheduled job in this [post]({% link _posts/2019-09-02-spring-schdule-cron-job-tutorial.md %}). Most of us probably faced with a use case to make only one specific cron job to run at a time in a distributed environment. For example; we don't want to run multiple cron jobs to send same email to customers or charging them multiple times. To avoid this we have to find a way to make a cron job run only in one instance. For this we will walk through how to use [Shedlock](https://github.com/lukas-krecan/ShedLock) which provides a distributed lock mechanism for cron jobs. Well for this we also need to have a database so I will use MySQL in this example to run locally.

<!--more-->

<br>

<h2>Dependencies</h2>

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.6.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
</parent>

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
        <artifactId>spring-boot-starter-data-jpa</artifactId>
        <version>2.2.2.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.18</version>
    </dependency>

    <dependency>
        <groupId>net.javacrumbs.shedlock</groupId>
        <artifactId>shedlock-spring</artifactId>
        <version>4.0.3</version>
    </dependency>

    <dependency>
        <groupId>net.javacrumbs.shedlock</groupId>
        <artifactId>shedlock-provider-jdbc-template</artifactId>
        <version>4.0.3</version>
    </dependency>
</dependencies>
```

<br>

<h2>Enabling Shedlock</h2>
First step is creating a table for locks to be used by Shedlock. I assume you already have a MySQL running locally (Or any other JDBC compliant database like MariaDB, Postgres etc). Then we have to a table;

```sql
CREATE TABLE shedlock(
    name VARCHAR(64), 
    lock_until TIMESTAMP(3) NULL, 
    locked_at TIMESTAMP(3) NULL, 
    locked_by  VARCHAR(255), 
    PRIMARY KEY (name)
)
```

Later on, we have to add one annotation and a bean producer for LockProvider for enabling Shedlock. So, our Application class will look like;

```java
@SpringBootApplication
@EnableScheduling
@EnableSchedulerLock(defaultLockAtMostFor = "30s")
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public LockProvider lockProvider(DataSource dataSource) {
        return new JdbcTemplateLockProvider(dataSource);
    }
}
```

* `EnableSchedulerLock` annotation enables Shedlock in Spring context so that we can use it for our jobs. `defaultLockAtMostFor` attribute is required but it can be overridden by individual tasks.
* We need to create a Bean for LockProvider which will be used by Shedlock to update lock table we created.

<br>

<h2>Running a Job with Shedlock</h2>
All jobs which need distributed lock should be annotated with `@SchedulerLock` annotation, otherwise, jobs will run in parallel. There are three main parameters to configure.

* **name** - Every job has to have a unique name.  Remember that `name` field from `shedlock` table was the primary key
* **lockAtLeastFor** - This attribute is for ensuring the current job will hold the lock at least given amount of time. For example; if we configure it to 20 seconds then the lock won't released even though job finished before 20 seconds.
* **lockAtMostFor** - This attribute is for making sure lock is released in case of executing instance dies. As it is suggested from the documentation itself it will be better to set this attribute larger than the maximum estimated execution time. If a task takes longer than this value than an unexpected behaviour might happen because some other instance will also run the same job since lock will be released.

I will create a job called `AwesomeJob` with `lockAtLeastFor` attribute set to 15 seconds and `lockAtMostFor` attribute set to 20 seconds. The actual time it will take to run will be just 5 seconds and it will run at every minute.

```java
import net.javacrumbs.shedlock.spring.annotation.SchedulerLock;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class JobHandler {
    Logger logger = LoggerFactory.getLogger(JobHandler.class);

    @Scheduled(cron = "0 * * * * *")
    @SchedulerLock(name = "AwesomeJob", lockAtLeastFor = "15S", lockAtMostFor = "20S")
    public void awesomeJob() throws InterruptedException {

        for (int i = 0; i < 5; i++) {
            logger.info("Processing {}", i);
            Thread.sleep(1000L);
        }
    }
}
```

Now I configured this Spring application to run as two different instances via Docker with a MySQL instance so that we can verify one of the instance will hold the lock and other instance won't run any job. Here is the docker-compose.yml file I used to spin up docker network.

```yml
version: '3.2'
services:
  spring-shedlock-service_1:
    build:
      dockerfile: Dockerfile
      context: .
    volumes:
      - .:/spring-shedlock
      - ~/.m2:/root/.m2
    working_dir: /spring-shedlock
    command:
      - bash
      - -c
      - cd /spring-shedlock && mvn clean spring-boot:run
    tty: true
    ports:
      - "8080:8080"

  spring-shedlock-service_2:
    build:
      dockerfile: Dockerfile
      context: .
    volumes:
      - .:/spring-shedlock
      - ~/.m2:/root/.m2
    working_dir: /spring-shedlock
    command:
      - bash
      - -c
      - cd /spring-shedlock && mvn clean spring-boot:run
    tty: true
    ports:
      - "8090:8090"

  mysql:
    build:
      dockerfile: mysql.docker
      context: .
    ports:
      - "3306:3306"
```

<br>

The whole project with docker files can be found [here](https://github.com/flexelem/spring-tutorials/tree/master/spring-shedlock)

> docker ps

```
CONTAINER ID        IMAGE                                      COMMAND                  CREATED             STATUS              PORTS                    NAMES
938e2903934b        springshedlock_spring-shedlock-service_2   "/usr/local/bin/mvn-…"   8 minutes ago       Up 2 minutes        0.0.0.0:8090->8090/tcp   springshedlock_spring-shedlock-service_2_1
f1177afa6b27        springshedlock_spring-shedlock-service_1   "/usr/local/bin/mvn-…"   8 minutes ago       Up 2 minutes        0.0.0.0:8080->8080/tcp   springshedlock_spring-shedlock-service_1_1
9db3e35db22c        springshedlock_mysql                       "bash /tmp/mysql-sta…"   8 minutes ago       Up 2 minutes        0.0.0.0:3306->3306/tcp   springshedlock_mysql_1
```

So we have 2 spring application with container Ids of `938e2903934b` and `f1177afa6b27`. One of them will hold the lock and rum the job. I started two applications at the same time and shared the logs below;

```
2019-12-20 11:09:44.904  INFO 1 --- [           main] o.h.e.j.e.i.LobCreatorBuilderImpl        : HHH000422: Disabling contextual LOB creation as connection was null
2019-12-20 11:09:46.381  INFO 1 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
2019-12-20 11:09:48.257  INFO 1 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2019-12-20 11:09:48.633  WARN 1 --- [           main] aWebConfiguration$JpaWebMvcConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
2019-12-20 11:09:49.489  INFO 1 --- [           main] o.s.s.c.ThreadPoolTaskScheduler          : Initializing ExecutorService 'taskScheduler'
2019-12-20 11:09:49.687  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-12-20 11:09:49.695  INFO 1 --- [           main] com.buraktas.Application                 : Started Application in 28.553 seconds (JVM running for 58.811)
```

```
2019-12-20 11:09:47.215  INFO 1 --- [           main] o.s.s.c.ThreadPoolTaskScheduler          : Initializing ExecutorService 'taskScheduler'
2019-12-20 11:09:47.619  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-12-20 11:09:47.638  INFO 1 --- [           main] com.buraktas.Application                 : Started Application in 28.894 seconds (JVM running for 56.711)
2019-12-20 11:10:00.241  INFO 1 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Processing 0
2019-12-20 11:10:01.249  INFO 1 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Processing 1
2019-12-20 11:10:02.250  INFO 1 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Processing 2
2019-12-20 11:10:03.250  INFO 1 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Processing 3
2019-12-20 11:10:04.251  INFO 1 --- [   scheduling-1] com.buraktas.jobs.JobHandler             : Processing 4
```

Now as you can see one of them started to run the job. Lets see the table we created for lock information.

```
mysql> select * from shedlock;
+------------+-------------------------+-------------------------+--------------+
| name       | lock_until              | locked_at               | locked_by    |
+------------+-------------------------+-------------------------+--------------+
| AwesomeJob | 2019-12-20 11:10:15.014 | 2019-12-20 11:10:00.152 | f1177afa6b27 |
+------------+-------------------------+-------------------------+--------------+
1 row in set (0.00 sec)
```

As we can see that lock `AwesomeJob` held by instance `f1177afa6b27` for 15 seconds. After some time, instance `938e2903934b` will hold the lock and run the job.

```
mysql> select * from shedlock;
+------------+-------------------------+-------------------------+--------------+
| name       | lock_until              | locked_at               | locked_by    |
+------------+-------------------------+-------------------------+--------------+
| AwesomeJob | 2019-12-20 11:16:20.000 | 2019-12-20 11:16:00.007 | 938e2903934b |
+------------+-------------------------+-------------------------+--------------+
1 row in set (0.00 sec)
```

I hope this illustration was enough and helpful to show how to use Shedlock for distributed lock mechanism in Spring Applications. Additionally, you can find the whole Dockerized project [here](https://github.com/flexelem/spring-tutorials/tree/master/spring-shedlock)