# The infamous Traceback (most recent call last)

We've all seen it at some point or another. When first starting out, it can be daunting, and that's understandable; there's some output you might have never learned even existed, and whatever course or tutorial you followed might have thought it would be a great idea to pretend all programs are error-free (spoiler alert: they're not)

Troubleshooting is an essential skill to have, which is why we're starting off strong with this section. We'll begin by learning how to read the traceback output. First things first: the hint is in the output; `(most recent call last)`. What it means is the most recent call is at the bottom of the output, and so we'll be reading it from the bottom upwards. Let's have a look at a nonsensical example with the following code:

```py
#!/usr/bin/env python3

# Filename: tberr.py

from lxml import html
import requests
from requests.exceptions import ConnectionError

def do_request():
    try:
        resp = requests.get('http://127.1').text
    except ConnectionError:
        resp = None
    return resp

try:
    tree = html.fromstring(do_request())
except:
    print(type(tree))
```

Assuming there's no HTTP server running on localhost (yes; loopback address can go by localhost, 127.0.0.1, or 127.1), one can safely assume the `requests.get()` method is bound to fail, raising a `ConnectionError` exception. That call was made as part of another function call `do_request()`. As we can see inside the function body, there's _some_ sort of exception handling, but it doesn't actually handle anything correctly. Neither do the statements that follow the function in the script, but we're doing this for ~~dramatic effect~~ illustration purposes

Next, there's a try/except block that _tries_ to assign the supposedly parsed HTML element (returned from the call to `do_request()`, which _should have_ returned valid a HTML string, but ended up returning `None` instead. One can make a very educated guess that `html.fromstring()` expects a string as an argument, so surely providing `None` is bound to raise an exception. Here comes the fun part: the `except` portion of the block _should_ handle such exception, right? Not in this example, I'll tell you what there!

Since `html.fromstring()` will never come to pass (it received `None`, remember?), `tree` will never have been declared in the first place, taking us to the _second_ exception where we try to print the type of `tree`; a nonexistent Name (i.e. variable in this context). Let's see that in action:

```sh
$ chmod +x tberr.py; ./tberr.py
Traceback (most recent call last):
  File "/home/$USER/./test.py", line 16, in <module>
    tree = html.fromstring(do_request())
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/lib/python3/dist-packages/lxml/html/__init__.py", line 849, in fromstring
    is_full_html = _looks_like_full_html_unicode(html)
                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
TypeError: expected string or bytes-like object, got 'NoneType'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/home/$USER/./test.py", line 18, in <module>
    print(type(tree))
               ^^^^
NameError: name 'tree' is not defined. Did you mean: 'True'?
```

One might think that the very bottom of that output is the actual issue, but as you've read, it's only one part of an otherwise dependency of issues. Let's start reading from the bottom upwards:

```py
Traceback (most recent call last):
  File "/home/$USER/./test.py", line 18, in <module>
    print(type(tree))
               ^^^^
NameError: name 'tree' is not defined. Did you mean: 'True'?
```

- `File "/home/$USER/./test.py", line 18`: line 18 in `/home/$USER/test.py` is where the exact issue took place
- `NameError: name 'tree' is not defined. Did you mean: 'True'?`: `NameError` is the exception, stating that the name `tree` is not defined. Python can be helpful sometimes in providing suggestions, but that may not always be as helpful as what meets the eye

Next, we'll move up to the _previous_/_older_ call:

```py
Traceback (most recent call last):
  File "/home/$USER/./test.py", line 16, in <module>
    tree = html.fromstring(do_request())
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/lib/python3/dist-packages/lxml/html/__init__.py", line 849, in fromstring
    is_full_html = _looks_like_full_html_unicode(html)
                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
TypeError: expected string or bytes-like object, got 'NoneType'
```

Here, we have to understand how the calls were invoked. `File "/home/$USER/./test.py", line 16` calls `html.fromstring()` on a `NoneType`. _Internally_, `html.fromstring()` makes another call as follows:

```py
  File "/usr/lib/python3/dist-packages/lxml/html/__init__.py", line 849, in fromstring
    is_full_html = _looks_like_full_html_unicode(html)
                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
TypeError: expected string or bytes-like object, got 'NoneType'
```

As we can see, `html.fromstring()` expected a string (supposedly HTML), but got `NoneType` instead

Let's recap, annotating the code block this time:

```py
#!/usr/bin/env python3

# Filename: tberr.py

# The following 3 lines import the required libraries
from lxml import html
import requests
from requests.exceptions import ConnectionError

# Nothing happens here upon executing the script _until_
# there's a call to do_request()
def do_request():
    try:
        resp = requests.get('http://127.1').text
    except ConnectionError:
        resp = None
    return resp

try:
    # Try to assign an HTML element from the string returned by
    # do_request(). This will fail because do_request() returns 
    # NoneType on ConnectionError
    tree = html.fromstring(do_request()) # this is where execution jumps up to do_request()
except:
    # If the above statement fails, execute the following line(s)
    # This will also fail since tree will have never been assigned
    print(type(tree))
```

So, logically, our execution flow is as follows:

1. `tree = html.fromstring()` => `do_request()`
2. `do_request()` => `requests.get()` => returns `NoneType` as per handling `ConnectionError`, rendering step 1 moot
3. `print(type(tree))` => raises `NameError` since step 1 never took place (i.e. the assignment of an HTML element to the variable `tree`)

While this example makes no sense in terms of logic, or common sense for that matter, it should give you an idea on how to read traceback output

Remember:
- Walk your way up the traceback, going from the most recent call to the oldest
- Pay close attention to file names; know what's relevant in _your_ script, verses what's happening _internally_ inside functions provided by libraries
- Pay close attention to line numbers so you can quickly navigate to the offending lines
- Print-debugging is great and all, but debugging with actual debugging tools/libs is paramount and will help you a great deal
- Don't just skim through traceback output, however long it might be. You need to understand what's happening, what's relevant, and why it happened in the first place. In time, your eyes will be trained to tune out the involved libraries, and you'll be mindful of the data types being passed around
- Last, but not least: _READ THE DOCS_. Familiarize yourself with exceptions so you know what `TypeError`, `ValueError`, `NameError`, and what the entire assortment of exception classes is
