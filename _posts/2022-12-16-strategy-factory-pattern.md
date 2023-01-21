---
title: Combination of Strategy and Factory Pattern using Guice MapBinder to eliminate if-else ladders
date: 2022-12-16 12:00:00
categories: [Tech Blogs, Design Patterns]
tags: [designpatterns, programming, bestpractices, java]
---

# Combination of Strategy and Factory Pattern using Guice MapBinder to eliminate if-else ladders

### Introduction

Even though if-else ladders can be very useful and often the right tool for the job, they can also lead to poor design choices when writing extensible code.

Let’s consider an example

```java
public class SmartRemoveService {
 
  @Inject
  private final TeamService teamService;
  
  @Inject
  private final EmployeeService employeeService;
  
  public void smartRemove(String entity, int id) {
    if(entity.equals("Team")) {
      if(teamService.exists(id)) {
      	teamService.remove(id)
      }
    } else if(entity.equals("Employee")) {
      if(employeeService.exists(id)) {	
      	employeeService.remove(id)
      }
    }
    
    throw new Exception("Invalid entity")
  }
}
```

Code of this type has a few problems with it:

1. **Single Responsibility Principle Violation**: The class and in turn, the smartRemove() method has the responsibility to deal with the smart removal of multiple entities.

1. **High Coupling:** This has led to highly coupling with two other Service classes: TeamService and EmployeeService.

1. **Maintenance issues in the long run:** Let’s say I want to add smart remove logic for an additional entity I will have to again couple this class with another Service Layer object and add another if condition in the smartRemove() method leading to unmaintainable and unreadable code.

1. **Low Abstraction:** This class also is cluttered with logic for each entity’s smart remove logic which can be abstracted away.

There are multiple ways to eliminate the if-else ladders, one of the ways which I found efficient and easy is by using a combination of Strategy Pattern with Google Guice’s MapBinder as a Factory.

### Strategy Pattern

The Strategy pattern suggests that you take a class that does something specific in a lot of different ways and extract all of these algorithms into separate classes called *strategies*. It is a behavioural design pattern that lets you define a family of algorithms, put each of them into a separate class, and make their objects interchangeable.

![](https://upload.wikimedia.org/wikipedia/commons/3/39/Strategy_Pattern_in_UML.png)

You can read more about Strategy Pattern here: [https://refactoring.guru/design-patterns/strategy](https://refactoring.guru/design-patterns/strategy)

### Google Guice’s MapBinder

MapBinder in Google Guice is especially intended for plugin-type architectures. Neither the plugin consumer nor the plugin author needs to write much setup code for extensible applications. Simply define an interface, bind implementations, and inject a map of implementations into the consumer class.

You can read more about MapBinder here: [https://github.com/google/guice/wiki/Multibindings](https://github.com/google/guice/wiki/Multibindings)

### Refactoring the SmartRemove Logic

Let’s refactor our code now.

**Step 1: Implementing the Strategy Pattern:** So the first step in eliminating the if-else ladder would be abstracting the smart remove logic into a SmartRemoveStrategy interface.

```java
public interface SmartRemoveStrategy {
	int smartRemove(int id);
}
```

Now we can concrete classes that implement this interface for each of our entities:

```java
public class TeamSmartRemoveStrategy implements SmartRemoveStrategy {
	@Inject
  	private final TeamService teamService;
  
  	void smartRemove(int id) {
      //logic
    }
}


public class EmployeeSmartRemoveStrategy implements SmartRemoveStrategy {
	@Inject
  	private final EmployeeService employeeService;
  
  	void smartRemove(int id) {
      //logic
    }
}
```

Now each individual strategy class has the responsibility of dealing with its own entity. This will also help us comply with the Single Responsibility Principle and reduce coupling.

**Step 2: Injecting and Invoking the Strategies from our SmartRemoveSerivce class: **MapBinder helps us in injecting a map of strategies into our *SmartRemoveSerivce* class thus acting as a **factory**.

The class, after refactoring, looks like this:

```java
public class SmartRemoveService {
 
  private Map<String, SmartRemoveStrategy> smartRemoveStrategyMap;
  
  @Inject
  public SmartRemoveDaoService(Map<String, SmartRemoveStrategy> smartRemoveStrategyMap) {
    this.smartRemoveStrategyMap = smartRemoveStrategyMap;
  }
  
  public void smartRemove(String entity, int id) {
    SmartRemoveStrategy smartRemoveStrategy = smartRemoveStrategyMap.get(entity);
    
    if(Objects.isNull(smartRemoveStrategy))
      throw new Exception("Invalid entity")
      
    smartRemoveStrategy.smartRemove(id);
  }
}
```

The injection of the *smartRemoveStrategyMap* will be taken care of by Guice and now the smartRemove() method has only one responsibility: **Retrieve the strategy from the map and execute it**.

In this way, the logic of each smart remove strategy is abstracted away.

We set up the injection of the Map as follows in the Module class:

```java
MapBinder<String, SmartRemoveStrategy> smartRemoveStrategyMapBinder =  
    MapBinder.newMapBinder(binder(), String.class, SmartRemoveStrategy.class);
smartRemoveStrategyMapBinder.addBinding("Team").to(TeamService.class);
smartRemoveStrategyMapBinder.addBinding("Employee").to(EmployeeService.class);
```

### Extensibility and Testability

We can now achieve extensibility as any new Entity’s smart removal logic can simply be added by doing the following:

1. Add a new concrete strategy class that implements the strategy interface.

1. Add the binding in Module class for it. No need to touch any logic in SmartRemoveService class.

Individual strategy classes are testable in nature. Test for the SmartRemoveService can also be written by injecting a stub / mocked HashMap.

This was just one of the ways via which we can refactor the if-else ladders. There are plenty more that you can explore.

*Hoping that this proves useful in helping you write better and extensible code. Thanks for reading and Happy Coding!*

~ Aditya :)
