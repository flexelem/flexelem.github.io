---
title: CDI Dependency Injection Producer Method Example
author: buraktas
layout: post
permalink: /cdi-dependency-injection-producer-method-example/
dsq_thread_id:
- 3519616668
categories:
- Java-EE
tags:
- cdi
- dependency-injection
- java
- java-annotations
- java-ee
---

<p>
  In this tutorial we will see how to use producer methods for injecting CDI beans by
  using <blognewcode>@Produces</blognewcode> annotation. We will use Weld CDI implementation.
  Using producer methods provides a programmatic way to solve disambiguation by using injection
  points. We have already seen how to solve disambiguation by using qualifiers in this
  <a href="{{ site.url }}/create-qualifiers-cdi-beans/">tutorial</a>.
  As a first step, we will create a disambiguation.
</p>

<p>
  Here we have one interface (Bank), one implementation (BankOfAmerica) ,and a simple producer method.
</p>

<pre class="line-numbers"><code class="language-java">
public interface Bank {
    public void withdrawal();
    public void deposit();
}</code>
</pre>

<pre class="line-numbers"><code class="language-java">
public class BankOfAmerica implements Bank {

    @Override
    public void withdrawal() {
        System.out.println("Withdrawal from Bank of America");
    }

    @Override
    public void deposit() {
        System.out.println("Deposit to Bank of America");
    }
}</code>
</pre>

<pre class="line-numbers"><code class="language-java">
public class BankFactory {

    @Produces
    public Bank createBank() {
        return new BankOfAmerica();
    }
}</code>
</pre>

<p>
  Now, if we try to inject a bank bean like
</p>

<pre><code class="language-java">
@Inject
private Bank bankOfAmerica
</code></pre>

<p>
  Our container will not be able to decide to return correct implementation of Bank bean and will throw a Ambiguous dependencies error.
</p>

<pre><code class="language-markup">
org.jboss.weld.exceptions.DeploymentException: Exception List with 2 exceptions:
Exception 0 :
org.jboss.weld.exceptions.DeploymentException: WELD-001409 Ambiguous dependencies for type [Bank] with qualifiers [@Default] at injection point [[field] @Inject private produces_example.ProducesExample.bankOfAmerica]. Possible dependencies [[Managed Bean [class produces_example.BankOfAmerica] with qualifiers [@Default @Any], Producer Method [Bank] with qualifiers [@Default @Any] declared as [[method] @Produces public produces_example.BankFactory.createBank()]]]
  at org.jboss.weld.bootstrap.Validator.validateInjectionPoint(Validator.java:277)
  at org.jboss.weld.bootstrap.Validator.validateInjectionPoint(Validator.java:243)
  at org.jboss.weld.bootstrap.Validator.validateBean(Validator.java:106)
  at org.jboss.weld.bootstrap.Validator.validateRIBean(Validator.java:126)
  at org.jboss.weld.bootstrap.Validator.validateBeans(Validator.java:345)
  at org.jboss.weld.bootstrap.Validator.validateDeployment(Validator.java:330)
  at org.jboss.weld.bootstrap.WeldBootstrap.validateBeans(WeldBootstrap.java:366)
  at org.jboss.arquillian.container.weld.ee.embedded_1_1.mock.TestContainer.startContainer(TestContainer.java:257)
  at org.jboss.arquillian.container.weld.ee.embedded_1_1.WeldEEMockContainer.deploy(WeldEEMockContainer.java:98)
</code></pre>

<p>
  Now it is time solve our simple ambiguous dependency problem.
  We are going to create a qualifier called <blognewcode>@BankProducer</blognewcode>.
</p>

<pre><code class="language-java">
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({FIELD, METHOD, TYPE})
public @interface BankProducer {
}</code>
</pre>

<p>
   If we annotate our producer method with
   <blognewcode>BankProducer</blognewcode>
   qualifier, container will know that when to call producer method when needed. There are two conditions now,
</p>

<div class="bullet list">
   <ul>
      <li>
         If a Bank bean has qualifier BankProducer then container will call the producer method.
      </li>
      <li>
         If a Bank bean has not a qualifier then container will inject directly the BankOfAmerica bean.
      </li>
   </ul>
</div>

<pre class="line-numbers"><code class="language-java">
public class ProducesExample {

    @Inject
    private Bank beanFromBankImplementation;

    @Inject
    @BankProducer
    private Bank beanFromBankProducer;

    public void callBanksWithdrawal() {
        beanFromBankImplementation.withdrawal();
        beanFromBankProducer.withdrawal();
    }
}</code>
</pre>

   ![]({{ site.url }}/images/2015/02/disambiguation1.png)

<p>
   Now what if we had more than one Bank implementation and we want to resolve disambiguation in producer method. Assume we have HSBC and Chase.
</p>

<pre class="line-numbers"><code class="language-java">
public class Chase implements Bank {
    @Override
    public void withdrawal() {
        System.out.println("Withdrawal from Chase");
    }

    @Override
    public void deposit() {
        System.out.println("Deposit to Chase");
    }
}</code>
</pre>

<pre class="line-numbers"><code class="language-java">
public class HSBC implements Bank {
    @Override
    public void withdrawal() {
        System.out.println("Withdrawal from HSBC");
    }

    @Override
    public void deposit() {
        System.out.println("Deposit to HSBC");
    }
}</code>
</pre>

<p>
   Additionally, lets define a java annotation and an Enum to separate our Bank implementations in their injection points.
</p>

<pre class="line-numbers"><code class="language-java">
public enum BankName {

    HSBC (HSBC.class),
    Chase (Chase.class),
    BankOfAmerica (BankOfAmerica.class);

    private Class&lt;? extends Bank&gt; bankType;

    private BankName(Class&lt;? extends Bank&gt; bankType) {
        this.bankType = bankType;
    }

    public Class&lt;? extends Bank&gt; getBankType() {
        return bankType;
    }
}</code>
</pre>

<pre class="line-numbers"><code class="language-java">
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.TYPE})
public @interface BankType {

    BankName value();
}</code>
</pre>

<p>
   Producer method should have two parameters which are; <blognewcode>Instance&lt;Bank&gt;</blognewcode>
   and <blognewcode>InjectionPoint</blognewcode>.
</p>

<pre class="line-numbers"><code class="language-java">
public class BankFactory {

    @Produces
    @BankProducer
    public Bank createBank(@Any Instance&lt;Bank&gt; instance, InjectionPoint injectionPoint) {

        Annotated annotated = injectionPoint.getAnnotated();
        BankType bankTypeAnnotation = annotated.getAnnotation(BankType.class);
        Class&lt;? extends Bank&gt; bankType = bankTypeAnnotation.value().getBankType();
        return instance.select(bankType).get();
    }
}</code>
</pre>

<p>
   <blognewcode>Instance</blognewcode> parameter is parameterized by <blognewcode>BankType</blognewcode>
   and uses <blognewcode>@Any</blognewcode> annotation which refers to any Bank implementation. So, we
   are able to return any type of Bank implementation. A CDI bean has <blognewcode>@Any</blognewcode>
   qualifier as a default. Moreover, <blognewcode>InjectionPoint</blognewcode> refers to the place where
   the bean is injected (Simply where @Inject is used for requested bean type). Thus, producer method knows
   where to inject the requested bean due to <blognewcode>InjectionPoint</blognewcode>. On the other hand,
   we can return the new instances of Bank implementations by checking their class types. For example;
</p>

<pre class="line-numbers"><code class="language-java">
if (bankType == BankOfAmerica.class) {
    // init new BankOfAmerica object and return it
    return new BankOfAmerica();
} else if (bankType == Chase.class) {
    // init new Chase object and return it
    return new Chase();
} else {
    // init new HSBC object and return it
    return new HSBC();
}</code>
</pre>

<p>
   Finally, our ProducesExample will look like;
</p>

<pre class="line-numbers"><code class="language-java">
public class ProducesExample {

    @Inject
    @BankType(BankName.BankOfAmerica)
    @BankProducer
    private Bank bankOfAmerica;

    @Inject
    @BankType(BankName.Chase)
    @BankProducer
    private Bank chase;

    @Inject
    @BankType(BankName.HSBC)
    @BankProducer
    private Bank hsbc;

    public void callBanksWithdrawal() {
        bankOfAmerica.withdrawal();
        chase.withdrawal();
        hsbc.withdrawal();
    }
}</code>
</pre>

<p>
   When we call callBanksWithdrawal() method the output will be;
</p>

<pre class="line-numbers"><code class="language-markup">
Withdrawal from Bank of America
Withdrawal from Chase
Withdrawal from HSBC</code>
</pre>

<p>
   <b><u>Additional Notes</u></b>
</p>
<div class="bullet list">
   <ul>
      <li>
         Producer method can be either static or non-static method.
      </li>
      <li>
         Producer methods let us to do custom initialization on beans.
      </li>
      <li>
         The injection point should have the same type and qualifier with producer method so that injection point will resolved to the proper producer method.
      </li>
   </ul>
</div>
[1]: http://localhost:4000/create-qualifiers-cdi-beans/
