---
title: Java CDI Dependency Injection Example
author: buraktas
layout: post
permalink: /java-cdi-dependency-injection-example/
dsq_thread_id:
  - 3368735856
categories:
  - Java-EE
tags:
  - cdi
  - dependency-injection
  - java
  - java-ee
---
CDI (Context and Dependency Injection) is a specification defined in [JSR-299][1]. Major aim is loose coupling by [dependency injection][2].In this tutorial we will see how to use CDI Dependency Injection in java with three different ways;

<div class="bullet list">
  <ul>
    <li>
      Field injection
    </li>
    <li>
      Constructor injection
    </li>
    <li>
      Setter method injection
    </li>
  </ul>
</div>

We are going to use <blognewcode>@Inject</blognewcode> alongside <blognewcode>@Named</blognewcode> annotations from CDI of Java EE. <blognewcode>@Named</blognewcode> annotation is used for giving names for classes which implements the interface, and it is optional. Otherwise, we can specify alternatives and a default bean by <blognewcode>@Alternative</blognewcode>, and <blognewcode>@Default</blognewcode> annotations. However, I am going to show them in this tutorial. Moreover, these are the example interface and implementation classes we are going to use.

**<u>Notes:</u>**

<div class="bullet list">
  <ul>
    <li>
      <blognewcode>@Named</blognewcode> annotation is commonly used if there are more than one implementation for an interface. Thus, it provides to give and inject by their names.
    </li>
    <li>
      If there is only one implementation of an interface and <blognewcode>@Named</blognewcode> annotation is used, then the name of the bean is determined as camelCase style of class name.
    </li>
    <li>
      We can use <blognewcode>@Default</blognewcode> and <blognewcode>@Alternative</blognewcode> annotations instead of giving names to them.
    </li>
    <li>
      If there is only one implementation of an interface, compiler will inject it as the default one. So, there is no need to use <blognewcode>@Named</blognewcode>, <blognewcode>@Default</blognewcode> or <blognewcode>@Alternative</blognewcode> annotations.
    </li>
  </ul>
</div>

**<u>Important</u>**

We have to create an empty <blognewcode>beans.xml</blognewcode> file in <blognewcode>src/main/resources/META-INF</blognewcode> if it is a jar application or <blognewcode>src/main/webapp/WEB-INF</blognewcode> if it is a war application.

<pre class="lang:xhtml decode:true " >&lt;beans xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="

http://java.sun.com/xml/ns/javaee

http://java.sun.com/xml/ns/javaee/beans_1_0.xsd"&gt;

&lt;/beans&gt;
</pre>

<div class="class-names">
  <b>AutoService.java</b> </p> 
  
  <pre class="lang:java decode:true " >public interface AutoService {
    void getService();
}</pre>
</div>

<div class="class-names">
  <b>BMWAutoService.java</b></p> 
  
  <pre class="lang:java mark:1 decode:true " >@Named("bmwAutoService")
public class BMWAutoService implements AutoService{

    @Override
    public void getService() {
        System.out.println("You chose BMW auto service");
    }
}</pre>
</div>

<div class="class-names">
  <b>FordAutoService.java</b> </p> 
  
  <pre class="lang:java mark:1 decode:true " >@Named("fordAutoService")
public class Ford implements AutoService{

    @Override
    public void getService() {
        System.out.println("You chose Ford auto service");
    }
}</pre>
</div>

<div class="class-names">
  <b>HondaAutoService.java</b></p> 
  
  <pre class="lang:java mark:1 decode:true " >@Named("hondaAutoService")
public class HondaAutoService implements AutoService{

    @Override
    public void getService() {
        System.out.println("You chose Honda auto service");
    }
}</pre>
</div>

<div class="class-names">
  <b>AutoServiceCaller.java</b></p> 
  
  <pre class="lang:java decode:true " >public interface AutoServiceCaller {

    void callAutoService();
}</pre>
  
  <h3>
    1. Injection Through Fields
  </h3>
  
  <p>
    Beans are injected through fields.
  </p>
  
  <pre class="lang:java decode:true " >public class AutoServiceCallerImp implements AutoServiceCaller{

    @Inject
    @Named("bmwAutoService")
    private AutoService bmwAutoService;

    @Inject
    @Named("hondaAutoService")
    private AutoService hondaAutoService;

    @Inject
    @Named("fordAutoService")
    private AutoService fordAutoService;

    @Override
    public void callAutoService() {
        // get bmw's auto service
        bmwAutoService.getService();

        // get ford's auto service
        fordAutoService.getService();

        // get honda's auto service
        hondaAutoService.getService();
    }
}</pre>
  
  <h3>
    2. Injection Through Setter Methods
  </h3>
  
  <p>
    We can also inject our beans via setters
  </p>
  
  <pre class="lang:java decode:true " >public class AutoServiceCallerImp implements AutoServiceCaller{

    private AutoService bmwAutoService;
    private AutoService hondaAutoService;
    private AutoService fordAutoService;

    @Override
    public void callAutoService() {

        bmwAutoService.getService();
        fordAutoService.getService();
        hondaAutoService.getService();
    }

    @Inject
    public void setBmwAutoService(@Named("bmwAutoService") AutoService bmwAutoService) {
        this.bmwAutoService = bmwAutoService;
    }

    @Inject
    public void setHondaAutoService(@Named("hondaAutoService") AutoService hondaAutoService) {
        this.hondaAutoService = hondaAutoService;
    }

    @Inject
    public void setFordAutoService(@Named("fordAutoService") AutoService fordAutoService) {
        this.fordAutoService = fordAutoService;
    }
}</pre>
  
  <h3>
    3. Injection Through Constructor
  </h3>
  
  <p>
    Finally, beans can be injected through the constructor of the class.
  </p>
  
  <pre class="lang:java decode:true " >public class AutoServiceCallerImp implements AutoServiceCaller{

    private AutoService bmwAutoService;
    private AutoService hondaAutoService;
    private AutoService fordAutoService;

    @Inject
    public AutoServiceCallerImp(@Named("bmwAutoService") AutoService bmwAutoService,
                             @Named("hondaAutoService") AutoService hondaAutoService,
                             @Named("fordAutoService") AutoService fordAutoService) {

        this.bmwAutoService = bmwAutoService;
        this.fordAutoService = fordAutoService;
        this.hondaAutoService = hondaAutoService;
    }

    @Override
    public void callAutoService() {
        // get bmw's auto service
        bmwAutoService.getService();

        // get ford's auto service
        fordAutoService.getService();

        // get honda's auto service
        hondaAutoService.getService();
    }
}</pre>

 [1]: https://jcp.org/en/jsr/detail?id=299
 [2]: http://martinfowler.com/articles/injection.html