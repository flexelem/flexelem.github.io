---
title: Java Enums Tutorial
author: buraktas
layout: post
permalink: /java-enums-tutorial/
dsq_thread_id:
  - 3539669404
categories:
  - java
tags:
  - enums
  - java
comments: true
---
Enums (Enumeration) are introduced as a new reference type in Java 1.5 with a bunch of beneficial features. They consist of fix set of constants (each constant is implicitly labeled as <code>public static final</code>) which each constant can have a data-set. Lets see the simplest example of an Enum;

<!--more-->

```java
public enum Books {

    HARRY_POTTER,
    GAME_OF_THRONES,
    DRAGONLANCE;
}
```

Enums are classes that return one instance (like singletons) for each enumeration constant declared by public static final field (immutable) so that <code>==</code> operator could be used to check their equality rather than using <code>equals()</code> method. Additionally, we can use <code>toString()</code> method to get their string representation.

```java
public class BookExample {

    public static void main(String[] args) {
        Books dragonlance = Books.DRAGONLANCE;

        // toString() method could be used to get their string representation.
        System.out.println(dragonlance.toString());

        // they can use '==' operator to check their equality
        if (dragonlance == Books.DRAGONLANCE) {
            System.out.println(true);
        }
    }
}
```

Output;

```text
DRAGONLANCE
true
```

Before going deeper we should understand why and where to use Enums. Before Enums are introduced, int enum or String enum pattern were used to represent enumerated types, and these techniques had a bunch disadvantages. First of all, they are not type-safe. For example; we have a Languages class with int enum pattern ,and a method which returns a greetings message to its associated language as a String.

```java
public class Languages {
    public static final int ENGLISH = 1;
    public static final int FRENCH = 2;
    public static final int GERMAN = 3;
    public static final int TURKISH = 4;
}
```

```java
public String greet(int lang) {
        switch (lang) {
            case 1: // English
                return "Hello";
            case 2: // German
                return "Hallo";
            case 3: // French
                return "Salut";
            case 4: // Turkish
                return "Selam";
        }

        // now what ?
        throw new IllegalArgumentException("There is no associated language for given number");
}
```

As we can see if an undefined int parameter is send, method will throw an exception to inform us there is no any defined language for the given int. Thus, this is not a safe way. Another problem is we can not take the string representation for any language, so we have to hardcoded it. On the other hand, if we would use String constants (known as String enum pattern) than we had to use string comparisons to define each behavior which is not good for performance. Thanks to Enums so that we can overcome these problems. We should use Enums if we have to define a set of fixed constants

```java
public enum Languages {
    ENGLISH,
    GERMAN,
    FRENCH,
    TURKISH;
}
```

```java
// Enums are compiled-time type safe
public String greet(Languages lang) {
        if (lang == Languages.ENGLISH) {
            return "English";
        } else if (lang == Languages.GERMAN) {
            return "Hallo";
        } else if (lang == Languages.FRENCH) {
            return "Salut";
        } else {
            return "Selam";
        }
}
```

Now there is no way to send an undefined parameter into greet() function since Enums are compile-time type safe which means passing a wrong value will cause a compiler error.

<h3> More on Enums </h3>

We have seen simple examples and usages of Enums so far, now it is time see their advanced usages.

<b><u>1. Enums can have associated data</u></b>

Enums can have any kind of associated data that is initialized via a constructor. Since Enums are immutable, fields should be defined with private accessor (They can be public but better to use private).

```java
public enum Books {

    HARRY_POTTER ("Harry Potter", 856, 12.99),
    THE_SOULFORGE ("The Soulforge", 123, 12.11),
    GAME_OF_THRONES ("Game of Thrones", 567, 10.00),
    DRAGONLANCE ("Dragonlance", 456, 6.77);

    private final String representations;
    private final int pages;
    private final double price;

    // order of fields matter
    Books(String representations, int pages, double price) {
        this.representations = representations;
        this.pages = pages;
        this.price = price;
    }

    public String getRepresentations() {
        return representations;
    }

    public int getPages() {
        return pages;
    }

    public double getPrice() {
        return price;
    }
}
```

```java
for (Books book : Books.values()) {
            System.out.println("Book Name: " + book.getRepresentations()
                                + " Number of Pages: " + book.getPages()
                                + " Price: " + book.getPrice());
}
```

```text
Book Name: Harry Potter     Number of Pages: 856	Price: 12.99
Book Name: The Soulforge	Number of Pages: 100	Price: 12.11
Book Name: Game of Thrones	Number of Pages: 567	Price: 10.0
Book Name: Dragonlance		Number of Pages: 456	Price: 6.77
```

<b><u>2. Enums can have abstract methods</u></b>

Enums can have constant-specific method implementations which simply means they can have abstract methods and they can override it.

```java
public enum GameCharacterTypes {

    ARCHER {
        @Override
        public double attack() {
            return new Random().nextInt(20);
        }
    },
    WIZARD {
        @Override
        public double attack() {
            return new Random().nextInt(30);
        }
    },
    KNIGHT {
        @Override
        public double attack() {
            return new Random().nextInt(40);
        }
    };

    // calculates damage related to character type
    public abstract double attack();
}
```

<b><u>3. Enums can implement interfaces</u></b>

Like classes in Java, Enums can implement interfaces. The important point is they can not extend from any other class because Enums implicitly extend from java.lang.Enum. Additionally, Enums implicitly implement Comparable and Serializable interfaces but you can override them if you want.

```java
public interface Price {
    public double price();
}
```

```java
public enum Books implements Price {

    HARRY_POTTER (12.99),
    THE_SOULFORGE (12.11),
    GAME_OF_THRONES (10.00),
    DRAGONLANCE (6.77);

    private final double price;

    Books(double price) {
        this.price = price;
    }

    @Override
    public double price() {
        return this.price;
    }
}
```

<b><u>Additional Notes</u></b>

<div>
  <ul>
    <li>
      Classes can not extend Enums since they are labeled as final implicitly.
    </li>
    <li>
      Enums are classes that return one instance for each enumeration constant declared by public static final field.
    </li>
    <li>
      Enums are implicitly extend from java.lang.Enum, thus they can not extend from another classes.
    </li>
    <li>
      Enums are implicitly implement Comparable and Serializable interfaces. Any sort operation on an Enum array/list will provide their natural order (the order in which constants are declared).
    </li>
    <li>
      Enums can not created by <code>new</code> keyword since their constructor is <code>private</code>. You don&#8217;t need to declare its constuctor as <code>private</code>. Due to <a href="http://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html#jls-8.8.3">JLS</a>
    </li>
    <blockquote>
        If no access modifier is specified for the constructor of an enum type, the constructor is private.
    </blockquote>

    <li>
      Enums can have and implement abstract methods.
    </li>
    <li>
      Like other classes enums can have inner class and enums.
    </li>
    <li>
      Since enums are immutable <code>==</code> operator could be used to check their equality.
    </li>
    <li>
      If you override <code>toString()</code> method in an enum type than consider to write a fromString method to get enum constant related to its string representation.
    </li>
</ul>

```java
public enum Books {

    HARRY_POTTER ("Harry Potter"),
    THE_SOULFORGE ("The Soulforge"),
    GAME_OF_THRONES ("Game of Thrones"),
    DRAGONLANCE ("Dragonlance");

    private static final Map&lt;String, Books&gt; fromString = new HashMap&lt;&gt;();
    static {
        for (Books book : values()) {
            fromString.put(book.toString(), book);
        }
    }

    private  String representation;

    Books(String representation) {
        this.representation = representation;
    }

    @Override
    public String toString() {
        return representation;
    }

    public static Books fromString(String rep) {
        return enumMap.get(rep);
    }
}
```
</div>
