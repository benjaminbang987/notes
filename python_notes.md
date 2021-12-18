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

##### Pickle
I could go on about this, but [others have done it better than I could hope to](https://www.benfrederickson.com/dont-pickle-your-data/).
The main point here is: “using pickle is still a terrible idea that should be avoided whenever possible”.


##### Mutating Arguments

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

##### Modifying internal state

In a pattern very similar to the one above, there are a few cases of folks modifying
the internal state of a class sequentially--this looks something like this:

```python
transaction_processor = TransactionProcessor(config=config)
transaction_processor.get_data()
transaction_processor.proces_data()
transaction_processor.train_mode(model_config)
reults = transaction_processor.results
```

The reason this is bad/confusing is that this completely hides what is going on from the caller,
and totally modifies the underlying class instance. Have a look at the get_data function: transaction_proccessor
looks totally different before and after that is called!

If I, someone who did not write this code, got a TransactionProcessor instance without knowing anything about
its internal state, I would be hard pressed to know what to do with it.


Instead, it is recommended to be explicit about inputs and outputs. The code above would be more explicit if we re-wrote it as:


```python
transaction_processor = TransactionProcessor(config=config)
raw_data = transaction_processor.get_data()
processed_data = transaction_processor.proces_data(raw_data)
return transaction_processor.train_mode(processed_data=processed_data, model_config=model_config)
```

##### Returning many values
In general, try not to return many values from a function. This is pretty bug prone--consider the example:
```python
def split_name(name):
    s = name.split(" ")
    return (s[1], s[0])

first_name, last_name = split_name("Andre Menck")
assert last_name == "Andre"

```
As you can see, the way you deconstruct the return values can lead to bugs in the code--
this happens because the return values are not quite explicit.

Instead, it is recommended to use a dataclass to return an object composed of multiple elements:

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class FullName:
    first: str
    last: str

def split_name(name):
    s = name.split(" ")
    return FullName(first=s[0], last=s[1])

full_name = split_name("Andre Menck")
assert full_name.last == "Menck"

```

##### import *
In general, we do not recommend using from <module> import *, since this obscures what you are actually using
from the module. Instead, it is best to explicitly declare each import.

##### Formatting Strings
We do not recommend using “old-style” string formatting anywhere in your python code. Specifically, instead of doing this:

```python
message = "Balance %s below minimum %s." % (balance, min_balance)
```
You should be using python3’s new “f-strings” (except when logging, see Logging section below):
```python
message = f"Balance {balance} below minimum {min_balance}."
```
This puts your arguments into your strings, making them much more readable!
If you’re curious, you can find out more about string formatting in python [here](https://realpython.com/python-f-strings/).

##### Inheriting from (object)

In Python 3, there is no need to inherit from objectclass MyClass(object):
. Instead of:
```python
class MyClass(object):
```
Do:
```python
class Myclass
```

##### Logging
Use 
```python
logger.info("Some message", details=details)
```
instead of
```python
logger.info(f"Some message: {details}")
```
This ensure that the log is formatted correctly in whichever environment you are using.
Using kwargs also allows logger to obtain JSON format and pass the data as a dict rather than just a formatted string.

##### Comparing to None
Whenever doing `None`-checking in python, always compare using `is` instead of `==`. In otherwords,
do `x is None` instead of `x == None`.

##### Comparing to Boolean Values
Whenever you are checking a boolean value, you should prefer to use `if x:` instead of `if x == True:`.

##### Unnecessary Else When Returning
Whenever you have an `if`/`else` statement with a `return` at the end, you can remove the unnecessary `else`. For example:
```python
if x:
    return 2
else:
    return 3
```
can be reduced to
```python
if x:
    return 2
return 3
```

##### Argument Naming
There have been a few cases in our codebase when folks name their function arguments the same as some python built-in name.
Specifically, the name input tends to be used a lot as a function argument.

This is not recommended, since `input` is a [built-in python name](https://www.w3schools.com/python/ref_func_input.asp).
If using pycharm (recommended) your IDE should warn you against such function argument names.


##### Rule of Two
If your code repeats at least two times (copy/paste), you should create a helper function for it.
