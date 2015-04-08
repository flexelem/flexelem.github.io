---
title: Adapter Design Pattern in Java
author: buraktas
layout: post
permalink: /adapter-design-pattern-java/
dsq_thread_id:
  - 3440392317
categories:
  - Design Patterns
tags:
  - adapter-design-pattern
  - design-pattern
  - java
---
Adapter design pattern is a structural design pattern that provides two unrelated interfaces to work together. The most common illustration about adapter design pattern is sockets from different countries. Well most of us faced with laptop adapter issues when traveling, and looked for appropriate adapter for our laptops. So, by using an adapter we are able to plug our laptop chargers to sockets. Or using a card reader to plug our memory card from our camera to our computer. Thus, we make two unrelated interfaces to work together. The elements of adapter design pattern are Client, Target, Adapter and Adaptee. We can see the Gang of Four definition for ADP, and description of each element of it below;

> Convert the interface of class into another interface clients expect.Adapter lets class work together that couldn&#8217;t otherwise because of incompatible interfaces.

<div class="bullet list">
  <b><u>Elements</u></b></p> 
  
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

<div class="bullet list">
  <b><u>Types</b></u></p> 
  
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

[<img src="http://www.buraktas.com/wp-content/uploads/2015/01/adapter-design-pattern.png" alt="adapter-design-pattern" width="1550" height="820" class="aligncenter size-full wp-image-737" />][1]

Now let&#8217;s implement an example of ADP. Suppose that we have implemented a RPG (role playing game) application by using a third party API that has only **Fighter** interface.

<div class="class-names">
  <b>Fighter.class</b></p> 
  
  <pre class="lang:java decode:true " >public interface Fighter {

    public void attack();
    public void defend();
    public void escape();
}</pre>
</div>

And a basic implementation of our Fighter interface.

<div class="class-names">
  <b>Knight.class</b></p> 
  
  <pre class="lang:java decode:true " >public class Knight implements Fighter {

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
}</pre>
</div>

After a while, a Wizard interface/class has published but we still want to use it again as a Fighter. So, what we have to do is adapting this **Wizard** class (adaptee) into a **Fighter** (Target) via an adapter class because Wizard has different functionality than Fighter. Here is the implementation of Wizard class;

<div class="class-names">
  <b>Wizard.java</b></p> 
  
  <pre class="lang:java decode:true " >public class Wizard {

    public void castDestructionSpell() {
        System.out.println("Fireball is coming!!!");
    }

    public void shield() {
        System.out.println("Wizard casted shild spell...");
    }

    public void openPortal() {
        System.out.println("No need to run lets open a portal");
    }
}</pre>
</div>

So, our WizardAdapter class will implement Fighter interface and call appropriate functions of Wizard class.

<div class="class-names">
  <b>WizardAdapter.java</b></p> 
  
  <pre class="lang:java decode:true " >public class WizardAdapter implements Fighter {

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
}</pre>
</div>

And finally our Client is ready to use adapted Wizard class.

<pre class="lang:java decode:true " >public class Main {

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
}</pre>

This is how we used the adapter design pattern by our example.

[<img src="http://www.buraktas.com/wp-content/uploads/2015/01/adapter_design_pattern_2_2.png" alt="adapter_design_pattern_2_2" width="1568" height="800" class="aligncenter size-full wp-image-729" />][2]

### Advanced Example

We saw the simplest implementation of ADP, however, due to the reason that we are dealing with huge systems we may need many adapters for different kind of adaptees. Consider that we have a system which provides paying customers bills, calculating their balance, credit history etc. through external APIs of some banks and other companies. And these kind of operations will be made by sending requests to their systems. So, we have an Accounting adapter for Banks (Bank of America and HSBC) and Credit History Service adapter for a company called CompX. We can see the UML diagram for this system below.

[<img src="http://www.buraktas.com/wp-content/uploads/2015/01/advance_adp_structure.png" alt="advance_adp_structure" width="1660" height="772" class="aligncenter size-full wp-image-739" />][3]

Actually, there is no need to show adaptee objects. Each adapter contains an appropriate adaptee object and sends a http request to process it to the related systems. This is where adaptors providing to deal with unrelated interfaces.

[<img src="http://www.buraktas.com/wp-content/uploads/2015/01/adp_flow_chart.png" alt="adp_flow_chart" width="1482" height="818" class="aligncenter size-full wp-image-740" />][4]

 [1]: http://www.buraktas.com/wp-content/uploads/2015/01/adapter-design-pattern.png
 [2]: http://www.buraktas.com/wp-content/uploads/2015/01/adapter_design_pattern_2_2.png
 [3]: http://www.buraktas.com/wp-content/uploads/2015/01/advance_adp_structure.png
 [4]: http://www.buraktas.com/wp-content/uploads/2015/01/adp_flow_chart.png