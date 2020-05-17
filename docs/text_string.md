## Splitting Strings on Any of Multiple Delimiters

##### Problem
You need to split a string into fields, but the delimiters (and spacing around them) aren’t consistent throughout the string.
##### Solution
The ````` split() ````` method of string objects is really meant for very simple cases, and does not allow for multiple delimiters or account for possible whitespace around the delimiters. In cases when you need a bit more flexibility, use the `````re.split()````` method:
````` python
>>> line = 'asdf fjdk; afed, fjek,asdf,     foo'
>>> import re
>>> re.split(r'[;,\s]\s*', line)
['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']
`````
##### Discussion
In the above example the separator is either a comma (,),semicolon (;), or whitespace followed by any amount of extra whitespace.

When using `````re.split()````` , you need to be a bit careful should the regular expression pattern involve a capture group enclosed in parentheses. If capture groups are used,
then the matched text is also included in the result. For example, watch what happens here:
````` python
>>> line = 'asdf fjdk; afed, fjek,asdf,     foo'
>>> fields = re.split(r'(;|,|\s)\s*', line)
>>> fields
['asdf', ' ', 'fjdk', ';', 'afed', ',', 'fjek', ',', 'asdf', ',', 'foo']
>>> values = fields[::2]
>>> delimiters = fields[1::2] + ['']
>>> values
['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']
>>> delimiters
[' ', ';', ',', ',', ',', '']
>>> ''.join(v+d for v,d in zip(values, delimiters)) #Reform the line using the same delimiters
'asdf fjdk;afed,fjek,asdf,foo'
>>>
`````
If you don’t want the separator characters in the result, but still need to use parentheses to group parts of the regular expression pattern, make sure you use a noncapture group,
specified as `````(?:...)````` . For example:
````` python
>>> re.split(r'(?:,|;|\s)\s*', line)
['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']
>>>
`````

## Matching Text at the Start or End of a String
##### Problem
You need to check the start or end of a string for specific text patterns, such as filename extensions, URL schemes, and so on.
##### Solution
A simple way to check the beginning or end of a string is to use the str.starts `````with()````` or `````str.endswith() `````methods.

````` python
>>> import os
>>> filenames = os.listdir('.')
>>> filenames
[ 'Makefile', 'foo.c', 'bar.py', 'spam.c', 'spam.h' ]
>>> [name for name in filenames if name.endswith(('.c', '.h')) ]
['foo.c', 'spam.c', 'spam.h'
>>> any(name.endswith('.py') for name in filenames)
True
>>>
`````
!> Oddly, this is one part of Python where a tuple is actually required as input. For example:
````` python
>>> choices = ['http:', 'ftp:']
>>> url = 'http://www.python.org'
>>> url.startswith(choices)
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
TypeError: startswith first arg must be str or a tuple of str, not list
>>> url.startswith(tuple(choices))
True
>>>
`````
Similar operations can be performed with slices or regular expressions but are far less elegant. For example:
````` python
>>> filename = 'spam.txt'
>>> filename[-4:] == '.txt'
True
>>> url = 'http://www.python.org'
>>> url[:5] == 'http:' or url[:6] == 'https:' or url[:4] == 'ftp:'
True
>>>
>>> import re
>>> url = 'http://www.python.org'
>>> re.match('http:|https:|ftp:', url)
<_sre.SRE_Match object at 0x101253098>
>>>
`````

## Matching Strings Using Shell Wildcard Patterns
##### Problem
You want to match text using the same wildcard patterns as are commonly used when working in Unix shells 
##### Solution
The `````fnmatch````` module provides two functions— `````fnmatch()````` and `````fnmatchcase()````` —that can be used to perform such matching. The usage is simple:
`````
>>> from fnmatch import fnmatch, fnmatchcase
>>> fnmatch('foo.txt', '*.txt')
True
`````

!> Normally, `````fnmatch()````` matches patterns using the same case-sensitivity rules as the system’s underlying filesystem (which varies based on operating system). For example:
````` python
>>> # On OS X (Mac)
>>> fnmatch('foo.txt', '*.TXT')
False
>>> # On Windows
>>> fnmatch('foo.txt', '*.TXT')
True
>>>
`````
If this distinction matters, use ````` fnmatchcase()````` instead. It matches exactly based on the lower  and uppercase conventions that you supply.
?> If you’re actually trying to write code that matches filenames, use the `````glob````` module instead.

## Matching and Searching for Text Patterns
##### Problem
You want to match or search text for a specific pattern.
## Solution
If the text you’re trying to match is a simple literal, you can often just use the basic string methods, such as `````str.find()````` , `````str.endswith()````` , `````str.startswith()````` , or similar. For more complicated matching, use regular expressions and the `````re````` module.
Here is a sample of how you would do it:
````` python
text = 'yeah, but no, but yeah, but no, but yeah'
text.find('no') # Search for the location of the first occurrence
#10
text1 = '11/27/2012'
import re
# Simple matching: \d+ means match one or more digits
if re.match(r'\d+/\d+/\d+', text1):
    print('yes')
else:
    print('no')
`````
If you’re going to perform a lot of matches using the same pattern, it usually pays to precompile the regular expression pattern into a pattern object first.
<br>
`````match()````` always tries to find the match at the start of a string. If you want to search text for all occurrences of a pattern, use the `````findall()````` method instead.

When defining regular expressions, it is common to introduce capture groups by enclosing parts of the pattern in parentheses. 
`````
>>> datepat = re.compile(r'(\d+)/(\d+)/(\d+)')
`````

?> Capture groups often simplify subsequent processing of the matched text because the contents of each group can be extracted individually. For example:
`````
>>> m = datepat.match('11/27/2012')
>>> m
<_sre.SRE_Match object at 0x1005d2750>
>>> # Extract the contents of each group
>>> m.group(0)
'11/27/2012'
>>> m.group(1)
'11'
>>> m.group(2)
'27'
>>> m.group(3)
'2012'
>>> m.groups()
('11', '27', '2012')
>>> month, day, year = m.groups()
>>>
`````
The `````findall()````` method searches the text and finds all matches, returning them as a list.
If you want to find matches iteratively, use the `````finditer()````` method instead
!> If you want an exact match, make sure the pattern includes the end-marker `````( $ )`````

Last, if you’re just doing a simple text matching/searching operation, you can often skip the compilation step and use module-level functions in the re module instead. For
example:
`````
>>> re.findall(r'(\d+)/(\d+)/(\d+)', text)
`````
?> The module-level functions keep a cache of recently compiled patterns, so there isn’t a huge performance hit, but you’ll save a few lookups and extra processing by using your own compiled pattern.

## Searching and Replacing Text
##### Problem
You want to search for and replace a text pattern in a string.
##### Solution
For simple literal patterns, use the `````str.replace()````` method. For example:
`````
>>> text = 'yeah, but no, but yeah, but no, but yeah'
>>> text.replace('yeah', 'yep')
'yep, but no, but yep, but no, but yep'
>>>
`````
For more complicated patterns, use the `````sub()````` functions/methods in the re module.
Here is a sample of how to do it:
`````
>>> text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
>>> import re
>>> re.sub(r'(\d+)/(\d+)/(\d+)', r'\3-\1-\2', text)
'Today is 2012-11-27. PyCon starts 2013-3-13.'
>>>
`````
The first argument to sub() is the pattern to match and the second argument is the replacement pattern. Backslashed digits such as \3 refer to capture group numbers in the pattern.
<br>
If you’re going to perform repeated substitutions of the same pattern, consider compiling it first for better performance.
<br>
For more complicated substitutions, it’s possible to specify a substitution callback function instead. For example:
`````
>>> from calendar import month_abbr
>>> def change_date(m):
...
mon_name = month_abbr[int(m.group(1))]
...
return '{} {} {}'.format(m.group(2), mon_name, m.group(3))
...
>>> datepat.sub(change_date, text)
'Today is 27 Nov 2012. PyCon starts 13 Mar 2013.'
>>>
`````
If you want to know how many substitutions were made in addition to getting the replacement text, use `````re.subn()````` instead. For example:
`````
>>> newtext, n = datepat.subn(r'\3-\1-\2', text)
>>> newtext
'Today is 2012-11-27. PyCon starts 2013-3-13.'
>>> n
2
>>>
`````

## Searching and Replacing Case-Insensitive Text
##### Problem
You need to search for and possibly replace text in a case-insensitive manner.

##### Solution
To perform case-insensitive text operations, you need to use the re module and supply the `````re.IGNORECASE````` flag to various operations. For example:
````` python
>>> text = 'UPPER PYTHON, lower python, Mixed Python'
>>> re.findall('python', text, flags=re.IGNORECASE)
['PYTHON', 'python', 'Python']
>>> re.sub('python', 'snake', text, flags=re.IGNORECASE)
'UPPER snake, lower snake, Mixed snake'
>>>
`````
The last example reveals a limitation that replacing text won’t match the case of the matched text. If you need to fix this, you might have to use a support function, as in the following:
````` python
def matchcase(word):
    def replace(m):
        text = m.group()
        if text.isupper():
            return word.upper()
        elif text.islower():
            return word.lower()
        elif text[0].isupper():
            return word.capitalize()
        else:
            return word
        return replace

re.sub('python', matchcase('snake'), text, flags=re.IGNORECASE)
#'UPPER SNAKE, lower snake, Mixed Snake'
`````

## Specifying a Regular Expression for the Shortest Match
##### Problem
You’re trying to match a text pattern using regular expressions, but it is identifying the longest possible matches of a pattern. Instead, you would like to change it to find the shortest possible match.
##### Solution
`````
>>> str_pat = re.compile(r'\"(.*)\"')
>>> text2 = 'Computer says "no." Phone says "yes."'
>>> str_pat.findall(text2)
['no." Phone says "yes.']
>>>
`````

However, the * operator in a regular expression is greedy, so matching is based on finding the longest possible match.

To fix this, add the ? modifier after the * operator in the pattern, like this:
`````
>>> str_pat = re.compile(r'\"(.*?)\"')
>>> str_pat.findall(text2)
['no.', 'yes.']
>>>
`````
?> This - ? - makes the matching nongreedy, and produces the shortest match instead.

## Writing a Regular Expression for Multiline Patterns

##### Problem
You’re trying to match a block of text using a regular expression, but you need the match to span multiple lines.
##### solution 
To fix the problem, you can add support for newlines. For example:
`````
>>> comment = re.compile(r'/\*((?:.|\n)*?)\*/')
>>> comment.findall(text2)
[' this is a\n       multiline comment ']
>>>
`````
In this pattern, `````(?:.|\n)````` specifies a noncapture group (i.e., it defines a group for the purposes of matching, but that group is not captured separately or numbered).

##### Discussion
The `````re.compile()````` function accepts a flag, `````re.DOTALL````` , which is useful here. It makes the `````.````` in a regular expression match all characters, including newlines. For example:
`````
>>> comment = re.compile(r'/\*(.*?)\*/', re.DOTALL)
>>> comment.findall(text2)
[' this is a\n       multiline comment ']
`````
!> Using the `````re.DOTALL````` flag works fine for simple cases, but might be problematic if you’re working with extremely complicated patterns or a mix of separate regular expressions
that have been combined together for the purpose of tokenizing.

## Normalizing Unicode Text to a Standard Representation
##### Problem
You’re working with Unicode strings, but need to make sure that all of the strings have the same underlying representation.
##### Solution

!> In Unicode, certain characters can be represented by more than one valid sequence of code points. Having multiple representations is a problem for programs that compare strings. In order to fix this, you should first normalize the text into a standard representation using
the `````unicodedata````` module. For example:
`````
>>> s1 = 'Spicy Jalape\u00f1o'
>>> s2 = 'Spicy Jalapen\u0303o'
>>> s1
'Spicy Jalapeño'
>>> s2
'Spicy Jalapeño'
>>> s1 == s2
False
>>> import unicodedata
>>> t1 = unicodedata.normalize('NFC', s1)
>>> t2 = unicodedata.normalize('NFC', s2)
>>> t1 == t2
True
`````
?> The first argument to `````normalize()````` specifies how you want the string normalized.
- `````NFC````` means that characters should be fully composed (i.e., use a single code point if possible).
- `````NFD````` means that characters should be fully decomposed - `````NFKC````` and `````NFKD````` , which add extra compatibility features for dealing with certain kinds of characters.

The `````combining()````` function tests a character to see if it is a combining character. There are other functions in
the module for finding character categories, testing digits, and so forth.
````` python
>>> t1 = unicodedata.normalize('NFD', s1)
>>> ''.join(c for c in t1 if not unicodedata.combining(c))
'Spicy Jalapeno'
>>>
`````

## Working with Unicode Characters in Regular Expressions
##### Problem
You are using regular expressions to process text, but are concerned about the handling of Unicode characters.
##### Solution
By default, the `````re````` module is already programmed with rudimentary knowledge of certain Unicode character classes. For example, \d already matches any unicode digit character:
If you need to include specific Unicode characters in patterns, you can use the usual escape sequence for Unicode characters. For example :
`````
>>> arabic = re.compile([\u0600-\u06ff\u0750-\u077f\u08a0-\u08ff]+')
>>>
`````
When performing matching and searching operations, it’s a good idea to normalize and possibly sanitize all text to a standard form first .However, it’s also important to be aware of special cases. For example : 
````` python
>>> pat = re.compile('stra\u00dfe', re.IGNORECASE)
>>> s = 'straße'
>>> pat.match(s)
# Matches
<_sre.SRE_Match object at 0x10069d370>
>>> pat.match(s.upper())
# Doesn't match
>>> s.upper()
# Case folds
'STRASSE'
>>>
`````

## Stripping Unwanted Characters from Strings
##### Problem
You want to strip unwanted characters, such as whitespace, from the beginning, end, or middle of a text string.
##### Solution
The ````` strip() ````` method can be used to strip characters from the beginning or end of a string. `````lstrip()````` and `````rstrip()````` perform stripping from the left or right side, respectively.
By default, these methods strip whitespace, but other characters can be given. For example:
````` python
>>> # Whitespace stripping
>>> s = 'hello world \n'
>>> s.strip()
'hello world'
>>> s.lstrip()
'hello world \n'
>>> s.rstrip()
'hello world'
>>> # Character stripping
>>> t = '-----hello====='
>>> t.strip('-=')
'hello'
>>>
`````
##### Discussion
!> Be aware that stripping does not apply to any text in the middle of a string.

?> If you needed to do something to the inner space, you would need to use another technique, such as using the `````replace()````` method or a regular expression substitution.

?> It is often the case that you want to combine string stripping operations with some other kind of iterative processing, such as reading lines of data from a file. If so, this is one area where a generator expression can be useful. For example:
`````
with open(filename) as f:
    lines = (line.strip() for line in f)
    for line in lines:
        ...
`````

## Sanitizing and Cleaning Up Text
##### Problem
Some bored script kiddie has entered the text “pýtĥöñ” into a form on your web page and you’d like to clean it up somehow.
##### Solution

Simple replacements using `````str.replace()````` or `````re.sub()````` can focus on removing or changing very specific character sequences. You can also normalize text using unicode `````data.normalize()````` ,

However, you might want to take the sanitation process a step further. Perhaps, for example, you want to eliminate whole ranges of characters or strip diacritical marks. To
do so, you can turn to the often overlooked `````str.translate()````` method. To illustrate, suppose you’ve got a messy string such as the following:
`````
>>> s = 'pýtĥöñ\fis\tawesome\r\n'
>>> s
'pýtĥöñ\x0cis\tawesome\r\n'
>>> remap = {
...ord('\t') : ' ',
...ord('\f') : ' ',
...ord('\r') : None # Deleted
... }
>>> a = s.translate(remap)
>>> a
'pýtĥöñ is awesome\n'
>>>
`````
For example, let’s remove all combining characters:
````` python
>>> import unicodedata
>>> import sys
>>> cmb_chrs = dict.fromkeys(c for c in range(sys.maxunicode)if unicodedata.combining(chr(c)))
>>> b = unicodedata.normalize('NFD', a)
>>> b
'pýtĥöñ is awesome\n'
>>> b.translate(cmb_chrs)
'python is awesome\n'
>>>
`````
As another example, here is a translation table that maps all Unicode decimal digit characters to their equivalent in ASCII:
`````
>>> digitmap = { c: ord('0') + unicodedata.digit(chr(c))
                for c in range(sys.maxunicode)
                if unicodedata.category(chr(c)) == 'Nd'
                }
>>> len(digitmap)
460
>>> # Arabic digits
>>> x = '\u0661\u0662\u0663'
>>> x.translate(digitmap)
'123'
>>>
`````
Yet another technique for cleaning up text involves I/O decoding and encoding functions. The idea here is to first do some preliminary cleanup of the text, and then run it
through a combination of encode() or decode() operations to strip or alter it. For
example:
`````
>>> a
'pýtĥöñ is awesome\n'
>>> b = unicodedata.normalize('NFD', a)
>>> b.encode('ascii', 'ignore').decode('ascii')
'python is awesome\n'
>>>
`````
##### Discussion
As a general rule, the simpler it is, the faster it will run. For simple replacements, the `````str.replace()````` method
is often the fastest approach—even if you have to call it multiple times.
<br>

On the other hand, the `````translate()````` method is very fast if you need to perform any kind of nontrivial character-to-character remapping or deletion.

## Aligning Text Strings
##### Problem
You need to format text with some sort of alignment applied.
##### Solution
For basic alignment of strings, the `````ljust()````` , `````rjust()````` , and `````center()````` methods of strings
can be used.All of these methods accept an optional fill character as well. For example:
`````
>>> text = 'Hello World'
>>> text.ljust(20)
'Hello World         '
>>> text.center(20)
'    Hello World     '
>>> text.rjust(20,'=')
'=========Hello World'
`````
The `````format()````` function can also be used to easily align things. All you need to do is use the `````<````` , `````>````` , or `````^````` characters along with a desired width. For example:
`````
>>> format(text, '>20')
'         Hello World'
>>> format(text, '<20')
'Hello World         '

>>> format(text, '^20')
'    Hello World     '
`````
If you want to include a fill character other than a space, specify it before the alignment character:
`````
>>> format(text, '=>20s')
'=========Hello World'
>>> format(text, '*^20s')
'****Hello World*****'
>>>
`````
These format codes can also be used in the format() method when formatting multiple values. For example:
`````
>>> '{:>10s} {:>10s}'.format('Hello', 'World')
'     Hello      World'
>>>
`````
One benefit of `````format()````` is that it is not specific to strings. 
<br>

In older code, you will also see the `````%````` operator used to format text. For example:
`````
>>> '%-20s' % text
'Hello World         '
>>> '%20s' % text
'         Hello World'
>>>
`````
?> `````format()````` is a lot more powerful than what is provided with the `````%````` operator.
Moreover, `````format()````` is more general purpose than using the `````jlust()````` , `````rjust()````` , or `````center()````` method of strings in that it works with any kind of object.


## Combining and Concatenating Strings
##### Problem
You want to combine many small strings together into a larger string.
##### Solution
If the strings you wish to combine are found in a sequence or iterable, the fastest way to combine them is to use the `````join()````` method. For example:
````` python
>>> parts = ['Is', 'Chicago', 'Not', 'Chicago?']
>>> ' '.join(parts)
'Is Chicago Not Chicago?'
>>> ','.join(parts)
'Is,Chicago,Not,Chicago?'
`````
this odd syntax is because the objects you want to join could come from any number of different data sequences (e.g., lists, tuples, dicts, files, sets, or generators), and it would be redundant to have join() implemented as a method on all of
those objects separately.
<br>

If you’re only combining a few strings, using `````+````` usually works .
##### Discussion
The most important thing to know is that using the + operator to join a lot of strings together is grossly inefficient due to the memory copies and garbage collection that occurs.
<br>
One related (and pretty neat) trick is the conversion of data to strings and concatenation at the same time using a generator expression. For example:
`````
>>> data = ['ACME', 50, 91.1]
>>> ','.join(str(d) for d in data)
'ACME,50,91.1'
>>>
`````
!> Sometimes programmers get carried away with concatenation when it’s really not technically necessary. For example:

````` python
print(a + ':' + b + ':' + c) # Ugly
print(':'.join([a, b, c])) # Still ugly
print(a, b, c, sep=':') # Better
`````
Mixing I/O operations and string concatenation is something that might require study in your application. For example, consider the following two code fragments:
`````
# Version 1 (string concatenation)
f.write(chunk1 + chunk2)
# Version 2 (separate I/O operations)
f.write(chunk1)
f.write(chunk2)
`````
- If the two strings are small, the first version might offer much better performance due to the inherent expense of carrying out an I/O system call. 
- If the two strings are large, the second version may be more efficient, since it avoids making a large temporary result and copying large blocks of memory around. 
Last, but not least, if you’re writing code that is building output from lots of small strings,you might consider writing that code as a generator function, using yield to emit fragments. For example:
````` python
def sample():
    yield 'Is'
    yield 'Chicago'
    yield 'Not'
    yield 'Chicago?'
`````
For example, you could simply join the fragments using `````join()````` :
`````
text = ''.join(sample())
`````
Or you could come up with some kind of hybrid scheme that’s smart about combining
I/O operations:
````` python
def combine(source, maxsize):
    parts = []
    size = 0
    for part in source:
        parts.append(part)
        size += len(part)
        if size > maxsize:
            yield ''.join(parts)
            parts = []
            size = 0
    yield ''.join(parts)
for part in combine(sample(), 32768):
    f.write(part)
`````


## Interpolating Variables in Strings
##### Problem
You want to create a string in which embedded variable names are substituted with a string representation of a variable’s value.
##### Solution
Python has no direct support for simply substituting variable values in strings. However, this feature can be approximated using the `````format()````` method of strings. For example:
`````
>>> s = '{name} has {n} messages.'
>>> s.format(name='Guido', n=37)
'Guido has 37 messages.'
>>>
`````
Alternatively, if the values to be substituted are truly found in variables, you can use the combination of `````format_map()````` and `````vars()````` , as in the following:
`````
>>> name = 'Guido'
>>> n = 37
>>> s.format_map(vars())
'Guido has 37 messages.'
>>>
`````
?> One subtle feature of vars() is that it also works with instances.
`````
>>> class Info:
        def __init__(self, name, n):
        self.name = name
        self.n = n
>>> a = Info('Guido',37)
>>> s.format_map(vars(a))
'Guido has 37 messages.'
>>>
`````
!> One downside of `````format()````` and `````format_map()````` is that they do not deal gracefully with missing values

One way to avoid this is to define an alternative dictionary class with a `````__missing__()````` method, as in the following:
`````
class safesub(dict):
    def __missing__(self, key):
    return '{' + key + '}'
del n # Make sure n is undefined
s.format_map(safesub(vars()))
#'Guido has {n} messages.'
`````
If you find yourself frequently performing these steps in your code, you could hide the variable substitution process behind a small utility function that employs a so-called “frame hack.” For example:
````` python
import sys
def sub(text):
    return text.format_map(safesub(sys._getframe(1).f_locals))
name = 'Guido'
n = 37
print(sub('Hello {name}'))
#Hello Guido
print(sub('You have {n} messages.'))
#You have 37 messages.
print(sub('Your favorite color is {color}'))
#Your favorite color is {color}
`````

##### Discussion
As an alternative to the solution presented in this recipe, you will sometimes see string formatting like this:
`````
>>> name = 'Guido'
>>> n = 37
>>> '%(name) has %(n) messages.' % vars()
'Guido has 37 messages.'
>>>
`````
You may also see the use of template strings:
`````
>>> import string
>>> s = string.Template('$name has $n messages.')
>>> s.substitute(vars())
'Guido has 37 messages.'
>>>
`````
?> However, the `````format()````` and `````format_map()````` methods are more modern than either of these alternatives, and should be preferred. One benefit of using format() is that you
also get all of the features related to string formatting (alignment, padding, numerical formatting, etc.)
The little-known __\_\_missing\_\_()__ method of mapping/dict classes is a method that you can define to handle missing values. In the safesub class, this method has been defined to return missing values back as a placeholder. Instead of getting a KeyError exception.

<br>

The `````sub()````` function uses `````sys._getframe(1)````` to return the stack frame of the caller. From that, the `````f_locals````` attribute is accessed to get the local variables. It goes without saying that messing around with stack frames should probably be avoided in most code.

<br>

it’s probably worth noting that `````f_locals````` is a dictionary that is a copy of the local variables in the calling function. Although you can modify the contents of `````f_locals````` . the modifications don’t actually have any lasting effect. Thus, even though accessing a
different stack frame might look evil, it’s not possible to accidentally overwrite variables or change the local environment of the caller.

## Reformatting Text to a Fixed Number of Columns
##### Problem
You have long strings that you want to reformat so that they fill a user-specified number of columns.
##### Solution
Use the `````textwrap````` module to reformat text for output. For example:
````` python
s = "Look into my eyes, look into my eyes, the eyes, the eyes, \
the eyes, not around the eyes, don't look around the eyes, \
look into my eyes, you're under."
import textwrap
>>> print(textwrap.fill(s, 40))
Look into my eyes, look into my eyes,
the eyes, the eyes, the eyes, not around
the eyes, don't look around the eyes,
look into my eyes, you're under.
>>> print(textwrap.fill(s, 40, initial_indent='    '))
#    Look into my eyes, look into my
#eyes, the eyes, the eyes, the eyes, not
#around the eyes, don't look around the
#eyes, look into my eyes, you're under.
>>> print(textwrap.fill(s, 40, subsequent_indent='    ))
#Look into my eyes, look into my eyes,
#    the eyes, the eyes, the eyes, not
#    around the eyes, don't look around
#    the eyes, look into my eyes, you're
#    under.
`````
##### Discussion
On the subject of the terminal size,you can obtain it using `````os.get_terminal_size()````` . For example:
`````
>>> import os
>>> os.get_terminal_size().columns
80
>>>
`````
?> The `````fill()````` method has a few additional options that control how it handles tabs, sentence endings, and so on.

## Handling HTML and XML Entities in Text
##### Problem
You want to replace HTML or XML entities such as &entity; or &#code; with their corresponding text. Alternatively, you need to produce text, but escape certain characters (e.g., < , > , or & ).
##### Solution
If you are producing text, replacing special characters such as < or > is relatively easy if you use the `````html.escape()````` function. For example:
````` python
s = 'Elements are written as "<tag>text</tag>".'
import html
print(s)    #Elements are written as "<tag>text</tag>".
print(html.escape(s))
#Elements are written as &quot;&lt;tag&gt;text&lt;/tag&gt;&quot;.
print(html.escape(s, quote=False)) # Disable escaping of quotes
#Elements are written as "&lt;tag&gt;text&lt;/tag&gt;".
`````
If you’re trying to emit text as ASCII and want to embed character code entities for non-ASCII characters, you can use the `````errors='xmlcharrefreplace'````` . For example:
````` python
s = 'Spicy Jalapeño'
s.encode('ascii', errors='xmlcharrefreplace')
#b'Spicy Jalape&#241;o'
`````
If, for some reason, you’ve received bare text with some entities in it and you want them replaced manually, you can usually do it using various utility functions/methods associated with HTML or XML parsers. For example:
````` python
s = 'Spicy &quot;Jalape&#241;o&quot.'
from html.parser import HTMLParser
p = HTMLParser()
p.unescape(s) #'Spicy "Jalapeño".'
t = 'The prompt is &gt;&gt;&gt;'
from xml.sax.saxutils import unescape
unescape(t) #'The prompt is >>>'
````` 
##### Discussion
Proper escaping of special characters is an easily overlooked detail of generating HTML or XML. However, you really need to investigate the use of a proper parser. For example, if processing HTML or XML, using a parsing module such as `````html.parser````` or `````xml.etree.ElementTree````` should already take care of details related to replacing entities in the input text for you.

## Tokenizing Text
##### Problem
You have a string that you want to parse left to right into a stream of tokens.
##### Solution
To tokenize the string, you need to do more than merely match patterns. You need to have some way to identify the kind of pattern as well.

- the first step is to define all of the possible tokens, including whitespace, by regular expression patterns using named capture groups.
- Next, to tokenize, use the little-known `````scanner()````` method of pattern objects. This method creates a scanner object in which repeated calls to `````match()````` step through the
supplied text one match at a time. Here is an interactive example of how a scanner object works:
````` python
import re
NAME = r'(?P<NAME>[a-zA-Z_][a-zA-Z_0-9]*)'
NUM = r'(?P<NUM>\d+)'
PLUS = r'(?P<PLUS>\+)'
TIMES = r'(?P<TIMES>\*)'
EQ = r'(?P<EQ>=)'
WS = r'(?P<WS>\s+)'
master_pat = re.compile('|'.join([NAME, NUM, PLUS, TIMES, EQ, WS]))
text = 'foo = 23 + 42 * 10'
scanner = master_pat.scanner('foo = 42')
_ = scanner.match()
_.lastgroup, _.group() #('NAME', 'foo')
_ = scanner.match()
_.lastgroup, _.group() #('WS', ' ')
_ = scanner.match()
_.lastgroup, _.group() #('EQ', '=')
_ = scanner.match()
_.lastgroup, _.group() #('WS', ' ')
_ = scanner.match()
_.lastgroup, _.group() #('NUM', '42')
scanner.match()
`````
?> the `````?P<TOKENNAME>````` convention is used to assign a name to the pattern.

it can be cleaned up and easily packaged into a generator like this:
````` python
from collections import namedtuple
Token = namedtuple('Token', ['type','value'])
def generate_tokens(pat, text):
    scanner = pat.scanner(text)
    for m in iter(scanner.match, None):
        yield Token(m.lastgroup, m.group())
# Example use
for tok in generate_tokens(master_pat, 'foo = 42'):
    print(tok)
# Produces output
# Token(type='NAME', value='foo')
# Token(type='WS', value=' ')
# Token(type='EQ', value='=')
# Token(type='WS', value=' ')
# Token(type='NUM', value='42')
`````

You can either define more generator functions or use a generator expression. For example, here is how you might filter out all whitespace tokens.
````` python
tokens = (tok for tok in generate_tokens(master_pat, text) if tok.type != 'WS')
for tok in tokens:
    print(tok)
`````
##### Discussion
To use the scanning technique shown, there are a few important details to keep in mind.
- First, you must make sure that you identify every possible text sequence that might appear in the input with a correponding re pattern. If any nonmatching text is found, scanning simply stops. This is why it was necessary to specify the whitespace ( WS ) token in the example.
- The order of tokens in the master regular expression also matters. When matching, re tries to match pattens in the order specified. Thus, if a pattern happens to be a substring
of a longer pattern, you need to make sure the longer pattern goes first. 
- Last, but not least, you need to watch out for patterns that form substrings. For example,
````` python
PRINT = r'(P<PRINT>print)'
NAME = r'(P<NAME>[a-zA-Z_][a-zA-Z_0-9]*)'
master_pat = re.compile('|'.join([PRINT, NAME]))
for tok in generate_tokens(master_pat, 'printer'):
    print(tok)
# Outputs :
# Token(type='PRINT', value='print')
# Token(type='NAME', value='er')
`````
?> For more advanced kinds of tokenizing, you may want to check out packages such as PyParsing or PLY.

## Writing a Simple Recursive Descent Parser
##### Problem
You need to parse text according to a set of grammar rules and perform actions or build an abstract syntax tree representing the input. The grammar is small, so you’d prefer to
just write the parser yourself as opposed to using some kind of framework.
##### Solution
In order to do this, you should probably start by having a formal specification of the grammar in the form of a `````BNF````` or `````EBNF`````.
<br>
For example, a grammar for simple arithmetic expressions might look like this:
`````
expr ::= expr + term
    |expr - term
    |term
term ::= term * factor
    |term / factor
    |factor
factor ::= ( expr )
    |NUM
`````

Or, alternatively, in `````EBNF````` form:
`````
expr ::= term { (+|-) term }*
term ::= factor { (*|/) factor }*
factor ::= ( expr )
    |   NUM
`````
?> In an EBNF, parts of a rule enclosed in { ... }* are optional.

?> The * means zero or more repetitions (the same meaning as in a regular expression).
Think of `````BNF````` as a specification of substitution or replacement rules where symbols on the left side can be
replaced by the symbols on the right (or vice versa). Generally, what happens during parsing is that you try to match the input text to the grammar by making various substitutions and expansions using the BNF.

<br>
example:

``` python
import re
import collections
# Token specification
NUM = r'(?P<NUM>\d+)'
PLUS = r'(?P<PLUS>\+)'
MINUS = r'(?P<MINUS>-)'
TIMES = r'(?P<TIMES>\*)'
DIVIDE = r'(?P<DIVIDE>/)'
LPAREN = r'(?P<LPAREN>\()'
RPAREN = r'(?P<RPAREN>\))'
WS = r'(?P<WS>\s+)'
master_pat = re.compile('|'.join([NUM, PLUS, MINUS, TIMES,DIVIDE, LPAREN, RPAREN, WS]))
Token = collections.namedtuple('Token', ['type','value']) # Tokenizer
def generate_tokens(text):
    scanner = master_pat.scanner(text)
    for m in iter(scanner.match, None):
        tok = Token(m.lastgroup, m.group())
        if tok.type != 'WS':
            yield tok
# Parser
class ExpressionEvaluator:
    '''
    Implementation of a recursive descent parser.
    Each methodimplements a single grammar rule. Use the ._accept() method
    to test and accept the current lookahead token. Use the ._expect()
    method to exactly match and discard the next token on on the input
    (or raise a SyntaxError if it doesn't match).
    '''
    def parse(self,text):
        self.tokens = generate_tokens(text)
        self.tok = None # Last symbol consumed
        self.nexttok = None # Next symbol tokenized
        self._advance() # Load first lookahead token
        return self.expr()
    def _advance(self):
        'Advance one token ahead'
        self.tok, self.nexttok = self.nexttok, next(self.tokens, None)
    def _accept(self,toktype):
        'Test and consume the next token if it matches toktype'
        if self.nexttok and self.nexttok.type == toktype:
            self._advance()
            return True
        else:
            return False
    def _expect(self,toktype):
        'Consume next token if it matches toktype or raise SyntaxError'
        if not self._accept(toktype):
            raise SyntaxError('Expected ' + toktype)
            # Grammar rules follow
    def expr(self):
        "expression ::= term { ('+'|'-') term }*"
        exprval = self.term()
        while self._accept('PLUS') or self._accept('MINUS'):
            op = self.tok.type
            right = self.term()
            if op == 'PLUS':
                exprval += right
            elif op == 'MINUS':
                exprval -= right
        return exprval
    def term(self):
        "term ::= factor { ('*'|'/') factor }*"
        termval = self.factor()
        while self._accept('TIMES') or self._accept('DIVIDE'):
            op = self.tok.type
            right = self.factor()
            if op == 'TIMES':
                termval *= right
            elif op == 'DIVIDE':
                termval /= right
        return termval
    def factor(self):
        "factor ::= NUM | ( expr )"
        if self._accept('NUM'):
            return int(self.tok.value)
        elif self._accept('LPAREN'):
            exprval = self.expr()
            self._expect('RPAREN')
            return exprval
        else:
            raise SyntaxError('Expected NUMBER or LPAREN')


e = ExpressionEvaluator()
e.parse('2') #2
e.parse('2 + 3') #5
e.parse('2 + 3 * 4') #14
e.parse('2 + (3 + 4) * 5') #37
e.parse('2 + (3 + * 4)') #Throw error
```

If you want to do something other than pure evaluation, you need to change the
ExpressionEvaluator class to do something else.

##### Discussion
If you are seeking background knowledge about grammars, parsing algorithms, and other information, a compilers book is where you should turn.
- To start, you take every grammar rule and you turn it into a function or method. Thus, if your grammar looks like this:
`````
    expr ::= term { ('+'|'-') term }*
    term ::= factor { ('*'|'/') factor }*
    factor ::= '(' expr ')'
            |NUM
`````
You start by turning it into a set of methods like this:
`````
class ExpressionEvaluator:
...def expr(self):
...def term(self):
...def factor(self):
`````
- The task of each method is simple—it must walk from left to right over each part of the grammar rule, consuming tokens in the process. In a sense, the goal of the method is to either consume the rule or generate a syntax error if it gets stuck. To do this, the following implementation techniques are applied:
    - If the next symbol in the rule is the name of another grammar rule (e.g., term or factor ), you simply call the method with the same name. This is the “descent” part of the algorithm—control descends into another grammar rule. Sometimes rules
    will involve calls to methods that are already executing (e.g., the call to expr in the
    factor ::= '(' expr ')' rule). This is the “recursive” part of the algorithm.
    - If the next symbol in the rule has to be a specific symbol (e.g., ( ), you look at the next token and check for an exact match. If it doesn’t match, it’s a syntax error. The _expect() method in this recipe is used to perform these steps.
    - If the next symbol in the rule could be a few possible choices (e.g., + or - ), you have to check the next token for each possibility and advance only if a match is made. This is the purpose of the _accept() method in this recipe. It’s kind of like a weaker version of the _expect() method in that it will advance if a match is made, but if not, it simply backs off without raising an error (thus allowing further checks to be made).
    - For grammar rules where there are repeated parts (e.g., such as in the rule expr ::=term { ('+'|'-') term }* ), the repetition gets implemented by a while loop.
    The body of the loop will generally collect or process all of the repeated items until no more are found.
    - Once an entire grammar rule has been consumed, each method returns some kind of result back to the caller. This is how values propagate during parsing. For example, in the expression evaluator, return values will represent partial results of the expression being parsed. Eventually they all get combined together in the topmost grammar rule method that executes.

?> Python code itself is interpreted by a recursive descent parser. If you’re so inclined, you can look at the underlying grammar by inspecting the file Grammar/Grammar in the Python source.

?> For really complicated grammars, you are often better off using parsing tools such as PyParsing or PLY. This is what the expression evaluator code looks like using PLY:

``` python
from ply.lex import lex
from ply.yacc import yacc
# Token list
tokens = [ 'NUM', 'PLUS', 'MINUS', 'TIMES', 'DIVIDE', 'LPAREN', 'RPAREN' ]
# Ignored characters
t_ignore = ' \t\n'
# Token specifications (as regexs)
t_PLUS = r'\+'
t_MINUS = r'-'
t_TIMES = r'\*'
t_DIVIDE = r'/'
t_LPAREN = r'\('
t_RPAREN = r'\)'
# Token processing functions
def t_NUM(t):
    r'\d+'
    t.value = int(t.value)
    return t
# Error handler
def t_error(t):
    print('Bad character: {!r}'.format(t.value[0]))
    t.skip(1)
# Build the lexer
lexer = lex()
# Grammar rules and handler functions
def p_expr(p):
    '''
    expr : expr PLUS term
    | expr MINUS term
    '''
    if p[2] == '+':
        p[0] = p[1] + p[3]
    elif p[2] == '-':
        p[0] = p[1] - p[3]
def p_expr_term(p):
    '''
    expr : term
    '''
    p[0] = p[1]
def p_term(p):
    '''
    term : term TIMES factor
    | term DIVIDE factor
    '''
    if p[2] == '*':
        p[0] = p[1] * p[3]
    elif p[2] == '/':
        p[0] = p[1] / p[3]
def p_term_factor(p):
    '''
    term : factor
    '''
    p[0] = p[1]
def p_factor(p):
    '''
    factor : NUM
    '''
    p[0] = p[1]
def p_factor_group(p):
    '''
    factor : LPAREN expr RPAREN
    '''
    p[0] = p[2]
def p_error(p):
    print('Syntax error')

parser = yacc()
print(parser.parse('2+(3+4)*5'))
```

## Performing Text Operations on Byte Strings
##### Problem
You want to perform common text operations (e.g., stripping, searching, and replacement) on byte strings.
##### Solution
Byte strings already support most of the same built-in operations as text strings. For example:
````` python
>>> data = b'Hello World'
>>> data[0:5]
b'Hello'
>>> data.startswith(b'Hello')
True
>>> data.split()
[b'Hello', b'World']
>>> data.replace(b'Hello', b'Hello Cruel')
b'Hello Cruel World'
>>>
`````

Such operations also work with byte arrays. For example:
````` python
>>> data = bytearray(b'Hello World')
>>> data[0:5]
bytearray(b'Hello')
>>> data.startswith(b'Hello')
True
>>> data.split()
[bytearray(b'Hello'), bytearray(b'World')]
>>> data.replace(b'Hello', b'Hello Cruel')
bytearray(b'Hello Cruel World')
>>>
`````
You can apply regular expression pattern matching to byte strings, but the patterns
themselves need to be specified as bytes. For example:
````` python
>>> data = b'FOO:BAR,SPAM'
>>> import re
>>> re.split('[:,]',data)
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "/usr/local/lib/python3.3/re.py", line 191, in split
        return _compile(pattern, flags).split(string, maxsplit)
    TypeError: can't use a string pattern on a bytes-like object
>>> re.split(b'[:,]',data) # Notice: pattern as bytes
[b'FOO', b'BAR', b'SPAM']
>>> 
`````
##### Discussion
For the most part, almost all of the operations available on text strings will work on byte
strings. However, there are a few notable differences to be aware of. 
- First, indexing of byte strings produces integers, not individual characters. For example:
- Second, byte strings don’t provide a nice string representation and don’t print cleanly
unless first decoded into a text string. For example:
- Third , there are no string formatting operations available to byte strings.If you want to do any kind of formatting applied to byte strings, it should be done using
normal text strings and encoding. For example:
`````
>>> '{:10s} {:10d} {:10.2f}'.format('ACME', 100, 490.1).encode('ascii')
'ACME              100     490.10'
>>>
`````
- Finally, you need to be aware that using a byte string can change the semantics of certain
operations—especially those related to the filesystem. For example, if you supply a file‐
name encoded as bytes instead of a text string, it usually disables filename encoding/
decoding. For example:
`````
>>> # Get a directory listing
>>> import os
>>> os.listdir('.') # Text string (names are decoded)
['jalapeño.txt']
>>> os.listdir(b'.') # Byte string (names left as bytes)
[b'jalapen\xcc\x83o.txt']
>>>
`````

?> Frankly, if you’re working with text, use normal text strings in your program, not byte strings.