---
title: "Python call functions written in C"
excerpt_separator: "<!--more-->"
categories:
  - python
tags:
  - python
  - C
  - programming
---

Calling functions written purely with C using python code.

> Python is one of the best languages to write code if you want to develop something quickly.

From what i can see (get it? see --> C), i would only use this in cases where with python you are not able to access some system resource or C gives you the performance boost that you require.

The following is the C code that i will use for this example.

```c
// filename : c_tools.c
// author   : eshan shafeeq
// date     : 2017-10-06

int addAll( int value );
int addAll( int value ){
    int total=0;
    while( value > 0 ){
        total+=value;
        value--;
    }
    return total;
}
```

I compiled the above the code into `c-tools.so` using the following command.

```bash
gcc -shared -fPIC c_tools.c -o c-tools.so
```

The code above just keeps adding `value` to `total` while `value` gets deducted and `value` is still greater than 0. Nothing too complicated, since this is just for demonstration. Here is the python code which calls the `addAll` function.

```python
# filename  : python-call-c-functions.py
# author    : eshan shafeeq
# description : Use python to call function written using c

from ctypes import *

# Loading the shared object file
tools = CDLL('./c-tools.so')

value = 6
total = tools.addAll( value )
print('Sum of the values added up : ' + str(total) )

```

The code above is also pretty straight forward. We can see how it's able to load the shared C object file, and call the function within the object file as well. Doing so while passing in a value which is defined in python. This gives us very interesting possibilities in the future. I am so going to use this method to control GPIO pins on the raspberry pi just because i can!

Below is the output generated from the program.

```console
python python-call-c-functions.py
Sum of the values added up : 21
```
