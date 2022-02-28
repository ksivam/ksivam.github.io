---
layout: post 
title: "Flag Arguments"
date: 2022-02-26 
categories: blogging 
tags: programming
comments: true 
analytics: true 
excerpt_separator: <!--more-->
---

Flag Arguments are bad programming practice. Well known experts like Martin Fowler and Robert C. Martin denounce using
Flag Arguments, and advise to avoid them at all cost.

In this blog post, we will be walking through a sample code which uses Flag Argument, and see why it's a code smell.
Then the bad code will be refactored to follow the SOLID principles.

<!--more-->

Let's take a trivial example of a `Workflow` which migrates user from an old to a new system. The workflow performs
series of activities, and these activities are encapsulated in a `Activity` class. The `Workflow` provides a `dryRun`
option to simulate the migration activities without doing the actual migration.

```java
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

If we see the above `Workflow` code, the `userExist` method is called irrespective of the `dryRun` flag, but the rest of
activities are branched with `if/else` check.

This is clearly a code smell, and it will violate the `Closed for modifiaction` SOLID principle as soon as new
requirement to add another activity is introduced. Plus, testing the `if/else` branch becomes a nightmare.

A general rule of thumb when we see `if/else` or `switch`, we should try exploiting `polymorphism`.

In the above case, `dryRun` is a requirement, and we cannot avoid it. But using `polymorphism` and `factory` pattern, we
can bury the `dryRun` logic deep down, and avoid changing it.

Let's introduce an activity factory `ActivityFactory` which create a dry run activity or real migration activity based
on the `dryRun` flag. This activity factory should not change foreseeably.

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

Now, we can move the common `userExist` to an abstract activity class, and the concrete implementations can provide the
correct logic without the dry run flag.

```java
abstract class Activity {
  User user;

  boolean userExist() {
    // call the data store to check if the user exist or not. 
  }

  abstract Status migrateUser();

  abstract void updateUserMigrationStatus(Status status);
}
```

```java
class DryRunActivityImp extends Activity {
  Status migrateUser() {
    LOGGER.info("This is dry run. User would have been migrated if it's an actual run");
    return Status.MIGRATED;
  }

  void updateUserMigrationStatus(Status status) {
    LOGGER.info("This is dry run. User migration status would have been updated if it's an actual run");
  }
}
```

```java
class ActivityImp extends Activity {
  Status migrateUser() {
    // some logic to migrate the user to the new system.
    return getStatusOfUserInNewSystem();
  }

  void updateUserMigrationStatus(Status status) {
    // some logic to update the migration status of the user in old system. 
  }
}
```

With the introduction of `polymorphims` and `factory` pattern, the `Workflow` class can be refactored as below

```java
public class Workflow {

  void run(User user, boolean dryRun) {
    Activity activity = ActivityFactory.create(user, dryRun);

    if (!activity.userExist()) {
      throw new EntityNotFound();
    }

    Status status = activity.migrateUser();
    activity.updateUserMigrationStatus(status);
  }
}
```

As we can see now, the `Workflow` is much simplified, adheres to SOLID principle and testing the class becomes much
efficient. 

