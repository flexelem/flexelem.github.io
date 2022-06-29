---
title: Convert objects to/from JSON by Jackson example
author: buraktas
layout: post
permalink: /convert-objects-to-from-json-by-jackson-example/
dsq_thread_id:
  - 3425397232
categories:
  - serialization
tags:
  - jackson
  - java
  - json
comments: true
---
We will see a simple example about converting objects to/from JSON objects in Java by Jackson. First, add maven dependency for jackson.

<pre><code class="language-apacheconf">&lt;dependency&gt;
        &lt;groupId&gt;com.fasterxml.jackson.core&lt;/groupId&gt;
        &lt;artifactId&gt;jackson-databind&lt;/artifactId&gt;
        &lt;version&gt;2.4.4&lt;/version&gt;
&lt;/dependency&gt;</code>
</pre>

<!--more-->

And two typical Person and Address classes.

<b>Person.java</b>

<pre><code class="language-java">public class Person {

    private String name;
    private String lastName;
    private int age;
    private List&lt;Address&gt; addressList;

    @JsonCreator
    public Person(@JsonProperty("name") String name,
                  @JsonProperty("lastName") String lastName,
                  @JsonProperty("age") int age,
                  @JsonProperty("addressList") List&lt;Address&gt; addressList) {
        this.name = name;
        this.lastName = lastName;
        this.age = age;
        this.addressList = addressList;
    }

    // getters and setters

    @Override
    public String toString(){
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.append("name: ")
                .append(this.name).append("\n")
                .append("lastName: ")
                .append(this.lastName).append("\n")
                .append("age: ")
                .append(this.age).append("\n");

        for (Address address: this.addressList) {
            stringBuilder.append(address.toString());
        }

        return stringBuilder.toString();
    }
}</code>
</pre>

<b>Address.java</b>

<pre><code class="language-java">public class Address {

    private int zipcode;
    private String street;

    @JsonCreator
    public Address(@JsonProperty("zipcode") int zipcode,
                   @JsonProperty("street") String street) {
        this.zipcode = zipcode;
        this.street = street;
    }

    // getters and setters

    @Override
    public String toString() {
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.append("zipcode: ")
                .append(this.zipcode).append("\n")
                .append("street: ")
                .append(this.street).append("\n");

        return stringBuilder.toString();
    }
}</code>
</pre>

<b><u>Notes</u></b>

<div>
    <ul>
      <li>
        <code>@JsonCreator</code> annotation is used for constructors or static factory methods to construct instances from Json. This is called <b>property base creators</b>. Property base creators can have one more or more parameters which all of them have to annotated by <code>@JsonProperty</code> annotations.
      </li>

      <li>
        <code>@JsonProperty</code> annotation is used to bind data by a given name.
      </li>
    </ul>
</div>

<h3> 1.Object to JSON </h3>

<pre><code class="language-java">public class JacksonExample {

    public static void main(String[] args) throws JsonProcessingException {
        Address homeAddress = new Address(12345, "Stenhammer Drive");
        Address workAddress = new Address(7986, "Market Street");
        List&lt;Address&gt; addressList = new ArrayList&lt;&gt;();
        addressList.add(homeAddress);
        addressList.add(workAddress);
        Person person = new Person("Sawyer", "Bootstrapper", 23, addressList);

        // object mapper is the main object to marshall (write) data into JSON
        ObjectMapper objectMapper = new ObjectMapper();
        String value = objectMapper.writeValueAsString(person);
        System.out.println(value);
    }
}</code>
</pre>

Output will be;

<pre><code class="language-java">{
   "name":"Sawyer",
   "lastName":"Bootstrapper",
   "age":23,
   "addressList":[
      {
         "zipcode":12345,
         "street":"Stenhammer Drive"
      },
      {
         "zipcode":7986,
         "street":"Market Street"
      }
   ]
}</code>
</pre>

<h3> 2.JSON to Object </h3>

<pre><code class="language-java">public class JacksonExample {

    private static String jsonValue = "{\"name\":\"Sawyer\",\"lastName\":\"Bootstrapper\",\"age\":23,\"addressList\":[{\"zipcode\":12345,\"street\":" +
            "\"Stenhammer Drive\"},{\"zipcode\":7986,\"street\":\"Market Street\"}]}";

    public static void main(String[] args) throws IOException {
        Person personValue = objectMapper.readValue(jsonValue, Person.class);
        System.out.println(personValue.toString());
    }
}</code>
</pre>

Output will be;

<pre>name: Sawyer
lastName: Bootstrapper
age: 23
zipcode: 12345
street: Stenhammer Drive
zipcode: 7986
street: Market Street
</pre>
