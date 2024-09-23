+++
title = "Python and PyTest"
LastModifierDisplayName = "Alain Bouchard"
LastModifierEmail = "abouchard@live.ca"
disableToc = "false"
+++

{{< toc >}}

## Python and PyTest

## PyTest

### Why PyTest?

- allow to run a standalone test function as its own case
- easy to read syntax, allowing you to use the standard assert method
- powerful CLI
- automates test setup, teardown, and common test scenarios (uses fixtures)
- Great to use with CI tools like Jenkins, Travis, Circle CI, etc.
- actively maintained with participatory open-source community

### requirements.txt content for pytest

```text
coverage===xxx
pytest===xxx
pytest-cov===xxx
pytest-flakes===xxx
pytest-pep8===xxx
pytest-pythonpath==xxx
docker
```

- pytest-flakes will make pytest use [PyFlake and Flake8].  It will make pytest and python use a Linter and code style checker.

### Run a test

Get `pytest` help:

```bash
pytest -h
```

Run all tests:

```bash
pytest
```

Run test using keyword for filename

```bash
pytest -k <test_name_keyword>
```

Explore test coverage of a script - requires the `pytest-cov` package to be installed:

```bash
pytest --cov scripts
```

### Test file skeleton

```py
from path.to.class.to.test import ClassName
from pytest # to use pytest Context Manager


def test_name():
    obj = ClassName("value1", "value2")
    assert obj.value1 == "value1"
    assert obj.value2 == "value2"

def test_with_exception_context_manager():
    with pytest.raises(ValueError) as ex:
        # Code that will raise an exception.
        obj = ClassName("value1", "value2")
        obj.raises_exception()

    assert str(ex.value) == "this is an exception!"
```

### Fixtures

A [test fixture] is a concept used in both electronics and software. It’s a piece of software or device that sets up a system to satisfy certain preconditions of the process. Its biggest advantage is that it provides consistent results so that the test results can be repeatable. Examples of fixtures could be loading test set to the database, reading a configuration file, setting up environment variables, etc.

A pytest fixture has a specific scope.  By default, the scope is a function. Pytest fixtures have five different scopes: function, class, module, package, and session. The scope basically controls how often each fixture will be executed.

Order of priority:
1. session (higher priority)
1. package
1. module
1. class
1. function (lower priority)

#### Function

THe default scope is function: `scope="function" and therefore may be omited.

```py
import pytest
from datetime import datetime

@pytest.fixture()
def only_used_once():
    with open("app.json") as f:
        config = json.load(f)
    return config

@pytest.fixture()
def light_operation():
    return "I'm a constant"

@pytest.fixture()
def need_different_value_each_time():
    return datetime.now()
```

#### Class

The `scope="class"` run _before_ any function or test of the Test Class.

```py
@pytest.fixture(scope="class")
def dummy_data(request):
    request.cls.num1 = 10
    request.cls.num2 = 20
    logging.info("Execute fixture")

@pytest.mark.usefixtures("dummy_data")
class TestCalculatorClass:
    def test_distance(self):
        logging.info("Test distance function")
        assert distance(self.num1, self.num2) == 10

    def test_sum_of_square(self):
        logging.info("Test sum of square function")
        assert sum_of_square(self.num1, self.num2) == 500
```

Special usage of the keywork `yield` in a fixture: the code before the `yield` keyword will be executed before the test functions of the Test Class while the code after the `yield` keyword will be executed after the test functions of the Test Class.

```py
@pytest.fixture(scope="class")
def prepare_db(request):
    # pseudo code
    connection = db.create_connection()
    request.cls.connection = connection
    yield
    connection = db.close()

@pytest.mark.usefixtures("prepare_db")
class TestDBClass:
    def test_query1(self):
        assert self.connection.execute("..") == "..."

    def test_query2(self):
        assert self.connection.execute("..") == "..."
```

#### Module and package

The `scope="module"` run the fixture per module while the `scope="package"` run by package.  The scope _module_ is usualy used more often than the scope `package`. The difference between scope `function` and scope `module` is that the scope module will only be run once, even if used in many functions in the module.

```py
@pytest.fixture(scope="module")
def read_config():
    with open("app.json") as f:
        config = json.load(f)
        logging.info("Read config")
    return config

def test1(read_config):
    logging.info("Test function 1")
    assert read_config == {}

def test2(read_config):
    logging.info("Test function 2")
    assert read_config == {}
```

#### Session

The `scope="session"` is only run once every time `pytest` is run.  A per-directory `conftest.py` file will be executed once per `pytest` execution.

```py
# test/conftest.py
@pytest.fixture(scope="session")
def read_config():
    with open("app.json") as f:
        config = json.load(f)
        logging.info("Read config")
    return config

# test/test_code1.py
def test1(read_config):
    logging.info("Test function 1")
    assert read_config == {}

def test2(read_config):
    logging.info("Test function 2")
    assert read_config == {}

# test/test_code2.py
def test3(read_config):
    logging.info("Test function 3")
    assert read_config == {}

def test4(read_config):
    logging.info("Test function 4")
    assert read_config == {}
```

Using `conftest.py` for common functions:
- stores common utlity test fixtures and extension code often referred to as _hooks_
- pytest collects the fixtures in this file so they are globally accessible within the testing directory
- it must be placed under your `/tests` directory
- good practice to cross-reference ths file when reading a testing suite

#### conftest.py modularization

It is possible to modularize the conftest.py file when it is getting too big.

```py
# referring to modules:
# tests/utils/db.py
# tests/utils/network.py

# in conftest.py
pytest_plugins = [
    "tests.utils.db",
    "tests.utils.network"
]
```

The `autouse=True` fixtures must stay in the `conftest.py` file.

#### Autouse

The fixture parameter `autouse=True` will make the fixture used automatically even if the fixture isn't called by the test function.

```py
@pytest.fixture(autouse=True)
def function_autouse():
    logging.info("scope function with autouse")
    
def test_autouse():
    assert True
```

#### Parametrize

The [pytest.mark.parametrize] fixture allows to run the same test, multiple time, by modifying the input parameters.

```py
@pytest.mark.parametrize("num, output",[(1,11),(2,22),(3,35),(4,44)])
def test_multiplication_11(num, output):
   assert 11*num == output
```

## Python

### Class file skeleton

```py
class ClassName():
    """
    This is a multi-line comment (used for header in this example)
    This is a second line of comment..
    """
    def __init__(self, var1: str, var2: str): # the "var1: str" format will require the var1 to be a string type
        self._var1 = var1 # the _ is to make the variable "protected"
        self._var2 = var2

    @property # Using property decorator as a getter function
    def var1(self) -> str: # the -> specifies the return type
        return self._var1

    @var1.setter # using setter decorator for setter function
    def var1(self, value: str):
        self.var1 = value

    def raises_exception():
        raise ValueError("this is an exception!")
```

It is possible to user decorators for [getters and setters].

### Commonly used functions and examples

#### Verify variable type:

```py
if not isinstance(variable, str):
    raise ValueError("wrong type!")
```

#### Open file with context manager

```py
with open("test.txt", 'w', encoding='utf-8') as f:
   f.write("my first file\n")
   f.write("This file\n\n")
   f.write("contains three lines\n")
```

#### Iterate a list

```py
data = ["abc", "def", "ghi"]

for each_data in data:
  assert each_data in data
```

#### try except

```py
try:
    # Some Code
except:
    # Executed if error in the
    # try block
else:
    # execute if no exception
finally:
    # Some code .....(always executed)
```

<!-- References and links -->

[getters and setters]: https://www.geeksforgeeks.org/getter-and-setter-in-python/
[fixture.mark.parametrize]: https://www.tutorialspoint.com/pytest/pytest_parameterizing_tests.htm
[PyFlake and Flake8]: https://medium.com/python-pandemonium/what-is-flake8-and-why-we-should-use-it-b89bd78073f2
[test fixture]: https://betterprogramming.pub/understand-5-scopes-of-pytest-fixtures-1b607b5c19ed