# Python Notes
#### Primer:
This is an aggregation of all the lessons I learned on the way regarding best practices, standards around coding in Python.

I'm mostly aggregating in a single place because I tend to forget what I learned a while ago.

### Lesson Resources

* [Common Gotchas - The Hitchhiker's Guide to Python](https://brexhq.atlassian.net/wiki/spaces/ENGDOC/pages/1602126112/Python+Style+Guide)
* [Effective Python](https://effectivepython.com/) 


### Personal Lessons

#### Abstraction

#### Common Pitfalls/Gotchas
Some of this is from the internal Brex Python Style Guide document.

1. Pickle
I could go on about this, but [others have done it better than I could hope to](https://www.benfrederickson.com/dont-pickle-your-data/).
The main point here is: “using pickle is still a terrible idea that should be avoided whenever possible”.


2. Mutating Arguments

It is relatively common to see functions which mutate their own arguments, such as:

```python
def concat(a, b):
    a += b

a = [1, 2]
b = [4, 5]
concat(a, b)
assert a == [1, 2, 4, 5]
```
This is **not recommended**, since it makes it very unclear what the inputs and outputs of your function are.
For someone who is not familiar with the concat, they might not know that it modifies an input.
The recommended strategy is to **not modify your inputs**. Instead, you should **return** from your function:

```python
def concat(a, b):
    return a + b

a = [1, 2]
b = [4, 5]
assert concat(a, b) == [1, 2, 4, 5]
assert a == [1, 2] # a is not modified
```

3. Modifying internal state



