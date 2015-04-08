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
---
In this tutorial we will implement a simple web service with RESTEasy implementation -it&#8217;s an implementation of the JAX-RS specification by JBoss- without using a web.xml file. I&#8217;ve used

<div class="bullet list">
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

tools and technologies. The purpose of not using web.xml file is the fact that we don&#8217;t have to deal with servlet mappings within a xml file. So, this feature comes with any Servlet 3.x complaint container.

#### 1. Creating a Maven Project

First thing to do is creating a maven project, so enter the following code into command line;

<pre class="lang:default decode:true " >mvn archetype:generate -DgroupId=buraktas.com -DartifactId=rest-helloworld</pre>

command line will prompt some questions while it is initializing your project like; choosing an archetype version of maven. If you continue to press Enter it will automatically set to default options.

<div class="admonition important">
  <p class="first admonition-title">
    IMPORTANT
  </p>
  
  <p>
    <packaging>jar</packaging> is set on default in pom.xml, however, we will use <b>war</b> as packaging, so please change it to war.
  </p>
</div>

After we created our maven project we have to import initialized project into eclipse via *import&#8211;>Existing Maven Projects*

<div class="hentry img">
  <a href="http://buraktas.com/wp-content/uploads/2014/03/eclipse_import-maven_project.png"><img src="http://buraktas.com/wp-content/uploads/2014/03/eclipse_import-maven_project.png" alt="eclipse_import-maven_project" width="623" height="481" class="aligncenter size-full wp-image-87" /></a>
</div>

Then browse to the directory where you created your maven project, and select the root folder.

<div class="hentry img">
  <a href="http://buraktas.com/wp-content/uploads/2014/03/import-maven-projects.png"><img src="http://buraktas.com/wp-content/uploads/2014/03/import-maven-projects.png" alt="import-maven-projects" width="487" height="484" class="aligncenter size-full wp-image-162" /></a>
</div>

Now our directory structure should look like this

<div class="hentry img">
  <a href="http://buraktas.com/wp-content/uploads/2014/03/directory.png"><img src="http://buraktas.com/wp-content/uploads/2014/03/directory.png" alt="directory" width="290" height="343" class="aligncenter size-full wp-image-152" /></a>
</div>

#### 2. Dependencies 

We are going to use 2 dependencies which **junit** is optional cause I&#8217;m not going to show any test methods in this tutorial 

<pre class="lang:xhtml decode:true " >&lt;dependencies&gt;
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
&lt;/dependencies&gt;</pre>

<div class="bullet list">
  <ul>
    <li>
      <b>resteasy-jaxrs</b> is defined with <strong><scope>provided</scope></strong> so that JBoss will provide this dependency in runtime. Also here you can find more information about <a href="http://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html" title="maven dependency scopes" target="_blank">maven dependency scopes</a>
    </li>
    <li>
      <b>junit</b> is for testing and it comes with default maven project </ul> </div> <p>
        Also we have to add 2 maven plugins ;
      </p>
      
      <pre class="lang:default mark:4-9,13-19 decode:true " >&lt;build&gt;
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
&lt;/build&gt;</pre>
      
      <div class="bullet list">
        <ul>
          <li>
            <b>maven-compiler-plugin</b> is for compiling your Java sources.
          </li>
          <li>
            <b>maven-war-plugin</b> is responsible for handling with dependencies and packaging them into a web application archive.
          </li>
          <ul>
            <li>
              <blogcode><failOnMissingWebXml>false</failOnMissingWebXml></blogcode> is the key point to let maven build our web application without using web.xml,otherwise mavens give an error, and it will look like;
            </li>
            <div class="admonition warning">
              <p class="first admonition-title">
                Error
              </p>
              
              <p>
                Failed to execute goal org.apache.maven.plugins:maven-war-plugin:2.3:war (default-war) on project rest-helloworld: Error assembling WAR: webxml attribute is required (or pre-existing WEB-INF/web.xml if executing in update mode)
              </p></p>
            </div>
            
            <li>
              <blogcode><warName>rest-helloworld</warName></blogcode> element will define root URI of our web application. For instance, our every resource will located on <b>localhost:8080/rest-helloworld</b>.
            </li>
          </ul>
        </ul>
      </div>
      
      <h4>
        3. Developing REST Service
      </h4>
      
      <p>
        We have to implement a base/root application which defines the base URI for all other sub-resources. Then, this root class has to extended from `Application`, with `@ApplicationPath(&#8220;&#8221;)` annotation.
      </p>
      
      <pre class="lang:java decode:true " >
package com.buraktas.application;

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
}</pre>
      
      <div class="bullet list">
        <ul>
          <li>
            <blogcode>@ApplicationPath(&#8220;&#8221;)</blogcode> annotation is mandatory, however, path URI is optional. It defines the root URI for all other resources. For instance, if I specify it as <b>root-service</b>, my other resources will be reachable from <blogcode>http://mydomain.com/rest-helloworld/root-service</blogcode>. Otherwise, you will face with an error which says;
          </li>
          <div class="admonition warning">
            <p class="first admonition-title">
              ERROR
            </p>
            
            <p>
              JBAS011203: No Servlet mappings found for JAX-RS application: com.buraktas.application.BaseApplication either annotate it with @ApplicationPath or add a servlet-mapping in web.xml
            </p></p>
          </div>
          
          <li>
            Overriding <b>getClasses</b> method is optional, if you are not going to map classes manually.
          </li>
        </ul>
      </div>
      
      <p>
        Finally, we can implement our simplest and first REST service now which is only composed by a <b>GET</b> method and returns a <b>Hello World</b> string.
      </p>
      
      <pre class="lang:java decode:true " >
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
}</pre>
      
      <p>
        Now our web-service is ready for getting build by maven with;
      </p>
      
      <pre class="lang:default decode:true " >mvn clean install</pre>
      
      <p>
        After the process is successfully finished, maven will create <b>.war</b> file under target directory. When we deploy it on JBoss, we could use our implemented resource on <b>http://localhost:8080/rest-helloworld/helloworld</b> which will print <b>Hello World</b>
      </p>
      
      <p>
        <a href="http://buraktas.com/wp-content/uploads/2014/03/localhost_hello_world.png"><img src="http://buraktas.com/wp-content/uploads/2014/03/localhost_hello_world.png" alt="localhost_hello_world" width="968" height="69" class="aligncenter size-full wp-image-35" /></a>
      </p>
      
      <p>
        Here you can find the original source code.<br /> <a href="https://github.com/flexelem/resteasy_hello_world" title="RESTEasy Hello World" target="_blank">RESTEasy Hello World</a>
      </p>