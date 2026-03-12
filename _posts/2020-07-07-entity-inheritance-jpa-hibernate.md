---
layout: post
title: "Entity inheritance with JPA and Hibernate"
excerpt: "There are four different ways to use inheritance with entity classes."
date: 2020-07-07
tags: [Development, Databases]
feature_image: __GHOST_URL__/content/images/2020/07/julia-sabiniarz-trJqgtb3oIE-unsplash.jpg
---

Very often many fields share a number of same fields. Instead of duplicating these fields in all entity classes it would be great if we can re-use code by extending a common type. Well, at least that is what OOP suggests. However, I didn't know how extensions affects JPA entities and how that reflects to the database table.

In order to find that answer, I ran a short experiment. I used a Spring Boot application with an in-memory H2 database and I compared the tables created for different types of inheritance.

## Default entity inheritance behavior

As the main super class I use a dummy Event class.

```java
@Getter
@Setter
@NoArgsConstructor
@Entity
public class Event {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String description;
}
```

![](__GHOST_URL__/content/images/2020/07/Event.PNG)

It has two sub classes, FinancialEvent and UserEvent.

```java
@Getter
@Setter
@NoArgsConstructor
@Entity
public class UserEvent extends Event {
    private Long userId;
}

@Getter
@Setter
@NoArgsConstructor
@Entity
public class FinancialEvent extends Event {
    private BigDecimal amount;
}
```

![](__GHOST_URL__/content/images/2020/07/EventWithTwoExtensions.PNG)

In the database, the two entities get represented by a single Event table. Then I wondered, what happens if the extended classes have a field with the same name but different types. I added a field named **clashing**, in one class as **long** and **int** in the other.

![](__GHOST_URL__/content/images/2020/07/ClashingLongInt.PNG)

It was represented by a single column in the table. Then I changed it to String in onе of the sub classes. That worked as well. Only this time, the type of the column was changed to varchar.

![](__GHOST_URL__/content/images/2020/07/ClashingDifferentTypes.PNG)

I saved a couple of entities, one of each type just to see how the data would look. And it looked as expected. The table has a lot of null columns and it is not in third normal form.

![](__GHOST_URL__/content/images/2020/07/ClashingFieldsTable.PNG)

That is what happens by default when we extend an entity. It is the single table strategy. However, there are two other inheritance strategies we can use, table per class and joined.

## Table per class strategy

The inheritance strategy is configured using the Inheritance annotation. Also, in order to better illustrate what happens to the tables, I added an addition date field to the Event entity.

```java
@Getter
@Setter
@NoArgsConstructor
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public class Event {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private Date date;
}
```

However, simply adding the annotation did not work and I got an exception.

```java
nested exception is org.hibernate.MappingException: Cannot use identity column key generation with <union-subclass> mapping for: org.igorski.jpaspringexamples.Event
```

This is solved by changing the id generation strategy from identity to table.

```java
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE)
    private Long id;
```

Since I am using the table per class strategy, unsurprisingly, a table is created for each class, plus on for the id.

![](__GHOST_URL__/content/images/2020/07/TablePerClass-1.PNG)

The date field is replicated in each of the tables. Each table that matches a sub class entity will get all fields of the super class entity.

## Joined strategy

The documentation says that when using this strategy,  fields that are specific to a subclass are mapped to a separate table than the fields that are common to the parent class, and a join is performed to instantiate the subclass.

When I switched to the joined strategy I noticed that **GenerationType.IDENTITY** works again.

![](__GHOST_URL__/content/images/2020/07/JoinedStrategy-1.PNG)

The date field is present only in the Event table. Again, as expected from the strategy name, in order to get all the fields I need to perform a join.

```java
SELECT e.*, fe.*
FROM event AS e
JOIN financial_event AS fe
ON e.id = fe.id
```

## Converting to MappedSuperclass

As the last experiment, I changed the Event class, from an **Entity** to a **MappedSuperclass**.

```java
@Getter
@Setter
@NoArgsConstructor
@MappedSuperclass
public class Event {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
```

![](__GHOST_URL__/content/images/2020/07/MappedSuperClassEvent.PNG)

After this, it is obvious that Event is not treated as an entity any more and it does not exist as a table.

## Summary

There are four different ways to use inheritance with entity classes. Those are single table, table per class, joined or to use a MappedSuperclass. They all have their benefits and flaws. I wouldn't say there is a preferred way of doing things. It will always depend on your needs and your end goal.

## Links

* Cover image by [Julia Sabiniarz](https://unsplash.com/@julczed?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/rows?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)