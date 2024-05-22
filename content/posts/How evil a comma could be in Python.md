---
publish: true
title: How evil a comma could be in Python
aliases: []
date: 2024-05-22T21:58:34Z
lastmod: 2024-05-22T23:10:49Z
tags: []
category: posts
summary: 
---

How bad a comma could be in Python?

Though I have been using Python for a couple of years, mainly as a utility to help develop software, I am still surprised by Python from time to time. Sometimes is the Sytanx sugar making me realise how elegant a Python code could be,  sometimes is the caveat annoying me.

Here is the story,

in the beginning, I had a  code like the one below (for the demo purpose, I simplify the code here): 

```python
class DataInt:
	def __init__(self, name, value, view, ranges, unit, resolution) -> None:
		...

data_dict = dict(
a = DataInt('a', 0, None,[0,5], 'm/s', 0.01),
b = DataInt('b',0,None,[0,100], 'm', 0.01),
)
```
I needed to rector an exisiting `data_dict` to a mutable object for storing data. I decided to use the `dataclass` and enjoy the beauty and simplicity of declaring data objects straightforwardly.

so I replace the `data_dict` with
```python
@dataclass
class Data:
    a: DataInt = field(default_factory=lambda: DataInt(
        'a', 0, None, [0, 5], 'm/s', 0.01)),
    b: DataInt = field(default_factory=lambda: DataInt(
        'a', 0, None, [0, 100], 'm/s', 0.01)),
    l: list[int] = field(default_factory=lambda: [])

    def as_dict(self):
        return asdict(self)


if __name__ == '__main__':
    test = Data()
    print(test.a)
    print(type(test.a))

    print(test.l)
    print(test.as_dict())
 
```

`default_factory` intends to initialise the mutable attributes of `dataclass` as per the documentation.

However, after running the test `main` above, I got

```
(Field(name=None,type=None,default=<dataclasses._MISSING_TYPE object at 0x100857710>,default_factory=<function Data.<lambda> at 0x10092ef20>,init=True,repr=True,hash=None,compare=True,metadata=mappingproxy({}),kw_only=<dataclasses._MISSING_TYPE object at 0x100857710>,_field_type=None),)

<class 'tuple'>

[]

Traceback (most recent call last):
  File "tmp.py", line 33, in <module>
    print(test.as_dict())

...
TypeError: cannot pickle 'mappingproxy' object
```

This didn't look right as I was expecting `test.a` should be a value of `DataInt` however `test.l` works as expected. I thought it was the way I used the `dataclass` and didn't find the bug of my code. I spent several hours to find out what's wrong with it. I googled it, I searched in the StackOverflow, I read the documentation of `dataclass` PEP, and I even use a different python version to see if it is a bug of Python itself (the answer always is not). 

After adding, deleting and re-ordering the attributes, I suddenly found it is the **comma** `,`  at the end of each line of attributes declarations! 

Removing the **comma**s at the end of the line,
```python
@dataclass
class Data:
    a: DataInt = field(default_factory=lambda: DataInt(
        'a', 0, None, [0, 5], 'm/s', 0.01))
    b: DataInt = field(default_factory=lambda: DataInt(
        'a', 0, None, [0, 100], 'm/s', 0.01))
    l: list[int] = field(default_factory=lambda: [])

    def as_dict(self):
        return asdict(self)
```
now it works!

```
<__main__.DataInt object at 0x10281e390>
<class '__main__.DataInt'>
[]
{'a': <__main__.DataInt object at 0x10281e690>, 'b': <__main__.DataInt object at 0x10281e990>, 'l': []}
```

This is how evil a comma could be! I have commas left here because I was refactoring from existing `dict` with  existing commas.

Recall the comma in python, the extra  comma  makes expression result in a `tuple`.

```python
>>> a = 1
>>> b = a,
>>> type(b)
<class 'tuple'>
>>> b
(1,)
```

In summary,  be careful when have a tailing comma in Python. It could be a bug taking you several hours to debug.