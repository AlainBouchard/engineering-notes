+++
title = "Python and PyTest"
LastModifierDisplayName = "Alain Bouchard"
LastModifierEmail = "abouchard@live.ca"
disableToc = "false"
+++

{{< toc >}}

## Python and PyTest

Working with multiple python versions may cause issues and confusion. It is recommended to specify the python version to use.

Working with python 3.9:

```bash
py -3.9 -m <command>
```

It is recommended to _upgrade_ the `pip` version:

```bash
py -3.9 -m pip install --upgrade pip
```

It is recommended to install `tox`, which is a virtual environment (venv) manager for Python.

```bash
py -3.9 -m pip install tox
```

To run tests with `tox`, the following example assumes that pytest has been configured by the `tox.ini` commands parameter:

```bash
py -3.9 -m tox --recursive -- <pytest parameters>
```

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

### pytest.ini configuration file example

```ini
[pytest]
# Configure the logging within PyTest (the configuration won't work if configured in test files)
log_cli = 1
log_cli_level = WARNING
log_cli_format = %(asctime)s [%(levelname)8s] %(message)s (%(filename)s:%(lineno)s)
log_cli_date_format=%Y-%m-%d %H:%M:%S

# Filter Warnings
filterwarnings =
    ignore::FutureWarning

# Uses classes with prefix Test as test files
python_files = Test*.py
```

## Tox

[Tox] aims to automate and standardize testing in Python. It is part of a larger vision of easing the packaging, testing and release process of Python software.

### tox.ini

Tox may be used along with a `tox.ini` configuration file.

```ini
[framework]
files = tests
coverages = --cov=src

[tox]
envlist = py3.9
skip_missing_interpreters = false
skipsdist = true
toxworkdir = tmp

[flake8]
max-line-length = 159
# E501: Ignore max line length
# ignore = E501

[pytest]
norecursedirs = .cache tmp

# pytest-spec configuration
spec_header_format = {module_path}:
spec_test_format = {result} {name}

[testenv]
deps = -rrequirements.txt

# Only forward the environment variables with the following prefix.
passenv = PYTHON_SANDBOX_*
commands =
    flake8 src tests --exclude=__init__.py
    pytest -p no:cacheprovider --spec --durations=5 --cov-config .coveragerc --cov-report term-missing {posargs} {[framework]coverages} {[framework]files}
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

### Test class skeleton

The Test Class will work like the fixtures from the test file.

```py
# conftest.py
import pytest
import logging


@pytest.fixture(scope="session", autouse=True)
def set_logging() -> None:
    logging.info("set_logging on conftest.py")


# TestExample.py
import logging


class TestExample:
    @classmethod
    def setup_class(cls):
        logging.info("starting class: {} execution".format(cls.__name__))

    @classmethod
    def teardown_class(cls):
        logging.info("starting class: {} execution".format(cls.__name__))

    def setup_method(self, method):
        logging.info("starting execution of tc: {}".format(method.__name__))

    def teardown_method(self, method):
        logging.info("starting execution of tc: {}".format(method.__name__))

    def test_tc1(self):
        logging.info("running tc1")
        assert True

    def test_tc2(self):
        logging.info("running tc2")
        assert True
```

The output of this example Test Class will be:

```log
============================= test session starts =============================
collecting ... collected 2 items

TestExample.py::TestExample::test_tc1
------------------------------- live log setup --------------------------------
2022-07-18 12:57:45 [    INFO] set_logging on conftest.py (conftest.py:7)
2022-07-18 12:57:45 [    INFO] starting class: TestExample execution (TestExample.py:7)
2022-07-18 12:57:45 [    INFO] starting execution of tc: test_tc1 (TestExample.py:14)
-------------------------------- live log call --------------------------------
2022-07-18 12:57:45 [    INFO] running tc1 (TestExample.py:20)
PASSED                                                                   [ 50%]
------------------------------ live log teardown ------------------------------
2022-07-18 12:57:45 [    INFO] starting execution of tc: test_tc1 (TestExample.py:17)

TestExample.py::TestExample::test_tc2
------------------------------- live log setup --------------------------------
2022-07-18 12:57:45 [    INFO] starting execution of tc: test_tc2 (TestExample.py:14)
-------------------------------- live log call --------------------------------
2022-07-18 12:57:45 [    INFO] running tc2 (TestExample.py:24)
PASSED                                                                   [100%]
------------------------------ live log teardown ------------------------------
2022-07-18 12:57:45 [    INFO] starting execution of tc: test_tc2 (TestExample.py:17)
2022-07-18 12:57:45 [    INFO] starting class: TestExample execution (TestExample.py:11)

============================== 2 passed in 0.02s ==============================

Process finished with exit code 0
```

A session configuration can be done in the `conftest.py` as shown in the example.  It will be run once only in the session setup.

### Fixtures

A [test fixture] is a concept used in both electronics and software. It’s a piece of software or device that sets up a system to satisfy certain preconditions of the process. Its biggest advantage is that it provides consistent results so that the test results can be repeatable. Examples of fixtures could be loading a test set to the database, reading a configuration file, setting up environment variables, etc.

A pytest fixture has a specific scope.  By default, the scope is a function. Pytest fixtures have five different scopes: function, class, module, package, and session. The scope basically controls how often each fixture will be executed.

Order of priority:

1. session (higher priority)
1. package
1. module
1. class
1. function (lower priority)

#### Function

THe default scope is function: `scope="function" and therefore may be omitted.

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

Special usage of the keyword `yield` in a fixture: the code before the `yield` keyword will be executed before the test functions of the Test Class while the code after the `yield` keyword will be executed after the test functions of the Test Class.

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

The `scope="module"` runs the fixture per module while the `scope="package"` runs by package.  The scope _module_ is usually used more often than the scope `package`. The difference between scope `function` and scope `module` is that the scope module will only be run once, even if used in many functions in the module.

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

- stores common utility test fixtures and extension code often referred to as _hooks_
- pytest collects the fixtures in this file so they are globally accessible within the testing directory
- it must be placed under your `/tests` directory
- good practice to cross-reference this file when reading a testing suite

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

The [pytest.mark.parametrize] fixture allows the user to run the same test, multiple times, by modifying the input parameters.

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

It is possible to use decorators for [getters and setters].

### Commonly used functions and examples

#### Verify variable type

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
[tox]: https://tox.wiki/en/latest/
