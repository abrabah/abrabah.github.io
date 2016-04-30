---
layout: post
title: Batch collecting java 8 Streams
excerpt: Figuring out a way to collect java 8 streams in batches
---


I guess most people have some kind of a love/hate relationship to lambdas and streams in java 8. Granted, the implementation is not as elegant as other pure programming languages out there, but these new tools have certainly made java programming easier and more fun for me. 

Lately I have been faced with the challenge of passing a  stream of elements  to a remote database;

{% highlight java %}
someHumongousDataStream
.map(element -> doSomeBlackVoodooMagic(element))
.forEach(element -> sendToSomeDatabase(element))
{% endhighlight %}

The code above is not very efficient. Sending one element at a time is a bad idea because of the overhead in initializing a connection to the database. An alternative is to collect the whole datastream and then send all the elements as one big chunk: 

{% highlight java %}
List<Element> humongousList = 
someHumongousDataStream
.map(element -> doSomeBlackVoodooMagic(element))
.collect(Collectors.toList());

sendToSomeDatabase(humongousList);
{% endhighlight %}

While this may speed up execution, it won't do much good if the list is too large to be held in memory. So I've written my own, neat batch collector which collects the elements into small batches. 

{% highlight java %}
someHumongousDataStream
.map(element -> doSomeBlackVoodooMagic(element))
.collect(BatchCollector.collect(
    (List<Element> elements) -> sendToSomeDatabase(elements),
    1000)
);
{% endhighlight %}

The example above batches the stream into lists holding maximum 1000 elements; small enough to be held in memory, and big enough to eliminate some of the database connection overhead. 

The source code is available at [Github](https://github.com/abrabah/java8-batchcollector). Feel free to do with it as you like :)
