---
layout: post
title: Groovify CUBA - An overview of Groovy
description: "In this two part blog posts i'll show how to groovify your CUBA app. The first part i'll give a brief overview of Groovy and the benefits of using it"
modified: 2016-01-11
tags: [cuba, groovy, java]
image:
  feature: groovify-cuba-app/feature.jpg
  feature_source: https://pixabay.com/de/neujahr-silvester-silvester-2015-1090770/
---

Developing an application in [CUBA](https://www.cuba-platform.com/) is mostly about productivity. It has other advantages, but productivity is probably one of the most important reasons. To ramp up productivity on all stages of CUBA app development, we can invite Groovy to the party.


<!-- more -->

In this first of two blog posts i will give you an overview about Groovy, so you get a understanding that this might be valuable for you. Why? Because in my opinion the productivity advantage of CUBA falls apart when turn from *generating the UI or creating the domain model* to the part of programming where you actually want to implement business logic. In this case you are back at the POJO model either in your controller logic or in the services that are just Spring beans.

Coming to CUBA from a Groovy background and haven't developed in Java for quite some time, it feels a little bit clumsy to go back. This is why i think it's probably worth thinking about raising the productivity gain (in the business logic side of things) just a little bit by getting the different benefits of the Groovy language onto our CUBA application.

<style type="text/css">
#the-elevator-pitch-for-groovy + p + .highlight pre {
	height:300px;
}
</style>

### The elevator pitch for Groovy

Ok, for an elevator pitch, the easiest thing is to come up with is syntax. We as programmers all love to care about syntax, so there you go. A customer class in Java (~100 LOC):

{% highlight java %}
import java.util.Collection;
import java.util.Date;

public class Customer {

    private String firstName;
    private String lastName;
    private Date birthday;
    private Collection<Order> orders;

    public Customer() {}

    public Customer(String firstName, String lastName, Date birthday, Collection<Order> orders) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.birthday = birthday;
        this.orders = orders;
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public Collection<Order> getOrders() {
        return orders;
    }

    public void setOrders(Collection<Order> orders) {
        this.orders = orders;
    }
    
    @Override
    public String toString() {
        return "Customer{" +
                "firstName='" + firstName + '\'' +
                ", lastName='" + lastName + '\'' +
                ", birthday=" + birthday +
                ", orders=" + orders +
                '}';
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Customer customer = (Customer) o;

        if (firstName != null ? !firstName.equals(customer.firstName) : customer.firstName != null) return false;
        if (lastName != null ? !lastName.equals(customer.lastName) : customer.lastName != null) return false;
        if (birthday != null ? !birthday.equals(customer.birthday) : customer.birthday != null) return false;
        return orders != null ? orders.equals(customer.orders) : customer.orders == null;

    }


    public int calculateTurnover() {
        int totalTurnover = 0;
        
        for(Order order: orders) {
            totalTurnover += order.getAmount();
        }
        
        return totalTurnover;
    }

    @Override
    public int hashCode() {
        int result = firstName != null ? firstName.hashCode() : 0;
        result = 31 * result + (lastName != null ? lastName.hashCode() : 0);
        result = 31 * result + (birthday != null ? birthday.hashCode() : 0);
        result = 31 * result + (orders != null ? orders.hashCode() : 0);
        return result;
    }
}


{% endhighlight %}

The equivalent in Groovy (~15 LOC):


<img style="width: 150px; float:right; padding: 10px; margin-right:-15px;" src="{{site.url}}/images/groovify-cuba-app/arrow-up.png">

{% highlight groovy %}
import groovy.transform.EqualsAndHashCode
import groovy.transform.ToString

@ToString
@EqualsAndHashCode
class Customer {
    String firstName
    String lastName
    Date birthday
    Collection<Order> orders = []

    int calculateTurnover() {
        orders*.amount.sum() ?: 0
    }
}
{% endhighlight %}


**That's it**. It has the same functionality. 

Just in case you think i'm kidding: The Unit test in this in this [article](https://www.accelebrate.com/blog/call-pogo-name/) proves that both variants are semantically equivalent.

Groovy basically has a significant better *signal to noice ratio*. I'm not sure if you noticed it, but there is a little bit of signal in this class. It's the method <code>calculateTurnover</code>, which is basically the "business logic" if you will. In the Java class it's very hard to find, because it's just not visible. 

### The differences of Groovy

<img style="float:left; padding: 10px; margin-left:-293px;" src="{{site.url}}/images/groovify-cuba-app/groovy-baby.png">

When comparing both examples, these are some of the things that are different:

* removed imports, because of more default imports in Groovy
* removed semicolons where not needed
* changed default visibility scope: <code>private</code> for fields and <code>public</code> for methods, because that's the defacto default in Java
* removed *Getters* and *Setters*, will be generated by the compiler in the default case
* removed *constructors* and added a map based constructor as well as a default one
* removed <code>toString</code>, <code>equals</code> and <code>hashCode</code> because of the AST-Transformation Annotation

We could even reduce it further with <code>@Canonical</code> instead of <code>@ToString</code> and <code>@EqualsAndHashCode</code>.
A running example of this you'll find [here](http://goo.gl/UmkYw2) (which is a very good playground to start with btw.)


### Groovy shines with Maps and Lists

Since bashing about Java Getters and Setters is not my main purpose here, let's have a look at some more interessting stuff like the implementation of <code>calculateTurnover</code>.

{% highlight groovy %}
int calculateTurnover() {
    orders*.amount.sum() ?: 0
}
{% endhighlight %}

Groovy has some very cool features regarding the Collections API. [In the official docs](http://www.groovy-lang.org/groovy-dev-kit.html) you'll find a good overview of the features (2. Working with collections). I'll guide you through a few of them.

Starting with the <code>*</code> Operator. The attribute orders is a collection. When doing a <code>*.</code> on a collection, it will execute the method call after the <code>.</code> for each items in the collection. In this case <code>getAmount()</code> on every item in the orders list will be called. The result of this is a List with the results of each entry. If you are familiar with functional programming, the [Map](https://en.wikipedia.org/wiki/Map_(higher-order_function)) operation would be something similar (the exact equivalent of it is the <code>collect</code> method in Groovy).

Then, since <code>orders*.amount</code> returns a list, we can call the operation <code>sum</code> on it, which will act according to it's name. The elvis operator <code>?:</code> will either return the expression on the left if it's not <code>null</code>, otherwise it will return the right side.


<img style="width: 150px; float:right; padding: 10px; margin-right:-250px;" src="{{site.url}}/images/groovify-cuba-app/arrow-side-down.png">

If this is to scary to you, another alternative implementation would be something like:
{% highlight groovy %}
int calculateTurnover() {
    def sum = 0
    orders.each {
        sum += it.amount
    }
    sum
}
{% endhighlight %}

This example is a little closer to what we saw in the Java class. <code>each</code> is a method on a List, that takes a [closure](http://www.groovy-lang.org/closures.html) (a function) as a parameter. On each element in the list the function will be executed and <code>it</code> will be the corresponding list item. Same story as Java 8 Lambdas.

There are two additional things that are worth mentioning. First, as i said, the each method has one argument: a closure. In Groovy, often times it is not required to use parentheses at all. In this case, it is equivalent to <code>orders.each({ sum += it.amount })</code> (see the [docs](http://www.groovy-lang.org/style-guide.html) - "5. Omitting parentheses" for more details). Second, <code>return</code> is an optional keyword. If it's not in place, Groovy will use the last expression of the method as the return value.

Since the headline talks about List and *Maps*, let's have a short look at this first class language construct of Groovy. Here we have a few examples on how Groovy handles Maps:

{% highlight groovy %}
def map = ["hello" : "world", "lorem" : "ipsum"]

assert map["hello"] == "world"
assert map.lorem == "ipsum"

// create a customer with a map constructor
def customer = new Customer([
	firstName: "Mario", 
	lastName: "David", 
	orders: [
		new Order(amount: 150)
	]
])

{% endhighlight %}

The map constructor gives you a cartesian product on the possible constructors. Using a map as the parameter list in general (not just in the constructor) can be a very good idea, because it drastically increases the readability of the code compared to parameter lists where the third and the firth parameter is a boolean, and nobody is able to know what that actually indicates. It literally gives you named parameters like in C# or Javascript (when passing a JSON object to a function).

More differences that differtiates Groovy from Java can be found in the [Groovy style guide](http://www.groovy-lang.org/style-guide.html).

By the way, <code>==</code> is not a reference comparison like in Java. Instead the <code>equals</code> method of the objects are called, because like before:

<div class="well">Groovy changes default Java behaviour to something that makes sence in most cases</div>

### Syntax is just Syntax - there are more things

Although this syntactic sugar compared to Java is neat, there are other things to keep in mind before switching your whole project from one language to another. Since this discussion would clearly go beyond this blog post, i'll just go over them and give you some resources to dig down further.

Performance is one of these issues. In the beginning, before [invokedynamic](http://docs.oracle.com/javase/7/docs/technotes/guides/vm/multiple-language-support.html) was build into the JVM, creating performant dynamic languages on the JVM was pretty hard. Nowadays Groovy gives the user choices about speed with <code>@CompileStatic</code>. More information you'll find at this [InfoQ article](http://www.infoq.com/articles/new-groovy-20) from 2012 as well as this SE Radio [podcast](http://www.se-radio.net/2015/10/se-radio-episode-240-the-groovy-language-with-cedric-champeau/) with Cédric Champeau. But whenever thinking about performance, keep in mind the two [rules of software optimization](http://c2.com/cgi/wiki?RulesOfOptimization).

Next up, we have dynamic-, static-, strong- and duck-typing. All of these attributes are correct for Groovy. You basically can <code>def</code> all the things if you want to. The runtime will figure out the rest on your behalf. This is potentially not the best thing to do, so Groovy gives you choices. When you want to use types and have that checked by the compiler there are options like <code>@TypeChecked</code> or <code>@CompileStatic</code>. Have a look at this article [optional typing in groovy](https://objectpartners.com/2013/08/19/optional-typing-in-groovy/) for a few insights. 

Aditionally there is a Metaobject Protocol (MOP), which lets you do runtime meta programming and since Groovy is compiled, you can do compile time metaprogramming with AST-Transformations. An example of runtime meta programming is the following example from a Grails (a Groovy based Web framework) Domain class: 

{% highlight groovy %}

class Post {
    
    String content
    String teaser
    Date dateCreated
    Date lastUpdated

    static hasMany = [comments : Comment]

}

myPost = Post.findByTeaserLike("%my teaser%")
{% endhighlight %}

<code>findByTeaserLike</code> is a method that does not exist at compile time. It is a method that is evaluated at runtime and build and execute a SQL Query that queries for a post entry where the column teaser is like %my teaser%. A good insight on this feature (which is based on <code>methodMissing</code>) you'll find in the article [Groovy Goodness: Create Dynamic Methods](http://mrhaki.blogspot.de/2009/11/groovy-goodness-create-dynamic-methods.html).

Based on this meta programming feature comes another one: [domain specific languages](http://docs.groovy-lang.org/docs/latest/html/documentation/core-domain-specific-languages.html). You can create languages that look like this:

{% highlight groovy %}
take 2.pills of chloroquinine after 6.hours

// equivalent to: take(2.pills).of(chloroquinine).after(6.hours)
{% endhighlight %}

To create a DSL in Groovy really is a breeze.
This is perhaps not directly relevant for the developers (although it could be), doing this right can give your process a dramatic additional performance boost and include other people be part of the process as well. Have a look at the book [domain specific languages](http://martinfowler.com/books/dsl.html) from [Martin Fowler](https://twitter.com/martinfowler) for further reading.

### Grooovy can make a difference

To wrap this up, i think you get a good feeling about the differences that Groovy can make. The syntactic sugar together with more options open up possibilities for the users of this language. 

As i already wrote last time, *software developers paychecks are the driving cost factors in most IT efforts*. This is especially true for software development. Any opportunity to get better in this regard will increase the overall outcome. CUBA has helped us with different things to go down this road pretty far. To use Groovy in this scenario is just another one of these steps that focuses on the business part of things.

If you want to take a deeper look into Groovy (and i hope i could encourage you to do so), there a very good book about Groovy, called [Programming Groovy 2](http://www.amazon.de/dp/1937785300) form [Venkat Subramaniam](https://twitter.com/venkat_s) which goes much deeper in the described topics.

In the [second part](http://www.road-to-cuba-and-beyond.com/groovify-cuba-app-integrate-with-cuba/) of this two part blog posts, i will go trough the actual integration. We'll have a look on how to enable Groovy in the CUBA app and take look at some other integration possibilities as well.