# Testing (pytest-django)

This readme explains how to configure pytest and use it with Django applications. 

# Setup 

## Installation
1. Install ```pytest```, ```pytest-django``` and ```pytest-pythonpath``` - ALL must be installed within the same environment as the program being tested.
1. At the workspace root, create a ```pytest.ini``` file and populate with the following 
```
    [pytest]
    DJANGO_SETTINGS_MODULE = djangoProjectName.settings 
    python_files=*test*.py
    python_paths = .
```
- ```DJANGO_SETTINGS_MODULE```: location of the main django project settings.py
- ```djangoProjectName```: the name of the django __project__ (_not_ just the django app)
- ```python_files = test*.py``` : search for test files beginning with ```test``` (other patterns also accepted eg ```*test```)
- ```python_paths = . ``` : correctly sets the PYTHONPATH environment variable to the workspace root - ensures testing modules can correctly access local module imports. 

Note - using pytest_pythonpath to set the PYTHONPATH from within pytest.ini was the only method I could find that worked, even when ```sys.path``` appeared to have the correct workspace root set, and the program was able find modules fine when not running pytest.

## Configuring VSCode

1. Add the following to the settings.json file to set the testing configuration to pytest (since pytest-django is installed, pytest is automatically configured for django projects)
```json
    "python.testing.pytestArgs": [
        "projectFolder"
    ],
    "python.testing.unittestEnabled": false,
    "python.testing.nosetestsEnabled": false,
    "python.testing.pytestEnabled": true
```
Replace ```"projectFolder"``` with the folder that contains the entire django project (the folder with ```manage.py```) pytest-django then automatically recognises the django project.
1. Tests can now be created within the tests.py file within the django application (or anywhere desired within ```"projectFolder"```).
1. In VSCode, select the testing tab from the sidebar, it may be hidden but can be re-enabled by right clicking the sidebar. Any discovered tests will now appear and can be ran/debugged from here.

# Creating tests

Using pytest-django, tests can be built inside classes (unittest method) or as standalone functions - unittest class based tests are fully compatible with pytest and can be useful for building larger tests with multiple test methods.

## Test anatomy:

Tests are (in most cases) implemented using the ```assert``` keyword, which simply takes a conditional statement and evaluates it. Whilst this seems trivial, it allows for large tests with multiple conditionals to be easily constructed without needing to write many ```if(condition == True)``` statements and keeping track of the result for each. If any assert statement fails this will be displayed in the output log, so allows for easy location of failed test aspects. The basic usage is as follows:
```python
def test_something(inputs, expectedOutputs):
    # someFunction imported into tests.py
    assert someFunction(inputs) == expectedOutput 
```
this test will only pass if the action of ```someFunction()``` on the inputs produces the expected output. This is the most versatile testing methodology, but more specific assert statements can also be used. Django specific asserts can be imported from pytest-django, such as 
```python
from pytest_django.asserts import assertContains, assertTemplateUsed, ... etc
```

## Test classes:
Tests can be created within a class, (name must begin with ```Test```) that derives from the builtin django test class ```TestCase``` (or ```SimpleTestCase``` for testing without needing the database). 
- Inheriting from ```TestCase``` is _not_ strictly necessary in pytest, however, it provides access to many of the needed objects such as the ```Client``` class for simulating GET and POST responses. 

Tests are then constructed as class methods within this class, for example 
```python
class TestClass(SimpleTestCase):
    
    # Class may contain any data required for the test(s) - inputs, expected outputs etc
    inputs = list(...)
    expectedResults = list(...)
    
    # Class method defining a single test - name must begin with test
    def test_something(self):
        ...
    # Multiple tests can exist within the same class
    def test_somethingElse(self)
        ... 
```
## Standalone methods (pytest)

In addition to creating a test class, standalone methods can also be used as tests (pytest only). From a standalone method, any required objects such as the django client must also be instantiated.
```python
def test_something_standalone(...):
    # Either pass in test data or create within this method 
    # Any required objects, like the client, must be instantiated
    c = Client()
    # Standard test implementation
    assert ... == ...
```

## The Django client

The client facilitates simulating GET and POST requests to the server. A simple example for testing whether a page returns the correct HTTP response is shown here:
```python
# Instantiate a client instance (shown here) OR use within a class derived from TestCase
from django.test import Client

def test_get_response():
    c = Client()
    # Simulate the HTTP response for the index page
    response = c.get('/index/')
    # Assert the response.status_code is OK
    assert response.status_code == 200
```
This test will thus fail if the index page does not return the correct HTTP response. 

### Simulating POST data

The client can also be used to test responses from given POST data. The client must receive the POST data as a dict. The following example creates a simulated POST request and the respective response

```python
def test_post_response():
    c = Client()
    # Simulated post data
    postData = {'key':value, ...}
    # The returned response
    response = c.post('/index/', postData)
    assert ...
```
