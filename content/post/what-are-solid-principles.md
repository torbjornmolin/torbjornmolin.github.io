+++
title = "What are SOLID principles"
description = "One of those acronyms"
date = 2022-08-18T08:13:50Z
author = "Torbjörn Molin"
+++

## Background

A while back I was at a technical interview for a consultant assignment and the interviewer asked about SOLID principles and Clean Code. I had heard of both and read a most of Clean Code, but I couldn't list what the acronym SOLID stands for so in order to perhaps make me remember it next time I decided to write this.

[SOLID](https://en.wikipedia.org/wiki/SOLID) is an acronym for a set of principles for development of object oriented software promoted by 'Uncle' Bob Martin who also wrote Clean Code. Some of them are quite self explanatory and others less so. Let's list them.

## Single responsibility principle

I've seen this stated both as "classes should only have one reason to change" and "classes should have a single responsibility".

### An example

Say that we have a simple web API that calculates a sum according to some rate and some formula. The logic of how to calculate the sum is implemented inside a controller type class that is sometimes used in web API frameworks.

If either the business logic of how to calculate the rate changes or we decide that we want the interface to be a command line interface instead of a web api the class needs to be rewritten and thus has multiple responsibilities.
To resolve this, calculating the rate can be moved to a separate class. The new class has the added benefit of being potentially easier to test since it will not have the dependencies that the controller class may have.

## The open-closed principle

This states that software entities should be open to extension and closed for modification. A class can be both using inheritance. It is closed for modification since we cannot change a class that we don't control, but it is open for extension since we can inherit from it and extend it's functionality.

Interfaces and abstract classes have a similar, but slightly different way of being open and closed. We can't modify the interface but we specify some or all of the implementation.

## Liskov substitution principle

The [Liskov substitution principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle), named after Barbera Liskov states that we should be able to substitute objects of type T with objects that are of a type that inherit T.

### Examples

Say that we have an abstract class or interface `Shape` with a method `Area` that returns the area of the shape, we can imagine inheriting or implementing `Shape` with the classes `Circle` and `Rectangle`. A piece of code, say a method `TotalArea`, that takes a `Shape s` and an `int n` and returns `s.Area() * n` will be able to take either `Circle` and `Rectangle` as parameters with the behavior being the expected.

If we instead have a class `Rectangle` with two methods `SetWidth` and `SetHeight` and have a new class `Square` inherit `Rectangle` we would run in to unexpected behavior if we try to use a `Square` as a `Rectangle`. This would be a violation of the Liskov substitution principle. It is also the sort of thing that comes up when people make the (valid) argument for favoring composition over inheritance.

## The interface segregation principle

The I in SOLID. This one perhaps feels a bit more dated than some of the others. Wikipedia cites [this](https://web.archive.org/web/20150905081110/http://www.objectmentor.com/resources/articles/isp.pdf) article. The general idea is that a superclass should not expose an interface that only a subset of it's subclasses implement.

## Dependency inversion principle

This principle states that

- High-level modules should not import anything from low-level modules. Both should depend on abstractions (e.g., interfaces).
- Abstractions should not depend on details. Details (concrete implementations) should depend on abstractions.

To give an example, consider a web API framework that has a request pipeline composed of a set of steps. Not following the dependency inversion principle you could implement the steps as a set of classes that the framework uses directly. Using the dependency inversion principle the framework would define an abstraction for this, say the interface `IPipelineStep`. The implementation(s) could be defined in another module, even supplied by the users of the framework. The framework would still only depend on the abstraction that it itself defines and thus the dependency relationship is inverted compared to the version without `IPiplineStep`.

Dependency inversion thus allows for loose coupling between classes and modules.
