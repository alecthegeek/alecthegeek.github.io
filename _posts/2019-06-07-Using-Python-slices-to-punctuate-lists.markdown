---
layout: post
title: "Using Python Slices to punctuate lists of words"
date: 2019-06-20 17:02
comments: true
categories: python
---

In the English language items in a list are separated (generally) by
commas, except the last item. For example


* Dog
* Dog and Cat
* Dog, Cat and Hamster

Sometimes we write

* Dog, Cat or Hamster

The "and" (or the "or" etc) are called conjunctions.

When python is being used to create human readable text from lists it would be very
nice to add the conjunctions correctly.

With Python [index](https://docs.python.org/3/reference/expressions.html?highlight=slice#subscriptions)
and [slicing](https://docs.python.org/3/reference/expressions.html?highlight=slice#slicings)
operations on lists this turns our to be easy.

```python
def listWithConjunctions(l, conjunction = "and"):
    """return a list of text items with the correct conjunction
    between the last item and all previous items in the list

    >>> listWithConjunctions("")
    ''
    >>> listWithConjunctions("Dog")
    'Dog'
    >>> listWithConjunctions(["Dog"])
    'Dog'
    >>> listWithConjunctions(["Dog", "Cat"])
    'Dog and Cat'
    >>> listWithConjunctions(["Dog", "Cat", "Hamster"])
    'Dog, Cat and Hamster'
    >>> listWithConjunctions(["Dog", "Cat", "Hamster"], "or")
    'Dog, Cat or Hamster'
    >>> listWithConjunctions(["Dog", "Cat", "Hamster"], "nor")
    'Dog, Cat nor Hamster'
    """

    if type(l) is not list:
        return f"{l}"  # Just try and return a string
    if len(l) < 2:
        return ''.join(l) # Convert list of 0 or 1 items to string

    return f"{', '.join(l[:-1])} {conjunction} {l[-1]}"
    
```
