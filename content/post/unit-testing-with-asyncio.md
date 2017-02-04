+++
date = "2017-01-26T00:28:12-06:00"
title = "unit testing with asyncio"
draft = false
tags = ["python", "testing", "asyncio"]
categories = ["post"]
readtime = "11"

+++

### The Problem

```python
# One method on this class is an asyncio coroutine.
class PersonClass(object):

    def __init__(self, name=None, cursor=None):
        """Constructor for PersonClass."""
        self.name = name
        self.cursor = cursor
        self.insert_sql = 'INSERT INTO person VALUES ?;'

    async def create(self):  # ....HOW TO TEST THIS METHOD??
        """Persist the person to the database."""
        return await self.cursor.execute(self.insert_sql, (self.name,))
```

Testing and even stubbing asyncio code can be confusing. And frustrating. I spent an embarrassing four hours yesterday trying to unit test my async/await code. After much googling (and screaming) I discovered two methods that are both neat and clean.

### My Environment

Inspiration for writing this article comes from my project [veggiecron-server](https://github.com/jacobbridges/veggiecron-server), which is written with async/await syntax from Python 3.5 and 3.6. For testing I am using [pytest](http://doc.pytest.org/en/latest/) as the test runner and [MagicMock](https://docs.python.org/3/library/unittest.mock.html#magicmock-and-magic-method-support) for stubbing. I also highly recommend the pytest plugin [pytest-asyncio](https://github.com/pytest-dev/pytest-asyncio) which reduces the event loop boilerplate code needed for each asynchronous test.

### With `unittest.TestClass`

```python
import asyncio

from unittest import TestCase
from unittest.mock import MagicMock

from src.person import PersonClass

class TestPersonClass(TestCase):

    def test_send_query_on_create(self):
        """Should send insert query to database on create()"""

        event_loop = asyncio.new_event_loop()
        asyncio.set_event_loop(event_loop)

        async def run_test():

            # Stub the database cursor
            database_cursor = MagicMock()

            # Stub the execute function with mocked results from the database
            execute_stub = MagicMock(return_value='future result!')
            # Wrap the stub in a coroutine (so it can be awaited)
            execute_coro = asyncio.coroutine(execute_stub)
            database_cursor.execute = execute_coro

            # Instantiate new person obj
            person = PersonClass(name='Johnny', cursor=database_cursor)

            # Call person.create() to trigger the database call
            person_create_response = await person.create()

            # Assert the response from person.create() is our predefined future
            assert person_create_response == 'future result!'
            # Assert the database cursor was called once
            execute_stub.assert_called_once_with(person.insert_sql, ('Johnny',))

        # Run the async test
        coro = asyncio.coroutine(run_test)
        event_loop.run_until_complete(coro())
        event_loop.close()
```

This is one of the simplest ways to test asynchronous code without external dependencies. What is going on here: 

 - A new event loop should be created for every test, otherwise it could become a point of mutated state across multiple tests and cause strange issues.
 - `await` can only be called from a function marked with `async`, so a coroutine must be created inside the test function to await the function call `person.create()`.
 - Again, I cannot `await run_test()` in the test function so I transform it into a coroutine object that I can send directly to the event loop.

### With `pytest.mark.asyncio`

```python
import asyncio
import pytest

from unittest.mock import MagicMock

from my_code import PersonClass

class TestPersonClass(object):

    @pytest.mark.asyncio
    async def test_send_query_on_create(self):
        """Should send insert query to database on create()"""

        # Stub the database cursor
        database_cursor = MagicMock()

        # Stub the execute function with mocked results from the database
        execute_stub = MagicMock(return_value='future result!')
        # Wrap the stub in a coroutine (so it can be awaited)
        execute_coro = asyncio.coroutine(execute_stub)
        database_cursor.execute = execute_coro

        # Instantiate new person obj
        person = PersonClass(name='Johnny', cursor=database_cursor)

        # Call person.create() to trigger the database call
        person_create_response = await person.create()

        # Assert the response from person.create() is our predefined future
        assert person_create_response == 'future result!'
        # Assert the database cursor was called once
        execute_stub.assert_called_once_with(person.insert_sql, ('Johnny',))
```

This code is much cleaner, and resembles any other unit test in the project. No setting up event loops or managing coroutines. Just one decorator and we are gold.

#### Important! Danger! EXTERMINATE!!

Please take note in the above code that `TestPersonClass` is **not** a child class of `unittest.TestCase`. If it was, the test would still succeed -- but the success would be a false positive because code after the `await` expression would not run.

Why is this happening? The answer is complex enough that it deserves a separate post, but the tl;dr version is that on [line 93](https://github.com/pytest-dev/pytest-asyncio/blob/master/pytest_asyncio/plugin.py#L93) of pytest-asyncio's [source](https://github.com/pytest-dev/pytest-asyncio) the author is expecting the event loop to be passed into the test from a pytest [fixture](http://doc.pytest.org/en/latest/fixture.html), while `unittest.TestCase` methods cannot directly receive fixture function arguments. Whew.

## External References

#### Articles

 - [Stefan Scherfke - Advanced asyncio testing](https://stefan.sofa-rockers.org/2016/03/10/advanced-asyncio-testing/)
 - [StackOverflow - How to test Python 3.4 asyncio code?](http://stackoverflow.com/questions/23033939/how-to-test-python-3-4-asyncio-code)

#### Docs

 - [pytest docs](http://doc.pytest.org/en/latest/contents.html#toc)
 - [pytest-asyncio (pytest plugin)](https://pypi.python.org/pypi/pytest-asyncio)
 - [Python unittest module](https://docs.python.org/3/library/unittest.html)


