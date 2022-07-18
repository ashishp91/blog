---
title: "Data Types in Python"
date: 2022-07-16T22:21:36+05:30
draft: false
---

All programming languages have data types. In this post, we're going to talk about 6 data types of Python.

- Mutable Data Types - Data types whose value *can* be changed.
> 1. List
> 2. Dictionary
> 3. Set
- Immutable Data Types - Data types whose value *cannot* be changed.
> 4. Tuple
> 5. String
> 6. Numeric
> 7. Boolean

## List

List is an ordered sequence of items. It's equivalent to an array in Python.

```python
# Defining a List
a = [1, 2, "Apple", 4.50, "Banana", 5, 6]

print("a[2] = ", a[2])
# a[2] =  Apple

print("a[0:3] = ", a[0:3])
# a[0:3] =  [1, 2, 'Apple']

# Removing element at an index
a.pop(1)
# 2

# Adding an element
a.append(7)
# [1, 2, 'Apple', 4.5, 'Banana', 5, 7]
```

## Dictionary

Dictionary is an unordered collection of key-value pairs. It’s equivalent to an map in Python.

```python
# Defining a Dictionary
d = { 128: 'value', 'key': 256 }

# Deleting a key
del d[128]
# { 'key': 256 }

# Assigning a key
d[128] = 'new_value'
# {'key': 256, 128: 'new_value'}
```

## Set

Set is an unordered collection of unique items.

```python
# Defining a Set
s = { 5, 2, "France", 1, "Germany" }

# Adding a value
s.add(10)
# {1, 2, 'France', 5, 10, 'Germany'}

# Removing a value
s.remove(5)
# {1, 2, 'France', 10, 'Germany'}

# Sets are not ordered
print(s[1])
# TypeError: 'set' object is not subscriptable
```

## Tuple

Tuple is an ordered sequence of items same as a List. The only difference being Tuple is Immutable.

```python
# Defining a Tuple
a = ( 1, 2, "Apple", 4.50, "Banana", 5, 6 )

print("a[2] = ", a[2])
# a[2] =  Apple

print("a[0:3] = ", a[0:3])
# a[0:3] =  [1, 2, 'Apple']

# Tuples are Immutable
a[0] = 0
# TypeError: 'tuple' object does not support item assignment
```

## String

String is a sequence of Unicode characters.

```python
# Defining a string
s = "Hello world"

# Defining a multi line string
s = '''A multiline
string'''

s = 'Hello world!'

print("s[4] = ", s[4])
# s[4] =  o

print("s[6:11] = ", s[6:11])
# s[6:11] =  world

# Strings are immutable
s[5] ='d'
# TypeError: 'str' object does not support item assignment
```

## Numeric
Numeric represents any data with numeric value. They can be further categorized into:

- Integer - Positive or Negative whole numbers (without fraction or decimal)
- Float - Real numbers with floating point representation.
- Complex Numbers - Complex number represented as (real part) + (imaginary part)j.

```python
num = 5
print(num, "is of type", type(num))
# 5 is of type <class 'int'>

num = 2.0
print(num, "is of type", type(num))
# 2.0 is of type <class 'float'>

num = 1+2j
print(num, "is complex number?", isinstance(1+2j,complex))
# (1+2j) is complex number? True
```

## Boolean

Boolean represents one of the two values i.e. True or False.

```python
a = True
print(type(a))
# <class 'bool'>

b = False
print(type(b))
# <class 'bool'>
```
