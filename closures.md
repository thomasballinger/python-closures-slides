


* Intro
  * Why do we need closure?
* What are closures?
  * "close" meaning "bind" meaning "figure out what value a variable has"
* Python functions are closures!
  * proof: same code, different functionality
  * how closures work:
    * hold onto reference to environments
    * figure out scope at definition time
      * need the global keyword to specify sometimes
* Python *really* has closures in 2.1
  * We couldn't 
* Python *really* *really* has closures in 3.0
  * We can't specify variables names as being from an outer scope yet!
* What do we use closures for?
* Why don't we use rebinding closures so much?
  * Mutable
  * we don't hide data, we have modules, and we have bound methods

* Finding our closure:
  * definition
  * Python functions are closures
  * how to tell if a functions uses closed over values















TO CUT:
* less code examples!
* remove bytecode references!
* any discussion of builtins
* implementation details about variable lookup (.__closure__)








---


^To remember the name,

---

#closure

---

#"close"

---

#bind, or figure out what value a variable has

---

##With closures the environment can used to close variables

^In constrast, in a pure function you can only use the arguments to the function.

---






^#[fit]Getting our closure
 closures are code coupled with environment
 Python functions are definitely closures
 it's ok that we don't use rebinding closures all that much







----------------cut content----------------

~~~python
>>> height.__closure__
(<cell at 0x10fcd62e8: dict object at 0x10fcb0548>,)
>>> height.__closure__[0].cell_contents
{'Burj Khalifa': 828, 'Shanghai Tower': 632, 'Abraj Al-Bait': 601}
~~~



---

~~~python
def make_formatters():
    template = '<space style="color:{}">{}</span>'
    return [lambda x: template.format(color, x)
            for color in ['red', 'green', 'blue']]

red, green, blue = make_formatters.value()
~~~

^Once again, declaring functions in a loop is a problem.

---

~~~python

def make_formatters():
    template = '<space style="color:{}">{}</span>'
    colors = ['red', 'green', 'blue']
    for color in colors:

        def colored(s):
            return template.format(color, s)

        formatters.append(colored)
    return formatters

red, green, blue = make_formatters.value()

~~~

^Once again, this doesn't work: the functions don't close over the value their
variables referred at function definition time, they refer to the bindings
themselves!

---












---

~~~python
import queue import Queue
from threading import Thread

def (url):
    attempt = 1
    result = Queue.queue()

    def on_retry():
        nonlocal attempt
        attempt += 1

    def on_success(data):
        result.put(data)

    Thread(target=lambda: retrying_download(on_retry, on_success).start()
    return (q.get(), attempts)
~~~

^This comes up in asyncio a lot, anything with a callback interface
The above isn't idiomatic for callbacks, but a common use of nonlocal
is at the interfaces between a callback interface and a synchronous one.

---

#[fit]But it isn't used very much!

^I've been thinking about why...

---

#[fit]But it isn't used very much!

* only in Python 3





---

#[fit]But it isn't used very much!

* only in Python 3
* simple mutable object shim
* ?

^But even so...

https://asmeurer.github.io/python3-presentation/slides.html

---

Aaron Meurer presents

#[fit]10 awesome features of Python 
#[fit]that you can't use because you refuse to upgrade to Python 3

https://asmeurer.github.io/python3-presentation/slides.html

---

* Matrix Multiplication with `@`
* `a, *rest, b = range(10)`
* keyword-only arguments
* chained exceptions
* more error subclasses
* more iterators
* no more 'a' < 1
* yield from

---

* asyncio
* new stdlib
* function annotations
* unicode and bytes

^nonlocal doesn't make the top twelve! This leads me to think - maybe this
pattern just isn't as important in Python!

---

#[fit]But it isn't used very much!

* only Python 3
* simple mutable object shim
* less important in Python (than in JavaScript)

^My point of comparison is JavaScript, where this pattern is more popular.

---

#[fit]But it isn't used very much!

* only Python 3
* simple mutable object shim
* less important in Python
    * we have modules

^We've had modules for a long time

---

#[fit]But it isn't used very much!

* only Python 3
* simple mutable object shim






---

#[fit]But it isn't used very much!

* only Python 3
* simple mutable object shim
* less important in Python
    * we have modules
    * we don't hide data
    * we have a nice object system

---








