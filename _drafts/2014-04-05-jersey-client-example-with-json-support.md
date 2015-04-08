---
title: Jersey Client Example with JSON Support
author: buraktas
layout: post
permalink: /jersey-client-example-with-json-support/
dsq_thread_id:
  - 2588884713
categories:
  - jax-rs
tags:
  - java
  - jax-rs
  - jersey-client
  - rest
  - web-service
---
In this tutorial we will implement a Jersey client example with JSON support. The tools and technologies that I have used are;

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
      JDK 1.7
    </li>
  </ul>
</div>

#### Project Structure 

Our project structure will look like below;

<div class="hentry img">
  <a href="http://www.buraktas.com/wp-content/uploads/2014/04/jersey-client-get-structure.png"><img src="http://www.buraktas.com/wp-content/uploads/2014/04/jersey-client-get-structure.png" alt="jersey-client-get-structure" width="322" height="417" class="aligncenter size-full wp-image-124" /></a>
</div>

#### Dependencies 

There are two dependencies to use Jersey client and JSON support

<pre class="lang:xhtml decode:true " >&lt;dependency&gt;
	&lt;groupId&gt;com.sun.jersey&lt;/groupId&gt;
	&lt;artifactId&gt;jersey-client&lt;/artifactId&gt;
	&lt;version&gt;1.18.1&lt;/version&gt;
&lt;/dependency&gt;

&lt;dependency&gt;
	&lt;groupId&gt;com.sun.jersey&lt;/groupId&gt;
	&lt;artifactId&gt;jersey-json&lt;/artifactId&gt;
	&lt;version&gt;1.18.1&lt;/version&gt;
&lt;/dependency&gt;</pre>

<div class="bullet list">
  <ul>
    <li>
      <b>jersey-client</b> dependency provides required jars for jersey client.
    </li>
    <li>
      <b>jersey-json</b> dependecy includes the jersey-json.jar which provides JSON support. </ul> </div> <h4>
        POJO Entity
      </h4>
      
      <p>
        We will simply use a Book entity which consists of 4 fields
      </p>
      
      <pre class="lang:java decode:true " >public class BookEntity {

    private int    id;
    private String title;
    private String author;
    private double price;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public double getPrice() {
        return price;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    @Override
    public String toString() {

        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.append("Id : " + this.id + '\n');
        stringBuilder.append("Title : " + this.title + '\n');
        stringBuilder.append("Author : " + this.author + '\n');
        stringBuilder.append("Price : " + this.price + '\n');

        return stringBuilder.toString();
    }
}</pre>
      
      <h4>
        Client Configuration
      </h4>
      
      <p>
        To enable JSON support we have to turn <blogcode>JSONConfiguration.FEATURE_POJO_MAPPING</blogcode> feature on for our jersey client. So we create our client with the following code;
      </p>
      
      <pre class="lang:java mark:2 decode:true " >ClientConfig clientConfig = new DefaultClientConfig();
clientConfig.getFeatures().put(JSONConfiguration.FEATURE_POJO_MAPPING, Boolean.TRUE);
Client client = Client.create(clientConfig);</pre>
      
      <h4>
        Resource Class
      </h4>
      
      <p>
        Here are the interface and the implementation class of our resource
      </p>
      
      <pre class="lang:java decode:true " >
@Path("/book-service")
public interface BookResource {

    @GET
    @Path("/list")
    @Produces({ MediaType.APPLICATION_JSON })
    public Map&lt;Integer, BookEntity&gt; getBookList();

    @GET
    @Path("/find")
    @Produces({ MediaType.APPLICATION_JSON })
    public BookEntity findBook(@QueryParam("id") int id);

    @POST
    @Path("/create")
    @Consumes("application/json")
    @Produces("application/json")
    public BookEntity createBook(BookEntity bookEntity);
}</pre>
      
      <pre class="lang:java decode:true " >
public class DefaultBookResource implements BookResource {

    private Map&lt;Integer, BookEntity&gt; list      = new HashMap&lt;Integer, BookEntity&gt;();
    private AtomicInteger            idCounter = new AtomicInteger();

    public DefaultBookResource() {

        BookEntity bookEntity1 = new BookEntity();
        bookEntity1.setId(idCounter.getAndIncrement());
        bookEntity1.setAuthor("MargaretWeis");
        bookEntity1.setTitle("TheSoulforge");
        bookEntity1.setPrice(7.99);

        BookEntity bookEntity2 = new BookEntity();
        bookEntity2.setId(idCounter.getAndIncrement());
        bookEntity2.setAuthor("Salvatore");
        bookEntity2.setTitle("Exile");
        bookEntity2.setPrice(9.99);

        list.put(bookEntity1.getId(), bookEntity1);
        list.put(bookEntity2.getId(), bookEntity2);
    }

    @Override
    public Map&lt;Integer, BookEntity&gt; getBookList() {
        return list;
    }

    @Override
    public BookEntity findBook(int id) {
        return list.get(id);
    }

    @Override
    public BookEntity createBook(BookEntity bookEntity) {

        bookEntity.setId(idCounter.getAndIncrement());
        list.put(bookEntity.getId(), bookEntity);

        return bookEntity;
    }
}</pre>
      
      <div class="bullet list">
        <ul>
          <li>
            <blogcode>Map<Integer, BookEntity> list</blogcode> is a list that it will have two entities initially.
          </li>
          <li>
            <blogcode>AtomicInteger</blogcode> provides to increment and decrement amotically an integer.
          </li>
        </ul>
      </div>
      
      <h4>
        GET Request
      </h4>
      
      <p>
        As I mentioned earlier we have to create our Jersey client with JSON support, then we can start to make requests to available resources. I will first send a GET request to return a BookEntity if it exists.
      </p>
      
      <pre class="lang:java decode:true " >public class JerseyClient {

    public static void main(String[] args) {

        // Create Jersey client
        ClientConfig clientConfig = new DefaultClientConfig();
        clientConfig.getFeatures().put(JSONConfiguration.FEATURE_POJO_MAPPING, Boolean.TRUE);
        Client client = Client.create(clientConfig);

        // GET request to findBook resource with a query parameter
        String getBookURL = "http://localhost:8080/jersey-client-json-example/services/book-service/find";
        WebResource webResourceGet = client.resource(getBookURL).queryParam("id", "1");
        ClientResponse response = webResourceGet.get(ClientResponse.class);
        BookEntity responseEntity = response.getEntity(BookEntity.class);

        if (response.getStatus() != 200) {
            throw new WebApplicationException();
        }

        System.out.println(responseEntity.toString());
    }
}</pre>
      
      <p>
        Output ;
      </p>
      
      <pre class="lang:default decode:true " >Id : 1
Title : Exile
Author : Salvatore
Price : 9.99</pre>
      
      <p>
        Now lets see how we can get the list from response which contains BookEntities
      </p>
      
      <pre class="lang:java mark:5 decode:true " >// GET request to list resource
String getListURL = "http://localhost:8080/jersey-client-json-example/services/book-service/list";
webResourceGet = client.resource(getListURL);
response = webResourceGet.get(ClientResponse.class);
Map&lt;Integer, BookEntity&gt; list = webResourceGet.get(new GenericType&lt;Map&lt;Integer, BookEntity&gt;&gt;() {});

if (response.getStatus() != 200) {
     throw new WebApplicationException();
}

for (Entry&lt;Integer, BookEntity&gt; entry : list.entrySet()) {

     BookEntity bookEntity = entry.getValue();
     System.out.println(bookEntity.toString());
}</pre>
      
      <div class="bullet list">
        <ul>
          <li>
            <blogcode>GenericType</blogcode> is the key object to get the list from the response.
          </li>
        </ul>
      </div>
      
      <p>
        Output will be ;
      </p>
      
      <pre class="lang:default decode:true " >Id : 0
Title : TheSoulforge
Author : MargaretWeis
Price : 7.99

Id : 1
Title : Exile
Author : Salvatore
Price : 9.99</pre>
      
      <h4>
        POST Request
      </h4>
      
      <p>
        Now we will create a BookEntity object to send it associated POST resource.
      </p>
      
      <pre class="lang:java decode:true " >public class JerseyClient {

    public static void main(String[] args) {

        // Create Jersey client
        ClientConfig clientConfig = new DefaultClientConfig();
        clientConfig.getFeatures().put(JSONConfiguration.FEATURE_POJO_MAPPING, Boolean.TRUE);
        Client client = Client.create(clientConfig);

        // Jersey client POST example
        BookEntity bookEntity = new BookEntity();
        bookEntity.setTitle("LOTR");
        bookEntity.setAuthor("Tolkien");
        bookEntity.setPrice(12.99);
        String postURL = "http://localhost:8080/jersey-client-json-example/services/book-service/create";
        WebResource webResourcePost = client.resource(postURL);
        response = webResourcePost.type("application/json").post(ClientResponse.class, bookEntity);
        responseEntity = response.getEntity(BookEntity.class);

        System.out.println(responseEntity.toString());
    }
}</pre>
      
      <p>
        Finally our output will be;
      </p>
      
      <pre class="lang:java decode:true " >Id : 2
Title : LOTR
Author : Tolkien
Price : 12.99</pre>
      
      <p>
        You can download the source code from <a href="http://www.buraktas.com/wp-content/uploads/2014/04/jersey-client-json-example.rar" target="_blank">here</a>.
      </p>