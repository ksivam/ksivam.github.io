---
layout: post title: "Flag Arguments"
date: 2022-02-26 categories: blogging tags: programming

comments: true analytics: true excerpt_separator: <!--more-->
---

Flag Arguments are bad programming practice. Well known experts like Martin Fowler and Robert C. Martin denounce using
Flag Arguments, and advise to avoid them at all cost.

In this blog post, we will be walking through a sample code which uses Flag Argument, and see why it's a code smell.
Then the bad code will be refactored to follow the SOLID principles.

<!--more-->

Let's take a trivial example of a `Workflow` which migrates user from an old to a new system. The workflow performs series 
of activities, and these activities are encapsulated in a `Activity` class.  The `Workflow` provides a `dryRun` option 
to simulate the migration activities without doing the actual migration.

```java
interface Activity {
  boolean userExist();
  
  Status migrateUser();

  updateUserMigrationStatus(Status status);
}

public class Workflow {
  
  void run(User user, boolean dryRun) {
    Activity activity = newActivityFor(user);
    
    if (!activity.userExist()) {
      throw new EntityNotFound();
    }

    Status status = Status.UNDEFINED;
    if (dryRun) {
      LOGGER.info("This is dry run. User would have been migrated if it's an actual run");
    } else {
      status = activity.migrateUser();
    }
    
    if (dryRun) {
      LOGGER.info("This is dry run. User migration status would have been updated if it's an actual run");
    } else {
      activity.updateUserMigrationStatus(status);
    }
  }
}
```

The above `Workflow` code clearly smells with `if/else`, and it will violate the `Closed for modifiaction` SOLID 
principle as soon as new requirement to add another activity is introduced, and testing the `if/else` branch 
becomes a nightmare.

A general rule of thumb when we see `if/else` or `switch`, we should try exploiting `polymorphism`. 

In the above case, `dryRun` is a requirement, and we cannot avoid it. But using `polymorphism` and `factory` pattern,
we can burry the `dryRun` logic deep down, and avoid changes to it.


```java
final class ActivityFactory {
  static Activity create(User user, boolean dryRun) {
    if (dryRun) {
      return new DryRunActivityImp(user);
    } else {
      return new ActivityImpl(user);
    }
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

