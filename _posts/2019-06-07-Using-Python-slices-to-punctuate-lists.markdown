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

When [Python](https://www.python.org/) (which has some excellent text processing features)
is being used to create human readable text from lists it would be excellent
to add the conjunctions correctly.

Python [index](https://docs.python.org/3/reference/expressions.html?highlight=slice#subscriptions)
and [slicing](https://docs.python.org/3/reference/expressions.html?highlight=slice#slicings)
operations on lists makes this easy.

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

The important piece of code is `return f"{', '.join(l[:-1])} {conjunction} {l[-1]}"`. How does it work?

Remember that we have checked previously and there are at least two items in the list.

1. `l[:-1]` is equivelant to taking a slice with `l[0:-1]`. The slice starts at the 1st element (index 0) and goes the end the list __minus one__. The index `-1` is shorthand for the last element and slices exclude the last item.
To prove this try `[0,1,2,3][:-1]` at the Python prompt. You should see the result `[0, 1, 2]`.
2. We then convert that list in a string with wth comma separators using the `str.join()` method. Again you can test this at the Python prompt with `', '.join(['0','1','2','3'][:-1])`
