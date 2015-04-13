---
title: CDI Dependency Injection @PostConstruct and @PreDestroy Example
author: buraktas
layout: post
permalink: /cdi-dependency-injection-postconstruct-predestroy-example/
dsq_needs_sync:
  - 1
categories:
  - Java-EE
tags:
  - cdi
  - dependency-injection
  - java
  - java-ee
comments: true
---
We have seen how to inject cdi beans by using [qualifiers][1] and [producers][2] so far. Additionally, every cdi bean has a lifecycle, and we can initialize and prepare to destroy any managed bean by using two common annotations; <code>@PostConstruct</code> and <code>@PreDestroy</code>. Methods annotated with these annotations are called bean lifecycle callbacks.

<!--more-->

<h3> @PostConstruct </h3>

Due to the [Weld Reference][3] injection and initialization happens in this order;

> <div>
>   <ul>
>     <li>
>       First, the container calls the bean constructor (the default constructor or the one annotated @Inject), to obtain an instance of the bean.
>     </li>
>     <li>
>       Next, the container initializes the values of all injected fields of the bean.
>     </li>
>     <li>
>       Next, the container calls all initializer methods of bean (the call order is not portable, don’t rely on it).
>     </li>
>     <li>
>       Finally, the @PostConstruct method, if any, is called.
>     </li>
>   </ul>
> </div>

So, the purpose of using <code>@PostConstruct</code> is clear; it gives you a chance to initialize injected beans, resources etc.

<pre><code class="language-java">public class Person {

    // you may have injected beans, resources etc.

    public Person() {
        System.out.println("Constructor is called...");
    }

    @PostConstruct
    public void init() {
        System.out.println("@PostConstruct is called...");
    }
}</code>
</pre>

So, the output by injecting a Person bean will be;

<pre>
Constructor is called...
@PostConstruct is called...
</pre>

One important point about <code>@PostConstruct</code> is, it won't be invoked if you try to inject and initialize bean through producer methods. Because using a producer method means that you are programmatically creating, initializing, and injecting your bean with <code>new</code> keyword.

<h3> @PreDestroy </h3>

The second lifecycle call back method for a bean is annotated with <code>@PreDestroy</code>. It prepares the managed bean to get destroyed. It is used for cleaning up operations, or releasing any resources. Finally, the method annotated with <code>@PreDestroy</code> is called by CDI container when the life cycle of bean is finished.

<pre><code class="language-java">public class Person {

    // you may have injected beans, resources etc.

    public Person() {
        System.out.println("Constructor is called...");
    }

    @PostConstruct
    public void init() {
        System.out.println("@PostConstruct is called...");
    }

    @PreDestroy
    public void dispose() {
        System.out.println("@PreDestroy is called...");
    }
}</code>
</pre>

We can see a very simple example about the life cycle a Person bean in a test method.

<pre><code class="language-java">@RunWith(Arquillian.class)
public class PersonTest {

    @Deployment
    public static WebArchive createJavaTestArchive() {
        return ShrinkWrap.create(WebArchive.class)
                .addPackage("predestroy_and_postconstruct")
                .addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml");
    }

    @Inject
    private Person person;

    @Test
    public void test() {
        System.out.println("running test");
    }
}</code>
</pre>

Output;

<pre>
Constructor is called...
@PostConstruct is called...
running test
@PreDestroy is called...
</pre>

Again like producer methods, if we try to destroy a bean with a disposer method then method annotated with <code>@PreDestroy</code> won't be invoked due to the same reason which applies for producer methods.

 [1]: http://www.buraktas.com/create-qualifiers-cdi-beans/
 [2]: http://www.buraktas.com/cdi-dependency-injection-producer-method-example/
 [3]: http://docs.jboss.org/weld/reference/latest/en-US/html/injection.html#_injection_points