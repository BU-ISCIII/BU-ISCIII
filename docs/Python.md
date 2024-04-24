# PEP-8 Guidelines ([Read the full PEP-8 documentation](https://peps.python.org/pep-0008/#introduction))

PEP 8 is the official style guide for Python code, and it covers various aspects of coding conventions. Here are some key points related to PEP 8 and variable naming best practices:

## Indentation

* Use 4 spaces per indentation level.
* Limit all lines to a maximum of 79 characters for code (72 for docstrings). In our case it is okay to increase the line length limit up to 89 characters
* For long lines, you can break lines using parentheses `()` or backslashes `\`.
* Continuation lines should align wrapped elements either vertically using Python’s implicit line joining inside parentheses, brackets and braces, or using a hanging indent
Example:

```python
result = some_function_that_takes_arguments(
    'a', 'b', 'c',
    'd', 'e', 'f',
     )
# or it may be lined up under the first character of the line that starts the multiline construct:
my_list = [
    1, 2, 3,
    4, 5, 6,
]
```

## Imports

* Imports are always put **at the top of the file**, just after any module comments and docstrings, and before module globals and constants.
* Imports should usually be on separate lines and should be grouped in the following order:
  * Standard library imports.
  * Related third-party imports.
  * Local application/library specific imports.
* You should put a blank line between each group of imports.
* Wildcard imports `from <module> import *` should be avoided.

## Whitespace in Expressions and Statements

* Avoid extraneous whitespace in the following situations:
  * Immediately inside parentheses, brackets, or braces.
  * Immediately before a comma, semicolon, or colon.
  * Immediately before the open parenthesis that starts the argument list of a function call.
  * Immediately before the open parenthesis that starts an indexing or slicing.
* Always surround these binary operators with a single space on either side: assignment `(=)`, augmented assignment `(+=, -= etc.)`, comparisons `(==, <, >, !=, <>, <=, >=, in, not in, is, is not)`, Booleans `(and, or, not)`.

## Function and Variable Names

* Class names should normally use the CapWords convention.
* Function names should be lowercase, with words separated by underscores as necessary to improve readability.
* Variable names follow the same convention as function names.
* Avoid using single-character variable names except for counters or iterators.
Example:

```python
class SchoolStudent:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def naming(self):
        print("Hello my name is " + self.name)

    def my_age(self, year):
        print("My age in ", year, "will be", self.age+year)  
```

## Blank lines

* Surround top-level function and class definitions with two blank lines.
* Method definitions inside a class are surrounded by a single blank line.
* Extra blank lines may be used (sparingly) to separate groups of related functions.
* Use blank lines in functions, sparingly, to indicate logical sections.

## Comments and docstring

* Comments should be complete sentences and should be used sparingly.
* Inline comments can be distracting if they state the obvious.
* Write docstrings for all public modules, functions, classes, and methods.

A docstring is a string literal that occurs as the first statement in a module, function, class, or method definition. Such a docstring becomes the **doc** special attribute of that object. We recommend using vs-code docstring extension for annotating your classes and methods.
Example:

```Python
def make_dir(path):
    """Create directory if it doesn't exist.
    Args:
        path (str): path where the directory will be created.
    Returns:
        None
    """
    try:
        os.makedirs(path)
    except OSError as exception:
        if exception.errno != errno.EEXIST:
            raise
```

[Read PEP-257 Docstring conventions for more details](https://peps.python.org/pep-0257/)

## Dictionary and list comprehension

List comprehensions provide a more concise way to create lists in situations where map() and filter() and/or nested loops would currently be used.
Example:

```python
# Nested loop used to create list of lists from 0 to 5 (e.g. a matrix)
for i in range(5): 
    matrix.append([])
    for j in range(5): 
        matrix[i].append(j)
# Same result using a list comprehension:
[[j for j in range(5)] for i in range(5)] 
```

[See more of list comprehensions and how they work here](https://docs.python.org/3/tutorial/datastructures.html#list-comprehensions)

Dict comprehensions are just like list comprehensions, except that you group the expression using curly braces instead of square braces:

```python
{k : v for k, v in some_dict.items()}
{"key1": value1, "key": value2, "key3": value3}
```

## Linting

Linting is the process of analyzing source code to identify potential errors, style violations, and other issues early in the development process.
When trying to commit changes to any BU-ISCIII repository you will encounter with a linting process that is automatically executed via Github actions. If this linting process fails, you won't be able to merge any changes. In BU-ISCIII we use two tools for linting: **Flake8** and **Black**

**Flake8** is a highly configurable tool that combines multiple linters to provide a comprehensive code analysis:

* PyFlakes: examines the code for static errors and undefined names.
* Pycodestyle: checks for adherence to PEP 8 style conventions.
* McCabe Complexity Checker: Complexity Checker identifies functions with high complexity.

**Black** is a code formatting tool for Python that follows an "opinionated" approach, meaning it imposes a specific style without much room for configuration. While Flake8 focuses on identifying issues and enforcing coding standards, Black focuses solely on automatically formatting code to adhere to a specific style, producing code that is visually appealing and adheres to a strict set of formatting rules.

Before commiting changes to one of our repositories, we recommend executing the linting process locally. You will need to install both Flake8 and black on your system, and then execute the linters with the following commands:

```Flake8
pip install flake8
# Move to folder containing files to be analyzed
flake8 
```

This code will analyze all the scripts in your folder and show an error report:

```
filename.py:3:1: E101 indentation contains mixed spaces and tabs
filename.py:5:10: W292 no newline at end of file
filename2.py:8:15: F821 undefined name 'example_variable'
...
Total Errors: 3
Total Warnings: 1
Total Flake Warnings: 1
```

For each file, Flake8 reports the line numbers where issues or violations were found. Flake8 uses error codes to identify the specific type of issue detected, where each type of error has a different level of severity over the code.

* E (Error): Indicates a more severe issue that may lead to runtime errors or unexpected behavior.
* W (Warning): Suggests potential problems or deviations from best practices but doesn't necessarily lead to errors.
* F (Flake): Identifies stylistic issues or deviations from coding conventions.

Black automatically reformats the targeted file. We recommend to always check the changes in the file before commiting to github.

```
pip install black
black script.py
```

You will encounter a report like this:

```
# If any of the files have been reformated
reformatted script.py
All done! ✨ 🍰 ✨
1 file reformatted.

# If no changes were done:
All done! ✨ 🍰 ✨
1 file left unchanged.
```

Some of the most common sources of linting errors are:

* Unfinished functions or blocks of code that are never executed.
* Variables called before being asigned
* Code not following PEP-8 style conventions

## Logging

Logging is an incredibly important feature of any application as it gives both programmers and people supporting the application key insight into what their systems are doing. Without proper logging we have no real idea as to why our applications fail and no real recourse for fixing these applications.

### Logging Configuration File

Create a file named logging_config.ini in your application to configure the information in the logging file.

Example of logging file (logging_config.ini):

```Python
[loggers]
keys=root

[handlers]
keys=stream_handler

[formatters]
keys=formatter

[loggers]
keys=root

[handlers]
keys=logfile

[formatters]
keys=logfile_formatter

[logger_root]
level=DEBUG
handlers=logfile

[formatter_logfile_formatter]
format=%(asctime)s %(name)-12s %(levelname)-8s %(message)s

[handler_logfile]
class=handlers.RotatingFileHandler
level=NOTSET
## args(log_file_name, 'a', maxBytes , backupCount)
args=('/srv/taranis/testing.log','a',150,5)
formatter=logfile_formatter

```

Then use **logging.config.fileConfig()** in the code:

```Python
import logging
from logging.config import fileConfig
    def open_log (logger_name)
    fileConfig('logging_config.ini')
    logger = logging.getLogger(logger_name)
    return logger

```

* `print` statement should not be used unless exceptional cases to inform the user when it runs the code from console.
* Write logging records everywhere with proper level and the information to debug the code.
* Use **name** as the logger name, when the application and the logger code is in the same file. When the logger code is in a separate directory/file than the main application then use a logger name that identify the application.
* Capture exceptions and record them with traceback. It is always a good practice to record when something goes wrong, but it won’t be helpful if there is no traceback. You should capture exceptions and record them with traceback. By calling logger methods with exc_info=True parameter, traceback is dumped to the logger.

Following is an example:

```Python
try:
    open('/path/to/does/not/exist', 'rb')
except (SystemExit, KeyboardInterrupt):
    raise
except Exception, e:
    logger.error('Failed to open file', exc_info=True)

```

* Use rotating file handler.
* Setup a central log server when you have multiple servers

## Exception Handling Errors

Exceptions are a means of breaking out of the normal flow of control of a code block to handle errors or other exceptional conditions.

When designing exceptions, it's important to remember that they should be targeted both at humans and computers. That's why they should include an explicit message, and embed as much information as possible.

That will help to debug and write resilient programs that can pivot their behaviour depending on the attributes of exception.

Silencing exceptions completely is to be considered as bad practice.
You should **not** write code like that:

### _Bad:_

```Python
try:
    do_something()
except Exception:
    # Whatever
    pass
```

Not having any kind of information in a program where an exception occurs is a nightmare to debug.

You should use the logging library. You can use the exc_info parameter to log a complete traceback when an exception occurs, which might help debugging on severe and unrecoverable failure:

### _Good:_

```python
try:
    do_something()
except Exception:
    logging.getLogger().error("Something bad happened", exc_info=True)
```

### Exceptions must follow certain conditions

* When an exception statement is written in the code this should be documented inside the function description help. Example:

```python
def connect_to_next_port(self, minimum):
    """Connects to the next available port.

    Args:
      minimum: A port value greater or equal to 1024.
    Raises:
      ValueError: If the minimum port specified is less than 1024.
      ConnectionError: If no available port is found.
    Returns:
      The new minimum port.
    """
```

* Provide details about the error. This is extremely valuable to be able to log correctly errors or take further action and try to recover.
* Raise exceptions like this: `raise MyError('Error message')` or `raise MyError()`. Do not use the two-argument form `(raise MyError, 'Error message')`.
* Make use of built-in exception classes when it makes sense. For example, raise a `ValueError` if you were passed a negative number but were expecting a positive one. Do not use `assert` statements for validating argument values of a public API. `assert` is used to ensure internal correctness, not to enforce correct usage nor to indicate that some unexpected event occurred. If an exception is desired in the latter cases, use a raise statement.
* Libraries or packages may define their own exceptions. When doing so they must inherit from an existing exception class. Exception names should end in `Error` and should not introduce stutter `(foo.FooError)`.
* Never use catch-all `except`: statements, or `catch Exception` or `StandardError`, unless you are re-raising the exception or in the outermost block in your thread (and printing an error message). Python is very tolerant in this regard and except: will really catch everything including misspelled names, sys.exit() calls, Ctrl+C interrupts, unittest failures and all kinds of other exceptions that you simply don’t want to catch.
* Minimize the amount of code in a `try/except` block. The larger the body of the `try`, the more likely that an exception will be raised by a line of code that you didn’t expect to raise an exception. In those cases, the `try/except` block hides a real error.
* When capturing an exception, use `as` rather than a comma. For example:

```python
try:
    file = open(filename)
except Exception as e:
    logging.getLogger().error("Something bad happened %s", e)
```

* Use the `finally` clause to execute code whether or not an exception is raised in the try block. This is often useful for cleanup, i.e., closing a file.

## References

* [Pyguide](http://google.github.io/styleguide/pyguide.html)
* [Exception handling guide](https://julien.danjou.info/python-exceptions-guide/)
* [Python Logging](https://docs.python-guide.org/writing/logging/)
* [Good logging practice in python](https://fangpenlin.com/posts/2012/08/26/good-logging-practice-in-python/)
