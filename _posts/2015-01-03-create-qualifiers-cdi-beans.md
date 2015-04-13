---
title: Create Qualifiers for CDI Beans
author: buraktas
layout: post
permalink: /create-qualifiers-cdi-beans/
dsq_thread_id:
  - 3421510595
categories:
  - Java-EE
tags:
  - cdi
  - dependency-injection
  - java
  - java-annotations
  - java-ee
  - qualifier
comments: true
---
As I described in my previous [post][1], we can define and inject cdi beans by <code>@Named</code> annotation. Well according to the CDI specification (JSR-299) injecting beans by their names is legacy and tend to cause issues (if a bean is tried to be injected by an undefined/wrong name then we will get errors which are hard to find). Thus, we are going to create different annotations (qualifiers) to inject different type of beans from same interface. Moreover, we will use Weld CDI implementation.

<!--more-->

<h3> Create a Qualifier </h3>

Creating your own qualifier is easy. We have to create a standard annotation and mark it with <code>@Qualifier</code>.

<pre><code class="language-java">import javax.inject.Qualifier;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import static java.lang.annotation.ElementType.*;

@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({FIELD, METHOD, TYPE, PARAMETER})
public @interface &lt;annotation_name&gt; {
}</code>
</pre>

So, according to the qualifier template we will need three annotations; Bmw, Honda, and Ford.

<b>Bmw.java</b>
  
<pre><code class="language-java">@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({FIELD, METHOD, TYPE, PARAMETER})
public @interface Bmw {
}</code>
</pre>

<b>Honda.java</b>
  
<pre><code class="language-java">@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({FIELD, METHOD, TYPE, PARAMETER})
public @interface Honda {
}</code>
</pre>

<b>Ford.java</b>
  
<pre><code class="language-java">@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({FIELD, METHOD, TYPE, PARAMETER})
public @interface Ford {
}</code>
</pre>

<b><u>Notes</u></b>

<div>
  <ul>
    <li>
      <b><em>Retention</b></em> determines at what point an annotation is discarded. More explanation can be seen from <a href="http://stackoverflow.com/questions/3107970/annotations-retention-policy">here.</a>
    </li> 
      
    <li>
      <b><em>Target</b></em> defines the type of our annotation. Explanation with clear examples can be seen from <a href="http://stackoverflow.com/questions/3550292/java-annotations-elementtype">here.</a>
    </li>
  </ul>
</div> 
        
I will again use the same code examples which I used in my previous posts. There is one interface called AutoService and three implementations of it.

<b>AutoService.java</b>
          
<pre><code class="language-java">public interface AutoService {
    String getService();
}</code>
</pre>

<b>BMWAutoService.java</b>
          
<pre><code class="language-java" >@Bmw
public class BMWAutoService implements AutoService{

    @Override
    public String getService() {
        return "You chose BMW auto service";
    }
}</code>
</pre>

<b>FordAutoService.java</b>
          
<pre><code class="language-java">@Ford
public class FordAutoService implements AutoService{

    @Override
    public String getService() {
        return "You chose Ford auto service";
    }
}</code>
</pre>

<b>HondaAutoService.java</b>
          
<pre><code class="language-java">@Honda
public class HondaAutoService implements AutoService{

    @Override
    public String getService() {
        return "You chose Honda auto service";
    }
}</code>
</pre>

Now we are completely ready to inject our cdi beans by their qualifiers! So, lets see three different injection examples &#8211; Field, Constructor and Setter method injection -.

<h3> 1. Injection Through Fields </h3>
        
<pre><code class="language-java" >public class Example {
 
    @Inject
    @Bmw
    private AutoService bmwAutoService;
 
    @Inject
    @Honda
    private AutoService hondaAutoService;
 
    @Inject
    @Ford
    private AutoService fordAutoService;
 
    ...
}</code>
</pre>
        
<h3> 2.Injection Through Setter Methods </h3>
        
<pre><code class="language-java">public class Example {
 
    private AutoService bmwAutoService;
    private AutoService hondaAutoService;
    private AutoService fordAutoService;
 
    @Inject
    public void setBmwAutoService(@Bmw AutoService bmwAutoService) {
        this.bmwAutoService = bmwAutoService;
    }
 
    @Inject
    public void setHondaAutoService(@Honda AutoService hondaAutoService) {
        this.hondaAutoService = hondaAutoService;
    }
 
    @Inject
    public void setFordAutoService(@Ford AutoService fordAutoService) {
        this.fordAutoService = fordAutoService;
    }
   
   ...
}</code>
</pre>

<h3> 3.Injection Through Constructor </h3>
        
<pre><code class="language-java">public class AutoServiceCallerImp 
 
    private AutoService bmwAutoService;
    private AutoService hondaAutoService;
    private AutoService fordAutoService;
 
    @Inject
    public AutoServiceCallerImp(@Bmw AutoService bmwAutoService,
                             @Honda AutoService hondaAutoService,
                             @Ford AutoService fordAutoService) {
 
        this.bmwAutoService = bmwAutoService;
        this.fordAutoService = fordAutoService;
        this.hondaAutoService = hondaAutoService;
    }

    ...
}</code>
</pre>

There is also a second way to use qualifiers to prevent from annotation overflow. In normal life we have tens of auto models. So, instead of creating lots of qualifiers, we can only create one qualifier and enum to define the type of annotation we want to use -enum is optional we can specify types by integer, string etc-. With this way we will be able to handle lots of annotations easily. Let us see;

<b>AutoType.java</b>
          
<pre><code class="language-java">public enum AutoType {
    Bmw, Ford, Honda
}</code>
</pre>


<b>Auto.java</b> 
          
<pre><code class="language-java">@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({METHOD, TYPE, PARAMETER, FIELD})
public @interface Auto {
    AutoType type() default AutoType.Bmw;
}</code>
</pre>

<code>default</code> property is optional here. It means if we dont specify a type then the default value will be used. Now let's an example by field injection.

<pre><code class="language-java">public class AutoServiceCallerImp implements AutoServiceCaller{

    @Inject
    @Auto(type = AutoType.Bmw)
    private AutoService bmwAutoService;

    @Inject
    @Auto(type = AutoType.Honda)
    private AutoService hondaAutoService;

    @Inject
    @Auto(type = AutoType.Ford)
    private AutoService fordAutoService;

    ...
}</code>
</pre>

So, as we can see we define the type of our cdi beans just by giving type value, and we have only used <code>@Auto</code> annotation.

 [1]: {{ site.url }}/java-cdi-dependency-injection-example/