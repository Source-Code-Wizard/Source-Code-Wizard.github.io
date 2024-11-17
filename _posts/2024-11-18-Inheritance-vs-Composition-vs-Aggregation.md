---
title: Inheritance vs Composition vs Aggregation
date: 2024-10-17 12:00:00 -500
categories: [ software-design ]
tags: [ java ]
---

## Inheritance
Inheritance is a mechanism where a new class is based on an existing class. The new class inherits the data and behaviors of the existing class, allowing it to reuse the code and extend its functionality. The new class is called the *derived* or *child* class, while the existing class is called the *base* or *parent* class.

Inheritance promotes code reuse and allows you to create a hierarchy of related classes. However, it can also lead to tight coupling between classes, making the system more rigid and harder to maintain.

## Composition
Composition is a design pattern where you build complex objects from simpler ones. In composition, an object contains an instance of another object as a member variable, and the containing object delegates certain responsibilities to the contained object.
In composition the parent object is responsible for the lifecycle of the objects that contains. 

Composition promotes flexibility and loose coupling, as the containing object can use different implementations of the contained object without affecting its own behavior. It also allows for better code organization and testability.

## Aggregation
Aggregation is a special case of composition, where the *part* has a *has-a* relationship with the *whole*. In aggregation, the *part* can exist independently of the *whole*, and the *whole* can exist without the *part*.

For example, a university *has-a* department, and a department *has-a* student. The department can exist without the university, and the student can exist without the department.

Aggregation is useful when you want to model a *part-whole* relationship where the *part* can have a separate lifecycle from the *whole*.

**Key Differences:**
1. **Inheritance:** A *is-a* relationship, where the child class inherits from the parent class.
2. **Composition:** A *has-a* relationship, where the containing object contains the contained object as a member variable.
3. **Aggregation:** A special case of composition, where the *part* can exist independently of the *whole*.

The choice between these three approaches depends on the specific requirements of your system and the relationship between the classes involved. Generally, composition and aggregation are preferred over inheritance due to their flexibility and loose coupling.

## Let's compare them with some code examples

### Inheritance Example in Java

```java
// Base class
class Animal {
    private String name;
    private int age;

    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void makeSound() {
        System.out.println("The animal makes a sound.");
    }
}

// Derived classes
class Dog extends Animal {
    private String breed;

    public Dog(String name, int age, String breed) {
        super(name, age);
        this.breed = breed;
    }

    @Override
    public void makeSound() {
        System.out.println("The dog barks.");
    }
}

class Cat extends Animal {
    private String color;

    public Cat(String name, int age, String color) {
        super(name, age);
        this.color = color;
    }

    @Override
    public void makeSound() {
        System.out.println("The cat meows.");
    }
}
```

Since we are all familiar with inheritance I provided a very simple Java example where the `Dog` and `Cat` classes inherit from the `Animal` class.
The derived classes can access the `name` and `age` properties and the `makeSound()` method from the base class, and they also add their own specific
attributes (`breed` for `Dog` and `color` for `Cat`) and override the `makeSound()` method.

### Composition Example in Java

In a true composition relationship, the lifecycle of the child objects is dependent on the parent object. This means that when the parent object is created, 
it is responsible for creating and initializing the child objects it needs. And when the parent object is destroyed, the child objects are also destroyed.

Let's look at an example that demonstrates this in Java:

```java
// Composed objects
class Engine {
    private int horsepower;

    public Engine(int horsepower) {
        this.horsepower = horsepower;
    }

    public void start() {
        System.out.println("The engine starts.");
    }

    public void shutdown() {
        System.out.println("The engine shuts down.");
    }
}

class Transmission {
    private int gears;

    public Transmission(int gears) {
        this.gears = gears;
    }

    public void shift(int gear) {
        System.out.println("The transmission shifts to gear " + gear + ".");
    }

    public void disconnect() {
        System.out.println("The transmission is disconnected.");
    }
}

// Composite object
class Car {
    private String make;
    private String model;
    private Engine engine;
    private Transmission transmission;

    public Car(String make, String model, int engineHP, int transmissionGears) {
        this.make = make;
        this.model = model;
        this.engine = new Engine(engineHP);
        this.transmission = new Transmission(transmissionGears);
    }

    public void start() {
        engine.start();
    }

    public void shiftGear(int gear) {
        transmission.shift(gear);
    }

    public void shutdown() {
        engine.shutdown();
        transmission.disconnect();
        System.out.println("The car is shut down.");
    }
}
```

In this example, the `Car` class is responsible for creating and initializing the `Engine` and `Transmission` objects that it needs. 
The `Car` class also has a `shutdown()` method that ensures the proper cleanup of the composed objects when the `Car` object is destroyed.

This demonstrates that in a composition relationship, the child objects' lifecycle is tightly coupled to the parent object. The parent object controls the creation, initialization, and destruction of the child objects.

Compared to the previous example, this showcases a more complete and accurate representation of the composition design pattern in Java.


### Inheritance vs Composition


```java
// INITIAL REQUIREMENTS:
// - Support basic audio and video playback
// - Include file loading and decryption

// With inheritance - Initial implementation
class MediaLibrary {
    protected void loadFile(String filename) {
        System.out.println("Loading file: " + filename);
    }
    
    protected void decrypt(String data) {
        System.out.println("Decrypting data");
    }
}

class AudioPlayer extends MediaLibrary {
    private int volume = 50;
    
    public void playAudio(String filename) {
        loadFile(filename);
        decrypt(filename);
        System.out.println("Playing audio at volume: " + volume);
    }
}

class VideoPlayer extends MediaLibrary {
    private int volume = 50;
    private int brightness = 70;
    
    public void playVideo(String filename) {
        loadFile(filename);
        decrypt(filename);
        System.out.println("Playing video");
    }
}

// NEW REQUIREMENT 1: Some media files don't need decryption anymore
// Problem: We're stuck with decrypt() in MediaLibrary, even when we don't need it
// Solution: Create new classes, breaking inheritance chain

class NonEncryptedAudioPlayer extends MediaLibrary {
    private int volume = 50;
    
    public void playAudio(String filename) {
        loadFile(filename);
        // Can't skip decrypt() without changing base class
        decrypt(filename); // Unnecessary operation!
        System.out.println("Playing audio");
    }
}

// NEW REQUIREMENT 2: Add streaming support
// Problem: MediaLibrary assumes file-based loading, need major restructuring
// Need to create parallel inheritance hierarchy!

class StreamingMediaLibrary {
    protected void initStream(String url) {
        System.out.println("Initializing stream: " + url);
    }
}

class StreamingAudioPlayer extends StreamingMediaLibrary {
    private int volume = 50;
    
    public void playAudio(String url) {
        initStream(url);
        System.out.println("Streaming audio");
    }
}
```

```java
MediaLibrary
↓
├── AudioPlayer (has volume, needs decrypt)
├── VideoPlayer (has volume and brightness, needs decrypt)
└── NonEncryptedAudioPlayer (has volume, doesn't need decrypt but forced to have it)
```


#### New Streaming Hierarchy:
```java
StreamingMediaLibrary
↓
└── StreamingAudioPlayer (has volume, completely different loading mechanism)
```

#### Problems of this design:
- Duplicate volume control in both hierarchies
- Can't share code between file-based and streaming players
- Must maintain two separate class hierarchies
- Adding new features requires updating both hierarchies

#### Better Solution with Composition:

```java
// Components that can be mixed and matched
interface MediaLoader {
    void load(String source);
}

class FileLoader implements MediaLoader { ... }
class StreamLoader implements MediaLoader { ... }
class Decryptor { ... }
class VolumeControl { ... }

// Single player class that can handle both cases
class ModernAudioPlayer {
    private final MediaLoader loader;      // Can be FileLoader or StreamLoader
    private final Decryptor decryptor;     // Optional
    private final VolumeControl volume;    // Shared functionality
}
```

#### Real-World Example:
```java

// With parallel hierarchies (problematic):
AudioPlayer filePlayer = new AudioPlayer();
filePlayer.playAudio("song.mp3");

StreamingAudioPlayer streamPlayer = new StreamingAudioPlayer();
streamPlayer.playAudio("http://stream.music.com/song");

// With composition (better):
ModernAudioPlayer player1 = new ModernAudioPlayer(new FileLoader(), new Decryptor());
ModernAudioPlayer player2 = new ModernAudioPlayer(new StreamLoader()); // No decryptor needed

// Both players use the same class but with different components!
```
#### This composition approach:

- Eliminates Duplication: Volume control is a single reusable component
- Flexible Loading: Can switch between file and streaming without inheritance
- Optional Features: Decryption can be included or excluded easily
- Easy to Extend: New features (like caching) just become new components

Think of it like building with LEGO blocks:

- Inheritance approach: You have two separate, incompatible sets of blocks
- Composition approach: All blocks work together, and you can pick the ones you need

This example perfectly demonstrates why parallel hierarchies are a code smell and why composition often provides a more flexible solution. Instead of maintaining multiple inheritance trees, you create reusable components that can be combined as needed.
In object-oriented design, choosing between inheritance and composition is a crucial decision. Here's why composition often proves to be the more flexible choice.

#### On the other hand there are cases where inheritance proves to be a better choice :

- The relationships are natural and stable
- The base behavior is unlikely to change
- Code reuse is significant and meaningful
- Polymorphism provides real benefits

Remember, the key is not to avoid inheritance completely, but to use it when it truly models the problem domain and relationships correctly.
