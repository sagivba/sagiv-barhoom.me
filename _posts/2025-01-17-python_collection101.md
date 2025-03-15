---
layout: post
title:  " python collections 101"
author: "Sagiv Barhoom"
date:   2025-01-17
categories: Python
background: '/img/posts/python.jpg'
---


# python collections 101
The collections module in Python provides specialized container datatypes, such as:
- Counter
- deque
- defaultdict
- namedtuple
- OrderedDict
Those datatypes extend the functionality of built-in data structures.
## Counter

### Description
`Counter` is a dictionary subclass designed for counting hashable objects. It maps elements to their counts.

### When to Use
Use `Counter` instead of a dictionary when you need:
- To count occurrences of items in an iterable.
- To perform arithmetic and set operations on counts.

### Examples

#### Basic Usage
```python
from collections import Counter

c = Counter("hello world")
print(c)  # Output: Counter({'l': 3, 'o': 2, 'h': 1, 'e': 1, ' ': 1, 'w': 1, 'r': 1, 'd': 1})
```

#### Arithmetic Operations
```python
c1 = Counter("hello")
c2 = Counter("world")
print(c1 + c2)  # Combined counts
print(c1 - c2)  # Subtracted counts
```

## namedtuple
`namedtuple` is a factory function in the `collections` module that creates tuple subclasses with named fields.
It is an immutable data structure that allows access to elements using field names instead of positional indices.

### When to Use
Use `namedtuple` instead of regular tuples when you need:
- To represent structured data with named fields for better readability.
- Immutable objects where field names improve code clarity.
- Lightweight alternatives to full-fledged classes.

### Examples

#### Basic Usage
```python
from collections import namedtuple

Point = namedtuple('Point', ['x', 'y'])
p = Point(10, 20)
print(p.x, p.y)  # Output: 10 20
```

#### Advanced Features
```python
p = p._replace(x=15)  # Replace a value
print(p._asdict())    # Convert to dictionary: {'x': 15, 'y': 20}
```

## deque

### Description
`deque` (double-ended queue) is a list-like container with fast appends and pops from both ends. It is implemented as a doubly-linked list.

### When to Use
Use `deque` instead of a list when you need:
- Fast append and pop operations from both ends.
- To implement a queue, stack, or circular buffer.
- Efficient rotations or operations requiring manipulation at both ends.

### Examples

#### Basic Usage
```python
from collections import deque

d = deque([1, 2, 3])
d.append(4)       # Append to the right
d.appendleft(0)   # Append to the left
print(d)          # Output: deque([0, 1, 2, 3, 4])

d.pop()           # Remove from the right
d.popleft()       # Remove from the left
print(d)          # Output: deque([1, 2, 3])
```

#### Advanced Features
```python
d = deque([1, 2, 3, 4])
d.rotate(1)  # Rotate to the right
print(d)     # Output: deque([4, 1, 2, 3])
```

## defaultdict

### Description
`defaultdict` is a dictionary subclass that provides a default value for non-existent keys.

### When to Use
Use `defaultdict` instead of a dictionary when you need:
- To avoid key errors by providing default values.
- Simplify code where keys are dynamically added.

### Examples

#### Basic Usage
```python
from collections import defaultdict

d = defaultdict(int)  # Default value of 0
d['a'] += 1
print(d)  # Output: defaultdict(<class 'int'>, {'a': 1})
```

#### Using Lists as Default Values
```python
d = defaultdict(list)
d['a'].append(1)
print(d)  # Output: defaultdict(<class 'list'>, {'a': [1]})
```
