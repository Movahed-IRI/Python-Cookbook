## Testing Output Sent to stdout
##### Problem
You have a program that has a method whose output goes to standard Output ( sys.stdout ). This almost always means that it emits text to the screen. You’d like to write a test for your code to prove that, given the proper input, the proper output is displayed.
##### Solution
Using the `unittest.mock` module’s `patch()` function, it’s pretty simple to mock out `sys.stdout` for just a single test, and put it back again, without messy temporary variables or leaking mocked-out state between test cases.

Consider, as an example, the following function in a module mymodule :
``` py
# mymodule.py
def urlprint(protocol, host, domain):
    url = '{}://{}.{}'.format(protocol, host, domain)
    print(url)
```
The built-in print function, by default, sends output to `sys.stdout` . In order to test that output is actually getting there, you can mock it out using a stand-in object, and then make assertions about what happened. Using the `unittest.mock` module’s `patch()` method makes it convenient to replace objects only within the context of a running test,returning things to their original state immediately after the test is complete.

Here’s the test code for mymodule :
``` py
from io import StringIO
from unittest import TestCase
from unittest.mock import patch
import mymodule
class TestURLPrint(TestCase):
    def test_url_gets_to_stdout(self):
        protocol = 'http'
        host = 'www'
        domain = 'example.com'
        expected_url = '{}://{}.{}\n'.format(protocol, host, domain)
        with patch('sys.stdout', new=StringIO()) as fake_out:
            mymodule.urlprint(protocol, host, domain)
            self.assertEqual(fake_out.getvalue(), expected_url)
```
##### Discussion
The __urlprint()__ function takes three arguments, and the test starts by setting up dummy arguments for each one. The expected_url variable is set to a string containing the expected output. To run the test, the `unittest.mock.patch()` function is used as a context manager to replace the value of `sys.stdout` with a StringIO object as a substitute. The fake_out variable is the mock object that’s created in this process. This can be used inside the body of the with statement to perform various checks. When the with statement completes, patch conveniently puts everything back the way it was before the test ever ran.

!> It’s worth noting that certain C extensions to Python may write directly to standard output, bypassing the setting of `sys.stdout` . This recipe won’t help with that scenario, but it should work fine with pure Python code (if you need to capture I/O from such C extensions, you can do it by opening a temporary file and performing various tricks involving file descriptors to have standard output temporarily redirected to that file).

## Patching Objects in Unit Tests
##### Problem
You’re writing unit tests and need to apply patches to selected objects in order to make assertions about how they were used in the test (e.g., assertions about being called with certain parameters, access to selected attributes, etc.).
##### Solution
The `unittest.mock.patch()` function can be used to help with this problem. It’s a little unusual, but `patch()` can be used as a decorator, a context manager, or stand-alone.

For example, here’s an example of how it’s used as a decorator:
``` py
from unittest.mock import patch
import example
@patch('example.func')
def test1(x, mock_func):
    example.func(x) # Uses patched example.func
    mock_func.assert_called_with(x)
```
It can also be used as a context manager:
``` py
with patch('example.func') as mock_func:
    example.func(x) # Uses patched example.func
    mock_func.assert_called_with(x)
```
Last, but not least, you can use it to patch things manually:
``` py
p = patch('example.func')
mock_func = p.start()
example.func(x)
mock_func.assert_called_with(x)
p.stop()
```
If necessary, you can stack decorators and context managers to patch multiple objects.

For example:
``` py
@patch('example.func1')
@patch('example.func2')
@patch('example.func3')
def test1(mock1, mock2, mock3):
    ...

def test2():
    with patch('example.patch1') as mock1, \
    patch('example.patch2') as mock2, \
    patch('example.patch3') as mock3:
        ...
```
##### Discussion
`patch()` works by taking an existing object with the fully qualified name that you provide and replacing it with a new value. The original value is then restored after the completion of the decorated function or context manager. By default, values are replaced with __MagicMock__ instances.

For example:
``` py
>>> x = 42
>>> with patch('__main__.x'):
...     print(x)
...
<MagicMock name='x' id='4314230032'>
>>> x
42
>>>
```
However, you can actually replace the value with anything that you wish by supplying
it as a second argument to `patch()` :
``` py
>>> x
42
>>> with patch('__main__.x', 'patched_value'):
...     print(x)
...
patched_value
>>> x
42
>>>
```
The __MagicMock__ instances that are normally used as replacement values are meant to mimic callables and instances. They record information about usage and allow you to
make assertions.

For example:
``` py
>>> from unittest.mock import MagicMock
>>> m = MagicMock(return_value = 10)
>>> m(1, 2, debug=True)
10
>>> m.assert_called_with(1, 2, debug=True)
>>> m.assert_called_with(1, 2)
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File ".../unittest/mock.py", line 726, in assert_called_with
        raise AssertionError(msg)
AssertionError: Expected call: mock(1, 2)
Actual call: mock(1, 2, debug=True)
>>>
>>> m.upper.return_value = 'HELLO'
>>> m.upper('hello')
'HELLO'
>>> assert m.upper.called
>>> m.split.return_value = ['hello', 'world']
>>> m.split('hello world')
['hello', 'world']
>>> m.split.assert_called_with('hello world')
>>>
>>> m['blah']
<MagicMock name='mock.__getitem__()' id='4314412048'>
>>> m.__getitem__.called
True
>>> m.__getitem__.assert_called_with('blah')
>>>
```
Typically, these kinds of operations are carried out in a unit test.

For example, suppose you have some function like this:
``` py
# example.py
from urllib.request import urlopen
import csv
def dowprices():
    u = urlopen('http://finance.yahoo.com/d/quotes.csv?s=@^DJI&f=sl1')
    lines = (line.decode('utf-8') for line in u)
    rows = (row for row in csv.reader(lines) if len(row) == 2)
    prices = { name:float(price) for name, price in rows }
    return prices
```
Normally, this function uses `urlopen()` to go fetch data off the Web and parse it. To unit test it, you might want to give it a more predictable dataset of your own creation, however. Here’s an example using patching:
``` py
import unittest
from unittest.mock import patch
import io
import example
sample_data = io.BytesIO(b'''\
"IBM",91.1\r
"AA",13.25\r
"MSFT",27.72\r
\r
''')
class Tests(unittest.TestCase):
    @patch('example.urlopen', return_value=sample_data)
    def test_dowprices(self, mock_urlopen):
        p = example.dowprices()
        self.assertTrue(mock_urlopen.called)
        self.assertEqual(p,
                        {'IBM': 91.1,
                         'AA': 13.25,
                         'MSFT' : 27.72})
if __name__ == '__main__':
    unittest.main()
```
In this example, the `urlopen()` function in the example module is replaced with a mock object that returns a `BytesIO()` containing sample data as a substitute.An important but subtle facet of this test is the patching of __example.urlopen__ instead of `urllib.request.urlopen` . When you are making patches, you have to use the names as they are used in the code being tested. Since the example code uses `from urllib.request import urlopen` , the `urlopen()` function used by the __dowprices()__ function is actually located in example .

This recipe has really only given a very small taste of what’s possible with the `unittest.mock` module. The official documentation is a must-read for more advanced features.

## Testing for Exceptional Conditions in Unit Tests
##### Problem
You want to write a unit test that cleanly tests if an exception is raised.
##### Solution
To test for exceptions, use the `assertRaises()` method.

For example, if you want to test that a function raised a ValueError exception, use this code:
``` py
import unittest
# A simple function to illustrate
def parse_int(s):
    return int(s)
class TestConversion(unittest.TestCase):
    def test_bad_int(self):
        self.assertRaises(ValueError, parse_int, 'N/A')
```
If you need to test the exception’s value in some way, then a different approach is needed.

For example:
``` py
import errno
class TestIO(unittest.TestCase):
    def test_file_not_found(self):
        try:
            f = open('/file/not/found')
        except IOError as e:
            self.assertEqual(e.errno, errno.ENOENT)
        else:
            self.fail('IOError not raised')
```
##### Discussion
The __assertRaises()__ method provides a convenient way to test for the presence of an exception. A common pitfall is to write tests that manually try to do things with exceptions on their own.

For instance:
``` py
class TestConversion(unittest.TestCase):
    def test_bad_int(self):
        try:
            r = parse_int('N/A')
        except ValueError as e:
            self.assertEqual(type(e), ValueError)
```
The problem with such approaches is that it is easy to forget about corner cases, such as that when no exception is raised at all. To do that, you need to add an extra check for that situation, as shown here:
``` py
class TestConversion(unittest.TestCase):
    def test_bad_int(self):
        try:
            r = parse_int('N/A')
        except ValueError as e:
            self.assertEqual(type(e), ValueError)
        else:
            self.fail('ValueError not raised')
```
The __assertRaises()__ method simply takes care of these details, so you should prefer to use it.

The one limitation of assertRaises() is that it doesn’t provide a means for testing the value of the exception object that’s created. To do that, you have to manually test it, as shown. Somewhere in between these two extremes, you might consider using the as `sertRaisesRegex()` method, which allows you to test for an exception and perform a regular expression match against the exception’s string representation at the same time.

For example:
``` py
class TestConversion(unittest.TestCase):
    def test_bad_int(self):
        self.assertRaisesRegex(ValueError, 'invalid literal .*',
                               parse_int, 'N/A')
```

A little-known fact about `assertRaises()` and `assertRaisesRegex()` is that they can
also be used as context managers:
``` py
class TestConversion(unittest.TestCase):
    def test_bad_int(self):
        with self.assertRaisesRegex(ValueError, 'invalid literal .*'):
        r = parse_int('N/A')
```
This form can be useful if your test involves multiple steps (e.g., setup) besides that of simply executing a callable.

## Logging Test Output to a File
##### Problem
You want the results of running unit tests written to a file instead of printed to standard output.
##### Solution
A very common technique for running unit tests is to include a small code fragment like this at the bottom of your testing file:
``` py
import unittest
class MyTest(unittest.TestCase):
    ...
if __name__ == '__main__':
    unittest.main()
```
This makes the test file executable, and prints the results of running tests to standard output. If you would like to redirect this output, you need to unwind the `main()` call a bit and write your own `main()` function like this:
``` py
import sys
def main(out=sys.stderr, verbosity=2):
    loader = unittest.TestLoader()
    suite = loader.loadTestsFromModule(sys.modules[__name__])
    unittest.TextTestRunner(out,verbosity=verbosity).run(suite)
if __name__ == '__main__':
    with open('testing.out', 'w') as f:
        main(f)
```
##### Discussion
The interesting thing about this recipe is not so much the task of getting test results redirected to a file, but the fact that doing so exposes some notable inner workings of the unittest module.

At a basic level, the unittest module works by first assembling a test suite. This test suite consists of the different testing methods you defined. Once the suite has been assembled, the tests it contains are executed.

These two parts of unit testing are separate from each other. The `unittest.TestLoader` instance created in the solution is used to assemble a test suite. The `loadTestsFromModule()` is one of several methods it defines to gather tests. In this case, it scans a module for TestCase classes and extracts test methods from them. If you want something more fine-grained, the `loadTestsFromTestCase()` method (not shown) can be used to pull test methods from an individual class that inherits from TestCase .

The `TextTestRunner` class is an example of a test runner class. The main purpose of
this class is to execute the tests contained in a test suite. This class is the same test runner that sits behind the `unittest.main()` function. However, here we’re giving it a bit of low-level configuration, including an output file and an elevated verbosity level.

> Although this recipe only consists of a few lines of code, it gives a hint as to how you might further customize the unittest framework. To customize how test suites are assembled, you would perform various operations using the `TestLoader` class. To customize how tests execute, you could make custom test runner classes that emulate the functionality of `TextTestRunner` . Both topics are beyond the scope of what can be covered here. However, documentation for the unittest module has extensive coverage
of the underlying protocols.

## Skipping or Anticipating Test Failures
##### Problem
You want to skip or mark selected tests as an anticipated failure in your unit tests.
##### Solution
The `unittest` module has decorators that can be applied to selected test methods to control their handling.

For example:
``` py
import unittest
import os
import platform
class Tests(unittest.TestCase):
    def test_0(self):
        self.assertTrue(True)
    @unittest.skip('skipped test')
    def test_1(self):
        self.fail('should have failed!')
    @unittest.skipIf(os.name=='posix', 'Not supported on Unix')
    def test_2(self):
        import winreg
        @unittest.skipUnless(platform.system() == 'Darwin', 'Mac specific test')
    def test_3(self):
        self.assertTrue(True)
    @unittest.expectedFailure
    def test_4(self):
        self.assertEqual(2+2, 5)

if __name__ == '__main__':
    unittest.main()
```
If you run this code on a Mac, you’ll get this output:
```
python3 testsample.py -v
test_0 (__main__.Tests) ... ok
test_1 (__main__.Tests) ... skipped 'skipped test'
test_2 (__main__.Tests) ... skipped 'Not supported on Unix'
test_3 (__main__.Tests) ... ok
test_4 (__main__.Tests) ... expected failure

----------------------------------------------------------------------
Ran 5 tests in 0.002s

OK (skipped=2, expected failures=1)
```
##### Discussion
- The `skip()` decorator can be used to skip over a test that you don’t want to run at all.
- skipIf() and skipUnless() can be a useful way to write tests that only apply to certain platforms or Python versions, or which have other dependencies.

Use the @expected Failure decorator to mark tests that are known failures, but for which you don’t want the test framework to report more information.

The decorators for skipping methods can also be applied to entire testing classes.

For example:
``` PY
@unittest.skipUnless(platform.system() == 'Darwin', 'Mac specific tests')
class DarwinTests(unittest.TestCase):
    ...
```
## Handling Multiple Exceptions
##### Problem
You have a piece of code that can throw any of several different exceptions, and you need to account for all of the potential exceptions that could be raised without creating duplicate code or long, meandering code passages.
##### Solution
If you can handle different exceptions all using a single block of code, they can be grouped together in a tuple like this:
``` py
try:
    client_obj.get_url(url)
except (URLError, ValueError, SocketTimeout):
    client_obj.remove_url(url)
```
In the preceding example, the __remove_url()__ method will be called if any one of the listed exceptions occurs. If, on the other hand, you need to handle one of the exceptions differently, put it into its own except clause:
``` py
try:
    client_obj.get_url(url)
except (URLError, ValueError):
    client_obj.remove_url(url)
except SocketTimeout:
    client_obj.handle_url_timeout(url)
```
Many exceptions are grouped into an inheritance hierarchy. For such exceptions, you can catch all of them by simply specifying a base class. For example, instead of writing code like this:
``` py
try:
    f = open(filename)
except (FileNotFoundError, PermissionError):
...
```
you could rewrite the except statement as:
``` py
try:
    f = open(filename)
except OSError:
    ...
```
This works because __OSError__ is a base class that’s common to both the __FileNotFoundError__ and __PermissionError__ exceptions.
##### Discussion
Although it’s not specific to handling multiple exceptions per se, it’s worth noting that you can get a handle to the thrown exception using the as keyword:
``` py
try:
    f = open(filename)
except OSError as e:
    if e.errno == errno.ENOENT:
        logger.error('File not found')
    elif e.errno == errno.EACCES:
        logger.error('Permission denied')
    else:
        logger.error('Unexpected error: %d', e.errno)
```
In this example, the __e__ variable holds an instance of the raised __OSError__ . This is useful if you need to inspect the exception further, such as processing it based on the value of an additional status code.

Be aware that except clauses are checked in the order listed and that the first match executes. It may be a bit pathological, but you can easily create situations where multiple except clauses might match.

For example:
``` py
>>> f = open('missing')
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
FileNotFoundError: [Errno 2] No such file or directory: 'missing'
>>> try:
...     f = open('missing')
... except OSError:
...     print('It failed')
... except FileNotFoundError:
...     print('File not found')
...
It failed
>>>
```
Here the except __FileNotFoundError__ clause doesn’t execute because the __OSError__ is more general, matches the __FileNotFoundError__ exception, and was listed first.

As a debugging tip, if you’re not entirely sure about the class hierarchy of a particular exception, you can quickly view it by inspecting the exception’s __\_\_mro\_\___ attribute.

For example:
``` py
>>> FileNotFoundError.__mro__
(<class 'FileNotFoundError'>, <class 'OSError'>, <class 'Exception'>,
<class 'BaseException'>, <class 'object'>)
>>>
```
Any one of the listed classes up to BaseException can be used with the except statement.

## Catching All Exceptions
##### Problem
You want to write code that catches all exceptions.
##### Solution
To catch all exceptions, write an exception handler for Exception , as shown here:
``` py
try:
    ...
except Exception as e:
    ...
    log('Reason:', e)   # Important!
```
This will catch all exceptions save __SystemExit__ ,__KeyboardInterrupt__ , and __GeneratorExit__ . If you also want to catch those exceptions, change Exception to BaseException .

##### Discussion
Catching all exceptions is sometimes used as a crutch by programmers who can’t remember all of the possible exceptions that might occur in complicated operations. As such, it is also a very good way to write undebuggable code if you are not careful.Because of this, if you choose to catch all exceptions, it is absolutely critical to log or report the actual reason for the exception somewhere (e.g., log file, error message printed to screen, etc.). If you don’t do this, your head will likely explode at some point.

Consider this example:
``` py
def parse_int(s):
    try:
        n = int(v)
    except Exception:
        print("Couldn't parse")
```
If you try this function, it behaves like this:
``` py
>>> parse_int('n/a')
Couldn't parse
>>> parse_int('42')
Couldn't parse
>>>
```
At this point, you might be left scratching your head as to why it doesn’t work. Now
suppose the function had been written like this:
``` py
def parse_int(s):
    try:
        n = int(v)
    except Exception as e:
        print("Couldn't parse")
        print('Reason:', e)
```
In this case, you get the following output, which indicates that a programming mistake has been made:
``` py
>>> parse_int('42')
Couldn't parse
Reason: global name 'v' is not defined
>>>
```
?> All things being equal, it’s probably better to be as precise as possible in your exception handling. However, if you must catch all exceptions, just make sure you give good diagnostic information or propagate the exception so that cause doesn’t get lost.

## Creating Custom Exceptions
##### Problem
You’re building an application and would like to wrap lower-level exceptions with custom ones that have more meaning in the context of your application.
##### Solution
Creating new exceptions is easy—just define them as classes that inherit from Exception (or one of the other existing exception types if it makes more sense).

For example,if you are writing code related to network programming, you might define some custom exceptions like this:
``` py
class NetworkError(Exception):
    pass
class HostnameError(NetworkError):
    pass
class TimeoutError(NetworkError):
    pass
class ProtocolError(NetworkError):
    pass
```
Users could then use these exceptions in the normal way. For example:
``` py
try:
    msg = s.recv()
except TimeoutError as e:
    ...
except ProtocolError as e:
    ...
```
##### Discussion
Custom exception classes should almost always inherit from the built-in Exception class, or inherit from some locally defined base exception that itself inherits from Exception .

!> Although all exceptions also derive from __BaseException__ , you should not use this as a base class for new exceptions. __BaseException__ is reserved for system-exiting exceptions, such as __KeyboardInterrupt__ or __SystemExit__ , and other exceptions that should signal the application to exit. Therefore, catching these exceptions is not theintended use case. Assuming you follow this convention, it follows that inheriting from BaseException causes your custom exceptions to not be caught and to signal an imminent application shutdown!

Having custom exceptions in your application and using them as shown makes your application code tell a more coherent story to whoever may need to read the code. One design consideration involves the grouping of custom exceptions via inheritance. In complicated applications, it may make sense to introduce further base classes that group different classes of exceptions together. This gives the user a choice of catching a narrowly specified error, such as this:
``` py
try:
    s.send(msg)
except ProtocolError:
    ...
```
It also gives the ability to catch a broad range of errors, such as the following:
``` py
try:
    s.send(msg)
except NetworkError:
    ...
```
If you are going to define a new exception that overrides the __\_\_init\_\_()__ method of Exception , make sure you always call __Exception.\_\_init\_\_()__ with all of the passed arguments.

For example:
``` py
class CustomError(Exception):
    def __init__(self, message, status):
        super().__init__(message, status)
        self.message = message
        self.status = status
```
This might look a little weird, but the default behavior of Exception is to accept all arguments passed and to store them in the `.args` attribute as a tuple. Various other libraries and parts of Python expect all exceptions to have the `.args` attribute, so if you skip this step, you might find that your new exception doesn’t behave quite right in certain contexts. 

To illustrate the use of `.args` , consider this interactive session with the built-in __RuntimeError__ exception, and notice how any number of arguments can be used with the raise statement:
``` py
>>> try:
...     raise RuntimeError('It failed')
... except RuntimeError as e:
...     print(e.args)
...
('It failed',)
>>> try:
...     raise RuntimeError('It failed', 42, 'spam')
... except RuntimeError as e:
...     print(e.args)
...
('It failed', 42, 'spam')
>>>
```
For more information on creating your own exceptions, see the Python documentation.
## Raising an Exception in Response to Another Exception
##### Problem
You want to raise an exception in response to catching a different exception, but want to include information about both exceptions in the traceback.
##### Solution
To chain exceptions, use the raise from statement instead of a simple raise statement.This will give you information about both errors.

For example:
``` py
>>> def example():
...     try:
...         int('N/A')
...     except ValueError as e:
...         raise RuntimeError('A parsing error occurred') from e...
>>>
example()
Traceback (most recent call last):
    File "<stdin>", line 3, in example
ValueError: invalid literal for int() with base 10: 'N/A'
```
The above exception was the direct cause of the following exception:
``` py
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "<stdin>", line 5, in example
RuntimeError: A parsing error occurred
>>>
```
As you can see in the traceback, both exceptions are captured. To catch such an exception, you would use a normal except statement. However, you can look at the __\_\_cause\_\___ attribute of the exception object to follow the exception chain should you wish.

For example:
``` py
try:
    example()
except RuntimeError as e:
    print("It didn't work:", e)
    if e.__cause__:
        print('Cause:', e.__cause__)
```
An implicit form of chained exceptions occurs when another exception gets raised in‐
side an except block.

For example:
``` py
>>> def example2():
...     try:
...         int('N/A')
...     except ValueError as e:
...         print("Couldn't parse:", err)
...
>>>
>>> example2()
Traceback (most recent call last):
    File "<stdin>", line 3, in example2
ValueError: invalid literal for int() with base 10: 'N/A'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "<stdin>", line 5, in example2
NameError: global name 'err' is not defined
>>>
```
In this example, you get information about both exceptions, but the interpretation is a bit different. In this case, the NameError exception is raised as the result of a programming error, not in direct response to the parsing error. For this case, the __\_\_cause\_\___ attribute of an exception is not set. Instead, a __\_\_context\_\___ attribute is set to the prior exception.

If, for some reason, you want to suppress chaining, use raise from None :
``` py
>>> def example3():
...     try:
...         int('N/A')
...     except ValueError:
...         raise RuntimeError('A parsing error occurred') from None...
>>>
example3()
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "<stdin>", line 5, in example3
RuntimeError: A parsing error occurred
>>>
```
##### Discussion
In designing code, you should give careful attention to use of the raise statement inside of other except blocks. In most cases, such raise statements should probably be changed to raise from statements. That is, you should prefer this style:
``` py
try:
    ...
except SomeException as e:
    raise DifferentException() from e
```
The reason for doing this is that you are explicitly chaining the causes together. That is, the DifferentException is being raised in direct response to getting a __SomeException__ . This relationship will be explicitly stated in the resulting traceback.If you write your code in the following style, you still get a chained exception, but it’s often not clear if the exception chain was intentional or the result of an unforeseen programming error:
``` py
try:
    ...
except SomeException:
    raise DifferentException()
```
When you use raise from , you’re making it clear that you meant to raise the second exception.

Resist the urge to suppress exception information, as shown in the last example. Although suppressing exception information can lead to smaller tracebacks, it also discards information that might be useful for debugging. All things being equal, it’s often best to keep as much information as possible.

## Reraising the Last Exception
##### Problem
You caught an exception in an except block, but now you want to reraise it.
##### Solution
Simply use the raise statement all by itself.

For example:
``` py
>>> def example():
...     try:
...         int('N/A')
...     except ValueError:
...         print("Didn't work")
...         raise
...
>>> example()
Didn't work
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "<stdin>", line 3, in example
ValueError: invalid literal for int() with base 10: 'N/A'
>>>
```
##### Discussion
This problem typically arises when you need to take some kind of action in response to an exception (e.g., logging, cleanup, etc.), but afterward, you simply want to propagate the exception along. A very common use might be in catch-all exception handlers:
``` py
try:
    ...
except Exception as e:
    # Process exception information in some way
    ...
    # Propagate the exception
    raise
```
## Issuing Warning Messages
##### Problem
You want to have your program issue warning messages (e.g., about deprecated features or usage problems).
##### Solution
To have your program issue a warning message, use the `warnings.warn()` function.

For example:
``` py
import warnings
def func(x, y, logfile=None, debug=False):
    if logfile is not None:
        warnings.warn('logfile argument deprecated', DeprecationWarning)
    ...
```
The arguments to `warn()` are a warning message along with a warning class, which is typically one of the following: __UserWarning__ , __DeprecationWarning__ , __SyntaxWarning__ , __RuntimeWarning__ , __ResourceWarning__ , or __FutureWarning__ .

The handling of warnings depends on how you have executed the interpreter and other configuration. For example, if you run Python with the `-W` all option, you’ll get output such as the following:
``` bash
python3 -W all example.py
example.py:5: DeprecationWarning: logfile argument is deprecated
    warnings.warn('logfile argument is deprecated', DeprecationWarning)
```
Normally, warnings just produce output messages on standard error. If you want to turn
warnings into exceptions, use the `-W` error option:
``` bash
python3 -W error example.py
Traceback (most recent call last):
    File "example.py", line 10, in <module>
        func(2, 3, logfile='log.txt')
    File "example.py", line 5, in func
        warnings.warn('logfile argument is deprecated', DeprecationWarning)
DeprecationWarning: logfile argument is deprecated
```
##### Discussion
Issuing a warning message is often a useful technique for maintaining software and assisting users with issues that don’t necessarily rise to the level of being a full-fledged exception. For example, if you’re going to change the behavior of a library or framework, you can start issuing warning messages for the parts that you’re going to change while still providing backward compatibility for a time. You can also warn users about problematic usage issues in their code.

As another example of a warning in the built-in library, here is an example of a warning message generated by destroying a file without closing it:
``` py
>>> import warnings
>>> warnings.simplefilter('always')
>>> f = open('/etc/passwd')
>>> del f
__main__:1: ResourceWarning: unclosed file <_io.TextIOWrapper name='/etc/passwd'
mode='r' encoding='UTF-8'>
>>>
```
By default, not all warning messages appear. The `-W` option to Python can control the output of warning messages. `-W all` will output all warning messages, `-W` ignore ignores all warnings, and `-W error` turns warnings into exceptions. As an alternative,

you can can use the `warnings.simplefilter()` function to control output, as just shown. An argument of `always` makes all warning messages appear, `ignore` ignores all
warnings, and `error` turns warnings into exceptions.

For simple cases, this is all you really need to issue warning messages. The `warnings` module provides a variety of more advanced configuration options related to the filtering and handling of warning messages.

See the Python documentation for more information.

## Debugging Basic Program Crashes
##### Problem
Your program is broken and you’d like some simple strategies for debugging it.
##### Solution
If your program is crashing with an exception, running your program as `python3 -i someprogram.py` can be a useful tool for simply looking around. The `-i` option starts an interactive shell as soon as a program terminates. From there, you can explore the environment. For example, suppose you have this code:
``` py
# sample.py
def func(n):
    return n + 10

func('Hello')
```
Running `python3 -i` produces the following:
``` bash
python3 -i sample.py
Traceback (most recent call last):
    File "sample.py", line 6, in <module>
        func('Hello')
    File "sample.py", line 4, in func
        return n + 10
TypeError: Can't convert 'int' object to str implicitly
>>> func(10)
20
>>>
```
If you don’t see anything obvious, a further step is to launch the Python debugger after a crash.

For example:
``` py
>>> import pdb
>>> pdb.pm()
> sample.py(4)func()
-> return n + 10
(Pdb) w
sample.py(6)<module>()
-> func('Hello')
> sample.py(4)func()
-> return n + 10
(Pdb) print n
'Hello'
(Pdb) q
>>>
```

If your code is deeply buried in an environment where it is difficult to obtain an interactive shell (e.g., in a server), you can often catch errors and produce tracebacks yourself.

For example:
``` py
import traceback
import sys
try:
    func(arg)
except:
    print('**** AN ERROR OCCURRED ****')
    traceback.print_exc(file=sys.stderr)
```
If your program isn’t crashing, but it’s producing wrong answers or you’re mystified by how it works, there is often nothing wrong with just injecting a few `print()` calls in places of interest. However, if you’re going to do that, there are a few related techniques of interest. First, the `traceback.print_stack()` function will create a stack track of your program immediately at that point.

For example:
``` py
>>> def sample(n):
...     if n > 0:
...         sample(n-1)
...     else:
...         traceback.print_stack(file=sys.stderr)
...
>>> sample(5)
File "<stdin>", line 1, in <module>
File "<stdin>", line 3, in sample
File "<stdin>", line 3, in sample
File "<stdin>", line 3, in sample
File "<stdin>", line 3, in sample
File "<stdin>", line 3, in sample
File "<stdin>", line 5, in sample
>>>
```
Alternatively, you can also manually launch the debugger at any point in your program using `pdb.set_trace()` like this:
``` py
import pdb
def func(arg):
    ...
    pdb.set_trace()
    ...
```
This can be a useful technique for poking around in the internals of a large program and answering questions about the control flow or arguments to functions. For instance, once the debugger starts, you can inspect variables using print or type a command such as __w__ to get the stack traceback.
##### Discussion
Don’t make debugging more complicated than it needs to be. Simple errors can often be resolved by merely knowing how to read program tracebacks (e.g., the actual error is usually the last line of the traceback). Inserting a few selected `print()` functions in your code can also work well if you’re in the process of developing it and you simply want some diagnostics (just remember to remove the statements later).

A common use of the debugger is to inspect variables inside a function that has crashed. Knowing how to enter the debugger after such a crash has occurred is a useful skill to know.Inserting statements such as `pdb.set_trace()` can be useful if you’re trying to unravel an extremely complicated program where the underlying control flow isn’t obvious. Essentially, the program will run until it hits the `set_trace()` call, at which point it will immediately enter the debugger. From there, you can try to make more sense of it. If you’re using an IDE for Python development, the IDE will typically provide its own debugging interface on top of or in place of pdb . Consult the manual for your IDE for more information.

## Profiling and Timing Your Program
##### Problem
You would like to find out where your program spends its time and make timing measurements.
##### Solution
If you simply want to time your whole program, it’s usually easy enough to use something like the Unix time command.

For example:
``` bash 
bash time python3 someprogram.py
real 0m13.937s
user 0m12.162s
sys 0m0.098s
```

On the other extreme, if you want a detailed report showing what your program is doing,you can use the `cProfile` module:
``` bash 
bash python3 -m cProfile someprogram.py
    859647 function calls in 16.016 CPU seconds

Ordered by: standard name

ncalls      tottime     percall     cumtime     percall     filename:lineno(function)
263169      0.080       0.000       0.080       0.000       someprogram.py:16(frange)
513         0.001       0.000       0.002       0.000       someprogram.py:30(generate_mandel)
262656      0.194       0.000       15.295      0.000       someprogram.py:32(<genexpr>)
1           0.036       0.036       16.077      16.077      someprogram.py:4(<module>)
262144      15.021      0.000       15.021      0.000       someprogram.py:4(in_mandelbrot)
1           0.000       0.000       0.000       0.000       os.py:746(urandom)
1           0.000       0.000       0.000       0.000       png.py:1056(_readable)
1           0.000       0.000       0.000       0.000       png.py:1073(Reader)
1           0.227       0.227       0.438       0.438       png.py:163(<module>)
512         0.010       0.000       0.010       0.000       png.py:200(group)
...
```
More often than not, profiling your code lies somewhere in between these two extremes.For example, you may already know that your code spends most of its time in a few selected functions. For selected profiling of functions, a short decorator can be useful.

For example:
``` py
# timethis.py
import time
from functools import wraps

def timethis(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        r = func(*args, **kwargs)
        end = time.perf_counter()
        print('{}.{} : {}'.format(func.__module__, func.__name__, end - start))
        return r
    return wrapper
```
To use this decorator, you simply place it in front of a function definition to get timings
from it.

For example:
``` py
>>> @timethis
... def countdown(n):
...     while n > 0:
...         n -= 1
...
>>> countdown(10000000)
__main__.countdown : 0.803001880645752
>>>
```
To time a block of statements, you can define a context manager.

For example:
``` py
from contextlib import contextmanager

@contextmanager
def timeblock(label):
    start = time.perf_counter()
    try:
        yield
    finally:
        end = time.perf_counter()
        print('{} : {}'.format(label, end - start))
```
Here is an example of how the context manager works:
``` py
>>> with timeblock('counting'):
...     n = 10000000
...     while n > 0:
...         n -= 1
...
counting : 1.5551159381866455
>>>
```
For studying the performance of small code fragments, the `timeit` module can be useful.

For example:
``` py
>>> from timeit import timeit
>>> timeit('math.sqrt(2)', 'import math')
0.1432319980012835
>>> timeit('sqrt(2)', 'from math import sqrt')
0.10836604500218527
>>>
```
timeit works by executing the statement specified in the first argument a million times and measuring the time. The second argument is a setup string that is executed to set up the environment prior to running the test. If you need to change the number of iterations, supply a number argument like this:
``` py
>>> timeit('math.sqrt(2)', 'import math', number=10000000)
1.434852126003534
>>> timeit('sqrt(2)', 'from math import sqrt', number=10000000)
1.0270336690009572
>>>
```
##### Discussion
When making performance measurements, be aware that any results you get are approximations. The `time.perf_counter()` function used in the solution provides the highest-resolution timer possible on a given platform. However, it still measures wallclock time, and can be impacted by many different factors, such as machine load.

If you are interested in process time as opposed to wall-clock time, use `time.process_time()` instead.

For example:
``` py
from functools import wraps
def timethis(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.process_time()
        r = func(*args, **kwargs)
        end = time.process_time()
        print('{}.{} : {}'.format(func.__module__, func.__name__, end - start))
        return r
    return wrapper
```
Last, but not least, if you’re going to perform detailed timing analysis, make sure to read the documentation for the `time` , `timeit` , and other associated modules, so that you have
an understanding of important platform-related differences and other pitfalls.See Recipe 13.13 for a related recipe on creating a `stopwatch` timer class.

## Making Your Programs Run Faster
##### Problem
Your program runs too slow and you’d like to speed it up without the assistance of more extreme solutions, such as __C extensions__ or a __just-in-time (JIT)__ compiler.
##### Solution
While the first rule of optimization might be to “not do it,” the second rule is almost certainly “don’t optimize the unimportant.” To that end, if your program is running slow, you might start by profiling your code as discussed in previous recipe. More often than not, you’ll find that your program spends its time in a few hotspots,such as inner data processing loops. Once you’ve identified those locations, you can use the no-nonsense techniques presented in the following sections to make your program run faster.
#### Use functions 
A lot of programmers start using Python as a language for writing simple scripts. When writing scripts, it is easy to fall into a practice of simply writing code with very little structure.

For example:
``` py
# somescript.py
import sys
import csv
with open(sys.argv[1]) as f:
    for row in csv.reader(f):
        # Some kind of processing
        ...
```
A little-known fact is that code defined in the global scope like this runs slower than code defined in a function. The speed difference has to do with the implementation of local versus global variables (operations involving locals are faster). So, if you want to make the program run faster, simply put the scripting statements in a function:
``` PY
# somescript.py
import sys
import csv
def main(filename):
    with open(filename) as f:
        for row in csv.reader(f):
            # Some kind of processing
...
main(sys.argv[1])
```
The speed difference depends heavily on the processing being performed, but in our experience, speedups of 15-30% are not uncommon.

#### Selectively eliminate attribute access

Every use of the dot (.) operator to access attributes comes with a cost. Under the covers,this triggers special methods, such as __\_\_getattribute\_\_()__ and __\_\_getattr\_\_()__ , which often lead to dictionary lookups. You can often avoid attribute lookups by using the `from module import name` form of import as well as making selected use of bound methods.

To illustrate, consider the following code fragment:
``` py
import math
def compute_roots(nums):
    result = []
    for n in nums:
        result.append(math.sqrt(n))
    return result
# Test
nums = range(1000000)
for n in range(100):
    r = compute_roots(nums)
```
When tested on our machine, this program runs in about 40 seconds. Now change the __compute_roots()__ function as follows:
``` py
from math import sqrt
def compute_roots(nums):
result = []
result_append = result.append
for n in nums:
    result_append(sqrt(n))
return result
```
This version runs in about 29 seconds. The only difference between the two versions of code is the elimination of attribute access. Instead of using `math.sqrt()` , the code uses `sqrt()` . The `result.append()` method is additionally placed into a local variable __result_append__ and reused in the inner loop. However, it must be emphasized that these changes only make sense in frequently executed code, such as loops. So, this optimization really only makes sense in carefully selected places.

#### Understand locality of variables
As previously noted, local variables are faster than global variables. For frequently accessed names, speedups can be obtained by making those names as local as possible.

For example, consider this modified version of the __compute_roots()__ function just discussed:
``` py
import math
def compute_roots(nums):
    sqrt = math.sqrt
    result = []
    result_append = result.append
    for n in nums:
        result_append(sqrt(n))
    return result
```
In this version, sqrt has been lifted from the math module and placed into a local variable. If you run this code, it now runs in about 25 seconds (an improvement over the previous version, which took 29 seconds). That additional speedup is due to a local lookup of sqrt being a bit faster than a global lookup of sqrt .

Locality arguments also apply when working in classes. In general, looking up a value such as __self.name__ will be considerably slower than accessing a local variable. In inner loops, it might pay to lift commonly accessed attributes into a local variable.

For example:
``` py
# Slower
class SomeClass:
    ...
    def method(self):
        for x in s:
            op(self.value)
# Faster
class SomeClass:
    ...
    def method(self):
        value = self.value
        for x in s:
            op(value)
```
#### Avoid gratuitous abstraction
Any time you wrap up code with extra layers of processing, such as decorators, properties, or descriptors, you’re going to make it slower. As an example, consider this class:
``` py
class A:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    @property
    def y(self):
        return self._y
    @y.setter
    def y(self, value):
        self._y = value
```
Now, try a simple timing test:
``` py
>>> from timeit import timeit
>>> a = A(1,2)
>>> timeit('a.x', 'from __main__ import a')
0.07817923510447145
>>> timeit('a.y', 'from __main__ import a')
0.35766440676525235
>>>
```
As you can observe, accessing the property y is not just slightly slower than a simple attribute x , it’s about 4.5 times slower. If this difference matters, you should ask yourself if the definition of y as a property was really necessary. If not, simply get rid of it and go back to using a simple attribute instead. Just because it might be common for programs in another programming language to use getter/setter functions, that doesn’t mean you should adopt that programming style for Python.

#### Use the built-in containers
Built-in data types such as strings, tuples, lists, sets, and dicts are all implemented in C, and are rather fast. If you’re inclined to make your own data structures as a replacement (e.g., linked lists, balanced trees, etc.), it may be rather difficult if not impossible to match the speed of the built-ins. Thus, you’re often better off just using them.

#### Avoid making unnecessary data structures or copies
Sometimes programmers get carried away with making unnecessary data structures when they just don’t have to. For example, someone might write code like this:
``` py
values = [x for x in sequence]
squares = [x*x for x in values]
```
Perhaps the thinking here is to first collect a bunch of values into a list and then to start applying operations such as list comprehensions to it. However, the first list is completely unnecessary. Simply write the code like this:
``` py
squares = [x*x for x in sequence]
```
Related to this, be on the lookout for code written by programmers who are overly paranoid about Python’s sharing of values. Overuse of functions such as `copy.deepcopy()` may be a sign of code that’s been written by someone who doesn’t fully understand or trust Python’s memory model. In such code, it may be safe to eliminate many of the copies.
##### Discussion
Before optimizing, it’s usually worthwhile to study the algorithms that you’re using first.You’ll get a much bigger speedup by switching to an O(n log n) algorithm than by trying to tweak the implementation of an an O(n**2) algorithm.

If you’ve decided that you still must optimize, it pays to consider the big picture. As a general rule, you don’t want to apply optimizations to every part of your program, because such changes are going to make the code hard to read and understand. Instead, focus only on known performance bottlenecks, such as inner loops. You need to be especially wary interpreting the results of micro-optimizations. For example, consider these two techniques for creating a dictionary:
``` py
a = {
    'name' : 'AAPL',
    'shares' : 100,
    'price' : 534.22
}
b = dict(name='AAPL', shares=100, price=534.22)
```
The latter choice has the benefit of less typing (you don’t need to quote the key names).However, if you put the two code fragments in a head-to-head performance battle, you’ll find that using `dict()` runs three times slower! With this knowledge, you might be inclined to scan your code and replace every use of `dict()` with its more verbose alternative. However, a smart programmer will only focus on parts of a program where it might actually matter, such as an inner loop. In other places, the speed difference just isn’t going to matter at all. 

If, on the other hand, your performance needs go far beyond the simple techniques in this recipe, you might investigate the use of tools based on just-in-time (JIT) compilation techniques. For example, the `PyPy` project is an alternate implementation of the Python interpreter that analyzes the execution of your program and generates native machine code for frequently executed parts. It can sometimes make Python programs run an order of magnitude faster, often approaching (or even exceeding) the speed of code written in C.You might also consider the `Numba` project. `Numba` is a dynamic compiler where you annotate selected Python functions that you want to optimize with a decorator. Those functions are then compiled into native machine code through the use of __LLVM__. It too can produce signficant performance gains.

> Last, but not least, the words of John Ousterhout come to mind: “The best performance improvement is the transition from the nonworking to the working state.” Don’t worry about optimization until you need to. Making sure your program works correctly is usually more important than making it run fast (at least initially).