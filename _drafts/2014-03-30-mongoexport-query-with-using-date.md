---
title: mongoexport query with using date
author: buraktas
layout: post
permalink: /mongoexport-query-with-using-date/
dsq_thread_id:
  - 2555546966
categories:
  - mongodb
tags:
  - date
  - mongodb
  - mongoexport
  - timestamp
---
Sometimes we might want to export only a specific part of our collection with query support of **mongoexport**.  
Suppose this is our notebook collection, and each document refers to a notebook with their production date.

<pre class="lang:js decode:true " >{
        "_id" : ObjectId("531ce460000000019b9643bc"),
        "company" : "Samsung",
        "date" : ISODate("2014-03-09T22:00:00Z"),
        "price" : 2000,
        "brand" : "Ultrabook",
}
{
        "_id" : ObjectId("531ce460000000019b9643ba"),
        "company" : "Sony",
        "date" : ISODate("2014-03-08T22:00:00Z"),
        "price" : 1500,
        "brand" : "Vaio",
}
{
        "_id" : ObjectId("531ce460000000019b9643bd"),
        "company" : "Apple",
        "date" : ISODate("2014-03-07T22:00:00Z"),
        "price" : 2250,
        "brand" : "MacbookPro",
}
{
        "_id" : ObjectId("531ce460000000019b9643be"),
        "company" : "Apple",
        "date" : ISODate("2014-03-06T22:00:00Z"),
        "price" : 1200,
        "brand" : "MacbookAir",
}
{
        "_id" : ObjectId("531ce460000000019b9643bf"),
        "company" : "Samsung",
        "date" : ISODate("2014-03-05T22:00:00Z"),
        "price" : 1000,
        "brand" : "Ultrabook",
}</pre>

The original way of mongoexport is defined by

<pre class="lang:default decode:true " >mongoexport --db &lt;database&gt; --collection &lt;collection&gt; --query &lt;JSON query&gt; --out &lt;file&gt;</pre>

The major problem is we can not use **ISODate(&#8220;&#8221;)** objects to represent dates within a query, so that we have to convert each of them object into a **Date** object.

For instance; if we try to find the notebooks produced between **2014-03-09T22:00:00Z** and **2014-03-07T22:00:00Z** by Apple with the given query; 

<pre class="wrap:true lang:sh decode:true " >mongoexport --db test --collection notebooks --query  '{ company:"Apple", date: { $lt: ISODate("2014-03-09T22:00:00Z") , $gte: ISODate("2014-03-07T22:00:00Z")} }' --out example.json</pre>

we will probably have an error like;

<div class="admonition warning">
  <p class="first admonition-title">
    ERROR
  </p>
  
  <p>
    ERROR: too many positional options
  </p>
</div>

There are two common ways to convert ISODates into Date objects which are;

<div class="bullet list">
  <ul>
    <li>
      we can use a simple javascript in mongo shell like;
    </li>
    <pre class="lang:js decode:true " >var a = ISODate('2014-03-10T22:00:00Z');
a.getTime()</pre>
    
    <li>
      we can convert the given date into milliseconds from the link <a href="http://www.ruddwire.com/handy-code/date-to-millisecond-calculators/#.Ux3CqPmSzO5">ISODate to milliseconds</a>
    </li>
  </ul>
</div>

Now we have correct Date times to use them in our query.

<pre class="wrap:true lang:default decode:true " >mongoexport --db test --collection notebooks --query  "{ company:"Apple", date: { $lt: new Date(1394402400000) , $gte: new Date(1394229600000)} }" --out example.jso</pre>

Finally, we will have a json file (example.json) in our current directory which includes only one document which is;

<pre class="lang:js decode:true " >{
        "_id" : ObjectId("531ce460000000019b9643bd"),
        "company" : "Apple",
        "date" : ISODate("2014-03-07T22:00:00Z"),
        "price" : 2250,
        "brand" : "MacbookPro",
}</pre>