[![Build Status](https://github.com/hukkinj1/typenv/workflows/Tests/badge.svg?branch=master)](https://github.com/hukkinj1/typenv/actions?query=workflow%3ATests+branch%3Amaster+event%3Apush)
[![codecov.io](https://codecov.io/gh/hukkinj1/typenv/branch/master/graph/badge.svg)](https://codecov.io/gh/hukkinj1/typenv)
[![PyPI version](https://img.shields.io/pypi/v/typenv)](https://pypi.org/project/typenv)

# typenv

<!--- Don't edit the version line below manually. Let bump2version do it for you. -->

> Version 0.1.6

> Typed environment variable parsing for Python

**Table of Contents**  *generated with [mdformat-toc](https://github.com/hukkinj1/mdformat-toc)*

<!-- mdformat-toc start --slug=github --maxlevel=6 --minlevel=2 -->

- [Background](<#background>)
- [Installing](<#installing>)
- [Usage](<#usage>)
  - [Basics](<#basics>)
  - [Supported types](<#supported-types>)
  - [Default values](<#default-values>)
  - [Name prefixes](<#name-prefixes>)
  - [Name character set](<#name-character-set>)
  - [Name uppercasing](<#name-uppercasing>)
  - [Validation](<#validation>)
  - [Reading from a `.env` file](<#reading-from-a-env-file>)
  - [Dumping parsed values](<#dumping-parsed-values>)
- [Acknowledgments](<#acknowledgments>)

<!-- mdformat-toc end -->

## Background<a name="background"></a>

Typenv does environment variable parsing with an API almost identical to the excellent [environs](https://github.com/sloria/environs).
There are a few reasons why typenv might be preferred:

- Type annotated typecast functions: type checkers are able to understand types of parsed environment variables.
- More flexible prefix manipulation of environment variable names.
- Validation of environment variable names.
- Optional automatic uppercasing of environment variable names.
- Ability to generate a .env.example that shows expected types of environment variables.
- Less dependencies. No [marshmallow](https://github.com/marshmallow-code/marshmallow) required.

## Installing<a name="installing"></a>

Installing from PyPI repository (https://pypi.org/project/typenv):

```bash
pip install typenv
```

## Usage<a name="usage"></a>

### Basics<a name="basics"></a>

Set environment variables:

```bash
export NAME='Harry Potter'
export AGE=14
export IS_WIZARD=true
export PATRONUM_SUCCESS_RATE=0.92
export BANK_BALANCE=134599.01
export LUCKY_NUMBERS=7,3,11
export EXTRA_DETAILS='{"friends": ["Hermione", "Ron"]}'
```

Parse the values in Python:

```python
from typenv import Env

env = Env()

NAME = env.str("NAME")  # => "Harry Potter"
AGE = env.int("AGE")  # => 14
IS_WIZARD = env.bool("IS_WIZARD")  # => True
PATRONUM_SUCCESS_RATE = env.float("PATRONUM_SUCCESS_RATE")  # => 0.92
BANK_BALANCE = env.decimal("BANK_BALANCE")  # => decimal.Decimal("134599.01")
LUCKY_NUMBERS = env.list("LUCKY_NUMBERS", subcast=int)  # => [7, 3, 11]
EXTRA_DETAILS = env.json("EXTRA_DETAILS")  # => {"friends": ["Hermione", "Ron"]}

# Optional settings must have a default value
IS_DEATH_EATER = env.bool("IS_DEATH_EATER", default=False)  # => False
```

### Supported types<a name="supported-types"></a>

The types supported by typenv are:

- `env.str`
- `env.int`
- `env.bool`
- `env.float`
- `env.decimal`
- `env.json`
- `env.list`
  - Takes a subcast argument for casting list items to one of `str`, `int` , `bool`, `float` or `decimal.Decimal`
- `env.bytes`

### Default values<a name="default-values"></a>

Normally, if an environment variable is not found, typenv raises an exception.
If a default value is provided, however, that will be returned instead of raising.

```python
from typenv import Env

env = Env()

BOOL = env.bool("NON_EXISTING_NAME", default=False)  # => False
LIST = env.list("NON_EXISTING_NAME", default=["a", "b"])  # => ["a", "b"]
OPTIONAL_INT = env.int("NON_EXISTING_NAME", default=None)  # => None
```

### Name prefixes<a name="name-prefixes"></a>

TODO: document here

### Name character set<a name="name-character-set"></a>

Typenv validates environment variable names.
By default, the set of allowed characters includes upper case ASCII letters, digits and the underscore (`_`).

The set of allowed characters can be configured:

```python
from typenv import Env

env = Env(allowed_chars="ABCDEFGHIJKLMNOPQRSTUVWXYZ")
```

### Name uppercasing<a name="name-uppercasing"></a>

```bash
export UPPER_CASE_NAME=true
```

```python
from typenv import Env

# Environment variable names in type cast methods will automatically be upper
# cased when `upper=True` is set here.
env = Env(upper=True)

NAME = env.bool("upper_casE_Name")
```

### Validation<a name="validation"></a>

```bash
export NAME='Harry Potter'
export AGE=14
```

```python
from typenv import Env

env = Env()

# A single validator function
NAME = env.str("NAME", validate=lambda n: n.startswith("Harry"))

# A validator function can signal error by raising an exception
def is_positive(num):
    if num <= 0:
        raise Exception("Number is not positive")


# A validator function can alternatively return `False` to signal an error
def is_less_than_thousand(num):
    if num >= 1000:
        return False
    return True


# Multiple validator functions can be passed as an iterable of callables
AGE = env.int("AGE", validate=(is_positive, is_less_than_thousand))
```

### Reading from a `.env` file<a name="reading-from-a-env-file"></a>

TODO: document here

### Dumping parsed values<a name="dumping-parsed-values"></a>

```bash
export SOME_STR=blaablaa
export SOME_INT=99
```

```python
from typenv import Env, ParsedValue

env = Env()

SOME_STR = env.str("SOME_STR")
SOME_INT = env.int("SOME_INT")

assert env.dump() == {
    "SOME_INT": ParsedValue(value=99, type="int", optional=False),
    "SOME_STR": ParsedValue(value="blaablaa", type="str", optional=False),
}
```

## Acknowledgments<a name="acknowledgments"></a>

The public API of this library is almost an exact copy of [environs](https://github.com/sloria/environs),
which is based on [envparse](https://github.com/rconradharris/envparse) and [django-environ](https://github.com/joke2k/django-environ).
Credit for the interface goes to the authors of those libraries.
