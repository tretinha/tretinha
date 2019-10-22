---
title: "Unit testing with python"
date: 2019-10-22T10:52:20-03:00
draft: false
---

There is no denying that unit testing is dropping our team's bug counter to
nearly zero, as opposed to the mess it once was. I won't be advocating for TDD
or trying to convince you that this is the only thing you can do to ensure 
quality software. It's not the only thing we did, and it's certainly not the 
most important one.

But, if you are looking to apply unit tests to your python program, this is 
sort of a guide to what I think are the most important things you should know.

The first thing would be choosing the testing lib. We have looked into the three
main ones, behave, unittest and pytest. We chose the latter because we thought
of it as a mix between the two. Behave and unittest are both good in what they 
offer, but we thought behave is more useful for integration tests as the unit 
tests started to get too verbose and unittest lacks test cases, which means you 
have to create a function for every case and corner case of the thing you are 
testing. Pytest does have test cases, using the parametrize decorator, and it's 
not too verbose, as opposed to behave. And I __think__ it's the most used by 
the community as well.

I am also going to use a lib called mock; for mocking stuff, obviously. Most
people that advocate for unit tests say you shouldn't use other components of
your software unnecessarily to test one of them. As an example, if you have a
method that saves a document into a database, you shouldn't access your database
third-party lib to test it. You can't verify the lib's integrity with your test 
and, if your test fails, who knows if this is because of the lib or your code.
This is where mock shines. You can create a mock of your database connector and
mimick its output based on the input you provide.

The mock lib comes with a function called patch, where you can mock the things
you imported in your to-be-tested function. With the database example, let's say
you use CouchDB and you are going to test a save method. Your function imports
CouchDB from cloudant, but you __don't__ want to use the real CouchDB object -
again, what if there's a problem in the lib?

```
from cloudant import CouchDB

class Connector:
    def __init__(self):
        self.couchdb = CouchDB(...)

    def save(document):
        couchdb.bulk_docs([document])  
```

To test the save method, we should mock the CouchDB class, which is the class 
that is imported inside the Connector module. Patch is useful for that. I'm 
assuming our Connector lives in a dummy path: `databases.couchdb`.


```
from mock import patch

@patch('database.couchdb.CouchDB')
def test_save(mocked_couchdb):
   ...
```

With that, the return value of the objects of the imported CouchDB class are all
`Mock()` objects. So everytime the `self.couchdb = CouchDB(...)` is executed,
`self.couchdb` receives a `Mock()` object. And, of course, you can fiddle with
the return value, it can be basically whatever you want. You could change it 
using a `return_value` parameter in the path or setting
`mocked_couchdb.return_value = ...`.


You can also use the `side_effect` parameter, which is basically a list
containing the sequence of return values that are attributed to the object as
the test cases run. And, lastly, speaking of test cases, to avoid calling the 
`save` method several times with different documents to test every possible 
case (and, frankly, turning the test basically unreadable), we can use the
parametrize decorator.

It has two parameters, a tuple containing the names of the test cases and a list
containing tuples with the test cases. Going on with the database save example,
we can store the `bulk_docs` (it's a real method from cloudant's CouchDB) 
responses in a variable and then assert that the response is actually what it's
supposed to be, that's what tests are for, right? So:

```
import pytest
from mock import patch

@patch('database.couchdb.CouchDB')
@pytest.mark.parametrize(
    ('document', 'expected_response'), [({'_id': '1'}, dummy_return), (...)])
def test_save(mocked_couchdb, document, expected_response):
    connector = Connector()
    response = connector.save(document)
    assert response == expected_return
```

Breaking it down a little bit: the tuple `('document', 'expected_response')`
represents what you are providing as test cases, and in the cases array, in the
first tuple: `({'_id': '1'}, dummy_return`), `{'_id': '1'}` is the document,
while `dummy_return` is the expected return.

You can add as many test cases as you want in the array of tests, e.g.:
`({'_id': '2'}, dummy_return2)`, `({'_id': '3'}, False)`, etc.

The test will run for every case you have defined in the parametrize. 
Therefore, you asserted that given an input, it returned the correct response. 
And this covers the basic stuff of testing with pytest and mock, how to use the
return values and the side effects. It should be enough for most applications.

Don't forget to check the documentations as well. There are plenty of other
things there.
