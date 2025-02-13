# Getting help()

Now that we've covered reading traceback output, we'll move on to another important skill; getting help

You might have learned the bare minimum, just enough to get you through basic scripting/programming tasks, and that's great, but what do you do when you're looking for help?

Ask {insert forum here}? Spam the same question on however many Discord servers you might be on? (seriously, don't do that; it's royally annoying, and wastes everyone's time)

The answer is simple: refer to the API docs. See, the very developers who make this language possible are the only ones who are capable of providing the most accurate technical documentation thereof. That is the first and foremost source everyone learning whatever {insert language/library/framework/API here} should check

With the rise of ~~LLMs~~ AI, things are unfortunately taking a sad turn, in my humble opinion. Everyone touts AI as some silver bullet to all life's problems and burning questions. It is not, however, any of that; it's something that fakes it just enough to sound like it's an SME. We can argue about how "you don't understand how it works" et al., but really, let's not kid ourselves. When there's a tiny little disclaimer that says any variation of the sentence "information may not be accurate", you know this is your first clue to not trust a word that's being generated out of this tin can's `/dev/null` (yes, that is exactly what you're thinking about right now)

I digress at this point, and we're not here to debate AI; I might write up an entry on everything I think is wrong with how we're dealing with it, but that's for another time. Let's get back to getting help quickly and effectively

## Ways to get help

There are at least 4 different ways to get help:
- Straight from the comfort of your terminal:
    - Using `pydoc <keyword>`
    - Inside your interpreter shell: `>>> help(<keyword>)`
    - [Inside Vim](../vim.md#misc) using `Shift+K` to bring up `pydoc <keyword>` (i.e. Python-specific documentation)
- Online: [https://docs.python.org/3/](https://docs.python.org/3/)

As a last resort, you can always look up keywords like "python multiprocessing docs" (or a variation thereof) using your favorite search engine. In that search, you might stumble upon some reply on some forum which cites the official documentation, or provide a decent technical writeup on how 'keyword' works, or explain its concept. Key is to know how to identify good answers and resources from bad ones, and you can only do that when you've read through enough online resources to tell the difference

Finally, let's say you wanted lookup documentation for a function/method that belongs to an _imported library_, like `requests`, for example, but you couldn't be bothered to check the Requests library API online. You can opt to do that interactively inside the interpreter shell as follows:

```py
Python 3.12.9 (main, Feb  5 2025, 01:31:18) [GCC 14.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import requests
>>> help(requests.Session) # help(requests.Session()) also works, but no need for extra chars
```

At this point, your default $PAGER (ideally LESS) is forked with the documentation pertinent to whichever object you passed to `help()` as follows: (output is truncated for brevity)

```py
Help on Session in module requests.sessions object:

class Session(SessionRedirectMixin)
 |  A Requests session.
 |
 |  Provides cookie persistence, connection-pooling, and configuration.
 |
 |  Basic Usage::
 |
 |    >>> import requests
 |    >>> s = requests.Session()
 |    >>> s.get('https://httpbin.org/get')
 |    <Response [200]>
 |
 |  Or as a context manager::
 |
 |    >>> with requests.Session() as s:
 |    ...     s.get('https://httpbin.org/get')
 |    <Response [200]>
 |
 |  Method resolution order:
 |      Session
 |      SessionRedirectMixin
 |      builtins.object
 |
 |  Methods defined here:
 |
 |  __enter__(self)
 |
 |  __exit__(self, *args)
 |
 |  __getstate__(self)
 |      Helper for pickle.
 |
 |  __init__(self)
 |      Initialize self.  See help(type(self)) for accurate signature.
 | ...
```

## Closing thoughs

Don't be afraid to ask for help, or feel discouraged when you first don't understand something. No one starts off an expert, and no one knows _all_ the ins and outs of a language save for those who invented it to begin with, and possibly a very small subset of people. The more you reference official API documentation, the easier and faster it gets for you, and you'll be positive that you're getting the most recent documentation on the topic at hand from the most verifiable source that can provide it
