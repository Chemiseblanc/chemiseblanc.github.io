---
title: "Interfaces and Base Classes"
date: 2021-11-05T10:16:17-04:00
cover: media/cover.jpg
draft: true
tags:
- programming
---
I was recommended this video on youtube by ArjanCodes detailing the differences between Protocols and Abstract Base Classes in Python.

While he tries to explain their differences, the problem is that for a language like Python the differences are entirely artificial.

Here is my take on the comparison and how it transfers to a statically typed language like C++.
<!--more-->

## Motivation for Static Types
To start from the same position as ArjanCodes, let's consider the differences in Python first by defining a class and some function that acts upon it.
```python
class Animal:
    def speak(self):
        return "Hello"

def says_what(animal):
    print(animal.speak())
```
What we want is to have multiple classes of animals with their own unique speak methods.

Now Python is a dynamically typed language so we don't actually need to worry about types or class hierarchies to achieve this behaviour
```python
class Dog:
    def speak(self):
        return "Woof"

class Cat:
    def speak(self):
        return "Meow"

def says_what(animal):
    print(animal.speak())

if __name__ == "__main__":
    dog = Dog()
    says_what(dog) # Prints "Woof"

    cat = Cat()
    says_what(cat) # Prints "Meow"
```

The problem is that for larger projects, the lack of information regarding argument types makes things more difficult.
If you see only the function ```says_what(animal)``` you have to infer the requirements of the animal parameter which is hopefully conveyed through written documentation, and that the documentation is up to date. 

By moving this information into type annotations, which are available as of Python 3.5+, it clarifies what the parameters are expected to be and moves the point of error in programs up the call stack to the function call site instead of where the incorrect type usage occurs.

## Typing with Abstract Base Classes
The first approach to adding static types is to introduce an abstract class that is inherited by concrete types that implement the methods defined in the abstract class.
```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def speak(self) -> str:
        pass

class Dog(Animal):
    def speak(self) -> str:
        return "Woof"

class Cat(Animal):
    def speak(self) -> str:
        return "Meow"

def says_what(animal: Animal):
    print(animal.speak())

if __name__ == "__main__":
    dog = Dog()
    cat = Cat()
    says_what(dog) # Prints "Woof"
    says_what(cat) # Prints "Meow"
```
The constraint this puts on the program is that now all the types must inherit from the base class and implement the abstract method. Failing to do either will introduce an error

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def speak(self) -> str:
        pass

class Iguana(Animal):
    pass

class Fish():
    def speak(self) -> str:
        return "Bloop"

def says_what(animal: Animal):
    print(animal.speak())

if __name__ == "__main__":
    iguana = Iguana() # Error, class Iguana does not implement all abstract methods of Animal
    says_what(iguana)

    fish = Fish()
    says_what(fish) # Error, fish is not of type Animal
```
Here we can see that for ```Iguana``` we get an error where the class is instantiated since it doesn't implement ```speak(self)```, but the type is compatible with the signature for ```says_what(animal: Animal)```.

For ```Fish``` we see that we don't get an error instantiating it, but we get an error when passing it to the function because even though it implements the required ```speak(self)``` method, it is not a subtype of ```Animal```


## Typing with Protocols
What protocols do is remove the requirement on inheriting a base class and conceptually change the requirements on the function signature from "Give me an Animal" to "Give me something with a method speak() that returns a string"

```python
from typing import Protocol

class Animal(Protocol):
    def speak(self) -> str:
        ...

class Iguana():
    pass

class Fish():
    def speak(self) -> str:
        return "Bloop"

def says_what(animal: Animal):
    print(animal.speak())

if __name__ == "__main__":
    iguana = Iguana()
    says_what(iguana) # Error, class Iguana doesn't implement protocol Animal

    fish = Fish()
    says_what(fish) # Prints "Bloop"
```

The constraints placed on parameters using protocols are weaker than using base classes since it acts on the capabilities of a class instead of its inheritance hierarchy. As a matter of fact, it's still possible to use abstract classes with protocols since they still implement the required functionality, but not vice-versa.

```python
from abc import ABC, abstractmethod
from typing import Protocol

class Animal(Protocol):
    def speak(self) -> str:
        ...

class AbstractAnimal(ABC):
    @abstractmethod
    def speak(self) -> str:
        pass

class Dog(AbstractAnimal):
    def speak(self) -> str:
        return "Woof"

class Cat(AbstractAnimal):
    def speak(self) -> str:
        return "Meow"

class Fish():
    def speak(self) -> str:
        return "Bloop"

def says_what(animal: Animal):
    print(animal.speak())

if __name__ == "__main__":
    dog = Dog()
    says_what(dog) # Prints "Woof"

    cat = Cat()
    says_what(cat) # Prints "Meow"

    fish = Fish()
    says_what(fish) # Prints "Bloop"
```

## Comparison to C++
Unlike Python, C++ is statically typed so the types of everything must be known at compile time.
If we recreate the same program we immediately run into an issue.
```c++
#include <iostream>
#include <string>

struct Dog {
    std::string speak() { return "Woof"; }
};

struct Cat {
    std::string speak() { return "Meow"; }
};

void says_what(/* What goes here? */ animal) {
    using std::cout, std::endl;
    cout << animal.speak() << endl;
}

int main(int, char**) {
    Cat cat{};
    says_what(cat);

    Dog dog{};
    says_what(dog);

    return 0;
}
```

For the function ```says_what()``` we have to specify what the type of the argument animal is.
There are several different ways to do this, each with their own strengths and weaknesses.

### Argument Dependant Lookup (ADL)
The first method is to use a feature called Argument Dependant Lookup, or ADL. What this does is that when the function ```says_what``` is called, the compiler will look for a version of ```says_what``` with a compatible type signature.
```c++
#include <iostream>
#include <string>

struct Dog {
    std::string speak() { return "Woof"; }
};

struct Cat {
    std::string speak() { return "Meow"; }
};

void says_what(Dog animal) {
    using std::cout, std::endl;
    cout << animal.speak() << endl;
}

void says_what(Cat animal) {
    using std::cout, std::endl;
    cout << animal.speak() << endl;
}


int main(int, char**) {
    Dog dog{};
    says_what(dog);

    Cat cat{};
    says_what(cat);

    return 0;
}
```
[Run on Compiler Explorer](https://godbolt.org/z/v33rh5W8r)

The problem using ADL in this fashion is that it creates a lot of duplication since it must be copied for each type.
In general, the benefit of ADL is when you want to group similair behaviour under the same function name, but have the behaviour customized to the specific type.

### Generic Functions
One techinique we can use to avoid duplicating ```says_what``` for each type is to make the function generic by using a template or auto parameter. This delegates generating each type of the function to the compiler and results in something very similair to the python code we had before we started using type annotations.

```c++
#include <iostream>
#include <string>

struct Dog {
    std::string speak() { return "Woof"; }
};

struct Cat {
    std::string speak() { return "Meow"; }
};

// // Says_what using a template paramter
// template<typename T>
// void says_what(T animal) {
//     using std::cout, std::endl;
//     cout << animal.speak() << endl;
// }

// Equivalent using auto
void says_what(auto animal) {
    using std::cout, std::endl;
    cout << animal.speak() << endl;
}

int main(int, char**) {
    Dog dog{};
    says_what(dog);

    Cat cat{};
    says_what(cat);

    return 0;
}
```
[Run on Compiler Explorer](https://godbolt.org/z/9jnPK4vEG)

Now, to see the motivation for using class inheritance to define interfaces consider what if we want to have a container that holds on to multiple implementations of an interface.

```c++
#include <iostream>
#include <string>
#include <vector>

struct Dog {
    std::string speak() { return "Woof"; }
};

struct Cat {
    std::string speak() { return "Meow"; }
};

void says_what(auto animal) {
    using std::cout, std::endl;
    cout << animal.speak() << endl;
}

int main(int, char**) {
    std::vector</* what goes here? */> animals{Dog{}, Cat{}};
    for (auto&& animal : animals) {
        says_what(animal);
    }
    return 0;
}
```
We've run into the same problem we had before where we need to know what type to use. Before we overcame this by using generics to automatically generate versions of ```says_what``` for each type, but in this case the container has a fixed type that each interface must belong to.

This is the real differentiating factor between using class and protocol based techniques for defining interfaces. 
In Python, containers are heterogeneous which means they can contain differing types so using protocols versus abstract base classes is ultimately an organization difference.

In C++ and other statically typed languages base classes and inheritance are necessary in order to be able to hold varing implementations of an interface in the same container or variable.

### Inheritance and Virtual Methods
To implement an interface using class inheritance in C++ it's necessary to introduce a new base class that uses the keyword ```virtual``` when defining the interface methods.

The prupose of a virtual method is to allow for dispatching calls to the method ```Animal::speak()``` to the implementation of it in the correct subclass.

There is also an additional requirement that the animal parameter of ```says_what``` must be either a reference ```Animal&``` or a pointer ```Animal*```. This is because the virtual dispatch can only happen when the parent type is accessed indirectly.
```c++
#include <iostream>
#include <string>

struct Animal {
    virtual std::string speak() = 0;
};

struct Dog : Animal {
    std::string speak() override { return "Woof"; }
};

struct Cat : Animal {
    std::string speak() override { return "Meow"; }
};

void says_what(Animal& animal) {
    using std::cout, std::endl;
    cout << animal.speak() << endl;
}

int main(int, char**) {
    Dog dog{};
    says_what(dog);

    Cat cat{};
    says_what(cat);

    return 0;
}
```
[Run on Compiler Explorer](https://godbolt.org/z/PKbqMjrP1)

### Constraining Generics
Now that we have covered the cases in which an interface must be defined through a base class, it's time to revisit the case of generic functions.

So far we've covered scenarios analogous to the untyped and abstract base class implementations in Python.

Remember that the key conceptual difference of protocols is that it moves from requiring A type that implements an interface, to requiring ANY type that implements an interface, that it is a superset of the constraint imposed by using an interface class.

In the first case with generics we used a template or auto parameter to allow the compiler to generate unique implementations of ```says_what``` for each of our animal types, and everything functioned as expected since each type implemented the ```speak``` method as expected.

Now, what if we were to pass a non-conforming type to the same function?

```c++
#include <iostream>
#include <string>

struct Plant {};

void says_what(auto animal) {
    using std::cout, std::endl;
    cout << animal.speak() << endl;
}

int main(int, char**) {
    Plant plant{};
    says_what(plant);

    return 0;
}
```
[Run on Compiler Explorer](https://godbolt.org/z/vMh3Y1n5z)

We get an error which, depending on what compiler you're using, will look something like
```
<source>:8:20: error: no member named 'speak' in 'Plant'
    cout << animal.speak() << endl;
            ~~~~~~ ^
<source>:13:5: note: in instantiation of function template specialization 'says_what<Plant>' requested here
    says_what(plant);
    ^
1 error generated.
ASM generation compiler returned: 1
<source>:8:20: error: no member named 'speak' in 'Plant'
    cout << animal.speak() << endl;
            ~~~~~~ ^
<source>:13:5: note: in instantiation of function template specialization 'says_what<Plant>' requested here
    says_what(plant);
    ^
1 error generated.
Execution build compiler returned: 1
```
Looking at the error, we can see the problem is that we're missing the ```speak()``` method.

We already know that the expected interface is that we need a speak method which returns a string, but if you look closely you'll realize that while the error mentions the speak method, it says nothing about the expected return type.

If we had no knowledge of the interface and only the error message it's possible to fix this error and still not satisfy the expected interface.
```c++
#include <iostream>
#include <string>

struct Plant {
    int speak() {return 0;}
};

void says_what(auto animal) {
    using std::cout, std::endl;
    cout << animal.speak() << endl;
}

int main(int, char**) {
    Plant plant{};
    says_what(plant);

    return 0;
}
```
[Run on Compiler Explorer](https://godbolt.org/z/E9q9avKq9)

In a more complex program where the interface may be used in several places this would either cause silent, incorrect behaviour or would set of a chain of other errors which we would have to resolve until we are hopefully able to deduce what the correct interface is.

A much better way to prevent this from happening in the first place is to come up with a constraint on our generic type which describes the required capabilities, serving the same purpose as a protocol does for types in Python.
```c++
#include <concepts>
#include <iostream>
#include <string>

struct Dog {
    std::string speak() { return "Woof"; }
};

struct Cat {
    std::string speak() { return "Meow"; }
};

// C++20 Concept
template<typename T>
concept Animal = requires(T a) {
    {a.speak()} -> std::convertible_to<std::string>;
};

// // Using a Constrained Template parameter
// template<Animal T>
// void says_what(T& animal) {
//     using std::cout, std::endl;
//     cout << animal.speak() << endl;
// }

// Equivalent with a constrained auto paramter
void says_what(Animal auto& animal) {
    using std::cout, std::endl;
    cout << animal.speak() << endl;
}

int main(int, char**) {
    Dog dog{};
    says_what(dog);

    Cat cat{};
    says_what(cat);

    return 0;
}
```
[Run on Compiler Explorer](https://godbolt.org/z/Y4sKqrnYh)

Here we use the concept syntax which is new as of C++20. It's possible to achieve the same functionality in earlier standards using SFINAE but it's more difficult to use and the errors much more verbose so we will ignore this alternative here.

Now if we use the Animal concept with the previous two erronious examples:

In the case where we were missing the ```speak()``` method we see that in the error message the expected return type is now included.
```c++
#include <concepts>
#include <iostream>
#include <string>

struct Plant {};

// C++20 Concept
template<typename T>
concept Animal = requires(T a) {
    {a.speak()} -> std::convertible_to<std::string>;
};

void says_what(Animal auto animal) {
    using std::cout, std::endl;
    cout << animal.speak() << endl;
}

int main(int, char**) {
    Plant plant{};
    says_what(plant);

    return 0;
}
```
```
<source>:21:5: error: no matching function for call to 'says_what'
    says_what(plant);
    ^~~~~~~~~
<source>:14:6: note: candidate template ignored: constraints not satisfied [with animal:auto = Plant]
void says_what(Animal auto animal) {
     ^
<source>:14:16: note: because 'Plant' does not satisfy 'Animal'
void says_what(Animal auto animal) {
               ^
<source>:11:8: note: because 'a.speak()' would be invalid: no member named 'speak' in 'Plant'
    {a.speak()} -> std::convertible_to<std::string>;
       ^
1 error generated.
ASM generation compiler returned: 1
```
[Run on Compiler Explorer](https://godbolt.org/z/aMj71M6W4)


And in the case where we implemented the correct method with the incorrect type the error includes details that the return type should be convertible to a string.
```c++
#include <concepts>
#include <iostream>
#include <string>

struct Plant {
    int speak() { return 0; }
};

// C++20 Concept
template<typename T>
concept Animal = requires(T a) {
    {a.speak()} -> std::convertible_to<std::string>;
};

void says_what(Animal auto animal) {
    using std::cout, std::endl;
    cout << animal.speak() << endl;
}

int main(int, char**) {
    Plant plant{};
    says_what(plant);

    return 0;
}
```
```
<source>:22:5: error: no matching function for call to 'says_what'
    says_what(plant);
    ^~~~~~~~~
<source>:15:6: note: candidate template ignored: constraints not satisfied [with animal:auto = Plant]
void says_what(Animal auto animal) {
     ^
<source>:15:16: note: because 'Plant' does not satisfy 'Animal'
void says_what(Animal auto animal) {
               ^
<source>:12:25: note: because type constraint 'std::convertible_to<int, std::string>' was not satisfied:
    {a.speak()} -> std::convertible_to<std::string>;
                        ^
/opt/compiler-explorer/gcc-11.2.0/lib/gcc/x86_64-linux-gnu/11.2.0/../../../../include/c++/11.2.0/concepts:72:30: note: because 'is_convertible_v<int, std::basic_string<char> >' evaluated to false
    concept convertible_to = is_convertible_v<_From, _To>
                             ^
1 error generated.
ASM generation compiler returned: 1
```
[Run on Compiler Explorer](https://godbolt.org/z/e63M8s59P)


## Bridging the Gap in C++
One of the biggest strengths and weaknesses of C++ is the level of control it gives you.

As I mentioned before the real difference between protocols and abstract base classes is that in a statically typed language, if you want to have a container holding multiple implementations of a common interface they must all inherit from a base class that defines the interface.

However, using the C++ template system it's possible to automatically generate these child classes in a hierarchy from different types that implement the correct interface.

I won't go into the details of how this works and the implementation has some rough corners, but this is a functional demo of how we can use some of the more advanced capabilities of C++ to add in features that aren't present in the core language in a manner which is transparent to the end user.
```c++
#include <concepts>
#include <iostream>
#include <memory>
#include <string>
#include <type_traits>
#include <utility>
#include <vector>

template<typename T>
concept Animal = requires(T a) {
    {a.speak()} -> std::convertible_to<std::string>;
};

struct dynAnimalImplBase {
    virtual ~dynAnimalImplBase(){}
    virtual std::unique_ptr<dynAnimalImplBase> clone() = 0;
    // Interface Methods
    virtual std::string speak() = 0;
};

template<Animal T, typename Base = std::remove_cvref_t<T>>
struct dynAnimalImpl : dynAnimalImplBase, Base {
    template<typename... Ts>
    dynAnimalImpl(Ts... args) : Base{std::forward<Ts>(args)...} {}

    std::unique_ptr<dynAnimalImplBase> clone() override {
        return std::make_unique<dynAnimalImpl>(static_cast<Base>(*this));
    }

    // Interface Methods
    std::string speak() override { return Base::speak(); }
};

struct dynAnimal {
    std::unique_ptr<dynAnimalImplBase> ptr;

    template<Animal T>
    dynAnimal(const T& t) : ptr{std::make_unique<dynAnimalImpl<T>>(t)} {}
    template<Animal T>
    dynAnimal(T&& t) : ptr{std::make_unique<dynAnimalImpl<T>>(t)} {}

    dynAnimal(const dynAnimal& other) : ptr{other.ptr->clone()} {}
    dynAnimal(dynAnimal&&) = default;
    dynAnimal& operator=(const dynAnimal& other) {
        ptr = other.ptr->clone();
        return *this;
    }
    dynAnimal& operator=(dynAnimal&& other) {
        ptr = std::exchange(other.ptr, nullptr);
        return *this;
    }

    // Interface methods
    std::string speak() { return ptr->speak();}
};

struct Dog {
    std::string speak() { return "Woof"; }
};

struct Cat {
    std::string speak() { return "Meow"; }
};

void says_what(Animal auto& animal) {
    using std::cout, std::endl;
    cout << animal.speak() << endl;
}

int main(int, char**) {
    using std::vector;

    Dog dog{};
    says_what(dog);

    Cat cat{};
    says_what(cat);

    vector<dynAnimal> animals{Dog{}, Cat{}};
    for (auto&& animal : animals) {
        says_what(animal);
    }
    
    return 0;
}
```
[Run on Compiler Explorer](https://godbolt.org/z/44o4x6ocd)

## Other Languages
It's worth mentioning that in languages which seperate the definition of a class's data from it's methods this behaviour can be achieved with less overhead.
For example look at how [interface types](https://tour.golang.org/methods/9) are handled in golang or the ["dyn" trait syntax](https://doc.rust-lang.org/std/keyword.dyn.html) in Rust.