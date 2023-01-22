---
title: Using Builder Pattern to avoid multiple constructors
date: 2022-10-16 12:00:00
categories: [Tech, Design Patterns]
tags: [designpatterns, programming, bestpractices, java]
---

# Using Builder Pattern to avoid multiple constructors

### Introduction

In most applications, we write model classes to represent real life entities like *Person, Employee, Customer,* etc. Whenever a new attribute is added to a model class we tend to create a new constructor with our newly added attribute to instantiate an object of that class. This leads to ***constructor cluttering*** as the class now contains multiple constructors each with different attributes.

Have a look at this class:

```java
@Getter
public class Team extends BaseEntity {
	private String teamId;
	private String teamName;
	private boolean isActive;
	private LocalDate startDate;
	private LocalDate endDate;
	private String optionalParam1;
	
	public Team(long id, String teamId, String teamName) {
		super(id);
		//Code to intialize attributes
	}
	
	public Team(long id, String teamId, String teamName, boolean isActive) {
		super(id);
		//Code to intialize attributes
	}
	
	public Team(long id, String teamId, String teamName,  boolean isActive, 
		    LocalDate startDate, LocalDate endDate) {
		super(id);
		//Code to intialize attributes
	}
	
	public Team(long id, String teamId, String teamName, boolean isActive, 
		    LocalDate startDate, LocalDate endDate, String optionalParam1) {
		super(id);
		//Code to intialize attributes
	}
	
	public Team(long id, String teamId, String teamName, String optionalParam1) {
		super(id);
		//Code to intialize attributes
	}
}

```

To instantiate an object of the Team class in different use cases, multiple constructors are created each with different attributes. Just observe the sheer amount of constructors there are! Now, if another property is added to the Team class, the new developer will again create a new constructor with the newly added property to satisfy his/her use case. **This makes the class unmaintainable in the long run**. This is also known as the ***telescoping constructors problem*.**

Surely there has to be a better way to write such classes, don’t you think? Yes! And that’s where the **Builder Pattern** comes in.

***The Builder Pattern proves beneficial in writing clean code if the class has many attributes, some of which are optional or may not be required in different scenarios.***

## What is the Builder Pattern?

The technical definition goes like this:
> **Builder** is a creational design pattern that** lets you construct complex objects step by step**. The pattern allows you to produce different types and representations of an object using the same construction code.

My goal is to keep this post practical, so I will not go into too much technical detail. You can read more about the Builder Pattern from the following sources:

* Effective Java by Joshua Bloch (Chapter 2)

* How to do in Java: [Builder Design Pattern](https://howtodoinjava.com/design-patterns/creational/builder-pattern-in-java/)

* Refactoring Guru: [Builder](https://refactoring.guru/design-patterns/builder)

## Implementation and practicality

You can very well implement Builders from scratch but [Lombok ](https://projectlombok.org/)provides a very handy annotation for it. Here is the documentation link for the same: [@Builder](https://projectlombok.org/features/Builder).

Now let’s see how we can optimize the above Team model class using the Lombok annotation:

```java
@Getter
@Builder
public class Team {
	private String teamId;
	private String teamName;
	private boolean isActive;
	private LocalDate startDate;
	private LocalDate endDate;
	private String optionalParam1;
}
```

That’s pretty much it! Now, in one of my use case, I only want to construct an object using only the *“teamId”, “teamName” and “isActive” *attributes:

```java
Team.builder()
.teamId("Team123")
.teamName("Sample Team")
.isActive(true)
.build();
```

***This turns the object creation into a clean & fluent API.***

## How to handle the initialization of the super class attributes?

The Team model class was extending the *BaseEntity* class which contains the *“id”* attribute. How do we initialize that *“id”* attribute? Lombok doesn’t support the initialization of the super class attributes directly by applying *@Builder* annotation on top of the child class.

If we look at the [*@Builder* documentation](https://projectlombok.org/features/Builder), we can see that it can also be applied to methods & constructors. Using that, this is how we can work our way around:

```java
@Getter
public class Team extends BaseEntity {
	private String teamId;
	private String teamName;
	private boolean isActive;
	private LocalDate startDate;
	private LocalDate endDate;
	private String optionalParam1;
	
	@Builder
	private Team(long id, String teamId, String teamName, boolean isActive, LocalDate startDate,
		LocalDate endDate, String optionalParam1) {
		super(id);
		//Code to initialize attributes
	}
}
```

Here I have [generated an AllArgs constructor using IntelliJ IDEA](https://www.jetbrains.com/idea/guide/tips/generate-getters-and-setters/) and simply put the *@Builder* annotation on top of it. Now, we can construct the object of the Team class using the same fluent API as mentioned before:

```java
Team.builder()
.id(12)
.teamId("Team123")
.teamName("Sample Team")
.isActive(true)
.build();
```

Now when a new attribute needs to be added to the Team class, the developer simply has to add the attribute to the class and append it to the *AllArgs constructor* and voila!

**An additional advantage of using Builders is that it does not break the existing code where the object was created using some other attributes of the class.**

*Hoping this proves useful in your development journey. ⚔️*

*~ Aditya Mahajan*