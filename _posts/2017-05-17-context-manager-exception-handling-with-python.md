---
title: "Handling exceptions with in the context manager"
excerpt_separator: "<!--more-->"
categories:
  - python
tags:
  - python
  - exceptions
  - context managers
  - programming
---

When dealing with context managers and exceptions, you can handle exceptions from within the context manager class. This will aid for a better control over the problems you face in context manager class. It will also make sure you can close all the operations gracefully and free the locked resources by the context manager class.

demo code
```python
# filename:     context-manager-exception-handling.py
# description:  demostration of context managers with exception handling
# author:       eshan shafeeq

# context manager demo class
class highspeed_file_writter:
    def __init__(self):
        print('Initialised the high speed file writter!')

    def __enter__(self):
        print('In the with block')

    def __exit__(self, exc_type, exc_value, traceback):
        if exc_type == None:
            print('Exiting the with block gracefully')
        else:
            print('Exiting the with block due to an error')

# normal execution of the code
with highspeed_file_writter() as hw:
    print('writing something to a file real fast!')
print('done with writing')

# demonstration of how it handles when an exception is thrown
with highspeed_file_writter() as hw:
    print('lets do something illegal')
    a = 10/0
print('done with the with block')

#done
```

The code above pretends to be a highspeed file writer. It demonstrates 2 scenarios, one is when the code runs normal, the other one is when some exception is raised. When the exception is caught, the `__exit__` method is able to execute the appropriate piece of code in case an exception was raised during the scope of the current context manager class. Below is the output from executing the code above.

```shell
$ python context-manager-exception-handling.py
Initialised the high speed file writter!
In the with block
writing something to a file real fast!
Exiting the with block gracefully
done with writing
Initialised the high speed file writter!
In the with block
lets do something illegal
Exiting the with block due to an error
Traceback (most recent call last):
  File "context-manager-exception-handling.py", line 27, in <module>
    a = 10/0
ZeroDivisionError: integer division or modulo by zero
```

As you can see, this is how we can handle exceptions that occur during the scope of a context manager class. This should help us write more robust pythonic code in the future.
