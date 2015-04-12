---
title: RESTEasy example without using a web.xml
author: buraktas
layout: post
permalink: /resteasy-example-without-using-a-web-xml/
dsq_thread_id:
  - 2555079485
categories:
  - jax-rs
tags:
  - java
  - jax-rs
  - rest
  - resteasy
  - web-service
comments: true
---
In this tutorial we will implement a simple web service with RESTEasy implementation -it's an implementation of the JAX-RS specification by JBoss- without using a web.xml file. I've used

<div>
  <ul>
    <li>
      Eclipse 4.3.0
    </li>
    <li>
      JBoss AS 7.1.1.Final
    </li>
    <li>
      Maven 3.1.1
    </li>
    <li>
      RESTEasy 3.0.6.Final
    </li>
  </ul>
</div>

tools and technologies. The purpose of not using web.xml file is the fact that we don't have to deal with servlet mappings within a xml file. So, this feature comes with any Servlet 3.x complaint container.

<h3> 1. Creating a Maven Project </h3>

First thing to do is creating a maven project, so enter the following code into command line;

```
mvn archetype:generate -DgroupId=buraktas.com -DartifactId=rest-helloworld
```

command line will prompt some questions while it is initializing your project like; choosing an archetype version of maven. If you continue to press Enter it will automatically set to default options.

<blockquote>
&lt;packaging&gt;jar&lt;/packaging&gt; is set on default in pom.xml, however, we will use <b>war</b> as packaging, so please change it to war.
</blockquote>

After we created our maven project we have to import initialized project into eclipse by <b>import->Existing Maven Projects</b>

![]({{ site.url }}/public/images/2014/03/eclipse_import-maven_project.png)

Then browse to the directory where you created your maven project, and select the root folder.

Now our directory structure should look like this

<a href="{{ site.url }}/public/images/2014/03/directory.png"><img src="{{ site.url }}/public/images/2014/03/directory.png" alt="directory" width="290" height="343" class="aligncenter size-full" /></a>

<h3> 2. Dependencies </h3>

We are going to use 2 dependencies which <code>junit</code> is optional cause I&#8217;m not going to show any test methods in this tutorial 

<pre>
<code class="language-default">&lt;dependencies&gt;
	&lt;dependency&gt;
		&lt;groupId&gt;junit&lt;/groupId&gt;
		&lt;artifactId&gt;junit&lt;/artifactId&gt;
		&lt;version&gt;3.8.1&lt;/version&gt;
		&lt;scope&gt;test&lt;/scope&gt;
	&lt;/dependency&gt;

	&lt;dependency&gt;
		&lt;groupId&gt;org.jboss.resteasy&lt;/groupId&gt;
		&lt;artifactId&gt;resteasy-jaxrs&lt;/artifactId&gt;
		&lt;version&gt;3.0.6.Final&lt;/version&gt;
		&lt;scope&gt;provided&lt;/scope&gt;
	&lt;/dependency&gt;
&lt;/dependencies&gt;</code>
</pre>

<div>
  <ul>
    <li>
      <b>resteasy-jaxrs</b> is defined with <code>&lt;scope&gt;provided&lt;/scope&gt;</code> so that JBoss will provide this dependency in runtime. Also here you can find more information about <a href="http://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html" title="maven dependency scopes" target="_blank">maven dependency scopes</a>
    </li>

    <li>
      <b>junit</b> is for testing and it comes with default maven project.
    </li> 
    </ul>
</div> 

Also we have to add 2 maven plugins ;
      
<pre>
<code class="language-default">&lt;build&gt;
	&lt;plugins&gt;
		&lt;plugin&gt;
			&lt;groupId&gt;org.apache.maven.plugins&lt;/groupId&gt;
			&lt;artifactId&gt;maven-compiler-plugin&lt;/artifactId&gt;
			&lt;configuration&gt;
				&lt;source&gt;1.7&lt;/source&gt;
				&lt;target&gt;1.7&lt;/target&gt;
			&lt;/configuration&gt;
		&lt;/plugin&gt;

		&lt;plugin&gt;
			&lt;groupId&gt;org.apache.maven.plugins&lt;/groupId&gt;
			&lt;artifactId&gt;maven-war-plugin&lt;/artifactId&gt;
			&lt;version&gt;2.3&lt;/version&gt;
			&lt;configuration&gt;
				&lt;failOnMissingWebXml&gt;false&lt;/failOnMissingWebXml&gt;
				&lt;warName&gt;rest-helloworld&lt;/warName&gt;
			&lt;/configuration&gt;
		&lt;/plugin&gt;
	&lt;/plugins&gt;
&lt;/build&gt;</code>
</pre>
      
<div>
  <ul>
    <li>
      <b>maven-compiler-plugin</b> is for compiling your Java sources.
    </li>

    <li>
      <b>maven-war-plugin</b> is responsible for handling with dependencies and packaging them into a web application archive.
    </li>

    <ul>
      <li><code><failOnMissingWebXml>false</failOnMissingWebXml></code> is the key point to let maven build our web application without     using web.xml,otherwise mavens give an error, and it will look like;</li>
      <pre>Failed to execute goal org.apache.maven.plugins:maven-war-plugin:2.3:war (default-war) on project rest-helloworld: Error assembling WAR: webxml attribute is required (or pre-existing WEB-INF/web.xml if executing in update mode)</pre>
      <li><code>&lt;warName&gt;rest-helloworld&lt;/warName&gt;</code> element will define root URI of our web application. For instance, our every resource will located on <b>localhost:8080/rest-helloworld</b>.</li>
    </ul>
  </ul>
</div>
      
<h3> 3. Developing a REST Service </h3>
We have to implement a base/root application which defines the base URI for all other sub-resources. Then, this root class has to extended from <code>Application</code>, with <code>@ApplicationPath(&#8220;&#8221;)</code> annotation.
      
<pre>
<code class="language-java">package com.buraktas.application;

import java.util.Collections;
import java.util.Set;
import javax.ws.rs.ApplicationPath;
import javax.ws.rs.core.Application;

@ApplicationPath("")
public class BaseApplication extends Application {

    @Override
    public Set&lt;Class&lt;?&gt;&gt; getClasses() {
        return Collections.emptySet();
    }
}</code>
</pre>
      
<div>
  <ul>
    <li>
      <code>@ApplicationPath(&#8220;&#8221;)</code> annotation is mandatory, however, path URI is optional. It defines the root URI for all other resources. For instance, if I specify it as <b>root-service</b>, my other resources will be reachable from <b>http://mydomain.com/rest-helloworld/root-service</b>. Otherwise, you will face with an error which says;
    </li>
<pre>
JBAS011203: No Servlet mappings found for JAX-RS application: com.buraktas.application.BaseApplication either annotate it with @ApplicationPath or add a servlet-mapping in web.xml
</pre>
          
    <li>
      Overriding <b>getClasses</b> method is optional, if you are not going to map classes manually.
    </li>
  </ul>
</div>
      
Finally, we can implement our simplest and first REST service now which is only composed by a <b>GET</b> method and returns a <b>Hello World</b> string.
      
<pre>
<code class="language-java">
package com.buraktas.helloworld;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.core.Response;

@Path("")
public class HelloWorld {

    @GET
    @Path("/helloworld")
    public Response getHelloWorld() {
        String value = "Hello World";
        return Response.status(200).entity(value).build();
    }
}</code>
</pre>
      
Now our web-service is ready for getting build by maven with;
      
```
mvn clean install
```

After the process is successfully finished, maven will create <b>.war</b> file under target directory. When we deploy it on JBoss, we could use our implemented resource on <b>http://localhost:8080/rest-helloworld/helloworld</b> which will print <b>Hello World</b>
      
<a href="{{ site.url }}/public/images/2014/03/localhost_hello_world.png"><img src="{{ site.url }}/public/images/2014/03/localhost_hello_world.png" alt="localhost_hello_world" width="968" height="69" class="aligncenter size-full" /></a>
      
Here you can find the original source code. <br>
<a href="https://github.com/flexelem/resteasy_hello_world" title="RESTEasy Hello World" target="_blank">RESTEasy Hello World</a>