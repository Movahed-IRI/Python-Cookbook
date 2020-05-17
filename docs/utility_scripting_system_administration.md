## Accepting Script Input via Redirection, Pipes, or Input Files
##### Problem
You want a script you’ve written to be able to accept input using whatever mechanism is easiest for the user. This should include piping output from a command to the script,redirecting a file into the script, or just passing a filename, or list of filenames, to the
script on the command line.
##### Solution
Python’s built-in `fileinput` module makes this very simple and concise. If you have a script that looks like this:
``` py
#!/usr/bin/env python3
import fileinput
with fileinput.input() as f_input:
    for line in f_input:
        print(line, end='')
```
Then you can already accept input to the script in all of the previously mentioned ways. If you save this script as filein.py and make it executable, you can do all of the following and get the expected output:
``` bash
$ ls | ./filein.py
# Prints a directory listing to stdout.
$ ./filein.py /etc/passwd # Reads /etc/passwd to stdout.
$ ./filein.py < /etc/passwd # Reads /etc/passwd to stdout.
```
##### Discussion
The `fileinput.input()` function creates and returns an instance of the FileInput class. In addition to containing a few handy helper methods, the instance can also be used as a context manager. So, to put all of this together, if we wrote a script that expected to be printing output from several files at once, we might have it include the filename and line number in the output, like this:
``` py
>>> import fileinput
>>> with fileinput.input('/etc/passwd') as f:
>>>     for line in f:
...         print(f.filename(), f.lineno(), line, end='')
...
/etc/passwd 1 ##
/etc/passwd 2 # User Database
/etc/passwd 3 #
<other output omitted>
```
Using it as a context manager ensures that the file is closed when it’s no longer being used, and we leveraged a few handy FileInput helper methods here to get some extra information in the output.

## Terminating a Program with an Error Message
##### Problem
You want your program to terminate by printing a message to standard error and returning a nonzero status code.
##### Solution
To have a program terminate in this manner, raise a __SystemExit__ exception, but supply the error message as an argument.

For example:
``` py
raise SystemExit('It failed!')
```
This will cause the supplied message to be printed to `sys.stderr` and the program to exit with a status code of 1.
##### Discussion
This is a small recipe, but it solves a common problem that arises when writing scripts. 

Namely, to terminate a program, you might be inclined to write code like this:
``` py
import sys
sys.stderr.write('It failed!\n')
raise SystemExit(1)
```
None of the extra steps involving import or writing to sys.stderr are neccessary if you simply supply the message to `SystemExit()` instead.

## Parsing Command-Line Options
##### Problem
You want to write a program that parses options supplied on the command line (found in `sys.argv` ).
##### Solution
The `argparse` module can be used to parse command-line options. A simple example will help to illustrate the essential features:
``` py
# search.py
'''
Hypothetical command-line tool for searching a collection of files for one or more text patterns.
'''
import argparse
parser = argparse.ArgumentParser(description='Search some files')
parser.add_argument(dest='filenames',metavar='filename', nargs='*')
parser.add_argument('-p', '--pat',metavar='pattern', required=True,
                    dest='patterns', action='append',
                    help='text pattern to search for')

parser.add_argument('-v', dest='verbose', action='store_true',
                    help='verbose mode')
parser.add_argument('-o', dest='outfile', action='store',
                    help='output file')
parser.add_argument('--speed', dest='speed', action='store',
                    choices={'slow','fast'}, default='slow',
                    help='search speed')
args = parser.parse_args()
# Output the collected arguments
print(args.filenames)
print(args.patterns)
print(args.verbose)
print(args.outfile)
print(args.speed)
```
This program defines a command-line parser with the following usage:
``` bash
python3 search.py -h
usage: search.py [-h] [-p pattern] [-v] [-o OUTFILE] [--speed {slow,fast}]
                [filename [filename ...]]
Search some files

positional arguments:
  filename
optional arguments:
  -h, --help                show this help message and exit
  -p pattern, --pat pattern
                            text pattern to search for
  -v                        verbose mode
  -o OUTFILE                output file
  --speed {slow,fast}       search speed
```
The following session shows how data shows up in the program. Carefully observe the output of the print() statements.
``` bash
python3 search.py foo.txt bar.txt
usage: search.py [-h] -p pattern [-v] [-o OUTFILE] [--speed {fast,slow}]
                 [filename [filename ...]]
search.py: error: the following arguments are required: -p/--pat

python3 search.py -v -p spam --pat=eggs foo.txt bar.txt
filenames = ['foo.txt', 'bar.txt']
patterns  = ['spam', 'eggs']
verbose   = True
outfile   = None
speed     = slow

python3 search.py -v -p spam --pat=eggs foo.txt bar.txt -o results
filenames = ['foo.txt', 'bar.txt']
patterns  = ['spam', 'eggs']
verbose   = True
outfile   = results
speed     = slow

python3 search.py -v -p spam --pat=eggs foo.txt bar.txt -o results --speed=fast
filenames = ['foo.txt', 'bar.txt']
patterns  = ['spam', 'eggs']
verbose   = True
outfile   = results
speed     = fast
```
Further processing of the options is up to the program. Replace the print() functions with something more interesting.
##### Discussion
The `argparse` module is one of the largest modules in the standard library, and has a huge number of configuration options. This recipe shows an essential subset that can be used and extended to get started.

To parse options, you first create an ArgumentParser instance and add declarations for the options you want to support it using the `add_argument()` method. In each `add_argument()` call, the dest argument specifies the name of an attribute where the result of parsing will be placed. The __metavar__ argument is used when generating help messages.The action argument specifies the processing associated with the argument and is often store for storing a value or append for collecting multiple argument values into a list. The following argument collects all of the extra command-line arguments into a list. It’s being used to make a list of filenames in the example:
``` py
parser.add_argument(dest='filenames',metavar='filename', nargs='*')
```
The following argument sets a Boolean flag depending on whether or not the argument was provided:
``` py
parser.add_argument('-v', dest='verbose', action='store_true',
                    help='verbose mode')
```
The following argument takes a single value and stores it as a string:
``` py
parser.add_argument('-o', dest='outfile', action='store',
                    help='output file')
```
The following argument specification allows an argument to be repeated multiple times and all of the values append into a list. The required flag means that the argument must be supplied at least once. The use of __-p__ and __--pat__ mean that either argument name is acceptable.
``` py
parser.add_argument('-p', '--pat',metavar='pattern', required=True,
                    dest='patterns', action='append',
                    help='text pattern to search for')
```
Finally, the following argument specification takes a value, but checks it against a set of possible choices.
``` py
parser.add_argument('--speed', dest='speed', action='store',
                    choices={'slow','fast'}, default='slow',
                    help='search speed')
```
Once the options have been given, you simply execute the `parser.parse()` method. This will process the `sys.argv` value and return an instance with the results. The results for each argument are placed into an attribute with the name given in the dest parameter to `add_argument()` .

There are several other approaches for parsing command-line options.

For example, you might be inclined to manually process `sys.argv` yourself or use the `getopt` module (which is modeled after a similarly named C library). However, if you take this approach, you’ll simply end up replicating much of the code that argparse already provides.

You may also encounter code that uses the `optparse` library to parse options. Although `optparse` is very similar to `argparse` , the latter is more modern and should be preferred in new projects.

## Prompting for a Password at Runtime
##### Problem
You’ve written a script that requires a password, but since the script is meant for interactive use, you’d like to prompt the user for a password rather than hardcode it into the script.
##### Solution
Python’s `getpass` module is precisely what you need in this situation. It will allow you to very easily prompt for a password without having the keyed-in password displayed on the user’s terminal.

Here’s how it’s done:
``` py
import getpass

user = getpass.getuser()
passwd = getpass.getpass()

if svc_login(user, passwd):     # You must write svc_login()
    print('Yay!')
else:
    print('Boo!')
```
In this code, the `svc_login()` function is code that you must write to further process the password entry. Obviously, the exact handling is application-specific.
##### Discussion
> Note in the preceding code that `getpass.getuser()` doesn’t prompt the user for their username. Instead, it uses the current user’s login name, according to the user’s shell environment, or as a last resort, according to the local system’s password database (on platforms that support the `pwd` module).

If you want to explicitly prompt the user for their username, which can be more reliable, use the built-in `input` function:
``` py
user = input('Enter your username: ')
```
It’s also important to remember that some systems may not support the hiding of the typed password input to the `getpass()` method. In this case, Python does all it can to forewarn you of problems (i.e., it alerts you that passwords will be shown in cleartext) before moving on.

## Getting the Terminal Size
##### Problem
You need to get the terminal size in order to properly format the output of your program.
##### Solution
Use the `os.get_terminal_size()` function to do this:
``` py
>>> import os
>>> sz = os.get_terminal_size()
>>> sz
os.terminal_size(columns=80, lines=24)
>>> sz.columns
80
>>> sz.lines
24
>>>
```
##### Discussion
There are many other possible approaches for obtaining the terminal size, ranging from reading environment variables to executing low-level system calls involving `ioctl()` and TTYs.

Frankly, why would you bother with that when this one simple call will
suffice?
## Executing an External Command and Getting Its Output
##### Problem
You want to execute an external command and collect its output as a Python string.
##### Solution
Use the `subprocess.check_output()` function. For example:
``` py
import subprocess
out_bytes = subprocess.check_output(['netstat','-a'])
```
This runs the specified command and returns its output as a byte string. If you need to interpret the resulting bytes as text, add a further decoding step.

For example:
``` py
out_text = out_bytes.decode('utf-8')
```
If the executed command returns a nonzero exit code, an exception is raised. Here is an example of catching errors and getting the output created along with the exit code:
``` py
try:
    out_bytes = subprocess.check_output(['cmd','arg1','arg2'])
except subprocess.CalledProcessError as e:
    out_bytes = e.output        # Output generated before error
    code = e.returncode         # Return code
```
By default, `check_output()` only returns output written to standard output. If you want both standard output and error collected, use the stderr argument:
``` py
out_bytes = subprocess.check_output(['cmd','arg1','arg2'],
                                    stderr=subprocess.STDOUT)
```
If you need to execute a command with a timeout, use the timeout argument:
``` py
try:
    out_bytes = subprocess.check_output(['cmd','arg1','arg2'], timeout=5)
except subprocess.TimeoutExpired as e:
    ...
```
Normally, commands are executed without the assistance of an underlying shell (e.g., sh , bash , etc.). Instead, the list of strings supplied are given to a low-level system command, such as `os.execve()` .

If you want the command to be interpreted by a shell, supply it using a simple string and give the __shell=True__ argument. This is sometimes useful if you’re trying to get Python to execute a complicated shell command involving
pipes, I/O redirection, and other features. For example:
``` py
out_bytes = subprocess.check_output('grep python | wc > out', shell=True)
```
!> Be aware that executing commands under the shell is a potential security risk if arguments are derived from user input. The `shlex.quote()` function can be used to properly quote arguments for inclusion in shell commands in this case.
##### Discussion
The `check_output()` function is the easiest way to execute an external command and get its output. However, if you need to perform more advanced communication with a subprocess, such as sending it input, you’ll need to take a difference approach. For that, use the `subprocess.Popen` class directly. 

For example:
``` py
import subprocess
# Some text to send
text = b'''
hello world
this is a test
goodbye
'''
# Launch a command with pipes
p = subprocess.Popen(['wc'],
                    stdout = subprocess.PIPE,
                    stdin = subprocess.PIPE)
# Send the data and get the output
stdout, stderr = p.communicate(text)
# To interpret as text, decode
out = stdout.decode('utf-8')
err = stderr.decode('utf-8')
```
The subprocess module is not suitable for communicating with external commands that expect to interact with a proper TTY. For example, you can’t use it to automate tasks that ask the user to enter a password (e.g., a ssh session). For that, you would need to turn to a third-party module, such as those based on the popular “expect” family of tools (e.g., pexpect or similar).

## Copying or Moving Files and Directories
##### Problem
You need to copy or move files and directories around, but you don’t want to do it by calling out to shell commands.
##### Solution
The `shutil` module has portable implementations of functions for copying files and directories. The usage is extremely straightforward. For example:
``` py
import shutil
# Copy src to dst. (cp src dst)
shutil.copy(src, dst)
# Copy files, but preserve metadata (cp -p src dst)
shutil.copy2(src, dst)
# Copy directory tree (cp -R src dst)
shutil.copytree(src, dst)
# Move src to dst (mv src dst)
shutil.move(src, dst)
```
The arguments to these functions are all strings supplying file or directory names. The underlying semantics try to emulate that of similar Unix commands, as shown in the comments.

By default, symbolic links are followed by these commands. For example, if the source file is a symbolic link, then the destination file will be a copy of the file the link points to. If you want to copy the symbolic link instead, supply the `follow_symlinks` keyword argument like this:
``` py
shutil.copy2(src, dst, follow_symlinks=False)
```
If you want to preserve symbolic links in copied directories, do this:
``` py
shutil.copytree(src, dst, symlinks=True)
```
The `copytree()` optionally allows you to ignore certain files and directories during the copy process. To do this, you supply an ignore function that takes a directory name and filename listing as input, and returns a list of names to ignore as a result.

For example:
``` py
def ignore_pyc_files(dirname, filenames):
    return [name in filenames if name.endswith('.pyc')]

shutil.copytree(src, dst, ignore=ignore_pyc_files)
```
Since ignoring filename patterns is common, a utility function `ignore_patterns()` has already been provided to do it.

For example:
``` py
shutil.copytree(src, dst, ignore=shutil.ignore_patterns('*~','*.pyc'))
```
##### Discussion
Using `shutil` to copy files and directories is mostly straightforward. However, one caution concerning file metadata is that functions such as copy2() only make a best effort in preserving this data. Basic information, such as access times, creation times,and permissions, will always be preserved, but preservation of owners, ACLs, resource forks, and other extended file metadata may or may not work depending on the underlying operating system and the user’s own access permissions.

!> You probably wouldn’t want to use a function like `shutil.copytree()` to perform system backups.

When working with filenames, make sure you use the functions in `os.path` for the greatest portability (especially if working with both Unix and Windows). 

For example:
``` py
>>> filename = '/Users/guido/programs/spam.py'
>>> import os.path
>>> os.path.basename(filename)
'spam.py'
>>> os.path.dirname(filename)
'/Users/guido/programs'
>>> os.path.split(filename)
('/Users/guido/programs', 'spam.py')
>>> os.path.join('/new/dir', os.path.basename(filename))
'/new/dir/spam.py'
>>> os.path.expanduser('~/guido/programs/spam.py')
'/Users/guido/programs/spam.py'
>>>
```
One tricky bit about copying directories with `copytree()` is the handling of errors. For example, in the process of copying, the function might encounter broken symbolic links, files that can’t be accessed due to permission problems, and so on. To deal with this, all exceptions encountered are collected into a list and grouped into a single exception that gets raised at the end of the operation. Here is how you would handle it:
``` py
try:
    shutil.copytree(src, dst)
except shutil.Error as e:
    for src, dst, msg in e.args[0]:
        # src is source name
        # dst is destination name
        # msg is error message from exception
        print(dst, src, msg)
```
If you supply the __ignore_dangling_symlinks=True__ keyword argument, then `copytree()` will ignore dangling symlinks.

The functions shown in this recipe are probably the most commonly used. However,`shutil` has many more operations related to copying data. The documentation is definitely worth a further look. See the Python documentation.

## Creating and Unpacking Archives
##### Problem
You need to create or unpack archives in common formats (e.g., .tar, .tgz, or .zip).
##### Solution
- The `shutil` module has two functions:
 - make_archive()
 - unpack_archive()

For example:
``` py
>>> import shutil
>>> shutil.unpack_archive('Python-3.3.0.tgz')
>>> shutil.make_archive('py33','zip','Python-3.3.0')
'/Users/beazley/Downloads/py33.zip'
>>>
```
The second argument to `make_archive()` is the desired output format. To get a list of supported archive formats, use `get_archive_formats()` .

For example:
``` py
>>> shutil.get_archive_formats()
[('bztar', "bzip2'ed tar-file"), ('gztar', "gzip'ed tar-file"),
('tar', 'uncompressed tar file'), ('zip', 'ZIP file')]
>>>
```
##### Discussion
Python has other library modules for dealing with the low-level details of various archive formats (e.g., tarfile , zipfile , gzip , bz2 , etc.). However, if all you’re trying to do is make or extract an archive, there’s really no need to go so low level. You can just use these high-level functions in `shutil` instead.

The functions have a variety of additional options for logging, dryruns, file permissions, and so forth. Consult the shutil library documentation for further details.
## Finding Files by Name
##### Problem
You need to write a script that involves finding files, like a file renaming script or a log archiver utility, but you’d rather not have to call shell utilities from within your Python script, or you want to provide specialized behavior not easily available by “shelling out.”
##### Solution
To search for files, use the `os.walk()` function, supplying it with the top-level directory.

Here is an example of a function that finds a specific filename and prints out the full path of all matches:
``` py
#!/usr/bin/env python3.3
import os
def findfile(start, name):
    for relpath, dirs, files in os.walk(start):
        if name in files:
            full_path = os.path.join(start, relpath, name)
            print(os.path.normpath(os.path.abspath(full_path)))
if __name__ == '__main__':
    findfile(sys.argv[1], sys.argv[2])
```
Save this script as findfile.py and run it from the command line, feeding in the starting point and the name as positional arguments, like this:
``` bash
./findfile.py . myfile.txt
```
##### Discussion
The `os.walk()` method traverses the directory hierarchy for us, and for each directory it enters, it returns a 3-tuple, containing the relative path to the directory it’s inspecting,a list containing all of the directory names in that directory, and a list of filenames in that directory.

For each tuple, you simply check if the target filename is in the files list. If it is, `os.path.join()` is used to put together a path. To avoid the possibility of weird looking paths like ././foo//bar, two additional functions are used to fix the result. The first is `os.path.abspath()` , which takes a path that might be relative and forms the absolute path, and the second is `os.path.normpath()` , which will normalize the path, thereby resolving issues with double slashes, multiple references to the current directory, and so on.

Although this script is pretty simple compared to the features of the find utility found on UNIX platforms, it has the benefit of being cross-platform. Furthermore, a lot of additional functionality can be added in a portable manner without much more work.

To illustrate, here is a function that prints out all of the files that have a recent modification time:
``` py
#!/usr/bin/env python3.3
import os
import time
def modified_within(top, seconds):
    now = time.time()
    for path, dirs, files in os.walk(top):
        for name in files:
            fullpath = os.path.join(path, name)
            if os.path.exists(fullpath):
                mtime = os.path.getmtime(fullpath)
                if mtime > (now - seconds):
                    print(fullpath)

if __name__ == '__main__':
    import sys
    if len(sys.argv) != 3:
        print('Usage: {} dir seconds'.format(sys.argv[0]))
        raise SystemExit(1)
    modified_within(sys.argv[1], float(sys.argv[2]))
```
It wouldn’t take long for you to build far more complex operations on top of this little function using various features of the os , os.path , glob , and similar modules.

## Reading Configuration Files
##### Problem
You want to read configuration files written in the common .ini configuration file format.
##### Solution
The `configparser` module can be used to read configuration files. For example, suppose you have this configuration file:
```
; config.ini
; Sample configuration file
[installation]
library=%(prefix)s/lib
include=%(prefix)s/include
bin=%(prefix)s/bin
prefix=/usr/local
# Setting related to debug configuration
[debug]
log_errors=true
show_warnings=False
[server]
port: 8080
nworkers: 32
pid-file=/tmp/spam.pid
root=/www/root
signature:
    =================================
    Brought to you by the Python Cookbook
    =================================
```
Here is an example of how to read it and extract values:
``` py
>>> from configparser import ConfigParser
>>> cfg = ConfigParser()
>>> cfg.read('config.ini')
['config.ini']
>>> cfg.sections()
['installation', 'debug', 'server']
>>> cfg.get('installation','library')
'/usr/local/lib'
>>> cfg.getboolean('debug','log_errors')
True
>>> cfg.getint('server','port')
8080
>>> cfg.getint('server','nworkers')
32
>>> print(cfg.get('server','signature'))
=================================
Brought to you by the Python Cookbook
=================================
>>>
```
If desired, you can also modify the configuration and write it back to a file using the `cfg.write()` method. For example:
``` py
>>> cfg.set('server','port','9000')
>>> cfg.set('debug','log_errors','False')
>>> import sys
>>> cfg.write(sys.stdout)
[installation]
library = %(prefix)s/lib
include = %(prefix)s/include
bin = %(prefix)s/bin
prefix = /usr/local
[debug]
log_errors = False
show_warnings = False
[server]
port = 9000
nworkers = 32
pid-file = /tmp/spam.pid
root = /www/root
signature =
    =================================
    Brought to you by the Python Cookbook
    =================================
>>>
```
##### Discussion
Configuration files are well suited as a human-readable format for specifying configuration data to your program. Within each config file, values are grouped into different sections (e.g., “installation,” “debug,” and “server,” in the example). Each section then specifies values for various variables in that section.

There are several notable differences between a config file and using a Python source file for the same purpose. First, the syntax is much more permissive and “sloppy.” For example, both of these assignments are equivalent:
```
prefix=/usr/local
prefix: /usr/local
```
The names used in a config file are also assumed to be case-insensitive.

For example:
``` py
>>> cfg.get('installation','PREFIX')
'/usr/local'
>>> cfg.get('installation','prefix')
'/usr/local'
>>>
```
When parsing values, methods such as `getboolean()` look for any reasonable value.
For example, these are all equivalent:
``` py
log_errors = true
log_errors = TRUE
log_errors = Yes
log_errors = 1
```
Perhaps the most significant difference between a config file and Python code is that, unlike scripts, configuration files are not executed in a top-down manner. Instead, the file is read in its entirety. If variable substitutions are made, they are done after the fact.

For example, in this part of the config file, it doesn’t matter that the prefix variable is assigned after other variables that happen to use it:
```
[installation]
library=%(prefix)s/lib
include=%(prefix)s/include
bin=%(prefix)s/bin
prefix=/usr/local
```
An easily overlooked feature of `ConfigParser` is that it can read multiple configuration files together and merge their results into a single configuration. 

For example, suppose a user made their own configuration file that looked like this:
``` 
; ~/.config.ini
[installation]
prefix=/Users/beazley/test
[debug]
log_errors=False
```
This file can be merged with the previous configuration by reading it separately.

For example:
``` py
>>> # Previously read configuration
>>> cfg.get('installation', 'prefix')
'/usr/local'
>>> # Merge in user-specific configuration
>>> import os
>>> cfg.read(os.path.expanduser('~/.config.ini'))
['/Users/beazley/.config.ini']
>>> cfg.get('installation', 'prefix')
'/Users/beazley/test'
>>> cfg.get('installation', 'library')
'/Users/beazley/test/lib'
>>> cfg.getboolean('debug', 'log_errors')
False
>>>
```
Observe how the override of the prefix variable affects other related variables, such as the setting of library . This works because variable interpolation is performed as late as possible. You can see this by trying the following experiment:
``` py
>>> cfg.get('installation','library')
'/Users/beazley/test/lib'
>>> cfg.set('installation','prefix','/tmp/dir')
>>> cfg.get('installation','library')
'/tmp/dir/lib'
>>>
```
Finally, it’s important to note that Python does not support the full range of features you might find in an .ini file used by other programs (e.g., applications on Windows). Make sure you consult the configparser documentation for the finer details of the syntax and supported features.

## Adding Logging to Simple Scripts
##### Problem
You want scripts and simple programs to write diagnostic information to log files.
##### Solution
The easiest way to add logging to simple programs is to use the `logging` module.

For example:
``` py
import logging
def main():
    # Configure the logging system
    logging.basicConfig(
    filename='app.log',
    level=logging.ERROR)
    # Variables (to make the calls that follow work)
    hostname = 'www.python.org'
    item = 'spam'
    filename = 'data.csv'
    mode = 'r'
    # Example logging calls (insert into your program)
    logging.critical('Host %s unknown', hostname)
    logging.error("Couldn't find %r", item)
    logging.warning('Feature is deprecated')
    logging.info('Opening file %r, mode=%r', filename, mode)
    logging.debug('Got here')
if __name__ == '__main__':
    main()
```
The five logging calls ( critical() , error() , warning() , info() , debug() ) represent different severity levels in decreasing order. The level argument to `basicConfig()` is a filter. All messages issued at a level lower than this setting will be ignored.

The argument to each logging operation is a message string followed by zero or more arguments. When making the final log message, the % operator is used to format the message string using the supplied arguments.

If you run this program, the contents of the file app.log will be as follows:
``` 
CRITICAL:root:Host www.python.org unknown
ERROR:root:Could not find 'spam'
```
If you want to change the output or level of output, you can change the parameters to the `basicConfig()` call.

For example:
``` py
logging.basicConfig(
    filename='app.log',
    level=logging.WARNING,
    format='%(levelname)s:%(asctime)s:%(message)s')
```
As a result, the output changes to the following:
```
CRITICAL:2012-11-20 12:27:13,595:Host www.python.org unknown
ERROR:2012-11-20 12:27:13,595:Could not find 'spam'
WARNING:2012-11-20 12:27:13,595:Feature is deprecated
```
As shown, the logging configuration is hardcoded directly into the program. If you want to configure it from a configuration file, change the ‍ call to the following:
``` py
import logging
import logging.config
def main():
    # Configure the logging system
    logging.config.fileConfig('logconfig.ini')
    ...
```
Now make a configuration file `logconfig.ini` that looks like this:
```
[loggers]
keys=root

[handlers]
keys=defaultHandler

[formatters]
keys=defaultFormatter

[logger_root]
level=INFO
handlers=defaultHandler
qualname=root

[handler_defaultHandler]
class=FileHandler
formatter=defaultFormatter
args=('app.log', 'a')

[formatter_defaultFormatter]
format=%(levelname)s:%(name)s:%(message)s
```
If you want to make changes to the configuration, you can simply edit the logconfig.ini file as appropriate.
##### Discussion
Ignoring for the moment that there are about a million advanced configuration options for the logging module, this solution is quite sufficient for simple programs and scripts.

Simply make sure that you execute the `basicConfig()` call prior to making any logging calls, and your program will generate logging output.

If you want the logging messages to route to standard error instead of a file, don’t supply any filename information to `basicConfig()` .

For example, simply do this:
``` py
logging.basicConfig(level=logging.INFO)
```
One subtle aspect of `basicConfig()` is that it can only be called once in your program.If you later need to change the configuration of the logging module, you need to obtain the root logger and make changes to it directly. 

For example:
``` py
logging.getLogger().level = logging.DEBUG
```
> It must be emphasized that this recipe only shows a basic use of the logging module. There are significantly more advanced customizations that can be made. An excellent resource for such customization is the “Logging Cookbook”.
## Adding Logging to Libraries
##### Problem
You would like to add a logging capability to a library, but don’t want it to interfere with programs that don’t use logging.
##### Solution
For libraries that want to perform logging, you should create a dedicated logger object, and initially configure it as follows:
``` py
# somelib.py
import logging
log = logging.getLogger(__name__)
log.addHandler(logging.NullHandler())
# Example function (for testing)
def func():
    log.critical('A Critical Error!')
    log.debug('A debug message')
```
With this configuration, no logging will occur by default.

For example:
``` py
>>> import somelib
>>> somelib.func()
>>>
```
However, if the logging system gets configured, log messages will start to appear.

For example:
``` py
>>> import logging
>>> logging.basicConfig()
>>> somelib.func()
CRITICAL:somelib:A Critical Error!
>>>
```
##### Discussion
!> Libraries present a special problem for logging, since information about the environment in which they are used isn’t known. As a general rule, you should never write library code that tries to configure the logging system on its own or which makes assumptions about an already existing logging configuration. Thus, you need to take great care to provide isolation.

The call to __getLogger(\_\_name\_\_)__ creates a logger module that has the same name as the calling module. Since all modules are unique, this creates a dedicated logger that is likely to be separate from other loggers.

The `log.addHandler(logging.NullHandler())` operation attaches a null handler to the just created logger object. A null handler ignores all logging messages by default. Thus, if the library is used and logging is never configured, no messages or warnings will appear.

One subtle feature of this recipe is that the logging of individual libraries can be independently configured, regardless of other logging settings.

For example, consider the following code:
``` py
>>> import logging
>>> logging.basicConfig(level=logging.ERROR)
>>> import somelib
>>> somelib.func()
CRITICAL:somelib:A Critical Error!
>>> # Change the logging level for 'somelib' only
>>> logging.getLogger('somelib').level=logging.DEBUG
>>> somelib.func()
CRITICAL:somelib:A Critical Error!
DEBUG:somelib:A debug message
>>>
```
Here, the root logger has been configured to only output messages at the ERROR level or higher. However, the level of the logger for somelib has been separately configured to output debugging messages. That setting takes precedence over the global setting.The ability to change the logging settings for a single module like this can be a useful debugging tool, since you don’t have to change any of the global logging settings—simply change the level for the one module where you want more output.

The “Logging HOWTO” has more information about configuring the logging module and other useful tips.

## Making a Stopwatch Timer
##### Problem
You want to be able to record the time it takes to perform various tasks.
##### Solution
The `time` module contains various functions for performing timing-related functions. However, it’s often useful to put a higher-level interface on them that mimics a stop watch.

For example:
``` py
import time
class Timer:
    def __init__(self, func=time.perf_counter):
        self.elapsed = 0.0
        self._func = func
        self._start = None
    def start(self):
        if self._start is not None:
            raise RuntimeError('Already started')
        self._start = self._func()
    def stop(self):
        if self._start is None:
            raise RuntimeError('Not started')
        end = self._func()
        self.elapsed += end - self._start
        self._start = None
    def reset(self):
        self.elapsed = 0.0
    @property
    def running(self):
        return self._start is not None
    def __enter__(self):
        self.start()
        return self
    def __exit__(self, *args):
        self.stop()
```
This class defines a timer that can be started, stopped, and reset as needed by the user. It keeps track of the total elapsed time in the elapsed attribute. Here is an example that shows how it can be used:
``` py
def countdown(n):
    while n > 0:
        n -= 1
# Use 1: Explicit start/stop
t = Timer()
t.start()
countdown(1000000)
t.stop()
print(t.elapsed)
# Use 2: As a context manager
with t:
    countdown(1000000)
print(t.elapsed)

with Timer() as t2:
    countdown(1000000)
print(t2.elapsed)
```
##### Discussion
This recipe provides a simple yet very useful class for making timing measurements and tracking elapsed time. It’s also a nice illustration of how to support the context-management protocol and the with statement.

One issue in making timing measurements concerns the underlying time function used to do it. As a general rule, the accuracy of timing measurements made with functions such as `time.time()` or `time.clock()` varies according to the operating system. In contrast, the `time.perf_counter()` function always uses the highest-resolution timer available on the system.As shown, the time recorded by the Timer class is made according to wall-clock time,and includes all time spent sleeping.

If you only want the amount of CPU time used by the process, use `time.process_time()` instead.

For example:
``` py
t = Timer(time.process_time)
with t:
    countdown(1000000)
print(t.elapsed)
```
Both the `time.perf_counter()` and `time.process_time()` return a “time” in fractional seconds. However, the actual value of the time doesn’t have any particular meaning. To make sense of the results, you have to call the functions twice and compute a time difference.


## Putting Limits on Memory and CPU Usage
##### Problem
You want to place some limits on the memory or CPU use of a program running on Unix system.
##### Solution
The `resource` module can be used to perform both tasks. For example, to restrict CPU time, do the following:
```py
import signal
import resource
import os
def time_exceeded(signo, frame):
    print("Time's up!")
    raise SystemExit(1)
def set_max_runtime(seconds):
    # Install the signal handler and set a resource limit
    soft, hard = resource.getrlimit(resource.RLIMIT_CPU)
    resource.setrlimit(resource.RLIMIT_CPU, (seconds, hard))
    signal.signal(signal.SIGXCPU, time_exceeded)
if __name__ == '__main__':
    set_max_runtime(15)
    while True:
        pass
```
When this runs, the __SIGXCPU__ signal is generated when the time expires. The program can then clean up and exit.

To restrict memory use, put a limit on the total address space in use.

For example:
``` py
import resource
def limit_memory(maxsize):
    soft, hard = resource.getrlimit(resource.RLIMIT_AS)
    resource.setrlimit(resource.RLIMIT_AS, (maxsize, hard))
```
With a memory limit in place, programs will start generating MemoryError exceptions when no more memory is available.
##### Discussion
In this recipe, the `setrlimit()` function is used to set a soft and hard limit on a particular resource.
- The soft limit is a value upon which the operating system will typically restrict or notify the process via a signal.
- The hard limit represents an upper bound on the values that may be used for the soft limit.

Typically, this is controlled by a system-wide parameter set by the system administrator. Although the hard limit can be lowered, it can never be raised by user processes (even if the process lowered itself).

The `setrlimit()` function can additionally be used to set limits on things such as the number of child processes, number of open files, and similar system resources.

!> Consult the documentation for the resource module for further details.Be aware that this recipe only works on Unix systems, and that it might not work on all of them. For example, when tested, it works on Linux but not on OS X.
## Launching a Web Browser
##### Problem
You want to launch a browser from a script and have it point to some URL that you specify.
##### Solution
The webbrowser module can be used to launch a browser in a platform-independent
manner.

For example:
``` py
>>> import webbrowser
>>> webbrowser.open('http://www.python.org')
True
>>>
```
This opens the requested page using the default browser. If you want a bit more control over how the page gets opened, you can use one of the following functions:
``` py
>>> # Open the page in a new browser window
>>> webbrowser.open_new('http://www.python.org')
True
>>>
>>> # Open the page in a new browser tab
>>> webbrowser.open_new_tab('http://www.python.org')
True
>>>
```
These will try to open the page in a new browser window or tab, if possible and supported by the browser.

If you want to open a page in a specific browser, you can use the `webbrowser.get()` function to specify a particular browser.

For example:
``` py
>>> c = webbrowser.get('firefox')
>>> c.open('http://www.python.org')
True
>>> c.open_new_tab('http://docs.python.org')
True
>>>
```
> A full list of supported browser names can be found in the Python documentation.
##### Discussion
Being able to easily launch a browser can be a useful operation in many scripts. For example, maybe a script performs some kind of deployment to a server and you’d like to have it quickly launch a browser so you can verify that it’s working. Or maybe a program writes data out in the form of HTML pages and you’d just like to fire up a browser to see the result. Either way, the webbrowser module is a simple solution.