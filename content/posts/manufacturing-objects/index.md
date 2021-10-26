---
title: Manufacturing Objects
date: 2021-10-26T16:10:00-04:00
draft: false
cover: media/cover.jpg
tags:
  - programming
resources: []
---
In object oriented programming, factory patterns are common design patterns for object creation. 
Online I've found some explainations of them lacking and often confused with a different concept entirely.

In this post I'm going to try and clarify the differences between these patterns and concepts and the purposes behind them.
<!--more-->
---

## Starting Conditions
To start, let's define an abstract class which presents an interface which provides access to the common functionality of an associated set of objects

### Abstract Class
```java
public abstract class Animal {
  public String makeSound();
}

public class Cat extends Animal {
  @override
  public String makeSound() {
    return "Meow";
  }
}

public class Dog extends Animal {
  @override
  public String makeSound() {
    return "Woof";
  }
}
```

### Something that uses the class
```java
public abstract class PetStore {
  private Animal onSale;
  public void useAnimal() {
    System.out.println(onSale.makeSound());
  }
}
```

### Goal
What we want is to get the correct concrete class of Animal to the PetStore instance in a way that can be configured by the user.

## Creation Strategies
### Basic Implementation
Our first implementation uses the concept of dependency injection to pass
the animal object to the pet store class when it is created
```java
public class BasicPetStore extends PetStore {
  public PetStore(Animal animal) {
    onSale = animal;
  }
}

public class Application {
  public static void main(String[] args) {
    String typeOfStore = args[1];
    PetStore store;

    if (typeOfStore.equals("Cat")) {
      store = new BasicPetStore(new Cat());
    } else if (typeOfStore.equals("Dog")) {
      store = new BasicPetStore(new Dog());
    }

    store.useAnimal();
  }
}
```
The weakness here is that while dependency injection is a very strong pattern, with no abstraction in the actual object creation process we end up having a point where everything must be created at once.

For more complex applications this point can become very large and for non-trivial initialization may end up containing logic about components that isn't easily visible to developers. 

For example:
```java
import java.time.*;
...
if (typeOfStore.equals("Cat")) {
  LocalDate today = LocalDate.now();
  // Bylaw 1234 - Can't sell cats on Friday the 13th so sell dogs instead
  if (today.getDayOfMonth() == 13 && today.getDayOfWeek() == DayOfWeek.FRIDAY) {
    store = new BasicPetStore(new Dog());
  } else {
    store = new BasicPetStore(new Cat());
  }
} else if (typeOfStore.equals("Dog")) {
  store = new BasicPetStore(new Dog());
}
...
```
While quick and easy to implement it complicates the creation logic in a way that isn't visible by looking at just the pet store class.

### Factory Method
Our first attempt at abstraction is to delegate Animal creation to a method in the PetStore class which can be overridden for different types of stores.
```java
public class FactoryMethodPetStore extends PetStore {
  public AbstractPetStore() {
    onSale = getAnimal();
  }
  protected abstract Animal getAnimal();
}

public class DogStore extends FactoryMethodPetStore {
  @override
  protected Animal getAnimal() {
    return new Dog();
  }
}

public class CatStore extends FactoryMethodPetStore {
  @override
  protected Animal getAnimal() {
    return new Cat();
  }
}
public class Application {
  public static void main(String[] args) {
    String typeOfStore = args[1];
    PetStore petStore;

    if (typeOfStore.equals("Cat"))
      petStore = new CatStore();
    } else if (typeOfStore.equals("Dog")) {
      petStore = new DogStore();
    }
  }
  petStore.useAnimal();
}
```
This encapsulates the creation of each animal object meaning we only have to initialize the type of PetStore in the main application instead of the PetStore and Animal

We can also extend this to support the same logic as before prohibiting the sale of cats on Friday the 13th

```java
import java.time.*;

public class CatStore extends FactoryMethodPetStore {
  @override
  protected Animal getAnimal() {
    if (today.getDayOfMonth() == 13 && today.getDayOfWeek() == DayOfWeek.FRIDAY) {
      return new Dog();
    else {
      return new Cat();
    }
  }
}
```
Unlike before this logic is now visible when inspecting the source code for the concrete class instead of being potentially hidden elsewhere.

This method still isn't perfect because now creating the instance of Animal is tied to the instance of FactoryMethodPetStore. To see the problem with this consider what happens if we now want to sell two types of Animals.

```java
public class DoubleFactoryMethodPetStore extends PetStore {
  private Animal alsoOnSale;
  public FactoryMethodPetStore() {
    onSale = getAnimal();
    alsoOnSale = getSecondAnimal();
  }
  protected abstract Animal getAnimal();
  protected abstract Animal getSecondAnimal();
}
```

By adding a second abstract method to the parent class we now have to change the implementation of the child classes to accomodate

```java
public class CatCatPetStore extends DoubleFactoryMethodPetStore {
  @override
  protected Animal getAnimal() {
    return new Cat();
  }
  @override
  protected Animal getSecondAnimal() {
    return new Cat();
  }
}
public class CatDogPetStore extends DoubleFactoryMethodPetStore {
  @override
  protected Animal getAnimal() {
    return new Cat();
  }
  @override
  protected Animal getSecondAnimal() {
    return new Dog();
  }
}
public class DogCatPetStore extends DoubleFactoryMethodPetStore {
  @override
  protected Animal getAnimal() {
    return new Dog();
  }
  @override
  protected Animal getSecondAnimal() {
    return new Cat();
  }
}
public class DogDogPetStore extends DoubleFactoryMethodPetStore {
  @override
  protected Animal getAnimal() {
    return new Dog();
  }
  @override
  protected Animal getSecondAnimal() {
    return new Dog();
  }
}
```
This means that the overhead of having the factory method within the calling class increases for every type of animal or additional abstract method.

### Abstract Factory
We saw that delegating the creation of the Animal objects to a method was able to encapsulate the creation logic but as we extended the abstract class it created overhead since every child class had to be adapted to those changes.

The Abstract Factory pattern is able to avoid this by refactoring the factory method into a dedicated abstract class which can be implemented for each type.
```java
public class AbstractAnimalFactory {
  public Animal create();
}

public class CatFactory extends AbstractAnimalFactory {
  @override
  public Animal create() {
    return new Cat();
  }
}

public class DogFactory extends AbstractAnimalFactory {
  @override
  public Animal create() {
    return new Dog();
  }
}

public class Application {
  public static void main(String[] args) {
    PetStore store;
    AbstractAnimalFactory factory;
    String typeOfStore = args[1];

    if (typeOfStore.equals("Cat")) {
      factory = new CatFactory();
    } else if (typeOfStore.equals("Dog")) {
      factory = new DogFactory();
    }

    store = new BasicPetStore(factory.create());
    store.useAnimal();
}
```
Here we can see that instead of modifying the PetStore class we can use the original one that receives the instance of Animal through dependency injection because it's the type of concrete factory that changes instead.

This technique still allows us to encapsulate the creation logic in one place

```java
import java.time.*;

public class CatFactory extends AbstractAnimalFactory {
  @override
  protected Animal create() {
    if (today.getDayOfMonth() == 13 && today.getDayOfWeek() == DayOfWeek.FRIDAY) {
      return new Dog();
    else {
      return new Cat();
    }
  }
}
```

But unlike when the factory method was located within the target class, we can extend the class without creating a multiplicitive amount of overhead

```java

public class DoublePetStore extends PetStore {
  private Animal alsoOnSale;
  DoublePetStore(Animal first, Animal second) {
    onSale = first;
    alsoOnSale = second;
  }
  public void useOtherAnimal() {
    System.out.println(alsoOnSale.makeSound());
  }
}

public class Application {
  public static void main(String[] args) {
    PetStore store;
    AbstractAnimalFactory firstFactory;
    AbstractAnimalFactory secondFactory;
    String typeOfStore = args[1];
    String otherTypeOfStore = args[2];

    if (typeOfStore.equals("Cat")) {
      firstFactory = new CatFactory();
    } else if (typeOfStore.equals("Dog")) {
      firstFactory = new DogFactory();
    }

    if (otherTypeOfStore.equals("Cat")) {
      secondFactory = new CatFactory();
    } else if (otherTypeOfStore.equals("Dog")) {
      secondFactory = new DogFactory();
    }

    store = new DoublePetStore(firstFactory.create(), secondFactory.create());

    store.useAnimal();
    store.useOtherAnimal();
  }
}
```
Now we have been able to successfully decouple the requirements of creating the Animal objects from the PetStore class which uses them but we still have the problem of selecting which concrete factory objects to create.

### Dynamic Class Instantiation
The ability to instantiate an object where the type name is stored in a parameter is called dynamic instantation. Programming languages such as Java and C++ don't support this at a syntax level so to achieve the same effect the behaviour must be implemented by hand.
```java
public class AnimalFactory {
  public Animal create(String name) {
    if (name.equals("Cat")) {
      return new Cat();
    } else if (name.equals("Dog")) {
      return new Dog();
    }
  }
}

public class Application {
  public static void main(String[] args) {
    String typeOfStore = args[1];
    AnimalFactory factory = new Factory;
    PetStore store = new BasicPetStore(factory.create(typeOfStore));

    store.useAnimal();
  }
}
```
Here we have a single factory method in the ```AnimalFactory``` class which takes the type of concrete Animal by name as a string parameter.
Like in the initial example, creation of the concrete Animal objects is not delegated to an object but we have been able to achieve the effect of being able to create the concrete class by passing the type name as a string.

## Putting it all together
Now that we have introduced the Factory Method pattern, the Abstract Factory pattern, and the concept of Dynamic Instantation we can show how these concepts fit together to create a piece of well organized, extensible software.
```java
import java.util.HashMap;

public abstract class AbstractAnimalFactory {
  public abstract Animal create();
}

public class AnimalRegistry {
  private HashMap<String, AbstractAnimalFactory> registry;
  public AnimalRegistry() {
    registry = new HashMap<String, AbstractAnimalFactory>;
  }
  public void register(String name AbstractAnimalFactory factory) {
    registry.put(name, factory);
  }
  public Animal create(String name) {
    return registry.get(name).create();
  }
}

public class CatFactory extends AbstractAnimalFactory {
  public Animal create() {
    return new Cat();
  }
}

public class DogFactory extends AbstractAnimalFactory {
  public Animal create() {
    return new Dog();
  }
}

public class Application {
  public static void main(String[] args) {
    AnimalRegistry registry = new AnimalRegistry();
    registry.register("Cat", new CatFactory());
    registry.register("Dog", new DogFactory());

    String typeOfStore = args[1];
    PetStore store = new BasicPetStore(registry.create(typeOfStore));

    store.useAnimal();
  }
}
```
Since the objects are still being created through a factory method we have encapsulation of the creation logic,
and using the AbstractFactory class with a class registry object give us the flexability to extend the application with minimal additional overhead.

## Conclusion
The takeaway from this is that a factory pattern is just a means for encapsulating object creation, and that dynamic instantation is a seperate, but related concept for selecting which classes to use at runtime.

To be complete, it's also worth looking at the overhead required to use these techniques.

### Overhead
To quantify the overhead of using these abstractions, we can measure how much overhead there is to add an additional type of Animal.

| Technique                                      | Lines of Overhead per Animal                        |
|:-----------------------------------------------|----------------------------------------------------:|
| Pure Dependency Injection                      |                                                   3 |
| Factory Method                                 | #Abstract Factory Methods * (#Concrete Objects + 4) |
| Abstract Factory                               |                                                   9 |
| Dependency Injection with Dynamic Instantation |                                                   3 |
| Abstract Factory with Class Registry           |                           7 (6 Factory, 1 Registry) |

The techniques with the lowest overhead are the two that don't encapsulate any of the creation logic, followed by our final pattern, the Abstract Factory, and finally the plain Factory Method.

This is to be expected since there will be overhead for any form of abstraction and one of the primary challenges when architecting software is how to manage the growth of a codebase so it stays manageable while not getting in the way of adding new functionality.
Another challenge is being able to recognize when it is appropriate to refactor and add these abstractions for maintainability.

For small code bases it doesn't make sense to have more code dedicated to building abstractions than the actual functionality but if you understand these patterns you'll be able to gracefully transition to them if or when the time comes.

### Final Code
```java
import java.util.HashMap;

public abstract class Animal {
  public String makeSound();
}

public class Cat extends Animal {
  @override
  public String makeSound() {
    return "Meow";
  }
}

public class Dog extends Animal {
  @override
  public String makeSound() {
    return "Woof";
  }
}

public class Iguana extends Animal {
  @override
  public String makeSound() {
    return "What sound does an Iguana make?";
  }
}

public abstract class AbstractAnimalFactory {
  public abstract Animal create();
}

public class AnimalRegistry {
  private HashMap<String, AbstractAnimalFactory> registry;
  public AnimalRegistry() {
    registry = new HashMap<String, AbstractAnimalFactory>;
  }
  public void register(String name AbstractAnimalFactory factory) {
    registry.put(name, factory);
  }
  public Animal create(String name) {
    return registry.get(name).create();
  }
}

public class CatFactory extends AbstractAnimalFactory {
  @override
  protected Animal create() {
    if (today.getDayOfMonth() == 13 && today.getDayOfWeek() == DayOfWeek.FRIDAY) {
      return new Dog();
    else {
      return new Cat();
    }
  }
}

public class DogFactory extends AbstractAnimalFactory {
  @override
  public Animal create() {
    return new Dog();
  }
}

public class IguanaFactory extends AbstractAnimalFactory {
  @override
  public Animal create() {
    return new Iguana();
  }
}

public abstract class PetStore {
  private Animal onSale;
  public void useAnimal() {
    System.out.println(onSale.makeSound());
  }
}

public class DoublePetStore extends PetStore {
  private Animal alsoOnSale;
  DoublePetStore(Animal first, Animal second) {
    onSale = first;
    alsoOnSale = second;
  }

  public void useOtherAnimal() {
    System.out.println(alsoOnSale.makeSound());
  }
}

public class Application {
  public static void main(String[] args) {
    AnimalRegistry registry = new AnimalRegistry();
    registry.register("Cat", new CatFactory());
    registry.register("Dog", new DogFactory());
    registry.register("Iguana", new IguanaFactory());

    String typeOfStore = args[1];
    String otherTypeOfStore = args[2];
    DoublePetStore store = new DoublePetStore(registry.create(typeOfStore), registry.create(otherTypeOfStore));

    store.useAnimal();
    store.useOtherAnimal();
  }
}
```