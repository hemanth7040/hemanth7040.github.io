---
title: "Phase 0: Python Basics for People Who Already Know Bash"
date: 2026-04-19T09:00:00+05:30
draft: false
categories: ["python"]
description: "Learn the fundamental building blocks of Python — variables, data types, strings, operators, control flow, data structures, functions, and error handling — explained simply with clear code examples."
video_url: "https://youtu.be/ygXn5nV5qFc?si=2ai-v1UT8qYrwvev"
series: ["Python Zero to MLOps"]
series_order: 3
tags: ["Beginners", "Concepts", "Syntax", "Functions", "Loops", "Ai"]
---

{{< youtube "ygXn5nV5qFc" >}}

> **Video Reference:** Watch the full course by Dave Ebbelaar on YouTube before or alongside this guide.


# Python Core Concepts for Beginners

This guide covers every major Python concept you need to start building real programs and AI applications. Every section uses simple language and practical examples. No experience required.

---

## What Is Programming?

Programming means giving a computer a set of instructions to follow. The computer does exactly what you say — nothing more, nothing less. Your job as a programmer is to write those instructions clearly.

Python is one of the most popular programming languages in the world — especially for AI. Its code reads almost like plain English, which makes it a great first language.

Here is a simple example of what Python code looks like:

```python
name = "Alice"
age = 25
print(f"Hello, my name is {name} and I am {age} years old.")
```

Output:
```
Hello, my name is Alice and I am 25 years old.
```

You will understand every part of that code by the end of this guide.

---

## Python Syntax and PEP 8

**Syntax** means the rules of how you write Python code — similar to grammar rules in English.

**PEP 8** is Python's official style guide. It tells you how to format your code so it is clean and easy for other people (and your future self) to read.

The most important rules to remember:

- Use **4 spaces** for indentation (not tabs)
- Put spaces around operators: `x = 1 + 2` not `x=1+2`
- Keep lines under 79 characters when possible
- Use lowercase with underscores for variable names: `model_name`, not `modelName`
- Leave one blank line between blocks of code

---

## Understanding Errors

Python will show you an error message when something goes wrong. Errors are normal — even experienced developers see them every day. The key skill is reading the error and understanding what it means.

Here are the most common types:

### SyntaxError — You wrote something Python cannot understand

```python
print("Hello"   # missing closing parenthesis
```

Error:
```
SyntaxError: '(' was never closed
```

Fix: Add the missing `)`.

---

### NameError — You used a variable that does not exist yet

```python
print(nmae)   # typo — 'nmae' instead of 'name'
```

Error:
```
NameError: name 'nmae' is not defined
```

Fix: Check your spelling.

---

### TypeError — You used the wrong type in an operation

```python
age = "25"
total = age + 5   # can't add a string and a number
```

Error:
```
TypeError: can only concatenate str (not "int") to str
```

Fix: Convert the string to a number first: `int(age) + 5`.

---

### How to Debug

When you see an error:
1. Read the last line of the error message — it usually says exactly what went wrong
2. Look at the line number it mentions
3. Check for typos, missing punctuation, or wrong types
4. Fix one thing at a time and run again

---

## Variables

A **variable** is a named container that holds a value. You can think of it like a labeled box — you put something inside and can retrieve it later using the label.

```python
name = "Claude"
age = 3
temperature = 0.7
is_active = True
```

### Rules for Variable Names

- Must start with a letter or underscore: `name`, `_name` ✅
- Cannot start with a number: `3name` ❌
- Cannot have spaces: `model name` ❌ — use `model_name` instead
- Cannot use Python keywords like `for`, `if`, `class`, `print`
- Are case-sensitive: `Name` and `name` are two different variables

### Assigning and Updating Variables

```python
score = 0       # create variable with value 0
score = score + 10    # update it to 10
score += 5      # shortcut for score = score + 5
print(score)    # prints 15
```

---

## Comments

Comments are notes you write in your code for yourself or other people. Python completely ignores them when running your program.

```python
# This is a single-line comment

# Calculate the total cost
price = 10
quantity = 5
total = price * quantity   # multiply price by quantity

"""
This is a multi-line comment.
You can write as many lines as you want.
Useful for longer explanations.
"""
```

Good comments explain **why** you did something, not just **what** the code does.

---

## Data Types

Every value in Python has a **type** — a category that tells Python what kind of data it is and what you can do with it.

You can always check the type of any value using `type()`:

```python
print(type(42))         # <class 'int'>
print(type(3.14))       # <class 'float'>
print(type("hello"))    # <class 'str'>
print(type(True))       # <class 'bool'>
```

---

## Numbers — Integers and Floats

### Integers (int)

Whole numbers — no decimal point.

```python
tokens = 1024
year = 2025
negative = -50
```

### Floats (float)

Numbers with a decimal point.

```python
temperature = 0.7
price = 19.99
pi = 3.14159
```

### Common Number Operations

```python
a = 10
b = 3

print(a + b)    # 13  — addition
print(a - b)    # 7   — subtraction
print(a * b)    # 30  — multiplication
print(a / b)    # 3.333...  — division (always returns float)
print(a // b)   # 3   — floor division (drops the decimal)
print(a % b)    # 1   — remainder (modulo)
print(a ** b)   # 1000  — exponent (10 to the power of 3)
```

### Converting Between Types

```python
x = "42"          # this is a string
y = int(x)        # converts to integer: 42
z = float(x)      # converts to float: 42.0

num = 3.9
print(int(num))   # 3 — drops the decimal, does NOT round
```

---

## Strings

A **string** is any text — letters, numbers, symbols, spaces — wrapped in quotes. You can use single quotes `'` or double quotes `"`.

```python
first_name = "Alice"
last_name = 'Smith'
sentence = "Python is great for AI!"
```

### Accessing Characters

Each character in a string has an **index** (position number). Python starts counting from 0.

```python
word = "Python"
print(word[0])    # P  — first character
print(word[1])    # y
print(word[-1])   # n  — last character (negative index counts from the end)
```

### Slicing Strings

You can extract a portion of a string using `[start:end]`. The end position is not included.

```python
text = "Hello, World!"
print(text[0:5])     # Hello
print(text[7:])      # World!  (from index 7 to the end)
print(text[:5])      # Hello   (from the start to index 5)
```

### String Methods

Python has many built-in functions (called **methods**) that work on strings.

```python
sentence = "  hello world  "

print(sentence.upper())         # "  HELLO WORLD  "
print(sentence.lower())         # "  hello world  "
print(sentence.strip())         # "hello world"  — removes spaces from both ends
print(sentence.strip().title()) # "Hello World"  — capitalizes first letter of each word

text = "Python is great"
print(text.replace("great", "amazing"))   # "Python is amazing"
print(text.split(" "))                    # ['Python', 'is', 'great']
print(text.startswith("Python"))          # True
print(text.endswith("code"))              # False
print(len(text))                          # 15  — number of characters
```

### F-Strings (String Formatting)

The most modern and readable way to put variables inside strings. Put an `f` before the opening quote and use `{}` to insert variables.

```python
name = "Claude"
version = 3
cost = 0.015

print(f"Model: {name}")
print(f"Version: {version}")
print(f"Cost per token: ${cost:.4f}")   # formats to 4 decimal places
print(f"2 + 2 = {2 + 2}")              # you can do math inside {}
```

Output:
```
Model: Claude
Version: 3
Cost per token: $0.0150
2 + 2 = 4
```

---

## Booleans

A **Boolean** has only two possible values: `True` or `False`. (Capital T and F — Python is case-sensitive.)

Booleans are the result of comparisons and are used in decision-making.

```python
is_trained = True
is_deployed = False

print(type(True))    # <class 'bool'>
```

### Comparison Operators

These produce a Boolean result:

```python
x = 10
y = 5

print(x == y)    # False — equal to
print(x != y)    # True  — not equal to
print(x > y)     # True  — greater than
print(x < y)     # False — less than
print(x >= 10)   # True  — greater than or equal to
print(x <= 9)    # False — less than or equal to
```

### Logical Operators

Combine multiple Boolean conditions:

```python
age = 25
has_account = True

# and — both conditions must be True
print(age > 18 and has_account)   # True

# or — at least one condition must be True
print(age > 30 or has_account)    # True

# not — reverses the Boolean
print(not has_account)            # False
```

---

## Operators

### Arithmetic Operators

| Operator | Example | Result |
|----------|---------|--------|
| `+` | `5 + 3` | `8` |
| `-` | `5 - 3` | `2` |
| `*` | `5 * 3` | `15` |
| `/` | `5 / 3` | `1.666...` |
| `//` | `5 // 3` | `1` |
| `%` | `5 % 3` | `2` |
| `**` | `5 ** 3` | `125` |

### Shortcut Assignment Operators

These update a variable in place:

```python
count = 10
count += 5    # same as count = count + 5  → 15
count -= 3    # same as count = count - 3  → 12
count *= 2    # same as count = count * 2  → 24
count /= 4    # same as count = count / 4  → 6.0
```

---

## Control Flow — Making Decisions

Control flow means controlling which parts of your code run based on conditions.

### If / Elif / Else

```python
score = 85

if score >= 90:
    print("Grade: A")
elif score >= 80:
    print("Grade: B")
elif score >= 70:
    print("Grade: C")
else:
    print("Grade: F")
```

Output:
```
Grade: B
```

**How it works:**
- Python checks each condition from top to bottom
- It runs the block under the **first** condition that is `True`
- If no condition is `True`, it runs the `else` block
- The indentation (4 spaces) is required — it tells Python which lines belong to each block

---

## Loops

Loops let you repeat a block of code multiple times without copying it over and over.

### For Loops

A **for loop** repeats code for each item in a sequence.

```python
fruits = ["apple", "banana", "cherry"]

for fruit in fruits:
    print(fruit)
```

Output:
```
apple
banana
cherry
```

### The range() Function

Use `range()` when you want to repeat code a specific number of times:

```python
for i in range(5):
    print(i)
# prints 0, 1, 2, 3, 4

for i in range(1, 6):
    print(i)
# prints 1, 2, 3, 4, 5

for i in range(0, 10, 2):
    print(i)
# prints 0, 2, 4, 6, 8 (step of 2)
```

### While Loops

A **while loop** keeps repeating as long as a condition is `True`.

```python
count = 0

while count < 5:
    print(f"Count is: {count}")
    count += 1    # important! must change the condition, or the loop never stops
```

> **Warning:** If the condition never becomes `False`, the loop runs forever. Always make sure something inside the loop will eventually make the condition `False`.

### Break and Continue

```python
# break — exit the loop immediately
for i in range(10):
    if i == 5:
        break     # stops the loop when i reaches 5
    print(i)      # prints 0, 1, 2, 3, 4

# continue — skip the rest of this iteration and go to the next
for i in range(10):
    if i % 2 == 0:
        continue  # skip even numbers
    print(i)      # prints 1, 3, 5, 7, 9
```

---

## Data Structures

Data structures are ways of organizing and storing multiple pieces of information together.

### Lists

A **list** is an ordered collection of items. Lists can hold anything — numbers, strings, other lists. Items can be changed (lists are **mutable**).

```python
models = ["GPT-4", "Claude", "Gemini", "Llama"]

# Access items by index (starts at 0)
print(models[0])     # GPT-4
print(models[-1])    # Llama (last item)

# Slice the list
print(models[1:3])   # ['Claude', 'Gemini']

# Add items
models.append("Mistral")         # adds to the end
models.insert(1, "Claude-3")     # inserts at position 1

# Remove items
models.remove("Gemini")          # removes by value
del models[0]                    # removes by index

# Check membership
print("Claude" in models)        # True or False

# Useful list operations
print(len(models))               # number of items
models.sort()                    # sorts alphabetically (in place)
models.reverse()                 # reverses the order
print(sorted(models))            # returns a sorted copy without changing original
```

### Dictionaries

A **dictionary** stores data as **key-value pairs** — like a real dictionary where you look up a word (key) to find its definition (value).

```python
model = {
    "name": "Claude",
    "provider": "Anthropic",
    "max_tokens": 200000,
    "temperature": 0.7,
    "supports_vision": True
}

# Access values by key
print(model["name"])             # Claude
print(model.get("provider"))     # Anthropic (safer — won't crash if key missing)

# Add or update a key
model["version"] = 3
model["temperature"] = 0.5

# Remove a key
del model["supports_vision"]

# Loop through a dictionary
for key, value in model.items():
    print(f"{key}: {value}")

# Check if a key exists
print("name" in model)           # True
```

### Tuples

A **tuple** is like a list, but it **cannot be changed** after creation (it is **immutable**). Use tuples for data that should not change — like coordinates or fixed configurations.

```python
coordinates = (40.7128, -74.0060)
rgb_color = (255, 128, 0)

# Access items the same way as lists
print(coordinates[0])    # 40.7128

# Unpack a tuple into variables
latitude, longitude = coordinates
print(latitude)          # 40.7128

# Tuples can be used as dictionary keys (lists cannot)
locations = {(0, 0): "origin", (1, 2): "point A"}
```

### Sets

A **set** is an unordered collection of **unique** items. Duplicates are automatically removed. Sets are great for checking membership quickly or finding unique values.

```python
languages = {"Python", "JavaScript", "Python", "Go", "Python"}
print(languages)    # {'Python', 'JavaScript', 'Go'} — duplicates removed

# Add and remove items
languages.add("Rust")
languages.remove("Go")

# Check membership (very fast)
print("Python" in languages)    # True

# Set operations
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

print(a | b)    # {1, 2, 3, 4, 5, 6}  — union (all items from both)
print(a & b)    # {3, 4}              — intersection (items in both)
print(a - b)    # {1, 2}              — difference (in a but not b)
```

### Which Data Structure Should You Use?

| Situation | Use |
|-----------|-----|
| Ordered list of items you may change | List |
| Key-value pairs / labeled data | Dictionary |
| Fixed, unchanging collection | Tuple |
| Unique items, fast lookup | Set |

---

## Functions

A **function** is a named block of code that does a specific task. You write it once and can use it (call it) as many times as you want.

Functions solve two big problems:
1. **Avoid repetition** — write the code once, use it everywhere
2. **Organization** — break a big program into small, manageable pieces

### Defining and Calling a Function

```python
# Define the function
def greet():
    print("Hello! Welcome to the Python for AI course.")

# Call (run) the function
greet()     # Hello! Welcome to the Python for AI course.
greet()     # You can call it as many times as you want
```

### Parameters and Arguments

**Parameters** are the variables listed in the function definition. **Arguments** are the actual values you pass in when calling the function.

```python
def greet_user(name):       # 'name' is a parameter
    print(f"Hello, {name}!")

greet_user("Alice")         # 'Alice' is an argument
greet_user("Bob")
```

### Multiple Parameters

```python
def calculate_cost(tokens, price_per_token):
    total = tokens * price_per_token
    print(f"Cost: ${total:.4f}")

calculate_cost(1000, 0.00002)   # Cost: $0.0200
```

### Default Parameter Values

You can give parameters a **default value** that is used when the caller does not provide one.

```python
def generate(prompt, model="claude-3", temperature=0.7):
    print(f"Model: {model}")
    print(f"Temperature: {temperature}")
    print(f"Prompt: {prompt}")

generate("Tell me a joke")                          # uses defaults
generate("Write code", model="gpt-4")              # overrides model
generate("Summarize", model="llama", temperature=0.1)  # overrides both
```

### Keyword Arguments

When calling a function, you can name the arguments — this makes the code clearer and lets you pass them in any order.

```python
def create_model(name, provider, max_tokens):
    print(f"{name} by {provider}, max {max_tokens} tokens")

# Using keyword arguments (order does not matter)
create_model(provider="Anthropic", max_tokens=200000, name="Claude")
```

### Return Values

Functions can **return** a value back to the caller. This is how you get a result out of a function.

```python
def add(a, b):
    return a + b

result = add(3, 5)    # result = 8
print(result)

# You can use the return value directly
print(add(10, 20))    # 30
```

A function without a `return` statement automatically returns `None`.

```python
def multiply(a, b):
    result = a * b
    return result

# You can return multiple values as a tuple
def min_max(numbers):
    return min(numbers), max(numbers)

low, high = min_max([3, 1, 8, 2, 9])
print(low, high)    # 1 9
```

---

## Variable Scope

**Scope** determines where a variable can be seen and used in your code.

### Local Variables

A variable created **inside** a function only exists inside that function. It disappears when the function finishes.

```python
def my_function():
    local_var = "I only exist inside this function"
    print(local_var)

my_function()       # works fine
print(local_var)    # ERROR: NameError — local_var doesn't exist here
```

### Global Variables

A variable created **outside** all functions is global — it can be read anywhere in the file.

```python
model_name = "Claude"   # global variable

def show_model():
    print(model_name)   # can read the global variable

show_model()    # prints "Claude"
```

### Modifying a Global Variable Inside a Function

If you need to change a global variable from inside a function, use the `global` keyword:

```python
call_count = 0

def make_call():
    global call_count
    call_count += 1

make_call()
make_call()
print(call_count)   # 2
```

> **Best practice:** Avoid using global variables as much as possible. It is cleaner to pass values as parameters and get results back with return.

---

## Modules and Imports

A **module** is a Python file that contains code — functions, variables, classes — that you can use in other files.

Python comes with many built-in modules. You can also install third-party modules with pip.

### Importing a Module

```python
import math

print(math.sqrt(25))      # 5.0
print(math.pi)            # 3.14159...
print(math.ceil(3.2))     # 4
print(math.floor(3.8))    # 3
```

### Importing Specific Items

```python
from math import sqrt, pi

print(sqrt(49))    # 7.0  — no need to write math.sqrt
print(pi)          # 3.14159...
```

### Importing with an Alias

```python
import numpy as np         # very common convention
import pandas as pd        # very common convention

# Now use np and pd instead of the full name
```

### The `random` Module

```python
import random

print(random.randint(1, 10))        # random integer between 1 and 10
print(random.choice(["a", "b", "c"]))   # random item from a list
print(random.random())              # random float between 0.0 and 1.0

my_list = [1, 2, 3, 4, 5]
random.shuffle(my_list)             # shuffles the list in place
print(my_list)
```

### The `datetime` Module

```python
from datetime import datetime, date

now = datetime.now()
print(now)                          # 2025-01-01 14:30:00.123456
print(now.year)                     # 2025
print(now.strftime("%B %d, %Y"))    # January 01, 2025

today = date.today()
print(today)                        # 2025-01-01
```

---

## Working with Files

Reading and writing files is a very common task — especially for loading data and saving results.

### Reading a Text File

```python
# Always use 'with' — it automatically closes the file when done
with open("notes.txt", "r") as file:
    content = file.read()    # read the whole file as one string
    print(content)

# Read line by line
with open("notes.txt", "r") as file:
    for line in file:
        print(line.strip())  # .strip() removes the newline character at the end
```

### Writing to a File

```python
with open("output.txt", "w") as file:
    file.write("Hello, file!\n")
    file.write("This is a second line.\n")
```

> `"w"` mode creates the file if it doesn't exist, and **overwrites** it if it does. Use `"a"` (append) to add to the end of an existing file.

### Working with JSON Files

JSON is the most common format for sharing data with APIs. Python's built-in `json` module handles it easily.

```python
import json

# Python dictionary
config = {
    "model": "claude-3",
    "temperature": 0.7,
    "max_tokens": 2048
}

# Save to a JSON file
with open("config.json", "w") as file:
    json.dump(config, file, indent=2)

# Read from a JSON file
with open("config.json", "r") as file:
    loaded_config = json.load(file)

print(loaded_config["model"])    # claude-3
```

---

## Error Handling

When something goes wrong in your program, Python raises an **exception** (an error). If you do not handle it, your program crashes.

**Try/except** lets you catch errors and respond to them gracefully instead of crashing.

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Error: Cannot divide by zero!")
```

### Handling Multiple Error Types

```python
def safe_convert(value):
    try:
        return int(value)
    except ValueError:
        print(f"'{value}' cannot be converted to an integer")
        return None
    except TypeError:
        print("Input must be a string or number")
        return None

print(safe_convert("42"))      # 42
print(safe_convert("hello"))   # error message, returns None
print(safe_convert(None))      # error message, returns None
```

### The `finally` Block

Code in `finally` always runs — whether an error happened or not. Use it for cleanup tasks like closing files.

```python
try:
    file = open("data.txt", "r")
    content = file.read()
except FileNotFoundError:
    print("File not found!")
finally:
    print("This always runs.")
```

### Common Exception Types

| Exception | When it happens |
|-----------|----------------|
| `ValueError` | Wrong value type, e.g. `int("hello")` |
| `TypeError` | Wrong data type, e.g. `"text" + 5` |
| `ZeroDivisionError` | Division by zero |
| `FileNotFoundError` | File does not exist |
| `KeyError` | Dictionary key does not exist |
| `IndexError` | List index is out of range |
| `AttributeError` | Calling a method that does not exist |

---

## Classes and Object-Oriented Programming

A **class** is a blueprint for creating objects. An **object** is an instance of a class — a specific thing built from that blueprint.

Think of a class as a cookie cutter and objects as the actual cookies.

### Creating a Class

```python
class AIModel:
    # __init__ is called automatically when you create an object
    # 'self' refers to the specific object being created
    def __init__(self, name, provider, max_tokens):
        self.name = name              # instance attribute
        self.provider = provider
        self.max_tokens = max_tokens
        self.call_count = 0          # starts at 0 for every new object

    def generate(self, prompt):
        self.call_count += 1
        print(f"{self.name}: Generating response for '{prompt}'")

    def get_info(self):
        return f"{self.name} by {self.provider} | Calls made: {self.call_count}"
```

### Creating Objects (Instances)

```python
# Create two separate objects from the same class
claude = AIModel("Claude", "Anthropic", 200000)
gpt = AIModel("GPT-4", "OpenAI", 128000)

# Each object is independent
claude.generate("Tell me about Python")
claude.generate("Explain neural networks")
gpt.generate("Write a haiku")

print(claude.get_info())   # Claude by Anthropic | Calls made: 2
print(gpt.get_info())      # GPT-4 by OpenAI | Calls made: 1
```

### Class Attributes vs Instance Attributes

```python
class AIModel:
    # Class attribute — shared by ALL instances
    total_models_created = 0

    def __init__(self, name):
        self.name = name                         # instance attribute
        AIModel.total_models_created += 1        # update class attribute

model1 = AIModel("Claude")
model2 = AIModel("GPT-4")

print(AIModel.total_models_created)    # 2
print(model1.name)                     # Claude
print(model2.name)                     # GPT-4
```

### Inheritance

A **child class** can inherit everything from a **parent class** and add or change things.

```python
class BaseModel:
    def __init__(self, name):
        self.name = name

    def respond(self, prompt):
        return f"Response to: {prompt}"


class ChatModel(BaseModel):
    def respond(self, prompt):
        base = super().respond(prompt)   # call the parent's method
        return f"[Chat] {base}"


class CodeModel(BaseModel):
    def respond(self, prompt):
        base = super().respond(prompt)
        return f"```\n{base}\n```"


chat = ChatModel("Claude-Chat")
code = CodeModel("Claude-Code")

print(chat.respond("Hello!"))
# [Chat] Response to: Hello!

print(code.respond("Write a loop"))
# ```
# Response to: Write a loop
# ```
```

### When to Use a Class vs a Function

Use a **function** when:
- You need to do one specific thing and return a result
- There is no need to store state between calls

Use a **class** when:
- You have data and multiple related operations that work together
- You need to keep track of state between operations
- You want to create multiple independent objects of the same type

---

## Quick Reference — Python Built-in Functions

These functions come with Python — no import needed:

| Function | What it does | Example |
|----------|-------------|---------|
| `print()` | Displays output | `print("hello")` |
| `input()` | Gets text from user | `name = input("Enter name: ")` |
| `len()` | Length of a string or list | `len("hello")` → 5 |
| `type()` | Type of a value | `type(42)` → int |
| `int()` | Converts to integer | `int("42")` → 42 |
| `float()` | Converts to float | `float("3.14")` → 3.14 |
| `str()` | Converts to string | `str(42)` → "42" |
| `range()` | Generates a sequence | `range(5)` → 0,1,2,3,4 |
| `list()` | Converts to list | `list(range(3))` → [0,1,2] |
| `sorted()` | Returns sorted copy | `sorted([3,1,2])` → [1,2,3] |
| `sum()` | Adds all numbers | `sum([1,2,3])` → 6 |
| `min()` | Smallest value | `min([3,1,2])` → 1 |
| `max()` | Largest value | `max([3,1,2])` → 3 |
| `abs()` | Absolute value | `abs(-5)` → 5 |
| `round()` | Rounds a number | `round(3.7)` → 4 |
| `enumerate()` | Index + value pairs | `for i, v in enumerate(list)` |
| `zip()` | Combines two lists | `for a, b in zip(list1, list2)` |

---

## Summary — Concepts at a Glance

| Concept | Key Idea |
|---------|----------|
| Variables | Named boxes that hold values |
| Data types | int, float, str, bool |
| Strings | Text data with many built-in methods |
| F-strings | Modern way to embed variables in text |
| Booleans | True/False values from comparisons |
| Operators | Math, comparison, and logical operations |
| if/elif/else | Run different code based on conditions |
| for loop | Repeat code for each item in a sequence |
| while loop | Repeat code while a condition is True |
| List | Ordered, changeable collection |
| Dictionary | Key-value pairs for labeled data |
| Tuple | Fixed, unchangeable collection |
| Set | Unique items, great for fast lookup |
| Function | Named block of code you can reuse |
| return | Sends a value back from a function |
| Scope | Where a variable can be seen |
| Module | A file of ready-made code to import |
| Try/except | Catch and handle errors gracefully |
| Class | Blueprint for creating objects |
| Inheritance | Child class reuses parent class code |

---

## What Comes Next

You now know the core building blocks of Python. The next step is combining these concepts to build real programs:

- **Working with APIs** — send requests to AI models like Claude or GPT
- **Data handling** — read CSV files, process data with pandas
- **Project structure** — organize code into multiple files and modules
- **Git and GitHub** — save and share your code professionally
- **AI applications** — build chatbots, summarizers, and AI-powered tools

> **Practice tip:** The best way to learn programming is to write code every day, even just 20 minutes. Take one concept from this guide and write a small program using it — that is how it really sticks.