---
layout: post
title:  "Dictionary tips and tricks"
author: "Sagiv Barhoom"
date:   2020-06-01
categories: Python
background: '/img/posts/python.jpg'
---

# python dictionary tips and tricks
Here ar some ```dict``` methods, tips and trick which I find useful

## The ```dict.clear``` method vs ```dict.new```
```python
d = {"a": "1"}
d2 = d
d = {}             # --> creates new dict into d
print (d2)         # --> outputs: {"a": "1"}
d = {"a": "1"}
d2 = d             # --> d2 is a referance to d
d.clear()          #  so now both d and d2  are cleared
print (d2)         # --> outputs: {}
Aassigning ```d = {}`` creates a new, empty dictionary.
This leaves d2 pointing at the old dictionary. so it contains the items of the original dictionary.
However, using ```d.clear()``` clears the same dictionary that d and d2 both point at.
```

## Convert two lists into a dictionary - using ```zip```
```python
keys = ['a', 'b', 'c']
values = [1, 2, 3]
if len(keys)==len(values):
        my_dict = dict(zip(keys, values))
        print(my_dict) # --> {'a': 1, 'b': 2, 'c': 3}
```
**Be careful!** - you must check that the length of ```key``` and ```values``` are the same, or else:
```python
keys = ['a']
values = [1, 2, 3]
my_dict = dict(zip(keys, values))
print(my_dict) # --> {'a': 1}

keys = ['a', 'b', 'c']
values = [1]
my_dict = dict(zip(keys, values))
print(my_dict) # --> {'a': 1}
```

## Convert a dictionary into tuples of keys, and values using ```zip```
This is less common use - an yet here it is:
```python
new_keys,new_values= zip( *my_dict.items() )
print (type(new_keys))                       # --> <class 'tuple'>
print (new_keys)                             # --> ('a', 'b', 'c')
print (new_values)                           # --> (1, 2, 3)
```

## Convert a List to Dictionary with same values
My preferred method:
```python
keys = ['a', 'b', 'c']
my dict = dict.fromkeys(keys , 1)
```
More pythonic way:
```python
keys = ['a', 'b', 'c']
my_dict = { k : 'same_value' for k in keys }
```


## Assignment (```=```) vs ```dict.copy```
Take a look here:
```python
d = {"a": "1"}
        
d["b"]=2
print (d,d2)           # --> outputs: {'a': '1', 'b': 2} {'a': '1', 'b': 2}

d = {"a": "1"}
d2 = d.copy()
d["b"]=2
print (d,d2)           # --> outputs: {'a': '1', 'b': 2} {'a': '1'}
```
The assignment operation (```=```) in the line:  ```d2 = d``` creates reference from ```d2``` which points to ```d```.
The method ```dict.copy``` creates exact copy of the dictionary ```d```.

## Set\Get value if not exists for a specific key
### Set... use ```setdefault(<key>, <defualt>)```
This is so useful - I and always forget to use it...
Get the value of the 'species' item...
- If the 'species' item does not exist, insert 'species' with the value Iris-setosa'.
- Else keep the current value

```python
print (iris) # --> {
             #      'sepal_length': 1, 
             #      'sepal_width': 5.1, 
             #      'petal_length': 3.5, 
             #      'petal_width': 1.4
             #      }
             
iris.setdefault("species", "Iris-setosa")

print (iris) # --> {
             #      'sepal_length': 1, 
             #      'sepal_width': 5.1, 
             #      'petal_length': 3.5, 
             #      'petal_width': 1.4
             #      'species': 'Iris-setosa'        <-- new key:value pair
             #       }

iris.setdefault("species","Iris-versicolor")

print (iris["species"]) # --> Iris-setosa           <-- the old value "Iris-setosa" did not chaneged
```

### Get... use ```get(<key>, <defualt>)```
```python
d = {'a': 1, 'b': 2, 'c': 3}

print(d.get('b'))      # --> 20
print(d.get('z'))      # --> None
print(d.get('z',777))  # -->777
```
BTW ```dict.get('my_none_exists_key')``` returns ```None```, but ```d['my_none_exists_key']``` will raise an excetion:
```d['my_none_exists_key']```


## Merging dictionaries
Here we try to merge two dictionaries which has different value for the key 'b':

```python
d1 = { 'a': 1, 'b': 2 }
d2 = {         'b': 3, 'c': 4 }

merged = { **d2, **d1 }
print (merged)                   # --> {'a': 1, 'b': 3, 'c': 4}    <-- d['b']=3
merged = { **d2, **d1 }
print (merged)                   # --> {'b': 2, 'c': 4, 'a': 1}    <-- d['b']=4 

merged = { **d1, **d2}
print (sorted(merged.items()))     # -->  [('a', 1), ('b', 3), ('c', 4)] <-- d['b'] =3            
merged = { **d2, **d1 }
print (sorted(merged.items()))     # -->  [('a', 1), ('b', 2), ('c', 4)] <-- d['b'] =2            
```
The creation of the merged dictionary  ```{d1,d2,...dn}``` add d2 after d1 and updates the values of a merged dictionary by the later dictionaries values.
In the first case ```d2['b']``` will overide  ```d1['b']``
