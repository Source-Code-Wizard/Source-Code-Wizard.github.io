---
title: Understanding and Implementing the Decorator Pattern in Spring Boot
date: 2024-12-30 12:00:00 -500
categories: [web-development]
tags: [server,spring-boot-3,java]
---


# Understanding and Implementing the Decorator Pattern in Spring Boot

## What is the Decorator Pattern?

The Decorator pattern is a structural design pattern that lets you dynamically add new behaviors to objects by placing them inside wrapper objects. Think of it like wrapping a gift - you start with a basic box (your core object), and you can keep adding layers of wrapping paper, ribbons, and bows (decorators) to enhance its appearance. Each wrapper adds something new while keeping the original gift unchanged.

In software terms, instead of creating a massive class with all possible feature combinations, you create a base class with core functionality and then create separate decorator classes that can add features one at a time. This approach follows the "Single Responsibility Principle" by keeping each feature in its own class and the "Open-Closed Principle" by allowing new behaviors without modifying existing code.

### Real-World Analogy

Imagine you're planning a trip to Japan. You start with a basic travel package (your base component) that includes standard accommodations and flights. Now, you can enhance this basic package with various luxury "decorators" to create your dream vacation:

Let's say you start with the Basic Package (¥3,000):
- First, you add a Supercar Rental (adds ¥5,000) - Now you can explore Japan's countryside in a GT-R or LFA
- Then add a Private Geisha Experience (adds ¥2,000) - Enhancing your cultural immersion with a traditional tea ceremony in Kyoto
- Next, you include Michelin Star Dining (adds ¥3,000) - Giving you access to exclusive restaurants like Sukiyabashi Jiro
- Finally, add Helicopter Tours (adds ¥7,000) - Providing breathtaking aerial views of Mt. Fuji

Each addition wraps around your previous choices, building upon them without altering the core travel package. The beauty of this system is that each decorator maintains awareness of what came before it. When you ask for the final price, the Helicopter Tours decorator asks the Michelin Star Dining decorator, which asks the Geisha Experience decorator, which asks the Supercar Rental decorator, which finally asks the Basic Package for its price. The same chain works for building the complete description of your custom luxury package.

This creates a seamless experience where your final package costs ¥20,000 and includes everything from ground transportation in a supercar to soaring above Japan's most iconic mountain. Each feature can be added or removed independently, and the system automatically recalculates the total price and updates the package description.

### Why Use the Decorator Pattern?

1. **Flexibility at Runtime**: Unlike inheritance, which adds behaviors at compile time, decorators let you add behaviors dynamically while the application is running. In our travel package system, this means customers can customize their trip packages on the fly.

2. **Avoiding Class Explosion**: Consider our Japan trip package system. With 7 optional features, using inheritance would require 2^7 = 128 different classes to cover all possible combinations! With decorators, we only need 7 decorator classes plus the base class.

3. **Single Responsibility**: Each decorator class handles one specific feature. This makes the code easier to understand, test, and maintain. If we need to modify how the helicopter tour pricing works, we only need to change one decorator class.

4. **Maintainable Combinations**: New features can be added by creating new decorators without touching existing code. Want to add a "Private Island Day Trip" feature? Just create a new decorator - no need to modify any existing classes.

### Common Pitfalls to Avoid

1. **Order Dependency**: Be careful if your decorators need to be applied in a specific order. This can create hidden dependencies.
2. **Complexity Creep**: While decorators are powerful, too many of them can make the system harder to understand and maintain.
3. **State Management**: When decorators maintain state, ensure proper synchronization in multi-threaded environments.

## Introduction
This project demonstrates a practical implementation of the Decorator pattern in a Spring Boot application. The system allows dynamic construction of travel packages by adding various luxury features to a base trip package. This approach showcases how the Decorator pattern can be used to extend functionality at runtime while maintaining single responsibility and open-closed principles.

## Understanding the Decorator Pattern
The Decorator pattern allows us to add new behaviors to objects dynamically by placing these objects inside wrapper objects that contain the behaviors. In our travel package system, we start with a basic Japan trip package and can enhance it with various luxury additions like supercar rentals, private helicopter tours, and personal butler services.

## Core Components

### Base Interface (TripService)
```java
public interface TripService {
    double getPrice();
    String getDescription();
}
```

This interface defines the core contract that all concrete services and decorators must follow. It specifies two fundamental operations: getting the price and description of a trip package.

**Base Implementation (BasicPackageTripServiceImpl)**
```java
@Service
public class BasicPackageTripServiceImpl implements TripService {
    @Override
    public double getPrice() {
        return 3000f;
    }

    @Override
    public String getDescription() {
        return "This is the standard package for traveling to Japan";
    }
}
```
This class provides the foundation of our travel package, implementing the basic functionality that can be enhanced by decorators.

**Abstract Decorator Base (TripServiceDecoratorBase)**

```java
abstract public class TripServiceDecoratorBase implements TripService {
  protected TripService tripService;

  public void setTripService(final TripService tripService) {
    this.tripService = tripService;
  }

  public abstract TripDecoratorType getType();
}
```
The abstract decorator class maintains a reference to a TripService object and provides the structure for concrete decorators.

**The Calculator Service** <br>
The TripExpensesCalcServiceImpl is where the magic happens. This service manages the decoration process and builds chains of decorators based on client requests. Let's break down its key components:
**Decorator Management**
```java
private Map<TripDecoratorType, TripServiceDecoratorBase> decoratorsMap;
private final @Qualifier("BasicPackageTripService") TripService standardTripService;
```
The service maintains a map of available decorators and a reference to the standard trip service. This allows for efficient lookup and application of decorators.
```java
private TripServiceDecoratorBase buildDecoratorChain(final List<TripDecoratorType> decoratorTypes) {
    TripServiceDecoratorBase tripServiceDecorator = null;
    for (int decoratorTypeIndex = 0; decoratorTypeIndex < decoratorTypes.size(); decoratorTypeIndex++) {
        tripServiceDecorator = decoratorsMap.get(decoratorTypes.get(decoratorTypeIndex));
        if (decoratorTypeIndex == 0) {
            tripServiceDecorator.setTripService(standardTripService);
        } else {
            TripServiceDecoratorBase previousTripServiceDecorator = 
                decoratorsMap.get(decoratorTypes.get(decoratorTypeIndex - 1));
            tripServiceDecorator.setTripService(previousTripServiceDecorator);
        }
    }
    return tripServiceDecorator;
}
```
This method creates a chain of decorators based on the requested types. It ensures proper linking between decorators and maintains the decoration order.

## Usage Example
**API Endpoint**
```json
POST /api/v1/trips/getPriceAndDesc
Content-Type: application/json

[
  "BASIC",
  "SUPERCAR_RENTAL",
  "MICHELIN_DINING",
  "HELICOPTER_TOURS"
]
```
**Response**
```json
{
  "price": 18000.0,
  "description": "Standard Japan Package + Supercar Rental + Michelin Star Dining + Helicopter Tours"
}
```

[Github repository](https://github.com/Source-Code-Wizard/SPDecoratorPattern)<br>
