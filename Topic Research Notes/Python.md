---
aliases: 
publish: 
---
%%
date:: [[2024-09-27]]
parent:: 
%%
# [[Python]]

In Python, an **object** is a core concept that represents an instance of a data structure defined by a class. An object encapsulates both data (attributes) and behaviour (methods) related to that data. Here's a breakdown:
- **Class**: A blueprint that defines the structure and behaviour (attributes and methods) that the objects created from it will have.
- **Object**: A specific instance of a class with actual values. When a class is defined, no memory is allocated until an object is created.
```
# Define a class
class Dog:
    def __init__(self, name, age):
        self.name = name  # Attribute
        self.age = age    # Attribute
    
    def bark(self):
        return f"{self.name} says woof!"

# Create an object
my_dog = Dog("Buddy", 5)

print(my_dog.name)   # Accessing attribute -> Output: Buddy
print(my_dog.bark()) # Calling method -> Output: Buddy says woof!
```
- **Basic Data Types**: Integers, floats, strings, lists, dictionaries, tuples, and even functions and classes themselves are all objects.
```
num = 10        # 'num' is an object of the 'int' class
text = "hello"  # 'text' is an object of the 'str' class
```
- **Functions and Methods**: Functions are objects of the `function` type, and methods are objects too.
```
def greet():
    return "Hello!"

print(type(greet))  # Output: <class 'function'>

```
## Object-Oriented Programming (OOP) 
OOP is a programming paradigm centred around **objects** and **classes**. It allows for organising code in a way that models real-world concepts, making it easier to manage and reuse.
### Key Concepts of OOP

- **Class**: A blueprint for creating objects, defining attributes (data) and methods (behaviour).
- **Object**: An instance of a class with specific values.
- **Encapsulation**: Bundling data (attributes) and methods within a class, restricting direct access to some components.
- **Inheritance**: Creating a new class from an existing one, inheriting its attributes and methods.
- **Polymorphism**: Allowing different classes to be treated as instances of the same class through a common interface.
- **Abstraction**: Hiding complex details and showing only the essential features of an object.
## Object-Oriented Programming (OOP) Overview

### 1. Class: Blueprint for creating objects
```
class Dog:
    def __init__(self, name, age):  # Constructor
        self.name = name            # Attribute
        self.age = age              # Attribute
    
    def bark(self):                 # Method
        return f"{self.name} says woof!"
```
### 2. Object: Instance of a class
```
my_dog = Dog("Buddy", 5)
```
### 3. Encapsulation: Data and methods are bundled in the class
```
print(my_dog.name)    # Accessing attribute
print(my_dog.bark())  # Calling method
```
### 4. Inheritance: Creating a new class from an existing one
```
class Puppy(Dog):
    def play(self):
        return f"{self.name} is playing!"

my_puppy = Puppy("Charlie", 1)
print(my_puppy.bark())   # Inherited method
print(my_puppy.play())   # New method
```
### 5. Polymorphism: Treating different objects through a common interface
```
def make_sound(animal):
    return animal.bark()

print(make_sound(my_dog))     # Works with Dog
print(make_sound(my_puppy))   # Works with Puppy
```
### 6. Abstraction: Hiding complex details (example not shown, concept explained above)

