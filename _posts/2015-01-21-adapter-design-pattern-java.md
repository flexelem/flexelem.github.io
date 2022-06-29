---
title: Adapter Design Pattern in Java
author: buraktas
layout: post
permalink: /adapter-design-pattern-java/
dsq_thread_id:
  - 3440392317
categories:
  - design-patterns
tags:
  - adapter-design-pattern
  - design-pattern
  - java
comments: true
---
Adapter design pattern is a structural design pattern that provides two unrelated interfaces to work together. The most common illustration about adapter design pattern is sockets from different countries. Well most of us faced with laptop adapter issues when traveling, and looked for appropriate adapter for our laptops. So, by using an adapter we are able to plug our laptop chargers to sockets. Or using a card reader to plug our memory card from our camera to our computer. Thus, we make two unrelated interfaces to work together. The elements of adapter design pattern are Client, Target, Adapter and Adaptee. We can see the Gang of Four definition for ADP, and description of each element of it below;

> Convert the interface of class into another interface clients expect.Adapter lets class work together that couldn&#8217;t otherwise because of incompatible interfaces.

<!--more-->

<b><u>Elements</u></b>
<div>
  <ul>
    <li>
      <b>Client:</b> collaborates with objects adaptable to the Target interface.
    </li>
    <li>
      <b>Target:</b> defines the domain-specific interface that Client uses.
    </li>
    <li>
      <b>Adapter:</b> adapts the interface of Adaptee to the Target interface.
    </li>
    <li>
      <b>Adaptee:</b> defines an existing interface that needs adapting.
    </li>
  </ul>
</div>

<b><u>Types</u></b>
<div>
  <ul>
    <li>
      <b>Object ADP:</b> Uses (wraps) an instance of the class it wants to adapt.
    </li>
    <li>
      <b>Class ADP:</b> Inherits from the class it wants to adapt.
    </li>
  </ul>
</div>

Here is the actual UML diagram for ADP

![]({{ site.url }}/assets/img/2015/01/adapter-design-pattern.png)

Now let's implement an example of ADP. Suppose that we have implemented a RPG (role playing game) application by using a third party API that has only <b>Fighter</b> interface.

<b>Fighter.class</b>

<pre><code class="language-java">public interface Fighter {
    public void attack();
    public void defend();
    public void escape();
}</code>
</pre>

And a basic implementation of our Fighter interface.

<b>Knight.class</b>

<pre><code class="language-java">public class Knight implements Fighter {

    @Override
    public void attack() {
        System.out.println("Knight attacks!!");
    }

    @Override
    public void defend() {
        System.out.println("Knight defends...");
    }

    @Override
    public void escape() {
        System.out.println("Run Knight run...");
    }
}</code>
</pre>

After a while, a Wizard interface/class has published but we still want to use it again as a Fighter. So, what we have to do is adapting this <b>Wizard</b> class (adaptee) into a <b>Fighter</b> (Target) via an adapter class because Wizard has different functionality than Fighter. Here is the implementation of Wizard class;

<b>Wizard.java</b>

<pre><code class="language-java">public class Wizard {

    public void castDestructionSpell() {
        System.out.println("Fireball is coming!!!");
    }

    public void shield() {
        System.out.println("Wizard casted shild spell...");
    }

    public void openPortal() {
        System.out.println("No need to run lets open a portal");
    }
}</code>
</pre>

So, our WizardAdapter class will implement Fighter interface and call appropriate functions of Wizard class.

<b>WizardAdapter.java</b>

<pre><code class="language-java">public class WizardAdapter implements Fighter {

    private Wizard wizard;

    public WizardAdapter(Wizard wizard) {
        this.wizard = wizard;
    }

    @Override
    public void attack() {
        this.wizard.castDestructionSpell();
    }

    @Override
    public void defend() {
        this.wizard.shield();
    }

    @Override
    public void escape() {
        this.wizard.openPortal();
    }
}</code>
</pre>

And finally our Client is ready to use adapted Wizard class.

<pre><code class="language-java">public class Main {

    public static void main(String args[]) {
        Fighter barbarian = new Barbarian();
        Wizard wizard = new Wizard();
        WizardAdapter wizardAdapter = new WizardAdapter(wizard);

        System.out.println("-----Barbarian's Action------");
        barbarian.attack();
        barbarian.defend();
        barbarian.escape();

        System.out.println("\n-----Wizard's Action------");
        wizardAdapter.attack();
        wizardAdapter.defend();
        wizardAdapter.escape();
    }
}</code>
</pre>

This is how we used the adapter design pattern by our example.

![]({{ site.url }}/assets/img/2015/01/adapter_design_pattern_2_2.png)

<h3> Advanced Example </h3>

We saw the simplest implementation of ADP, however, due to the reason that we are dealing with huge systems we may need many adapters for different kind of adaptees. Consider that we have a system which provides paying customers bills, calculating their balance, credit history etc. through external APIs of some banks and other companies. And these kind of operations will be made by sending requests to their systems. So, we have an Accounting adapter for Banks (Bank of America and HSBC) and Credit History Service adapter for a company called CompX. We can see the UML diagram for this system below.

![]({{ site.url }}/assets/img/2015/01/adapter_design_pattern_2_2.png)

Actually, there is no need to show adaptee objects. Each adapter contains an appropriate adaptee object and sends a http request to process it to the related systems. This is where adaptors providing to deal with unrelated interfaces.

![]({{ site.url }}/assets/img/2015/01/adp_flow_chart.png)
