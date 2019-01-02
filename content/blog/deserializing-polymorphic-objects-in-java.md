---
title: "(De)serializing Polymorphic Objects In Java"
date: 2019-01-02T12:11:17+02:00
draft: false
type: "posts"
---

TODO: What is polymorphism? Explain the problem in plain English.

> Polymorphism: The ability of something to take on many forms

Polymorphism is extremely popular in object-orientated programming. 
A typical use case is to reference instances of a subclass with the superclass.
Consider the class diagram below: 

![Class diagram of polymorphic objects](/images/polymorphic-objects-example-class-diagram.png "Class diagram of polymorphic types")

Since a `Dog` object is a `Dog`, `DomesticatedAnimal` and `Animal`, it is considered to be polymorphic with these types.
The examples below are focussed around cases where a list of polymorphic types is present, eg. A list containing a mixtureof `Dog`, `Cat` and `Goat` objects.
In Java, this can be portrayed as a `List` of `Animal`s:

```java
List<Animal> animals = new ArrayList<>();
animals.add(new Dog());
animals.add(new Cat());
animals.add(new Goat();
```

## JSON Binding
TODO: Explain why it is better to use JSON-B and not directly use Jackson or GSON.

## Serializing a List of Polymorphic Objects
In this context, serialization is the process of taking plain old Java objects (POJOs) and converting them to JSON.
With JSON-B, Jackson and GSON there is nothing special that needs to be done.
It is important to make sure that all the properties are visible to the serializer;
This is typically done with public getters.


## Deserializing a List of Polymorphic Objects

Deserialisation is when we want to go from JSON to POJOs. We typically have the JSON as a `String`.

[Sebastian Daschner mentions the `@JsonbTypeInfo` annotation](https://blog.sebastian-daschner.com/entries/json_mapping_polymorphism_support).

Sebastian also started a [thread on the JSR 367 mailing list](https://download.oracle.com/javaee-archive/jsonb-spec.java.net/users/2016/03/0389.html) regarding this.


Section 3.8 of the JSON-B specification (JSR 367) states:
> Deserialization into polymorphic types is not supported by default mapping.

TODO: How does Jackson support it?
TODO: How does GSON support it?

TODO: As an enterprise systems engineer, how do I want to write code to allow for deserialisation of a list of polymorphic objects?
