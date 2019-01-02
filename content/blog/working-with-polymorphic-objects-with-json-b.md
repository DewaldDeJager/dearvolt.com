---
title: "Working With Polymorphic Objects With JSON-B"
date: 2019-01-02T12:11:17+02:00
draft: true
---

What is polymorphism? Explain the problem in plain English.

[Sebastian Daschner mentions the `@JsonbTypeInfo` annotation](https://blog.sebastian-daschner.com/entries/json_mapping_polymorphism_support).

Sebastian also started a [thread on the JSR 367 mailing list](https://download.oracle.com/javaee-archive/jsonb-spec.java.net/users/2016/03/0389.html) regarding this.

![Class diagram for the example of polymorphic objects](//images/polymorphic-objects-example-class-diagram.png "Class diagram for the example")

Section 3.8 of the JSON-B specification (JSR 367) states:
> Deserialization into polymorphic types is not supported by default mapping.

TODO: How does Jackson support it?
TODO: How does GSON support it?

TODO: As an enterprise systems engineer, how do I want to write code to allow for deserialisation of a list of polymorphic objects?
