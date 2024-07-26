---
title: What is Dependency Injection, and how does it work?
tags: [Design Patterns]
style: border
color:
description: A brief explanation with examples of how one of the most common and widely used design patterns works.
---
**Dependency Injection** is one of the most widely used design patterns in modern software development, though it often goes unnoticed. It’s key to building scalable, maintainable, and testable applications.

Let's break down the term itself:

- **Dependency**: Refers to a relationship between two components where one relies on the other to function. In other words, a dependency is anything your code needs to work.
- **Injection**: Passing something in to be used.

So, **Dependency Injection** means providing an external dependency to a class, rather than the class creating it itself.

<br>

### Why is dependency injection important?

Consider the following example where a *WeatherController* class depends on a *WeatherService*:

```
public class WeatherController : ControllerBase
{
    private readonly IWeatherService _weatherService;

    public WeatherController()
    {
        _weatherService = new WeatherService();
    }
}
```

In this code, the controller has the **responsibility** of creating an instance of *WeatherService*, but, **why is this not recommended?**

1. **Tight Coupling:** The *WeatherController* is tightly coupled to the concrete implementation of *WeatherService*. This **lacks flexibility** because if we need to replace with a different implementation (e.g., replacing *WeatherService* with *MockWeatherService*) would require modifying the controller code, which can carry errors and consume valuable time.
2. **Hard to test**: Since *WeatherController* is responsible for creating the service instance, it's hard to mock the service class for testing purposes.
3. **Violates Dependency Inversion Principle (DPI):** It states that high-level modules should not depend on low-level modules, but rather should depend on abstractions (interfaces).
4. **Violates Single Responsibility Principle (SRP)**: According to SRP, a class should only have one reason to change. The controller is now responsible not just for handling HTTP requests but also for creating the service. If we change how the service is created (e.g., if new constructor parameters are added), we would need to modify the controller as well, which it shouldn’t be responsible for.

<br>

### How dependency injection solves these problems?

Now, consider this version:
```
public class WeatherController : ControllerBase
{
    private readonly IWeatherService _weatherService;

    public WeatherController(IWeatherService weatherService)
    {
        _weatherService = weatherService;
    }
}
```

Here, the dependency is injected into the controller's constructor, removing the responsibility of creating the instance. This gives us several benefits:
1. **Flexibility:** We can inject any implementation of the interface *IWeatherService* without changing the controller's code.
2. **Testability:** We can easily pass a mock in unit tests.
3. **Adherence to DIP and SRP:** The controller depends on an abstraction (interface), and it’s only responsible for its main job: handling requests.

<br>

### How does dependency injection work?

Dependency Injection relies on a principle called **Inversion of Control (IoC)**, that consists in delegating some responsibilities in our framework, in this case, the responsibility of creating instances of our implementations.

In C#, IoC containers are responsible for creating and managing the lifecycle of objects, like *WeatherService* in this case. We can register these dependencies in the IoC container:

```
// Program.cs
builder.Services.AddScoped<IWeatherService, WeatherService>();
```

Here, we tell the container that whenever *IWeatherService* is needed, it should provide an instance of WeatherService.

Furthermore, we need to know that there are different methods to register dependencies and to define how long instances should live and how they're reused during the application's lifecycle. In .NET, some of these methods are:
1. **AddScoped**: A new instance of the service is created once per HTTP request.
2. **AddTransient**: The instances are created each time they are requested.
3. **AddSingleton**: Creates a single instance throughout the application. It uses the same instance in all calls.

<br>

### Inversion of Control vs. Dependency Injection

It’s easy to confuse Inversion of Control with Dependency Injection. In fact, in the Dependency Injection pattern **we use Inversion of Control**, because we allow the framework to instantiate our classes.

On the other hand, Dependency Injection **specifically refers to the way we inject the concrete implementation of an interface into a class.** In other words, it's how we set the attribute in the class that uses the interface.

<br>

### Types of dependency injection

Finally, we need to know **there are different types of dependency injection**. In C# we have:
1. **Constructor Injection**: Dependencies are provided through the class constructor (as shown in the previous example). This is the most common and recommended approach.
2. **Setter Injection**: Dependencies are set via properties or setter methods after the object is constructed.

```
public class WeatherController : ControllerBase
{
    public IWeatherService WeatherService { get; set; }
    // ...
}
```

3. **Interface Injection**: The class implements an interface that defines methods for injecting dependencies.

```
// Interface that has a method to inject the WeatherService
public interface IWeatherServiceConsumer
{
    void SetWeatherService(IWeatherService weatherService);
}

// The controller implements the consumer interface and defines the method
public class WeatherController : ControllerBase, IWeatherServiceConsumer
{
    private IWeatherService _weatherService;

    public void SetWeatherService(IWeatherService weatherService)
    {
        _weatherService = weatherService;
    }
}
```

Both in Java and in C#, **constructor injection** is the most used for several reasons:
1. **Required Dependencies**: It ensures that all necessary dependencies are provided at the time of object creation.
2. **Readability**: All dependencies are clearly visible through the constructor parameters.
3. **Testability**: Makes it easy to pass mock dependencies during testing.
4. **Consistency**: Once injected, dependencies remain consistent throughout the object's lifecycle, preventing accidental changes.