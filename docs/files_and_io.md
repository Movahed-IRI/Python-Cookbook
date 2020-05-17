## Reading and Writing Text Data
##### Problem
You need to read or write text data, possibly in different text encodings such as __ASCII__, __UTF-8__, or __UTF-16__.
##### Solution
Use the `open()` function with mode `rt` to __read a text file__. For example:

``` py
# Read the entire file as a single string
with open('somefile.txt', 'rt') as f:
    data = f.read()

# Iterate over the lines of the file
with open('somefile.txt', 'rt') as f: 
    for line in f: # process line
    
...
```
Similarly, to write a text file, use `open()` with mode `wt` to __write a file__ .

clearing and overwriting the previous contents (if any). For example:
``` py
# Write chunks of text data
with open('somefile.txt', 'wt') as f:
    f.write(text1)
    f.write(text2)
...
# Redirected print statement
with open('somefile.txt', 'wt') as f:
    print(line1, file=f)
    print(line2, file=f)
...
```

?> To append to the end of an existing file, use `open()` with mode `at` .

?> By default, files are read/written using the system default text encoding, as can be found in `sys.getdefaultencoding()` . 

supply the optional encoding parameter to `open()` . For example:

``` py
with open('somefile.txt', 'rt', encoding='latin-1') as f:
...
```
- some of the more common encodings are :
    - ascii , __corresponds to the 7-bit characters in the range U+0000 to U+007F__
    - latin-1 , __direct mapping of bytes 0-255 to Unicode characters U+0000 to U+00FF__
    - utf-8 ,   __usually a safe bet if working with web applications__
    - utf-16 . 
    
?> latin-1 encoding is notable in that it will never produce a decoding error when reading text of a possibly unknown encoding. Reading a file as latin-1 might not produce a completely correct text decoding.

##### Discussion
- There are a number of subtle aspects to keep in mind:
    - The use of the with statement in the examples establishes a context in which the file will be used. When control leaves the with block, the file will be closed automatically. You don’t need to use the with statement, but if you don’t use it, make sure you remember to close the file:

    ``` py
    f = open('somefile.txt', 'rt')
    data = f.read()
    f.close()
    ```
    - Another minor complication concerns the recognition of newlines, which are different on Unix and Windows (i.e., \n versus \r\n ). By default, Python operates in what’s known as __“universal newline”__ mode. In this mode, all common newline conventions are recognized, and newline characters are converted to a single \n character while reading.
    - the newline character \n is converted to the system default newline character on output. If you don’t want this translation, supply the `newline=''` argument to
    `open()` , like this:
    ``` py
    # Read with disabled newline translation
    with open('somefile.txt', 'rt', newline='') as f:
    ...
    ```
    To illustrate the difference, here’s what you will see on a Unix machine if you read the contents of a Windows-encoded text file containing the raw data hello world!\r\n:

    ``` py
    >>> # Newline translation enabled (the default)
    >>> f = open('hello.txt', 'rt')
    >>> f.read()
    'hello world!\n'
    >>> # Newline translation disabled
    >>> g = open('hello.txt', 'rt', newline='')
    >>> g.read()
    'hello world!\r\n'
    >>>
    ```

    - A final issue concerns possible encoding errors in text files. When reading or writing a text file, you might encounter an encoding or decoding error. For instance:
    ``` py
    >>> f = open('sample.txt', 'rt', encoding='ascii')
    >>> f.read()
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
            File "/usr/local/lib/python3.3/encodings/ascii.py", line 26, in decode
                return codecs.ascii_decode(input, self.errors)[0]
    UnicodeDecodeError: 'ascii' codec can't decode byte 0xc3 in position 12: ordinal not in range(128)
    >>>
    ```
    - If encoding errors are still a possibility, you can supply an optional errors argument to `open()` to deal with the errors. Here are a few samples of
    common error handling schemes:
    ``` py
    >>> # Replace bad chars with Unicode U+fffd replacement char
    >>> f = open('sample.txt', 'rt', encoding='ascii', errors='replace')
    >>> f.read()
    'Spicy Jalape?o!'
    >>> # Ignore bad chars entirely
    >>> g = open('sample.txt', 'rt', encoding='ascii', errors='ignore')
    >>> g.read()
    'Spicy Jalapeo!'
    >>>
    ```
    
    ?> The number one rule with text is that you simply need to make sure you’re always using the proper text encoding. When in doubt, use the default setting (typically UTF-8).

## Printing to a File
##### Problem
You want to redirect the output of the `print()` function to a file.
##### Solution
Use the `file` keyword argument to `print()` , like this:
``` py
with open('somefile.txt', 'rt') as f:
    print('Hello World!', file=f)
```
##### Discussion
make sure that the file is opened in text mode. Printing will fail if the underlying file is in binary mode.

## Printing with a Different Separator or Line Ending
##### Problem
You want to output data using `print()` , but you also want to change the separator character or line ending.
##### Solution
Use the `sep` and `end` keyword arguments to `print()` to change the output as you wish.
<br>
For example:

``` py
>>> print('ACME', 50, 91.5)
ACME 50 91.5
>>> print('ACME', 50, 91.5, sep=',')
ACME,50,91.5
>>> print('ACME', 50, 91.5, sep=',', end='!!\n')
ACME,50,91.5!!
>>>
```
For Example :
``` py
>>> for i in range(5):
...     print(i)
...
0
1
2
3
4
>>> for i in range(5):
...     print(i, end=' ')
...
0 1 2 3 4 >>>
```
##### Discussion
Sometimes you’ll see programmers using `str.join()` to accomplish the same thing.
<br>
For example:

``` py
>>> print(','.join('ACME','50','91.5'))
ACME,50,91.5
>>>
```

The problem with `str.join()` is that it only works with strings.
<br>
For example:

``` py
>>> row = ('ACME', 50, 91.5)
>>> print(','.join(row))
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
TypeError: sequence item 1: expected str instance, int found
>>> print(','.join(str(x) for x in row))
ACME,50,91.5
>>>
```
Instead of doing that, you could just write the following:

``` py
>>> print(*row, sep=',')
ACME,50,91.5
>>>
```

## Reading and Writing Binary Data
##### Problem
You need to read or write binary data, such as that found in images, sound files, and so on.
##### Solution
Use the `open()` function with mode `rb` or `wb` to read or write binary data.
<br>
For example:

``` py
# Read the entire file as a single byte string
with open('somefile.bin', 'rb') as f:
    data = f.read()
# Write binary data to a file
with open('somefile.bin', 'wb') as f:
    f.write(b'Hello World')
```
!> When reading binary, it is important to stress that all data returned will be in the form of byte strings, not text strings.Similarly, when writing, you must supply data in the form of objects that expose data as bytes (e.g., byte strings, bytearray objects, etc.).

##### Discussion
In particular, be aware that indexing and iteration return integer byte values instead of byte strings. For example:
``` py
>>> t = 'Hello World'
>>> # Text string
>>> t[0]
'H'
>>> for c in t:
...     print(c)
...
H
e
l
l
o
...
>>> # Byte string
>>> b = b'Hello World'
>>> b[0]
72
>>> for c in b:
...     print(c)
...
72
101
108
108
111
>>>
```

!> If you ever need to read or write text from a binary-mode file, make sure you remember to decode or encode it. 
<br>
For example:

``` py
with open('somefile.bin', 'rb') as f:
    data = f.read(16)
    text = data.decode('utf-8')
with open('somefile.bin', 'wb') as f:
    text = 'Hello World'
    f.write(text.encode('utf-8'))
```
A lesser-known aspect of binary I/O is that objects such as __arrays__ and __C structures__ can be used for writing without any kind of intermediate conversion to a bytes object. 
<br>
For example:

``` py
import array
nums = array.array('i', [1, 2, 3, 4])
with open('data.bin','wb') as f:
    f.write(nums)
```

This applies to any object that implements the so-called __“buffer interface,”__ which directly exposes an underlying memory buffer to operations that can work with it. Writing binary data is one such operation.

Many objects also allow binary data to be directly read into their underlying memory using the `readinto()` method of files. For example:
``` py
>>> import array
>>> a = array.array('i', [0, 0, 0, 0, 0, 0, 0, 0])
>>> with open('data.bin', 'rb') as f:
...     f.readinto(a)
...
16
>>> a
array('i', [1, 2, 3, 4, 0, 0, 0, 0])
>>>
```

However, great care should be taken when using this technique, as it is often platform specific and may depend on such things as the word size and byte ordering (i.e., big endian versus little endian).


## Writing to a File That Doesn’t Already Exist
##### Problem
You want to write data to a file, but only if it doesn’t already exist on the filesystem.

##### Solution
This problem is easily solved by using the little-known `x` mode to `open()` instead of the usual `w` mode. For example:
``` py
>>> with open('somefile', 'wt') as f:
...     f.write('Hello\n')
...
>>> with open('somefile', 'xt') as f:
...     f.write('Hello\n')
...
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
FileExistsError: [Errno 17] File exists: 'somefile'
>>>
```

?> If the file is binary mode, use mode `xb` instead of `xt` .

##### Discussion
An alternative solution is to first test for the file like this:
``` py
import os
if not os.path.exists('somefile'):
... with open('somefile', 'wt') as f:
...     f.write('Hello\n')
... else:
...     print('File already exists!')
...
File already exists!
>>>
```

!> It is important to note that the `x` mode is a Python 3 specific extension to the `open()` function.

##  Performing I/O Operations on a String
##### Problem
You want to feed a text or binary string to code that’s been written to operate on file like objects instead.
##### Solution
Use the `io.StringIO()` and `io.BytesIO()` classes to create file-like objects that operate on string data.
For example:
``` py
>>> s = io.StringIO()
>>> s.write('Hello World\n')
12
>>> print('This is a test', file=s)
15
>>> # Get all of the data written so far
>>> s.getvalue()
'Hello World\nThis is a test\n'
>>> # Wrap a file interface around an existing string
>>> s = io.StringIO('Hello\nWorld\n')
>>> s.read(4)
'Hell'
>>> s.read()
'o\nWorld\n'
>>>
```
The `io.StringIO` class should only be used for text.
<br>

If you are operating with binary data, use the `io.BytesIO` class instead. For example:

``` py
>>> s = io.BytesIO()
>>> s.write(b'binary data')
>>> s.getvalue()
b'binary data'
>>>
```
##### Discussion
The `StringIO` and `BytesIO` classes are most useful in scenarios where you need to mimic a normal file for some reason. 

!> Be aware that `StringIO` and `BytesIO` instances don’t have a proper integer file descriptor. Thus, they do not work with code that requires the use of a real system-level file such as a file, pipe, or socket.

## Reading and Writing Compressed Datafiles
##### Problem
You need to read or write data in a file with __gzip__ or __bz2__ compression.
##### Solution
The `gzip` and `bz2` modules make it easy to work with such files. Both modules provide an alternative implementation of `open()` that can be used for this purpose.
<br>
For example :

``` py
# gzip compression
import gzip
with gzip.open('somefile.gz', 'rt') as f:
    text = f.read()
# bz2 compression
import bz2
with bz2.open('somefile.bz2', 'rt') as f:
    text = f.read()
```
Similarly, to write compressed data, do this:
``` py
# gzip compression
import gzip
with gzip.open('somefile.gz', 'wt') as f:
    f.write(text)
# bz2 compression
import bz2
with bz2.open('somefile.bz2', 'wt') as f:
    f.write(text)
```
all I/O will use text and perform Unicode encoding/decoding. If you want to work with binary data instead, use a file mode of `rb` or `wb` .
##### Discussion

!> Be aware that choosing the correct file mode is critically important.

!> If you don’t specify a mode, the default mode is __binary__, which will break programs that expect to receive text.

?> Both `gzip.open()` and `bz2.open()` accept the same parameters as the built-in `open()` function, including encoding , errors , newline , and so forth.

?> When writing compressed data, the compression level can be optionally specified using the `compresslevel` keyword argument. For example:
``` py
with gzip.open('somefile.gz', 'wt', compresslevel=5) as f:
    f.write(text)
```

?> The default level is 9, which provides the highest level of compression. 

?> Lower levels offer better performance, but not as much compression.

?> a little-known feature of `gzip.open()` and `bz2.open()` is that they can be layered on top of an existing file opened in binary mode.This allows the `gzip` and `bz2` modules to work with various file-like objects such as __sockets__, __pipes__, and __in-memory files__.

For example, this works:
``` py
import gzip
f = open('somefile.gz', 'rb')
with gzip.open(f, 'rt') as g:
    text = g.read()
```

## Iterating Over Fixed-Sized Records
##### Problem
Instead of iterating over a file by lines, you want to iterate over a collection of fixed sized records or chunks.
##### Solution
Use the `iter()` function and `functools.partial()` using this neat trick:
``` py
from functools import partial
RECORD_SIZE = 32
with open('somefile.data', 'rb') as f:
    records = iter(partial(f.read, RECORD_SIZE), b'')
    for r in records:
...
```
!> The last item may have fewer bytes than expected if the file size is not an exact multiple of the record size.

##### Discussion

?> A little-known feature of the `iter()` function is that it can create an iterator if you pass it a callable and a sentinel value. The resulting iterator simply calls the supplied callable over and over again until it returns the sentinel, at which point iteration stops.

-   In the solution:,
    - The `functools.partial` is used to create a callable that reads a fixed number of bytes from a file each time it’s called.
    - The sentinel of `b''` is what gets returned when a file is read but the end of file has been reached.

Last, but not least, the solution shows the file being opened in binary mode. For reading fixed-sized records, this would probably be the most common case.
<br>
For text files, reading line by line (the default iteration behavior) is more common.

## Reading Binary Data into a Mutable Buffer
##### Problem
You want to read binary data directly into a mutable buffer without any intermediate copying. Perhaps you want to mutate the data in-place and write it back out to a file.
##### Solution
To read data into a mutable array, use the `readinto()` method of files. For example:
``` py
import os.path
def read_into_buffer(filename):
    buf = bytearray(os.path.getsize(filename))
    with open(filename, 'rb') as f:
        f.readinto(buf)
    return buf


#Here is an example that illustrates the usage:
# Write a sample file
with open('sample.bin', 'wb') as f:
    f.write(b'Hello World')
buf = read_into_buffer('sample.bin')
print(buf) #bytearray(b'Hello World')
buf[0:5] = b'Hallo'
print(buf) #bytearray(b'Hallo World')

with open('newsample.bin', 'wb') as f:
    print(f.write(buf)) #11
```

##### Discussion
The `readinto()` method of files can be used to fill any preallocated array with data. This even includes arrays created from the array module or libraries such as `numpy` .
<br>

Unlike the normal `read()` method, `readinto()` fills the contents of an existing buffer rather than allocating new objects and returning them. Thus, you might be able to use it to avoid making extra memory allocations. 

For example, if you are reading a binary file consisting of equally sized records, you can write code like this:
``` py
record_size = 32
# Size of each record (adjust value)
buf = bytearray(record_size)
with open('somefile', 'rb') as f:
    while True:
        n = f.readinto(buf)
        if n < record_size:
            break
# Use the contents of buf
...
```

Another interesting feature to use here might be a `memoryview`, which lets you make zero-copy slices of an existing buffer and even change its contents. For example:

``` py
>>> buf
bytearray(b'Hello World')
>>> m1 = memoryview(buf)
>>> m2 = m1[-5:]
>>> m2
<memory at 0x100681390>
>>> m2[:] = b'WORLD'
>>> buf
bytearray(b'Hello WORLD')
>>>
```

One caution with using `f.readinto()` is that you must always make sure to check its return code, which is the number of bytes actually read.

!> If the number of bytes is smaller than the size of the supplied buffer, it might indicate truncated or corrupted data (e.g., if you were expecting an exact number of bytes to be read).

?> Finally, be on the lookout for other “into” related functions in various library modules (e.g., `recv_into()` , `pack_into()` , etc.). 

Many other parts of Python have support for direct I/O or data access that can be used to fill or alter the contents of arrays and buffers.

## Memory Mapping Binary Files
##### Problem
You want to memory map a binary file into a mutable byte array, possibly for random access to its contents or to make in-place modifications.
##### Solution
Use the `mmap` module to memory map files. Here is a utility function that shows how to open a file and memory map it in a portable manner:
``` py
import os
import mmap
def memory_map(filename, access=mmap.ACCESS_WRITE):
    size = os.path.getsize(filename)
    fd = os.open(filename, os.O_RDWR)
    return mmap.mmap(fd, size, access=access)
```

To use this function, you would need to have a file already created and filled with data. Here is an example of how you could initially create a file and expand it to a desired size:

``` py
>>> size = 1000000
>>> with open('data', 'wb') as f:
...     f.seek(size-1)
...     f.write(b'\x00')
...
>>>
```

Now here is an example of memory mapping the contents using the `memory_map()` function:

``` py
>>> m = memory_map('data')
>>> len(m)
1000000
>>> m[0:10]
b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'
>>> m[0]
0
>>> # Reassign a slice
>>> m[0:11] = b'Hello World'
>>> m.close()
>>> # Verify that changes were made
>>> with open('data', 'rb') as f:
...     print(f.read(11))
...
b'Hello World'
>>>
```

The mmap object returned by `mmap()` can also be used as a context manager, in which case the underlying file is closed automatically. For example:

``` py
>>> with memory_map('data') as m:
...     print(len(m))
...     print(m[0:10])
...
1000000
b'Hello World'
>>> m.closed
True
>>>
```

- By default, the `memory_map()` function shown opens a file for both reading and writing. Any modifications made to the data are copied back to the original file.

- If read-only access is needed instead, supply `mmap.ACCESS_READ` for the access argument. For example:

``` py
m = memory_map(filename, mmap.ACCESS_READ)
```

- If you intend to modify the data locally, but don’t want those changes written back to the original file, use `mmap.ACCESS_COPY` :

``` py
m = memory_map(filename, mmap.ACCESS_COPY)
```

##### Discussion
Using mmap to map files into memory can be an efficient and elegant means for randomly accessing the contents of a file.

?> instead of opening a file and performing various combinations of `seek()` , `read()` , and `write()` calls, you can simply map the file and access the data using slicing operations.

Normally, the memory exposed by `mmap()` looks like a bytearray object. However, you can interpret the data differently using a `memoryview`. For example:

``` py
>>> m = memory_map('data')
>>> # Memoryview of unsigned integers
>>> v = memoryview(m).cast('I')
>>> v[0] = 7
>>> m[0:4]
b'\x07\x00\x00\x00'
>>> m[0:4] = b'\x07\x01\x00\x00'
>>> v[0]
263
>>>
```
It should be emphasized that memory mapping a file does not cause the entire file to be read into memory. That is, it’s not copied into some kind of memory buffer or array. Instead, the operating system merely reserves a section of virtual memory for the file contents. As you access different regions, those portions of the file will be read and mapped into the memory region as needed. However, parts of the file that are never accessed simply stay on disk. This all happens transparently, behind the scenes.

?> If more than one Python interpreter memory maps the same file, the resulting mmap object can be used to exchange data between interpreters. That is, all interpreters can read/write data simultaneously, and changes made to the data in one interpreter will automatically appear in the others. Obviously, some extra care is required to synchronize things, but this kind of approach is sometimes used as an alternative to transmitting data in messages over pipes or sockets.

!> Be aware that there are some platform differences concerning the use of the `mmap()` call hidden behind the scenes.

## Manipulating Pathnames
##### Problem
You need to manipulate pathnames in order to find the base filename, directory name, absolute path, and so on.
##### Solution
To manipulate pathnames, use the functions in the `os.path` module. Here is an interactive example that illustrates a few key features:

``` PY
>>> import os
>>> path = '/Users/beazley/Data/data.csv'
>>> # Get the last component of the path
>>> os.path.basename(path)
'data.csv'
>>> # Get the directory name
>>> os.path.dirname(path)
'/Users/beazley/Data'
>>> # Join path components together
>>> os.path.join('tmp', 'data', os.path.basename(path))
'tmp/data/data.csv'
>>> # Expand the user's home directory
>>> path = '~/Data/data.csv'
>>> os.path.expanduser(path)
'/Users/beazley/Data/data.csv'
>>> # Split the file extension
>>> os.path.splitext(path)
('~/Data/data', '.csv')
>>>
```

##### Discussion
For any manipulation of filenames, you should use the `os.path` module instead of trying to cook up your own code using the standard string operations.
<br>

- ?> The `os.path` module knows about differences between Unix and Windows and can reliably deal with filenames such as Data/data.csv and Data\data.csv. 

- ?> Second, you really shouldn’t spend your time reinventing the wheel. It’s usually best to use the functionality that’s already provided for you.

- ?> It should be noted that the os.path module has many more features not shown in this recipe.

## Testing for the Existence of a File
##### Problem
You need to test whether or not a file or directory exists.

##### Solution
Use the `os.path` module to test for the existence of a file or directory. For example:

``` py
>>> import os
>>> os.path.exists('/etc/passwd')
True
>>> os.path.exists('/tmp/spam')
False
>>> # Is a regular file
>>> os.path.isfile('/etc/passwd')
True
>>> # Is a directory
>>> os.path.isdir('/etc/passwd')
False
>>> # Is a symbolic link
>>> os.path.islink('/usr/local/bin/python3')
True
>>> # Get the file linked to
>>> os.path.realpath('/usr/local/bin/python3')
'/usr/local/bin/python3.3'
>>>
```

If you need to get __metadata__ (e.g., the file size or modification date), that is also available in the `os.path` module.
``` py
>>> os.path.getsize('/etc/passwd')
3669
>>> os.path.getmtime('/etc/passwd')
1272478234.0
>>> import time
>>> time.ctime(os.path.getmtime('/etc/passwd'))
'Wed Apr 28 13:10:34 2010'
>>
```
##### Discussion

!> Probably the only thing to be aware of when writing scripts is that you might need to worry about permissions especially for operations that get metadata.

## Getting a Directory Listing
##### Problem
You want to get a list of the files contained in a directory on the filesystem.
##### Solution
Use the `os.listdir()` function to obtain a list of files in a directory:

``` py
import os
names = os.listdir('somedir')
```
This will give you the raw directory listing, including all files, subdirectories, symboliclinks, and so forth. 
<br>
If you need to filter the data in some way, consider using a list comprehension combined with various functions in the `os.path` library. For example:

``` py
import os.path
# Get all regular files
names = [name for name in os.listdir('somedir') if os.path.isfile(os.path.join('somedir', name))]
# Get all dirs
dirnames = [name for name in os.listdir('somedir') if os.path.isdir(os.path.join('somedir', name))]
```

The `startswith()` and `endswith()` methods of strings can be useful for filtering the contents of a directory as well. For example:
``` py
pyfiles = [name for name in os.listdir('somedir') if name.endswith('.py')]
```

For filename matching, you may want to use the `glob` or `fnmatch` modules instead. For example:

``` py
import glob
pyfiles = glob.glob('somedir/*.py')
from fnmatch import fnmatch
pyfiles = [name for name in os.listdir('somedir') if fnmatch(name, '*.py')]
```

##### Discussion
If you want to get additional metadata, such as file sizes, modification dates, and so forth, you either need to use additional functions in the `os.path` module or use
the `os.stat()` function. To collect the data. For example:

``` py
# Example of getting a directory listing
import os
import os.path
import glob
pyfiles = glob.glob('*.py')
# Get file sizes and modification dates
name_sz_date = [(name, os.path.getsize(name), os.path.getmtime(name)) for name in pyfiles]
for name, size, mtime in name_sz_date:
    print(name, size, mtime)
# Alternative: Get file metadata
file_metadata = [(name, os.stat(name)) for name in pyfiles]
for name, meta in file_metadata:
    print(name, meta.st_size, meta.st_mtime)
```
!> Be aware that there are subtle issues that can arise in filename handling related to encodings.  Normally, the entries returned by a function such as `os.list
dir()` are decoded according to the system default filename encoding. However, it’s possible under certain circumstances to encounter un-decodable filenames.


## Bypassing Filename Encoding
##### Problem
You want to perform file I/O operations using raw filenames that have not been decoded or encoded according to the default filename encoding.
##### Solution
By default, all filenames are encoded and decoded according to the text encoding returned by `sys.getfilesystemencoding()`. For example:
``` py
>>> sys.getfilesystemencoding()
'utf-8'
>>>
```

If you want to bypass this encoding for some reason, specify a filename using a raw byte string instead. For example:

``` py
>>> # Wrte a file using a unicode filename
>>> with open('jalape\xf1o.txt', 'w') as f:
...     f.write('Spicy!')
...
6
>>> # Directory listing (decoded)
>>> import os
>>> os.listdir('.')
['jalapeño.txt']
>>> # Directory listing (raw)
>>> os.listdir(b'.')
# Note: byte string
[b'jalapen\xcc\x83o.txt']
>>> # Open file with raw filename
>>> with open(b'jalapen\xcc\x83o.txt') as f:
...     print(f.read())
...
Spicy!
>>>
```

##### Discussion
?> Under normal circumstances, you shouldn’t need to worry about filename encoding and decoding—normal filename operations should just work.

?> many operating systems may allow a user through accident or malice to create files with names that don’t conform to the expected encoding rules. Such filenames may mysteriously break Python programs that work with a lot of files. Reading directories and working with filenames as raw undecoded bytes has the potential to avoid such problems, albeit at the cost of programming convenience.

## Printing Bad Filenames
##### Problem
Your program received a directory listing, but when it tried to print the filenames, it crashed with a UnicodeEncodeError exception and a cryptic message about “surrogates not allowed.”
##### Solution
When printing filenames of unknown origin, use this convention to avoid errors:
``` py
def bad_filename(filename):
    return repr(filename)[1:-1]

try:
    print(filename)
except UnicodeEncodeError:
    print(bad_filename(filename))
```
##### Discussion
?> This recipe is about a potentially rare but very annoying problem regarding programs that must manipulate the filesystem. By default, Python assumes that all filenames are encoded according to the setting reported by `sys.getfilesystemencoding()` . How ever, certain filesystems don’t necessarily enforce this encoding restriction, thereby allowing files to be created without proper filename encoding.

- When executing a command such as `os.listdir()` , bad filenames leave Python in a bind.
    - On the one hand, it can’t just discard bad names.
    - On the other hand, it still can’t turn the filename into a proper text string. 
    
Python’s solution to this problem is to take an undecodable byte value `\xhh` in a filename and map it into a so-called “surrogate encoding” represented by the Unicode character `\udchh` .

Here is an example of how a bad directory listing might look if it contained a filename bäd.txt, encoded as Latin-1 instead of UTF-8:

``` py
>>> import os
>>> files = os.listdir('.')
>>> files
['spam.py', 'b\udce4d.txt', 'foo.txt']
>>>
```

?> If you have code that manipulates filenames or even passes them to functions such as open() , everything works normally.

!> It’s only in situations where you want to output the filename that you run into trouble (e.g., printing it to the screen, logging it, etc.). Specifically, if you tried to print the preceding listing, your program will crash. The reason it crashes is that the character `\udce4` is technically invalid Unicode.It’s actually the second half of a two-character combination known as a surrogate pair. However, since the first half is missing, it’s invalid Unicode.

The only way to produce successful output is to take corrective action when a bad filename is encountered.
<br>
For example:

``` py
>>> for name in files:
...     try:
...         print(name)
...     except UnicodeEncodeError:
...         print(bad_filename(name))
...
spam.py
b\udce4d.txt
foo.txt
>>>
```

The choice of what to do for the `bad_filename()` function is largely up to you. Another option is to re-encode the value in some way, like this:

``` py
def bad_filename(filename):
    temp = filename.encode(sys.getfilesystemencoding(), errors='surrogateescape')
    return temp.decode('latin-1')
```
Usage :
``` py
>>> for name in files:
...     try:
...         print(name)
...     except UnicodeEncodeError:
...         print(bad_filename(name))
...
spam.py
bäd.txt
foo.txt
>>>
```

!> This recipe will likely be ignored by most readers. However, if you’re writing mission critical scripts that need to work reliably with filenames and the filesystem, it’s something to think about.

## Adding or Changing the Encoding of an Already Open File
##### Problem
You want to add or change the Unicode encoding of an already open file without closing it first.
##### Solution
If you want to add Unicode encoding/decoding to an already existing file object that’s opened in binary mode, wrap it with an `io.TextIOWrapper()` object. For example:
``` py
import urllib.request
import io
u = urllib.request.urlopen('http://www.python.org')
f = io.TextIOWrapper(u,encoding='utf-8')
text = f.read()
```

If you want to change the encoding of an already open text-mode file, use its `detach()` method to remove the existing text encoding layer before replacing it with a new one. Here is an example of changing the encoding on `sys.stdout` :
``` py
>>> import sys
>>> sys.stdout.encoding
'UTF-8'
>>> sys.stdout = io.TextIOWrapper(sys.stdout.detach(), encoding='latin-1')
>>> sys.stdout.encoding
'latin-1'
>>>
```

!> Doing this might break the output of your terminal. It’s only meant to illustrate.

##### Discussion
The I/O system is built as a series of layers. You can see the layers yourself by trying this simple example involving a text file:
``` py
>>> f = open('sample.txt','w')
>>> f
<_io.TextIOWrapper name='sample.txt' mode='w' encoding='UTF-8'>
>>> f.buffer
<_io.BufferedWriter name='sample.txt'>
>>> f.buffer.raw
<_io.FileIO name='sample.txt' mode='wb'>
>>>
```

- The `io.TextIOWrapper` is a text-handling layer that encodes and decodes Unicode.
- The `io.BufferedWriter` is a buffered I/O layer that handles binary data.
- The `io.FileIO` is a raw file representing the low-level file descriptor in the operating system.

?> Adding or changing the text encoding involves adding or changing the topmost `io.TextIOWrapper` layer.

?> As a general rule, it’s not safe to directly manipulate the different layers by accessing the attributes shown. For example, see what happens if you try to change the encoding using this technique:

``` py
>>> f
<_io.TextIOWrapper name='sample.txt' mode='w' encoding='UTF-8'>
>>> f = io.TextIOWrapper(f.buffer, encoding='latin-1')
>>> f
<_io.TextIOWrapper name='sample.txt' encoding='latin-1'>
>>> f.write('Hello')
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
ValueError: I/O operation on closed file.
>>>
```
!> It doesn’t work because the original value of f got destroyed and closed the underlying file in the process.

?> The `detach()` method disconnects the topmost layer of a file and returns the next lower layer.Afterward, the top layer will no longer be usable. For example:
```py
>>> f = open('sample.txt', 'w')
>>> f
<_io.TextIOWrapper name='sample.txt' mode='w' encoding='UTF-8'>
>>> b = f.detach()
>>> b
<_io.BufferedWriter name='sample.txt'>
>>> f.write('hello')
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
ValueError: underlying buffer has been detached
>>>
```
Once detached, however, you can add a new top layer to the returned result. For example:
``` py
>>> f = io.TextIOWrapper(b, encoding='latin-1')
>>> f
<_io.TextIOWrapper name='sample.txt' encoding='latin-1'>
>>>
```
Although changing the encoding has been shown, it is also possible to use this technique to change the __line handling__, __error policy__, and other aspects of file handling. For example:
``` py
>>> sys.stdout = io.TextIOWrapper(sys.stdout.detach(), encoding='ascii',
... errors='xmlcharrefreplace')
>>> print('Jalape\u00f1o')
Jalape&#241;o
>>>
```

!> Notice how the non-ASCII character ñ has been replaced by &#241; in the output.

## Writing Bytes to a Text File
##### Problem
You want to write raw bytes to a file opened in text mode.
##### Solution
Simply write the byte data to the files underlying buffer . For example:
``` py
>>> import sys
>>> sys.stdout.write(b'Hello\n')
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
TypeError: must be str, not bytes
>>> sys.stdout.buffer.write(b'Hello\n')
Hello
5
>>>
```
?> Similarly, binary data can be read from a text file by reading from its buffer attribute instead.

##### Discussion
?> Text files are constructed by adding a Unicode encoding/decoding layer on top of a buffered binary-mode file.The buffer attribute simply points at this underlying file. If you access it, you’ll bypass the text encoding/decoding layer.

?> The example involving `sys.stdout` might be viewed as a special case. By default, `sys.stdout` is always opened in text mode.

## Wrapping an Existing File Descriptor As a File Object
##### Problem
You have an integer file descriptor correponding to an already open I/O channel on the operating system (e.g., file, pipe, socket, etc.), and you want to wrap a higher-level Python file object around it.
##### Solution
A file descriptor is different than a normal open file in that it is simply an integer handle assigned by the operating system to refer to some kind of system I/O channel. If you happen to have such a file descriptor, you can wrap a Python file object around it using the `open()` function. However, you simply supply the integer file descriptor as the first argument instead of the filename. For example:
``` py
# Open a low-level file descriptor
import os
fd = os.open('somefile.txt', os.O_WRONLY | os.O_CREAT)
# Turn into a proper file
f = open(fd, 'wt')
f.write('hello world\n')
f.close()
```
?> When the high-level file object is closed or destroyed, the underlying file descriptor will also be closed. If this is not desired, supply the optional `closefd=False` argument to `open()` . For example:
``` py
# Create a file object, but don't close underlying fd when done
f = open(fd, 'wt', closefd=False)
...
```
##### Discussion
On Unix systems, this technique of wrapping a file descriptor can be a convenient means for putting a file-like interface on an existing I/O channel that was opened in a different way (e.g., pipes, sockets, etc.). 
<br>
For instance, here is an example involving sockets:

``` py
from socket import socket, AF_INET, SOCK_STREAM
def echo_client(client_sock, addr):
    print('Got connection from', addr)
# Make text-mode file wrappers for socket reading/writing
client_in = open(client_sock.fileno(), 'rt', encoding='latin-1',closefd=False)
client_out = open(client_sock.fileno(), 'wt', encoding='latin-1',closefd=False)
# Echo lines back to the client using file I/O
for line in client_in:
    client_out.write(line)
    client_out.flush()
    client_sock.close()

def echo_server(address):
    sock = socket(AF_INET, SOCK_STREAM)
    sock.bind(address)
    sock.listen(1)
    while True:
        client, addr = sock.accept()
    echo_client(client, addr)
```
It’s important to emphasize that the above example is only meant to illustrate a feature of the built-in open() function and that it only works on Unix-based systems.

?> If you are trying to put a file-like __interface__ on a socket and need your code to be cross platform, use the `makefile()` method of sockets instead. However, if portability is not a concern, you’ll find that the above solution provides much better performance than using `makefile()` .

You can also use this to make a kind of alias that allows an already open file to be used in a slightly different way than how it was first opened. For example, here’s how you could create a file object that allows you to emit binary data on stdout (which is normally opened in text mode):

``` py
import sys
# Create a binary-mode file for stdout
bstdout = open(sys.stdout.fileno(), 'wb', closefd=False)
bstdout.write(b'Hello World\n')
bstdout.flush()
```
- !> Although it’s possible to wrap an existing file descriptor as a proper file, be aware that not all file modes may be supported and that certain kinds of file descriptors may have funny side effects (especially with respect to error handling, end-of-file conditions, etc.).

- The behavior can also vary according to operating system. In particular, none of the examples are likely to work on non-Unix systems.

## Making Temporary Files and Directories
##### Problem
You need to create a temporary file or directory for use when your program executes. Afterward, you possibly want the file or directory to be destroyed.
##### Solution
The `tempfile` module has a variety of functions for performing this task. To make an unnamed temporary file, use `tempfile.TemporaryFile` :
``` py
from tempfile import TemporaryFile
with TemporaryFile('w+t') as f:
# Read/write to the file
    f.write('Hello World\n')
    f.write('Testing\n')
# Seek back to beginning and read the data
    f.seek(0)
    data = f.read()
# Temporary file is destroyed
```

Or, if you prefer, you can also use the file like this: 
``` py
f = TemporaryFile('w+t')
# Use the temporary file
...
f.close()
# File is destroyed
```

The first argument to `TemporaryFile()` is the file mode, which is usually `w+t` for text and `w+b` for binary. This mode simultaneously supports reading and writing, 

!> closing the file to change modes would actually destroy it. 

`TemporaryFile()` additionally accepts the same arguments as the built-in `open()` function. For example:
``` py
with TemporaryFile('w+t', encoding='utf-8', errors='ignore') as f:
...
```

On most Unix systems, the file created by `TemporaryFile()` is unnamed and won’t even have a directory entry. If you want to relax this constraint, use `NamedTemporaryFile()` instead. For example:
``` py
from tempfile import NamedTemporaryFile
with NamedTemporaryFile('w+t') as f:
    print('filename is:', f.name)
...
# File automatically destroyed
```

Here, the `f.name` attribute of the opened file contains the filename of the temporary file.

?> As with `TemporaryFile()` , the resulting file is automatically deleted when it’s closed. If you don’t want this, supply a `delete=False` keyword argument. For example:
``` py
with NamedTemporaryFile('w+t', delete=False) as f:
    print('filename is:', f.name)
...
```
To make a temporary directory, use `tempfile.TemporaryDirectory()` . For example:
``` py
from tempfile import TemporaryDirectory
with TemporaryDirectory() as dirname:
    print('dirname is:', dirname)
    # Use the directory
    ...
    # Directory and all contents destroyed
```

##### Discussion
- Probably the most convenient way to work with temporary files and directories : 
    - The `TemporaryFile()` 
    - The `NamedTemporaryFile()`
    - The `TemporaryDirectory()`

At a lower level, you can also use the `mkstemp()` and `mkdtemp()` to create temporary files and directories. For example:
``` py
>>> import tempfile
>>> tempfile.mkstemp()
(3, '/var/folders/7W/7WZl5sfZEF0pljrEB1UMWE+++TI/-Tmp-/tmp7fefhv')
>>> tempfile.mkdtemp()
'/var/folders/7W/7WZl5sfZEF0pljrEB1UMWE+++TI/-Tmp-/tmp5wvcv6'
>>>
```
!> However, these functions don’t really take care of further management.
<br>

For example, the `mkstemp()` function simply returns a raw OS file descriptor and leaves it up to you to turn it into a proper file. Similarly, it’s up to you to clean up the files if you want.

?> Normally, temporary files are created in the system’s default location, such as /var/tmp or similar. To find out the actual location, use the `tempfile.gettempdir()` function. For example:
``` py
>>> tempfile.gettempdir()
'/var/folders/7W/7WZl5sfZEF0pljrEB1UMWE+++TI/-Tmp-'
>>>
```
All of the temporary-file-related functions allow you to override this directory as well as the naming conventions using the __prefix__ , __suffix__ , and __dir__ keyword arguments. For example:
``` py
>>> f = NamedTemporaryFile(prefix='mytemp', suffix='.txt', dir='/tmp')
>>> f.name
'/tmp/mytemp8ee899.txt'
>>>
```

The `tempfile()` module creates temporary files in the most secure manner possible. This includes only giving access permission to the current user and taking steps to avoid race conditions in file creation. Be aware that there can be differences between platforms. Thus, you should make sure to check the official documentation for the finer points.

## Communicating with Serial Ports
##### Problem
You want to read and write data over a serial port, typically to interact with some kind of hardware device (e.g., a robot or sensor).
##### Solution
Although you can probably do this directly using Python’s built-in I/O primitives, your best bet for serial communication is to use the `pySerial` package.
You simply open up a serial port using code like this:
``` py
import serial
ser = serial.Serial('/dev/tty.usbmodem641',   # Device name varies
                    baudrate=9600,
                    bytesize=8,
                    parity='N',
                    stopbits=1)
```

The device name will vary according to the kind of device and operating system.
<br>

In Windows, you can use a device of 0, 1, and so on, to open up the communication ports such as “COM0” and “COM1.” Once open, you can read and write data using `read()` , `readline()` , and `write()` calls. For example:
``` py
ser.write(b'G1 X50 Y50\r\n')
resp = ser.readline()
```

##### Discussion
- serial communication can sometimes get rather messy :
    - One reason you should use a package such as `pySerial` is that it provides support for advanced features (e.g., __timeouts__, __control flow__, __buffer flushing__, __handshaking__, etc.). For instance, if you want to enable __RTS-CTS__ handshaking, you simply provide a `rtscts=True` argument to `Serial()` . 
    
    - Keep in mind that all I/O involving serial ports is binary. Thus, make sure you write your code to use bytes instead of text (or perform proper text encoding/decoding as needed). The `struct` module may also be useful should you need to create binary-coded commands or packets.

## Serializing Python Objects
##### Problem
You need to serialize a Python object into a byte stream so that you can do things such as save it to a file, store it in a database, or transmit it over a network connection.
##### Solution
The most common approach for serializing data is to use the `pickle` module.

?> pickle is a Python-specific self-describing data encoding.

To dump an object to a file, you do this:
``` py
import pickle
data = ... # Some Python object
f = open('somefile', 'wb')
pickle.dump(data, f)
```
To dump an object to a string, use `pickle.dumps()` :

``` py
s = pickle.dumps(data)
```
To recreate an object from a byte stream, use either the `pickle.load()` or `pickle.loads()` functions. For example:
``` py
# Restore from a file
f = open('somefile', 'rb')
data = pickle.load(f)
# Restore from a string
data = pickle.loads(s)
```
##### Discussion
For most programs, usage of the `dump()` and `load()` function is all you need to effectively use `pickle`.

?> If you’re working with any kind of library that lets you do things such as save/restore Python objects in databases or transmit objects over the network, there’s a pretty good chance that pickle is being used.

- the serialized data contains information related to:
    - The start and end of each object 
    - Information about its type.
    
For Example :
``` py
>>> import pickle
>>> f = open('somedata', 'wb')
>>> pickle.dump([1, 2, 3, 4], f)
>>> pickle.dump('hello', f)
>>> pickle.dump({'Apple', 'Pear', 'Banana'}, f)
>>> f.close()
>>> f = open('somedata', 'rb')
>>> pickle.load(f)
[1, 2, 3, 4]
>>> pickle.load(f)
'hello'
>>> pickle.load(f)
{'Apple', 'Pear', 'Banana'}
>>>
```
- You can pickle :
    - Functions
    - Classes
    - instances.

The resulting data only encodes name references to the associated code objects. For example:
``` py
>>> import math
>>> import pickle.
>>> pickle.dumps(math.cos)
b'\x80\x03cmath\ncos\nq\x00.'
>>>
```
?> When the data is unpickled, it is assumed that all of the required source is available. __Modules__, __classes__, and __functions__ will automatically be imported as needed. For applications where Python data is being shared between interpreters on different machines, this is a potential maintenance issue, as all machines must have access to the same source code.

!> `pickle.load()` should never be used on untrusted data.As a side effect of loading, pickle will automatically load modules and make instances.

However, an evildoer who knows how pickle works can create “malformed” data that causes Python to execute arbitrary system commands. Thus, it’s essential that `pickle`only be used internally with interpreters that have some ability to authenticate one another.

- Certain kinds of objects can’t be pickled. These are typically objects that involve some sort of external system state, such as : 
    - open files
    - open network connections
    - threads
    - processes
    - stack frames,
    - and so forth.
    
?> User-defined classes can sometimes work around these limitations by providing `__getstate__()` and `__setstate__()` methods. If defined, `pickle.dump()` will call `__getstate__()` to get an object that can be pickled. Similarly, `__setstate__()` will be invoked on unpickling.

To illustrate what’s possible, here is a class that internally defines a thread but can still be pickled/unpickled:
``` py
# countdown.py
import time
import threading
class Countdown:
    def __init__(self, n):
        self.n = n
        self.thr = threading.Thread(target=self.run)
        self.thr.daemon = True
        self.thr.start()
    def run(self):
        while self.n > 0:
            print('T-minus', self.n)
            self.n -= 1
            time.sleep(5)
    def __getstate__(self):
        return self.n
    def __setstate__(self, n):
        self.__init__(n)
```
Try the following experiment involving pickling:
``` py
>>> import countdown
>>> c = countdown.Countdown(30)
>>> T-minus 30
T-minus 29
T-minus 28
...
>>> # After a few moments
>>> f = open('cstate.p', 'wb')
>>> import pickle
>>> pickle.dump(c, f)
>>> f.close()
```
Now quit Python and try this after restart:
``` py
>>> f = open('cstate.p', 'rb')
>>> pickle.load(f)
countdown.Countdown object at 0x10069e2d0>
T-minus 19
T-minus 18
...
```

?> You should see the thread magically spring to life again, picking up where it left off when you first pickled it.

!> pickle is not a particularly efficient encoding for large data structures such as binary arrays created by libraries like the `array` module or `numpy` .

If you’re moving large amounts of array data around, you may be better off simply saving bulk array data in a file or using a more standardized encoding, such as HDF5 (supported by third-party libraries).

?> Frankly, for storing data in databases and archival storage, you’re probably better off using a more standard data encoding, such as __XML__, __CSV__, or __JSON__. These encodings are more standardized, supported by many different languages, and more likely to be better adapted to changes in your source code.

?> be aware that `pickle` has a huge variety of options and tricky corner cases.