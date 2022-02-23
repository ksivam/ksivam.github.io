---
layout: post 
title: "Java Function Chaining"
date: 2022-02-21 
categories: blogging
tags: java
comments: true
analytics: true
excerpt_separator: <!--more-->
---

Functional programming, being a subset of declarative programming, offers several constructs and `function chaining` is
one of them.

**Java function chaining** leads to very concise and readable programs, and we are going to see an example of using the
function chaining construct.

<!--more-->

Let's take a trivial example of a `Producer` and `Consumer`. We need to first start the producer to produce some data,
followed by the consumer to consume the data.

```java
class Producer {
  void start() {
    // start produce data
  }
}

public class Consumer {
  void consume() {
    // consume data
  }
}
```

In a regular java programming construct, a Driver code encapsulates starting the producer and consumer in a method

```java
class Drive {
  Producer producer;
  Consumer consumer;

  void run() {
    // do something else
    startProducerAndConsumer();
    // do something else
  }

  void startProducerAndConsumer() {
    producer.start();
    comsumer.consume();
  }
} 
```

Even though the method `startProducerAndConsumer` does only two operation, nothing prevents us to add more code in that
method than what it is supposed todo.

This method can be simplified using Java functional chain, and thus it becomes easier to read, reason about, test, and
maintain.

```java
class Drive {
  Producer producer;
  Consumer consumer;

  Function<Producer, Consumer> startProducer = p -> {
    p.start();
    return consumer;
  };

  Function<Consumer, Void> consume = c -> {
    c.consume();
    return null;
  };

  void run() {
    // do something else
    startProducer.andThen(consume).apply(producer);
    // do something else
  }
} 
```

