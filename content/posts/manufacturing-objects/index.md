---
title: Manufacturing Objects
date: 2021-10-23T09:20:38-04:00
draft: true
cover: media/060cd3895af729bdf52d6c9004a9653758fc5b7a_full.jpg
tags:
  - programming
resources: []
---
In object oriented programming factory patterns are common design patterns for object creation. They are also easy to confuse and can be presented with ambiguous meaning.

In this post I'm going to try and clarify the differences between these factory patterns and the purposes behind them.
<!--more-->

---

To start, let's define an abstract class which presents an interface which provides access to the common functionality of an associated set of objects

## Starting Conditions
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

### Something that uses the interface
```java
public abstract class PetStore {
  private Animal onSale;
  public void useAnimal() {
    System.out.println(this.onSale.makeSound());
  }
}
```

### Goal
What we want to do is to come up with an abstraction for creating a pet store where the type of pet sold is specified by the user at runtime.

## Creation Strategies
### Basic Implementation
Our first implementation uses the concept of dependency injection to pass
the animal object to the pet store class when it is created
```java
public class BasicPetStore extends PetStore {
  public PetStore(Animal animal) {
    this.onSale = animal;
  }
}

public class Application {
  public static void main(String[] args) {
    String typeOfStore = args[1];
    PetStore store;

    // We have to know how to create both the pet store and the type of animal
    if (typeOfStore.equals("Cat")) {
      store = new BasicPetStore(new Cat());
    } else if (typeOfStore.equals("Dog")) {
      store = new BasicPetStore(new Dog());
    }

    store.usesAnimal();
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
  // Bylaw 1234 - Can't sell cats on Friday the 13th
  if (!(today.getDayOfMonth() = 13 && today.getDayOfWeek() == DayOfWeek.FRIDAY)) {
    store = new BasicPetStore(new Cat());
  }
} else if (typeOfStore.equals("Dog")) {
  store = new BasicPetStore(new Dog());
}
...
```
While quick and easy to implement it complicates the creation logic in a way that isn't visible by looking at just the pet store class.

### Factory Method
Our first attempt at abstraction is to delegate creation to a method which is overridden by child classes that implement the logic for creating one type of animal each.
```java
public class FactoryMethodPetStore extends PetStore {
  public AbstractPetStore() {
    this.onSale = getAnimal();
  }
  protected Animal getAnimal();
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

    // Real type of petStore changes based on type of animal
    if (typeOfStore.equals("Cat"))
      petStore = new CatStore();
    } else if (typeOfStore.equals("Dog")) {
      petStore = new DogStore();
    }
  }
  petStore.usesAnimal();
}
```
This encapsulates the creation of each animal object meaning we only have to initialize the type of pet store in the main application instead of the pet store and animal


### Abstract Factory
```java
public class AbstractAnimalFactory {
  public Animal get();
}

public class CatFactory extends AbstractAnimalFactory {
  @override
  public Animal get() {
    return new Cat();
  }
}

public class DogFactory extends AbstractAnimalFactory {
  @override
  public Animal get() {
    return new Dog();
  }
}

public class AbstractFactoryPetStore extends PetStore {
  public PetStore(AbstractAnimalFactory factory) {
    this.onSale = factory.get();
  }
}

public class Application {
  public static void main(String[] args) {
    PetStore store;
    AbstractAnimalFactory factory;
    String typeOfStore = args[1];

    // Real type of factory changes, but type of petStore stays the same
    if (typeOfStore.equals("Cat")) {
      factory = new CatFactory();
    } else if (typeOfStore.equals("Dog")) {
      factory = new DogFactory();
    }

    store = new PetStore(factory);
    store.usesAnimal();
}
```

### Dynamic Class Instantiation
```java
public class AnimalFactory {
  public Animal create(string name) {
    //
    if (name.equals("Cat")) {
      return new Cat();
    } else if (name.equals("Dog")) {
      return new Dog();
    }
  }
}

public class DynamicPetStore extends PetStore {
  public DynamicPetStore(String type) {
    AnimalFactory factory;
    this.onSale = factory.create(type);
  }
}

public class Application {
  public static void main(String[] args) {
    String typeOfStore = args[1];
    AnimalFactory factory = new Factory;
    PetStore store = new BasicPetStore(factory.create(typeOfStore));

    store.usesAnimal();
  }
}
```

## Common threads
- Encapsulation of object creation
- Runtime selection of concrete class

## Takeaway
- Factory => encapsulating object creation
- Dynamic Instantation => selection of concrete class at runtime