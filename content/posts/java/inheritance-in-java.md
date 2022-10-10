---
title: "Inheritance in Java"
date: 2022-10-10T20:43:51+05:30
draft: false
---

Inheritance is a mechanism through which a class can inherit properties(fields and methods) of another class. This allows to resuse the fields and methods of the parent class.

Inheritance also represents the IS-A relationship which describes the parent-child relationships between the classes.

### Types of Inheritance

Java allows 5 types of Inheritance:

![types-of-inheritance-1](/java/inheritance-in-java/types-of-inheritance-1.png)
![types-of-inheritance-2](/java/inheritance-in-java/types-of-inheritance-2.png)

1. Single - One class inherits another class.
```java
class A {
  public void method() {
    // Do Something
  }
}

class B extends A {
  public void anotherMethod() {
    // Do Something
  }
}
```

2. Multilevel - A chain of inheritance.
```java
class A {
  public void method() {
    // Do Something
  }
}

class B extends A {
  public void anotherMethod() {
    // Do Something
  }
}

class C extends B {
  public void yetAnotherMethod() {
    // Do Something
  }
}
```
3. Hierarchical - Two or more classes inherit a class.
```java
class A {
  public void method() {
    // Do Something
  }
}

class B extends A {
  public void anotherMethod() {
    // Do Something
  }
}

class C extends A {
  public void yetAnotherMethod() {
    // Do Something
  }
}
```
4. Multiple - A class inherits multiple interfaces. Java does not allow inheriting multiple classes as if the two parent classes have same name properties then there will be a conflict.
```java
interface A {
  public void interfaceMethod();
}

interface B {
  public void interfaceMethod();
  public void anotherInterfaceMethod();
}

class C implements A, B {
  public void interfaceMethod() {
    // Implementation of interfaceMethod
  }
  public void anotherInterfaceMethod() {
    // Implementation of anotherInterfaceMethod
  }
}
```
5. Hybrid - Mix of Multiple and Hierarchical inheritance.
```java
interface A {
  public void interfaceMethod();
}

interface B extends A {
  public void anotherInterfaceMethod();
}
interface C extends A {
  public void yetAnotherInterfaceMethod();
}

class D implements B, C {
  public void interfaceMethod() {
    // Implementation of interfaceMethod
  }
  public void anotherInterfaceMethod() {
    // Implementation of anotherInterfaceMethod
  }

  public void yetAnotherInterfaceMethod() {
    // Implementation of yetAnotherInterfaceMethod
  }
}
```
