---
title: Arquillian Example for CDI Dependency Injection
author: buraktas
layout: post
permalink: /arquillian-example-for-cdi-beans/
deneme:
  - |
    @Named("hondaAutoService")
    public class HondaAutoService implements AutoService{
    
        @Override
        public void getService() {
            System.out.println("You chose Honda auto service");
        }
    }
dsq_thread_id:
  - 3421824012
categories:
  - Java-EE
tags:
  - arquillian
  - cdi
  - dependency-injection
  - java
  - java-ee
---
Arquillian is a platform which provides integration tests by deploying, running containers so that we can easily use cdi beans in tests. In this tutorial we will see how to inject and use cdi beans in test classes by running Arquillian.

As a first step we have to add Arquillian core library in our pom.xml in the <blognewcode>dependencyManagement</blognewcode> block. Well it is completely optional, and you can add it in <blognewcode>dependencies</blognewcode> section.

<pre class="lang:default decode:true " >&lt;dependencyManagement&gt;
    &lt;dependencies&gt;
        &lt;dependency&gt;
            &lt;groupId&gt;org.jboss.arquillian&lt;/groupId&gt;
            &lt;artifactId&gt;arquillian-bom&lt;/artifactId&gt;
            &lt;version&gt;1.1.5.Final&lt;/version&gt;
            &lt;scope&gt;import&lt;/scope&gt;
            &lt;type&gt;pom&lt;/type&gt;
        &lt;/dependency&gt;
    &lt;/dependencies&gt;
&lt;/dependencyManagement&gt;</pre>

Afterwards, add Arquillian Junit container in our pom.xml in <blognewcode>dependencies</blognewcode>.

<pre class="lang:default decode:true " >&lt;dependency&gt;
        &lt;groupId&gt;org.jboss.arquillian.junit&lt;/groupId&gt;
        &lt;artifactId&gt;arquillian-junit-container&lt;/artifactId&gt;
        &lt;scope&gt;test&lt;/scope&gt;
    &lt;/dependency&gt;</pre>

Finally, we have to add a container adapter and cdi dependency for that container.

<pre class="lang:java decode:true " >&lt;!-- Arquillian container adapter for the target container --&gt;
    &lt;dependency&gt;
        &lt;groupId&gt;org.jboss.arquillian.container&lt;/groupId&gt;
        &lt;artifactId&gt;arquillian-weld-ee-embedded-1.1&lt;/artifactId&gt;
        &lt;version&gt;1.0.0.CR3&lt;/version&gt;
        &lt;scope&gt;test&lt;/scope&gt;
    &lt;/dependency&gt;

    &lt;!-- CDI dependency for container --&gt;
    &lt;dependency&gt;
        &lt;groupId&gt;org.jboss.weld&lt;/groupId&gt;
        &lt;artifactId&gt;weld-core&lt;/artifactId&gt;
        &lt;version&gt;1.1.5.Final&lt;/version&gt;
        &lt;scope&gt;test&lt;/scope&gt;
    &lt;/dependency&gt;

    &lt;dependency&gt;
        &lt;groupId&gt;junit&lt;/groupId&gt;
        &lt;artifactId&gt;junit&lt;/artifactId&gt;
        &lt;version&gt;4.11&lt;/version&gt;
        &lt;scope&gt;test&lt;/scope&gt;
    &lt;/dependency&gt;</pre>

Here you can find very simple classes that I am going to use for dependency injection with the directory structure.

<pre class="lang:default decode:true " >.
├── pom.xml
├── src
   ├── main
   ├── java
   │   └── com
   │       └── buraktas
   │           ├── autoservice
   │               ├── AutoService.java
   │               ├── BMWAutoService.java
   │               ├── HondaAutoService.java
   │           
   ├── resources
   └── webapp
       └── WEB-INF
           └── beans.xml</pre>

<div class="class-names">
  <b>AutoService.java</b></p> 
  
  <pre class="lang:java decode:true " >public interface AutoService {
    String getService();
}</pre>
</div>

<div class="class-names">
  <b>BMWAutoService.java</b></p> 
  
  <pre class="lang:java decode:true " >@Named("bmwAutoService")
public class BMWAutoService implements AutoService{

    @Override
    public String getService() {
        return "You chose BMW auto service";
    }
}</pre>
</div>

<div class="class-names">
  <b>HondaAutoService.java</b></p> 
  
  <pre class="lang:java decode:true " >@Named("hondaAutoService")
public class HondaAutoService implements AutoService{

    @Override
    public String getService() {
        return "You chose Honda auto service";
    }
}</pre>
</div>

Finally, we can implement and test our cdi beans by using Arquillian platform.

<pre class="lang:java decode:true " >@RunWith(Arquillian.class)
public class AutoServiceTest {

    @Deployment
    public static WebArchive createJavaTestArchive() {

        return ShrinkWrap.create(WebArchive.class)
                .addPackage("com.buraktas.autoservice")
                .addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml");
    }

    @Inject
    @Named("bmwAutoService")
    private AutoService bmwAutoService;

    @Inject
    @Named("hondaAutoService")
    private AutoService hondaAutoService;

    @Test
    public void bmwAutoServiceTest() {
        Assert.assertEquals("You chose BMW auto service", bmwAutoService.getService());
    }

    @Test
    public void hondaAutoServiceTest() {
        Assert.assertEquals("You chose Honda auto service", hondaAutoService.getService());
    }
}</pre>

<div class="bullet list">
  <b><u>Notes</u></b></p> 
  
  <ul>
    <li>
      <blognewcode>@RunWith(Arquillian.class)</blognewcode> provides to run our test class with Arquillian features.
    </li>
    <li>
      We have to create a public static method to bootstrap a virtual context for our beans with <blognewcode>@Deployment</blognewcode> annotation. This method returns a <blognewcode>ShrinkWrap</blognewcode> archive.
    </li>
    <li>
      The classes which we are going to inject and test should be added into archive. On the other hand, we can add package(s) instead of adding classes.
    </li>
    <li>
      As specified in this <a href="http://www.buraktas.com/java-dependency-injection-example/">article</a> we have to create a beans.xml file to use CDI beans. So, the location of the beans.xml file will be different due to our project type.
    </li>
  </ul>
</div>

Finally, we will notice that our tests will run without any issues.