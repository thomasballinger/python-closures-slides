# [fit] Finding closure with closures

## Thomas Ballinger
## @ballingt

^Hello!

---

^
* "what are closures" interview question
* giving words to ideas you're already familar with
* answer "does Python even have closures?"
* why don't we see nonlocal all that much

---

## what closures are
## Python functions have always been closures
## how closures work in Python
## Python 2.1 functions really are closures
## Python 3 functions really really are closures
## How we harness closures in Python (or not)

^Then we'll look at how it's used and I'll speculate a bit about why it isn't used.

---

# Closure:

# code + environment

^The notion that code of some kind - we'll say functions here - are more than
their source code.

---

^Python functions are closures because the text of the function alone is not
sufficient for Python to run the function.
The environment where the function was defined is also used to determine its
behavior.

---

^I have two utility modules for formatting strings.
One is for html, the other is for outputting text to the terminal.
(you know, the way text in the terminal is sometimes different colors)

^I'm going to import these two functions, then look at their source code
with the inspect module, a Python standard library module.

---

~~~
>>> from htmlformat import bold as htmlbold
>>> from terminalformat import bold as termbold
~~~

---

~~~


>>> import inspect







~~~

---

~~~



>>> print(inspect.getsource(htmlbold))
def bold(text):
    return '{}{}{}'.format(BOLDBEFORE, text, BOLDAFTER)




~~~

---

~~~







>>> print(inspect.getsource(termbold))
def bold(text):
    return '{}{}{}'.format(BOLDBEFORE, text, BOLDAFTER)
~~~

---

~~~



>>> print(inspect.getsource(htmlbold))
def bold(text):
    return '{}{}{}'.format(BOLDBEFORE, text, BOLDAFTER)

>>> print(inspect.getsource(termbold))
def bold(text):
    return '{}{}{}'.format(BOLDBEFORE, text, BOLDAFTER)
~~~

^But when I use they, they're going to have different behavior.
There must be something beyond the source code that determines the behavior
of these functions, and I assert that's the respective environments in which
the functions were defined.

---

~~~
>>> htmlbold('eggplant')
'<b>eggplant</b>'


~~~

---

~~~
>>> htmlbold('eggplant')
'<b>eggplant</b>'
>>> termbold('eggplant')
'\x1b[1meggplant\x1b[0m'
~~~

^]

---

^Hold onto that question of how what those environments are for a moment,
we've got our first quiz:

---


    >>> from htmlformat import bold as htmlbold
    >>> BEFOREBOLD = 'Written in sharpie: '
    >>> htmlbold('eggplant')

^What will this print?

---

    >>> from htmlformat import bold as htmlbold
    >>> BEFOREBOLD = 'Written in sharpie: '
    >>> htmlbold('eggplant')
    '<b>eggplant</b>'

^Why is this?
That's right these are different global variables!

---

    #htmlformat.py                   #terminalformat.py

    BEFOREBOLD = '<b>'               BEFOREBOLD = '\x1b[1m'
    AFTERBOLD = '</b>'               AFTERBOLD = '\x1b[0m'

    def bold(text):                  def bold(text):
        return '{}{}{}'.format(          return '{}{}{}'.format(
          BOLDBEFORE,                      BOLDBEFORE,
          text,                            text,
          BOLDAFTER)                       BOLDAFTER)

^Different global variables, and it seems like they each use their own global variables!
"global" is not a great name

---

#global variables aren't "global"

^They aren't universal across your whole program,
they're really module level variables.

---

^Since we're stuck with the word "global," we should
think of each module as being a separate globe...


---

![](https://upload.wikimedia.org/wikipedia/commons/2/2d/Venus_Earth_Comparison.png)

     
     
    #htmlformat.py                      #terminalformat.py

    BEFOREBOLD = '<b>'                  BEFOREBOLD = '\x1b[1m'
    AFTERBOLD = '</b>'                  AFTERBOLD = '\x1b[0m'

    def bold(text):                     def bold(text):
        return '{}{}{}'.format(             return '{}{}{}'.format(
          BOLDBEFORE,                         BOLDBEFORE,
          text,                               text,
          BOLDAFTER)                          BOLDAFTER)

^Modules as being separate globes

---

![](./peridotplanet.png)

^And our function objects have a link to their home world,
they know where they came from.

---

```python
>>> from htmlformat import bold
>>> bold.__module__
'htmlformat'




```

^On the function object we find a reference to both the name of the module
where it was defined, and more importantly to the dictionary of the global
variables in the module.

---

```python
>>> from htmlformat import bold
>>> bold.__module__
'htmlformat'
>>> bold.__globals__.keys()
dict_keys(['BEFOREBOLD', 'AFTERBOLD', 'bold',
'__name__', '__builtins__', '__cached__', '__spec__',
'__package__', '__doc__', '__loader__', '__file__'])
```

^On the function object we find a reference to a dictionary of the global variables

---

```python
>>> from htmlformat import bold
>>> bold.__module__
'htmlformat'
>>> bold.__globals__.keys()
dict_keys(['BEFOREBOLD', 'AFTERBOLD', 'bold',
'__name__', '__builtins__', '__cached__', '__spec__',
'__package__', '__doc__', '__loader__', '__file__'])
>>> import htmlformat
>>> vars(htmlformat) is bold.__globals__
True

```

^The other important thing about this is that
this is a live link back to that home planet!
It's the very same object as the namespace of the module!


---

^Because these are the same namespace, what if we were to change a binding in that namespace?

---

    BEFOREBOLD = '<span class="bold">
    AFTERBOLD = '</span>'

^What happens when I add this code to the bottom on htmlformat?

---

![](https://upload.wikimedia.org/wikipedia/commons/2/2d/Venus_Earth_Comparison.png)

    #htmlformat.py                      

    BEFOREBOLD = '<b>'                  
    AFTERBOLD = '</b>'                  

    def bold(text):                     
        return '{}{}{}'.format(                                    
          BOLDBEFORE,                   
          text,                         
          BOLDAFTER)                    

    BEFOREBOLD = '<span class="bold">
    AFTERBOLD = '</span>'

^Modules as being separate globes

---

    >>> from htmlformat import bold
    >>> bold('eggplant')

---

    >>> from htmlformat import bold
    >>> bold('eggplant')
    '<span class="bold">eggplant</span>

^We're going to use the new values because it's about the values
of the variables when we call the function, not when we define the function.

---

^We can even modify these variables from outside the module!

---

    >>> from htmlformat import bold
    >>> import htmlformat
    >>> htmlformat.BEFOREBOLD = 'READ THIS--> '
    >>> bold('eggplant')

---

    >>> from htmlformat import bold
    >>> import htmlformat
    >>> htmlformat.BEFOREBOLD = 'READ THIS--> '
    >>> bold('eggplant')
    'READTHIS--> eggplant</span>

^Once again, it's about the value of the variable when the function is called.

---

^Sometimes it's hard not to expect templating behavior of using the value of a variable at the time the function was defined instead of at the time when the function is called.

---

~~~python
formatters = {}
colors = ['red', 'green', 'blue']
for color in colors:
    def colored(s):
        return ('<span style="color:' +
                color + '">' + s + '</span>')
    formatters[color] = colored

formatters['green']('hello')
~~~
^What will this do?

---

~~~python
formatters = {}
colors = ['red', 'green', 'blue']
for color in colors:
    def colored(s):
        return ('<span style="color:' +
                color + '">' + s + '</span>')
    formatters[color] = colored

print(color)
formatters['green']('hello')
~~~
^Does this help?

---

<span style="color:blue">hello</span>

---

#[fit]Python functions: totally closures!

^ By this broad definition of being a combination of code and environment.
One important detail and one big idea:
* Python functions hold references to their module evironments
* These are live links: current values of variables

^Next let's look at how Python figures out how to close the
variables we use in a function

---

# How closures work in Python

^At function definition time, Python analyzes our code
to decide which variables are going to be looked up in
the home module, and which ones don't need to be.
For as long as Python has been available on the internet,
there have been at least two kinds of variables:

---

* local variables: `func.__code__.co_varnames`
* "phone home" global variables: `func.__code__.co_names`

^When Python builds a function object, it categorizes names.
Let's be the interpreter and look for patterns in our decisions
about which is which.

---

#Be the interpreter

~~~python
>>> def movie_titleize(phrase):
...     capitalized = phrase.title()
...     return capitalized + ": The Untold Story"
~~~

^Is phrase a local variables, or module-level "global" variable?

---

#Be the interpreter

~~~python
>>> def movie_titleize(phrase):
...     capitalized = phrase.title()
...     return capitalized + ": The Untold Story"
~~~
#[fit]phrase

---

#Be the interpreter

~~~python
>>> def movie_titleize(phrase):
...     capitalized = phrase.title()
...     return capitalized + ": The Untold Story"
~~~
#[fit]capitalized

---

#Be the interpreter

~~~python
>>> def movie_titleize(phrase):
...     capitalized = phrase.title()
...     return capitalized + ": The Untold Story"
...
>>> movie_titleize.__code__.co_varnames
('phrase', 'capitalized')
>>> movie_titleize.__code__.co_names
()

~~~

---

#Be the interpreter

~~~python
>>> def shortened(phrase):
...     options = [phrase.title(), DEFAULT_TITLE]
...     options.sort(key=catchiness)
...     return options[1]
~~~

---

#Be the interpreter

~~~python
>>> def shortened(phrase):
...     options = [phrase.title(), DEFAULT_TITLE]
...     options.sort(key=catchiness)
...     return options[1]
~~~
#[fit]phrase

---

#Be the interpreter

~~~python
>>> def shortened(phrase):
...     options = [phrase.title(), DEFAULT_TITLE]
...     options.sort(key=catchiness)
...     return options[1]
~~~
#[fit]options

---

#Be the interpreter

~~~python
>>> def shortened(phrase):
...     options = [phrase.title(), DEFAULT_TITLE]
...     options.sort(key=catchiness)
...     return options[1]
~~~
#[fit]DEFAULT_TITLE

---

#Be the interpreter

~~~python
>>> def shortened(phrase):
...     options = [phrase.title(), DEFAULT_TITLE]
...     options.sort(key=catchiness)
...     return options[1]
~~~
#[fit]catchiness

---

#Be the interpreter

~~~python
>>> def shortened(phrase):
...     options = [phrase.title(), DEFAULT_TITLE]
...     options.sort(key=catchiness)
...     return options[1]
...
>>> shortened.__code__.co_varnames
('phrase', 'options')
>>> shortened.__code__.co_names
('DEFAULT_TITLE', 'catchiness')
~~~

^If its important that that function is defined in that file - i.e. you can't
copy and paste it into your file - then it's closing over the environment.

---

~~~python
>>> def shortened(phrase):
...     options = [phrase.title(), DEFAULT_TITLE]
...     options.sort(key=catchiness)
...     return options[1]
~~~

^Something tricky: one of the things Python uses to determine if it's a local
variables is whether assignment occurs. But what if you wanted to assign to a global variable?

---

~~~
>>> HIGH_SCORE = 1000
>>> def new_high_score(score):
...     print('congrats on the high score!')
...     print('old high score:', HIGH_SCORE)
...     HIGH_SCORE = score
... 
>>> new_high_score(1042)
~~~
^What error will we get here?

---

~~~
>>> HIGH_SCORE = 1000
>>> def new_high_score(score):
...     print('congrats on the high score!')
...     print('old high score:', HIGH_SCORE)
...     HIGH_SCORE = score
... 
>>> new_high_score(1042)
Traceback (most recent call last):
  File "<input>", line 1, in <module>
  File "<input>", line 2, in new_high_score
UnboundLocalError: local variable 'HIGH_SCORE'
referenced before assignment
~~~
^Why is that? Because HIGH_SCORE is a local variable!

---

~~~
>>> HIGH_SCORE = 1000
>>> def new_high_score(score):
...     print('congrats on the high score!')
...     print('old high score:', HIGH_SCORE)
...     HIGH_SCORE = score
...
>>> new_high_score.__code__.co_varnames
('score', 'HIGH_SCORE')
>>> new_high_score.__code__.co_names
('print',)
>>>
~~~

---

~~~
>>> HIGH_SCORE = 1000
>>> def new_high_score(score):
...     global HIGH_SCORE
...     print('congrats on the high score!')
...     print('old high score:', HIGH_SCORE)
...     HIGH_SCORE = score
... 
>>> new_high_score(1042)
congrats on the high score!
old high score: 1000
~~~

^So the global keyword lets us bypass Python's heuristic and specify the scope
of a variable

---

^How are we doing so far?

---

#In review, the
#[fit]Scope|Value
#of a variable is determined at
#[fit]Definition|Execution
#time

---

~~~
 
~~~
~~~
 
~~~

![](./certificate.jpg)
#[fit]Python 2.0 Scope Expert

#[fit]MM (the year 2000)
~~~
 
~~~
^BUT big changes were coming in Python 2.1

---

^Functions in Python have always closed over their module-level environment.
But before Python 2.1 in 2000, functions didn't close over scopes of outer
functions!

---

~~~python
def tallest_building():
    buildings = {'Burj Khalifa': 828,
                 'Shanghai Tower': 632,
                 'Abraj Al-Bait': 601}

    def height(name):
        return buildings[name]

    return max(buildings.keys(), key=height)
~~~

^let's be the interpreter here

---

~~~python
def tallest_building():
    buildings = {'Burj Khalifa': 828,
                 'Shanghai Tower': 632,
                 'Abraj Al-Bait': 601}

    def height(name):
        return buildings[name]

    return max(buildings.keys(), key=height)
~~~
#name

^if you're being the interpreter, is name a local or a global variable?

---

~~~python
def tallest_building():
    buildings = {'Burj Khalifa': 828,
                 'Shanghai Tower': 632,
                 'Abraj Al-Bait': 601}

    def height(name):
        return buildings[name]

    return max(buildings.keys(), key=height)
~~~
#buildings

^if you're being the interpreter, how do categorize the "buildings" variable?

---
^Sometimes when people talk about closures, they aren't including module-level
scope, or rather they might say that module-level "global" variables are a special case.
In Python their implementation is a special case; they're treated differently.

---

* local variables: `func.__code__.co_varnames`
* "phone home" global variables: `func.__code__.co_names`
* "phone home" outer non-global scopes: `func.__code__.co_freevars`

^A new kind of variable

---

~~~python
>>> height.__code__.co_varnames
('name',)
>>> height.__code__.co_names
()
>>> height.__code__.co_freevars
('buildings',)
~~~

^This was optional in Python 2.1, but

---

#[fit]Python 2.2 functions: definitely closures now!

^Also in the year 2001

---

^At thing point function objects got an attribute called "__closure__"!

---

~~~
 
~~~
~~~
 
~~~

![](./certificate.jpg)
#[fit]Python 2.7 Scope Expert

#[fit]MMVIII (the year 2008)
^One more change big change was coming in Python 3

---

^There's still one more thing we can't do, a way that our Python closures are more limited than those of some other languages.
Remember the problem with global variables, where if we want to assign to a global variable in a function we need to say "global"?

---

^That leads to the phrase "read-only" closures.
We can access those variables, and if they're mutable objects we can send them
messages, we can mutate them - but we can't rebind them.

---

~~~python
>>> def get_number_guesser(answer):
...     last_guess = None
...     def guess(n):
...         if n == last_guess:
...             print('already guessed that!')
...         last_guess = n
...         return n == answer
... 
>>> guess = get_number_guesser(12)
>>> guess(9)

~~~
^But this isn't going to work. Let's be the interpreter again to see why.

---

~~~python
>>> def get_number_guesser(answer):
...     last_guess = None
...     def guess(n):
...         if n == last_guess:
...             print('already guessed that!')
...         last_guess = n
...         return n == answer
~~~
#[fit]n

^If we're being the Python interpreter here, are these local variables, global
variables, or from an outer scope?

---

~~~python
>>> def get_number_guesser(answer):
...     last_guess = None
...     def guess(n):
...         if n == last_guess:
...             print('already guessed that!')
...         last_guess = n
...         return n == answer
~~~
#[fit]answer

---

~~~python
>>> def get_number_guesser(answer):
...     last_guess = None
...     def guess(n):
...         if n == last_guess:
...             print('already guessed that!')
...         last_guess = n
...         return n == answer
~~~
#[fit]last_guess

^It's going to think it's a local variable because we use assignment here.
So what error will we get when we run that inner function?

---

~~~python
>>> def get_number_guesser(answer):
...     last_guess = None
...     def guess(n):
...         if n == last_guess:
...             print('already guessed that!')
...         last_guess = n
...         return n == answer
... 
>>> guess = get_number_guesser(12)
>>> guess(9)
Traceback (most recent call last):
  File "<input>", line 1, in <module>
  File "<input>", line 4, in guess
UnboundLocalError: local variable 'last_guess' referenced before assignment
~~~

^One last puzzle piece: nonlocal!

---

~~~python
>>> def get_number_guesser(answer):
...     last_guess = None
...     def guess(n):
...         if n == last_guess:
...             print('already guessed that!')
...         last_guess = n  # modifies variable in outer scope
...         return n == answer
... 
~~~

^Add a hint to the Python interpreter when we define the function:
we mean the outer variable, not a new one.

---

~~~python
>>> def get_number_guesser(answer):
...     last_guess = None
...     def guess(n):
...         nonlocal last_guess
...         if n == last_guess:
...             print('already guessed that!')
...         last_guess = n  # modifies variable in outer scope
...         return n == answer
... 
>>> guess = get_number_guesser(12)
>>> guess(9)
False
>>> guess(9)
already guessed that!
False
~~~

---

#[fit]Python functions are definitely closures now!

^OK? Feeling alright with that?

---

~~~
 
~~~
~~~
 
~~~

![](./certificate.jpg)
#[fit]Python 3.6 Scope Expert

#[fit]MMVIII (the year 2016)

^Now you're totally up to date with scope in Python.

^What's left is how closures are used in Python.

---

#[fit]How closures are used in Python

^one answer is anytime you use a function, that's a closure

---

#[fit]Global variables
#[fit]`(`any function with`.__code__.co_names``)`

^Any time those phone-home variables are used, those are closures!
If you function calls other functions in the same module, that's using the
fact that the function is a closure!

---

^or maybe we aren't counting that, and it only counts when

---

#[fit]Variables from outer scopes besides globals
#[fit]`(`any function with`.__code__.co_freevars``)`

^Sometimes functions that use those module-level "global" variables aren't
considered closures because we're so used to that behavior, so when
do we use those `co_freevars` from outer functions?

---

* inner functions
* lambdas for key, cmp functions
* inner functions used as callbacks
  * threading.Thread(target=)
  * shutil.rmtree(..., onerror=)
  * signal.signal(..., handler)
  * atexit
  * ...

^Someimes these are "pure" functions, but often they use state from arround
Whenever you write a function that does something interesting.

---

* decorators

~~~
@retry(3)
def get_current_time():
    return requests.get('http://time.is')
~~~

^Decorators usually define a new function that references the old one.

---

#[fit]`(`any function with`.__code__.co_freevars``)`

---

^What about the definition of closure that says you have to be able to rebind
the variables in outer scopes that aren't the global scope?

---

#[fit]"read/write" closures
#[fit]"rebinding" closures
#[fit]nonlocal keyword

^These ones aren't used as much.

^One reason why not is that many people still write code
that works on both Python 3 and Python 2.

---

~~~python
def decorating_function(user_function):
    ...
    nonlocal_root = [root]  # make updateable non-locally

    def wrapper():
        nonlocal_root[0] = oldroot[NEXT]
        ...
~~~
heavily abridged [Django code](https://github.com/django/django/blob/master/django/utils/lru_cache.py#L80) from utils/lru_cache.py

^Real-world examples often use the *pattern* of nonlocal, but do it with
a workaround: stick the variable in a mutable object!


---

^We have modules, we have a nice object system (method binding)...

---

~~~python
def tallest_building():
    buildings = {'Burj Khalifa': 828,
                 'Shanghai Tower': 632,
                 'Abraj Al-Bait': 601}

    return max(buildings.keys(), key=buildings.get)
~~~

^Method binding, and we don't hide data...

---

^The next code sample is the most complicated one we've seen yet.

---

~~~python
>>> class Person(object): pass
>>> def create_person(name):
...     age = 10
...     p = Person()
...     def birthday():
...         nonlocal age
...         age = age + 1
...     p.birthday = birthday
...     p.greet = lambda: print("Hi, I'm", name, "and I'm", age, "years old")
...     return p
... 
>>> me = create_person('Tom')
>>> me.birthday()
>>> me.age
Traceback (most recent call last):
  File "<input>", line 1, in <module>
AttributeError: 'Problem' object has no attribute 'age'
~~~

^(explain this)

^Now this is totally possible in JavaScript and Python. But culturally we
don't do it in Python.

---

#[fit]Getting our closure

^And I think it's fine that we don't use rebinding closures all that much.

---

#[fit]use `nonlocal` when appropriate

^So why use nonlocal? Because it more precisely expresses what we mean!
No need to go looking for uses of it, but they do pop up!

---

#[fit]Getting our closure

^Closures are code + environment

^Functions in Python are closures because they rely on the environment where they were defined to execute

^A general definition includes module-level global variables,
more common definitions only include variables from outer scopes
that aren't the global scope.

^Sometimes we say only functions that use this capability are closures.

---

#[fit]Getting our closure

^Sometimes we mean rebinding closures, all of which Python has!

---

#[fit]Getting our closure

^When people that use other languages talk about the tremendous power of
closures, we can join right in!

^We use closures all the time in Python, just look for .co_freevars on your function objects if you're wondering if something is taking advantage of being a closure.

---

#[fit]Closure closures closed closures closure clojure closures.

^So hopefully I've helped you find closure with closures, I know writing this talk helped me get to that point.

^This doesn't make sense but I've used the word a lot so found it funny.

^Thank you!

---

#Thank you
Thomas Ballinger
@ballingt
ballingt.com

thanks to Ned Batchelder for some examples from "Loop like a Native" talk at PyCon NA 2013

Image credit:

`https://en.wikipedia.org/wiki/Colonization_of_Venus#/media/File:Venus_Earth_Comparison.png`

---

#Want to find out more?
* names - http://nedbatchelder.com/text/names.html
* builtins: last resort of failed global variable lookups
* `__code__.cell_vars`: I had to carefuly choose example to avoid these
* bytecode: what does "compiling" a function really mean?
* descriptors and method binding: the dark secret that turns functions into methods
* scopes of various comprehensions and generator expressions
