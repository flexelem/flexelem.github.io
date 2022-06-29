---
title: Java CDI Dependency Injection Example
author: buraktas
layout: post
permalink: /java-cdi-dependency-injection-example/
dsq_thread_id:
  - 3368735856
categories:
  - java-ee
tags:
  - cdi
  - dependency-injection
  - java
  - java-ee
comments: true
---
CDI (Context and Dependency Injection) is a specification defined in [JSR-299][1]. Major aim is loose coupling by [dependency injection][2].In this tutorial we will see how to use CDI Dependency Injection in java with three different ways;

<div>
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

<!--more-->

We are going to use <code>@Inject</code> alongside <code>@Named</code> annotations from CDI of Java EE. <code>@Named</code> annotation is used for giving names for classes which implements the interface, and it is optional. Otherwise, we can specify alternatives and a default bean by <code>@Alternative</code>, and <code>@Default</code> annotations. However, I am going to show them in this tutorial. Moreover, these are the example interface and implementation classes we are going to use.

<b><u>Notes:</u></b>

<div>
  <ul>
    <li>
      <code>@Named</code> annotation is commonly used if there are more than one implementation for an interface. Thus, it provides to give and inject by their names.
    </li>
    <li>
      If there is only one implementation of an interface and <code>@Named</code> annotation is used, then the name of the bean is determined as camelCase style of class name.
    </li>
    <li>
      We can use <code>@Default</code> and <code>@Alternative</code> annotations instead of giving names to them.
    </li>
    <li>
      If there is only one implementation of an interface, compiler will inject it as the default one. So, there is no need to use <code>@Named</code>, <code>@Default</code> or <code>@Alternative</code> annotations.
    </li>
  </ul>
</div>

<b><u>Important</u></b>

We have to create an empty <code>beans.xml</code> file in <code>src/main/resources/META-INF</code> if it is a jar application or <code>src/main/webapp/WEB-INF</code> if it is a war application.

<pre><code class="language-apacheconf">&lt;beans xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="

http://java.sun.com/xml/ns/javaee
http://java.sun.com/xml/ns/javaee/beans_1_0.xsd"&gt;

&lt;/beans&gt;</code>
</pre>

<b>AutoService.java</b>
<pre><code class="language-java">public interface AutoService {
    void getService();
}</code>
</pre>

<b>BMWAutoService.java</b>

<pre><code class="language-java">@Named("bmwAutoService")
public class BMWAutoService implements AutoService{

    @Override
    public void getService() {
        System.out.println("You chose BMW auto service");
    }
}</code>
</pre>

<b>FordAutoService.java</b>

<pre><code class="language-java">@Named("fordAutoService")
public class Ford implements AutoService{

    @Override
    public void getService() {
        System.out.println("You chose Ford auto service");
    }
}</code>
</pre>

<b>HondaAutoService.java</b>

<pre><code class="language-java">@Named("hondaAutoService")
public class HondaAutoService implements AutoService{

    @Override
    public void getService() {
        System.out.println("You chose Honda auto service");
    }
}</code>
</pre>

<b>AutoServiceCaller.java</b>
<pre><code class="language-java">public interface AutoServiceCaller {
    void callAutoService();
}</code>
</pre>

<h3> 1. Injection Through Fields </h3>

Beans are injected through fields.

<pre><code class="language-java">public class AutoServiceCallerImp implements AutoServiceCaller{

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
}</code>
</pre>

<h3> 2. Injection Through Setter Methods </h3>

We can also inject our beans via setters

<pre><code class="language-java">public class AutoServiceCallerImp implements AutoServiceCaller{

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
}</code>
</pre>

<h3> 3. Injection Through Constructor </h3>

Finally, beans can be injected through the constructor of the class.

<pre><code class="language-java">public class AutoServiceCallerImp implements AutoServiceCaller{

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
}</code>
</pre>

 [1]: https://jcp.org/en/jsr/detail?id=299
 [2]: http://martinfowler.com/articles/injection.html
