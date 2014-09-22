# Aspect-oriented Programming

If you've been a programmer for a while or have received formal training as a programmer, it's very likely you've heard of Object-Oriented programming, or OOP. This way of viewing code focuses on creating objects that represent the data and behavior of real-world objects. OOP programmers tend to organize modules into groups that inherit from each other and are (hopefully) loosely coupled or independent of one another.

## Cause for Concern

Occasionally, a technique or feature in an application doesn't seem to fit well in a single object. Rather than being encapsulated once, it seems to need to "cut across" many different parts of the architecture at once. Logging, error detection and handling, workflow auditing, and caching are some examples of cross-cutting concerns that find their way into most web applications.

Aspect-Oriented Programming, or AOP, is a technique that focuses on the organization and modularity of these cross-cutting concerns.

## AOP in li3

Aspect Oriented Programming finds a home in the li3 in at least two main pieces of functionality: filters and strategies.

An entire guide is devoted to li3's filtering system, but let's review it at a high level to see how it fits in the AOP paradigm. A filter is basically a bit of code you want applied to the existing flow of logic in the framework. For example, you might want a bit of logic that logs errors and sends them to a web service for reporting. This cross-cutting concern could really be utilized in many different parts of the code: in the model, view, or controller layers. Filters allow you to inject your logic into the flow, logging errors at whatever level they occur. This happens without having to extend or subclass core objects, nor implement messy beforeX afterX methods in your application classes.

Apart from filters, strategies are often used as a way to implement many different ways to accomplish a similar task. For example, the li3 session functionality is handled by the Session class. You can configure that class to use a number of different storage engines, each with it's own storage strategy. For example, you can configure sessions to be stored in memory either in JSON or Base64 format. Strategies create a cross-cutting solution that can be applied to a class that's been adapted to handle them.

## Summary

We enjoy the AOP flavor li3 has, mostly for it's centralized way to deal with cross-cutting concerns. These features usually start scattered across a codebase, and li3 solves this problem by facilitating a mechanism to house this logic. Though it does take some getting used to, embracing AOP in your application can help you organize these features in a concise and easily managed way.