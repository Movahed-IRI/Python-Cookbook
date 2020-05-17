## Reading and Writing CSV Data
##### Problem
You want to read or write data encoded as a CSV file.
##### Solution
For most kinds of CSV data, use the `csv` library. For example, suppose you have some stock market data in a file named stocks.csv like this:
``` py
Symbol,Price,Date,Time,Change,Volume
"AA",39.48,"6/11/2007","9:36am",-0.18,181800
"AIG",71.38,"6/11/2007","9:36am",-0.15,195500
"AXP",62.58,"6/11/2007","9:36am",-0.46,935000
"BA",98.31,"6/11/2007","9:36am",+0.12,104800
"C",53.08,"6/11/2007","9:36am",-0.25,360900
"CAT",78.29,"6/11/2007","9:36am",-0.23,225400
```
Here’s how you would read the data as a sequence of tuples:
``` py
import csv
with open('stocks.csv') as f:
    f_csv = csv.reader(f)
    headers = next(f_csv)
for row in f_csv:
    # Process row
...
```

?> In the preceding code, row will be a tuple. Thus, to access certain fields, you will need to use indexing.Since such indexing can often be confusing, this is one place where you might want to consider the use of named tuples.

For example:
``` py
from collections import namedtuple
with open('stock.csv') as f:
    f_csv = csv.reader(f)
    headings = next(f_csv)
    Row = namedtuple('Row', headings)
    for r in f_csv:
        row = Row(*r)
        # Process row
        ...
```

?> This would allow you to use the column headers such as `row.Symbol` and `row.Change` instead of indices.

!> It should be noted that this only works if the column headers are valid Python identifiers. If not, you might have to massage the initial headings (e.g., replacing nonidentifier characters with underscores or similar).

Another alternative is to read the data as a sequence of dictionaries instead. To do that, use this code:

``` py
import csv
with open('stocks.csv') as f:
    f_csv = csv.DictReader(f)
for row in f_csv:
# process row
...
```

In this version, you would access the elements of each row using the row headers. For example, `row['Symbol']` or `row['Change']` .

- To write CSV data, you also use the `csv` module but create a `writer` object. For example:

``` py
headers = ['Symbol','Price','Date','Time','Change','Volume']
rows = [('AA', 39.48, '6/11/2007', '9:36am', -0.18, 181800),
        ('AIG', 71.38, '6/11/2007', '9:36am', -0.15, 195500),
        ('AXP', 62.58, '6/11/2007', '9:36am', -0.46, 935000),
        ]
with open('stocks.csv','w') as f:
    f_csv = csv.writer(f)
    f_csv.writerow(headers)
    f_csv.writerows(rows)
```

If you have the data as a sequence of dictionaries, do this:

``` py
headers = ['Symbol', 'Price', 'Date', 'Time', 'Change', 'Volume']
rows = [{'Symbol':'AA', 'Price':39.48, 'Date':'6/11/2007','Time':'9:36am', 'Change':-0.18, 'Volume':181800},
        {'Symbol':'AIG', 'Price': 71.38, 'Date':'6/11/2007','Time':'9:36am', 'Change':-0.15, 'Volume': 195500},
        {'Symbol':'AXP', 'Price': 62.58, 'Date':'6/11/2007','Time':'9:36am', 'Change':-0.46, 'Volume': 935000},
        ]
with open('stocks.csv','w') as f:
    f_csv = csv.DictWriter(f, headers)
    f_csv.writeheader()
    f_csv.writerows(rows)
```

##### Discussion
You should almost always prefer the use of the `csv` module over manually trying to split and parse CSV data yourself. For instance, you might be inclined to just write some code like this:
``` py
with open('stocks.csv') as f:
    for line in f:
        row = line.split(',')
# process row
...
```

?> By default, the `csv` library is programmed to understand CSV encoding rules used by Microsoft Excel. This is probably the most common variant, and will likely give you thebest compatibility.

If you want to read tab-delimited data instead, use this:
``` py
# Example of reading tab-separated values
with open('stock.tsv') as f:
    f_tsv = csv.reader(f, delimiter='\t')
for row in f_tsv:
    # Process row
...
```
A CSV file could have a header line containing nonvalid identifier characters like this:

``` py
Street Address,Num-Premises,Latitude,Longitude
5412 N CLARK,10,41.980262,-87.668452
```
This will actually cause the creation of a namedtuple to fail with a ValueError exception.
<br>
To work around this, you might have to scrub the headers first. For instance, carrying a regex substitution on nonvalid identifier characters like this:

``` py
import re
with open('stock.csv') as f:
    f_csv = csv.reader(f)
    headers = [ re.sub('[^a-zA-Z_]', '_', h) for h in next(f_csv) ]
    Row = namedtuple('Row', headers)
    for r in f_csv:
        row = Row(*r)
        # Process row
...
```
?> It’s also important to emphasize that csv does not try to interpret the data or convert it to a type other than a string.

Here is one example of performing extra type conversions on CSV data:

``` py
col_types = [str, float, str, str, float, int]
with open('stocks.csv') as f:
    f_csv = csv.reader(f)
    headers = next(f_csv)
    for row in f_csv:
        # Apply conversions to the row items
        row = tuple(convert(value) for convert, value in zip(col_types, row))
...
```

Alternatively, here is an example of converting selected fields of dictionaries:
``` py
import csv
print('Reading as dicts with type conversion')
field_types = [ ('Price', float),('Change', float),('Volume', int) ]

with open('stocks.csv') as f:
    for row in csv.DictReader(f):
        row.update((key, conversion(row[key])) for key, conversion in field_types)
    print(row)

```
!> In the real world, it’s common for CSV files to have missing values, corrupted data, and other issues that would break type conversions. 

Finally, if your goal in reading CSV data is to perform data analysis and statistics, you might want to look at the `Pandas` package.
`Pandas` includes a convenient `pandas.read_csv()` function that will load CSV data into a DataFrame object. From there, you can generate various summary __statistics__, __filter the data__, and perform other kinds of high-level operations.

## Reading and Writing JSON Data
##### Problem
You want to read or write data encoded as JSON (JavaScript Object Notation).
##### Solution
The `json` module provides an easy way to encode and decode data in JSON. The two main functions are `json.dumps()` and `json.loads()` , mirroring the interface used in
other serialization libraries, such as `pickle` .
<br>

- Here is how you turn a Python data structure into JSON:

``` py
import json
data = {
    'name' : 'ACME',
    'shares' : 100,
    'price' : 542.23
}
json_str = json.dumps(data)
```
- Here is how you turn a JSON-encoded string back into a Python data structure:
``` py
data = json.loads(json_str)
```
If you are working with files instead of strings, you can alternatively use `json.dump()` and `json.load()` to encode and decode JSON data.
<br>
For example:

``` py
# Writing JSON data
with open('data.json', 'w') as f:
    json.dump(data, f)
# Reading data back
with open('data.json', 'r') as f:
    data = json.load(f)
```
Discussion
JSON encoding supports the basic types of __None__ , __bool__ , __int__ , __float__ , and __str__ , as well as __lists__, __tuples__, and __dictionaries__ containing those types.

?> For dictionaries, keys are assumed to be strings (any nonstring keys in a dictionary are converted to strings when encoding). To be compliant with the JSON specification, you should only encode Python __lists__ and __dictionaries__.

?> in web applications, it is standard practice for the top-level object to be a dictionary.

- The format of JSON encoding is almost identical to Python syntax except for a few minor changes.
   - True is mapped to true
   - False is mapped to false
   - None is mapped to null

For Example : 
``` py
>>> json.dumps(False)
'false'
>>> d = {'a': True,
...     'b': 'Hello',
...     'c': None}
>>> json.dumps(d)
'{"b": "Hello", "c": null, "a": true}'
>>>
```
If you are trying to examine data you have decoded from JSON, to assist with this, consider using the `pprint()` function in the `pprint` module. This will alphabetize the keys and output a dictionary in a more sane way.

Normally, JSON decoding will create dicts or lists from the supplied data. If you want to create different kinds of objects, supply the `object_pairs_hook` or `object_hook` to `json.loads()` . 
For example:
``` py
s = '{"name": "ACME", "shares": 50, "price": 490.1}'
from collections import OrderedDict
data = json.loads(s, object_pairs_hook=OrderedDict)
print(data) #OrderedDict([('name', 'ACME'), ('shares', 50), ('price', 490.1)])
```

Here is how you could turn a JSON dictionary into a Python object:
``` py
class JSONObject:
    def __init__(self, d):
        self.__dict__ = d

data = json.loads(s, object_hook=JSONObject)
print(data.name) #'ACME'
print(data.shares) #50
print(data.price) #490.1
```

There are a few options that can be useful for encoding JSON. If you would like the output to be nicely formatted, you can use the indent argument to `json.dumps()` . This causes the output to be pretty printed in a format similar to that with the `pprint()` function. 
<br>

For example:
``` py
>>> print(json.dumps(data))
{"price": 542.23, "name": "ACME", "shares": 100}
>>> print(json.dumps(data, indent=4))
{
"price": 542.23,
"name": "ACME",
"shares": 100
}
>>>
```

If you want the keys to be sorted on output, used the `sort_keys` argument:
``` py
>>> print(json.dumps(data, sort_keys=True))
{"name": "ACME", "price": 542.23, "shares": 100}
>>>
```
Instances are not normally serializable as JSON. For example:
``` py
>>> class Point:
...     def __init__(self, x, y):
...         self.x = x
...         self.y = y
...
>>> p = Point(2, 3)
>>> json.dumps(p)
Traceback (most recent call last):
  File "/home/sedgeek/Documents/Development/pytest/test.py", line 24, in <module>
    json.dumps(p)
  File "/usr/lib/python3.6/json/__init__.py", line 231, in dumps
    return _default_encoder.encode(obj)
  File "/usr/lib/python3.6/json/encoder.py", line 199, in encode
    chunks = self.iterencode(o, _one_shot=True)
  File "/usr/lib/python3.6/json/encoder.py", line 257, in iterencode
    return _iterencode(o, 0)
  File "/usr/lib/python3.6/json/encoder.py", line 180, in default
    o.__class__.__name__)
TypeError: Object of type 'Point' is not JSON serializable
>>>
```
If you want to serialize instances, you can supply a function that takes an instance as input and returns a dictionary that can be serialized. For example:
``` py
def serialize_instance(obj):
    d = { '__classname__' : type(obj).__name__ }
    d.update(vars(obj))
    return d
```
If you want to get an instance back, you could write code like this:
``` py
# Dictionary mapping names to known classes
classes = {
'Point' : Point
}
def unserialize_object(d):
    clsname = d.pop('__classname__', None)
    if clsname:
        cls = classes[clsname]
        obj = cls.__new__(cls) # Make instance without calling __init__

        for key, value in d.items():
            setattr(obj, key, value)
            return obj
    else:
        return d
```
Here is an example of how these functions are used:

``` py
>>> p = Point(2,3)
>>> s = json.dumps(p, default=serialize_instance)
>>> s
'{"__classname__": "Point", "y": 3, "x": 2}'
>>> a = json.loads(s, object_hook=unserialize_object)
>>> a
<__main__.Point object at 0x1017577d0>
>>> a.x
2
>>> a.y
3
>>>
```
?> The json module has a variety of other options for controlling the low-level interpretation of numbers, special values such as NaN , and more. Consult the documentation for further details.
## Parsing Simple XML Data
##### Problem
You would like to extract data from a simple XML document.
##### Solution
The `xml.etree.ElementTree` module can be used to extract data from simple XML documents.
<br>
To illustrate, suppose you want to parse and make a summary of the RSS feed on Planet Python. Here is a script that will do it:

``` py
from urllib.request import urlopen
from xml.etree.ElementTree import parse
# Download the RSS feed and parse it
u = urlopen('http://planet.python.org/rss20.xml')
doc = parse(u)
# Extract and output tags of interest
for item in doc.iterfind('channel/item'):
    title = item.findtext('title')
    date = item.findtext('pubDate')
    link = item.findtext('link')
    print(title)
    print(date)
    print(link)
    print()
```
If you run the preceding script, the output looks similar to the following:
````
PyCharm: Django Custom Tags in PyCharm Professional 2020.1
Wed, 15 Apr 2020 17:04:35 +0000
http://feedproxy.google.com/~r/Pycharm/~3/1A-5crBUefo/

Jacob Perkins: NLTK Trainer Updates
Wed, 15 Apr 2020 17:00:00 +0000
http://feedproxy.google.com/~r/StreamHackerPython/~3/sZ1leRc2s3M/

````

##### Discussion
Not only is __XML__ widely used as a format for exchanging data on the Internet, it is a common format for storing application data (e.g., word processing, music libraries, etc.). 

For example, the RSS feed from the example looks similar to the following:
``` xml
<rss xmlns:dc="http://purl.org/dc/elements/1.1/" version="2.0">
    <channel>
        <title>Planet Python</title>
        <link>http://planetpython.org/</link>
        <language>en</language>
        <description>Planet Python - http://planetpython.org/</description>
        <item>
            <title>
                PyCharm: Django Custom Tags in PyCharm Professional 2020.1
            </title>
            <guid>
                http://feedproxy.google.com/~r/Pycharm/~3/1A-5crBUefo/
            </guid>
            <link>
                http://feedproxy.google.com/~r/Pycharm/~3/1A-5crBUefo/
            </link>
            <description>
                ...
            </description>
            <pubDate>
                Wed, 15 Apr 2020 17:04:35 +0000
            </pubDate>
        </item>
        <item>
            <title>
                Jacob Perkins: NLTK Trainer Updates
            </title>
            <guid>
                http://feedproxy.google.com/~r/StreamHackerPython/~3/sZ1leRc2s3M/
            </guid>
            <link>
                http://feedproxy.google.com/~r/StreamHackerPython/~3/sZ1leRc2s3M/
            </link>
            <description>
                ...
            </description>
            <pubDate>
                Wed, 15 Apr 2020 17:00:00 +0000
            </pubDate>
        </item>
    </channel>
</rss>
```
The `xml.etree.ElementTree.parse()` function parses the entire XML document into a document object.From there, you use methods such as `find()` , `iterfind()` , and `findtext()` to search for specific XML elements. The arguments to these functions are the names of a specific tag, such as channel/item or title .

Each `find` operation takes place relative to a starting element. Likewise, the tagname that you supply to each operation is also relative to the start.

Each element represented by the `ElementTree` module has a few essential attributes and methods that are useful when parsing. The tag attribute contains the name of the tag, the text attribute contains enclosed text, and the `get()` method can be used to extract attributes (if any).
<br>
For example:

``` py
>>> doc
<xml.etree.ElementTree.ElementTree object at 0x101339510>
>>> e = doc.find('channel/title')
>>> e
<Element 'title' at 0x10135b310>
>>> e.tag
'title'
>>> e.text
'Planet Python'
>>> e.get('some_attribute')
>>>
```

?> For more advanced applications, you might consider `lxml` . It uses the same programming interface as `ElementTree` , so the example shown in this recipe works in the same manner. You simply need to change the first import to from `lxml.etree` import parse .

lxml provides the benefit of being fully compliant with XML standards. It is also extremely fast, and provides support for features such as __validation__, __XSLT__, and __XPath__.

## Parsing Huge XML Files Incrementally
##### Problem
You need to extract data from a huge XML document using as little memory as possible.
##### Solution
?> Any time you are faced with the problem of incremental data processing, you should think of iterators and generators.

Here is a simple function that can be used to incrementally process huge XML files using a very small memory footprint:

``` py
from xml.etree.ElementTree import iterparse
def parse_and_remove(filename, path):
    path_parts = path.split('/')
    doc = iterparse(filename, ('start', 'end'))
    # Skip the root element
    next(doc)
    tag_stack = []
    elem_stack = []
    for event, elem in doc:
        if event == 'start':
            tag_stack.append(elem.tag)
            elem_stack.append(elem)
        elif event == 'end':
            if tag_stack == path_parts:
                yield elem
                elem_stack[-2].remove(elem)
                try:
                    tag_stack.pop()
                    elem_stack.pop()
                except IndexError:
                    pass
```
Suppose you want to write a script that ranks ZIP codes by the number of pothole reports. To do it, you could write code like this:

``` py
from xml.etree.ElementTree import parse
from collections import Counter
potholes_by_zip = Counter()
doc = parse('potholes.xml')
for pothole in doc.iterfind('row/row'):
    potholes_by_zip[pothole.findtext('zip')] += 1
for zipcode, num in potholes_by_zip.most_common():
    print(zipcode, num)
```
!> The only problem with this script is that it reads and parses the entire XML file into memory. 

Using this recipe’s code, the program changes only slightly:
``` py
from collections import Counter
potholes_by_zip = Counter()
data = parse_and_remove('potholes.xml', 'row/row')
    for pothole in data:
        potholes_by_zip[pothole.findtext('zip')] += 1
    for zipcode, num in potholes_by_zip.most_common():
        print(zipcode, num)
```

##### Discussion
- This recipe relies on two core features of the `ElementTree` module.
    - The iter `parse()` method allows incremental processing of XML documents. To use it, you supply the filename along with an event list consisting of one or more of the following:
        - start
        - end
        - start-ns
        - end-ns

    - The iterator created by `iterparse()` produces tuples of the form (event, elem) , where event is one of the listed events and elem is the resulting XML element.
    For example:

``` py
>>> data = iterparse('potholes.xml',('start','end'))
>>> next(data)
('start', <Element 'response' at 0x100771d60>)
>>> next(data)
('start', <Element 'row' at 0x100771e68>)
>>> next(data)
('start', <Element 'row' at 0x100771fc8>)
>>> next(data)
('start', <Element 'creation_date' at 0x100771f18>)
>>> next(data)
('end', <Element 'creation_date' at 0x100771f18>)
>>> next(data)
('start', <Element 'status' at 0x1006a7f18>)
>>> next(data)
('end', <Element 'status' at 0x1006a7f18>)
>>>
```
- `start` events are created when an element is first created but not yet populated with any other data (e.g., child elements).
- `end` events are created when an element is completed.
- `start-ns` and `end-ns` events are used to handle XML namespace declarations.

In this recipe, the start and end events are used to manage stacks of elements and tags.

The stacks represent the current hierarchical structure of the document as it’s being parsed, and are also used to determine if an element matches the requested path given to the `parse_and_remove()` function. If a match is made, yield is used to emit it back to the caller.

The following statement after the yield is the core feature of ElementTree that makes this recipe save memory:
``` py
elem_stack[-2].remove(elem) #causes the previously yielded element to be removed from its parent
```
## Turning a Dictionary into XML
##### Problem
You want to take the data in a Python dictionary and turn it into XML.
##### Solution
Although the `xml.etree.ElementTree` library is commonly used for parsing, it can also be used to create XML documents.
<br>
For example, consider this function:

``` py
from xml.etree.ElementTree import Element
def dict_to_xml(tag, d):
    '''
    Turn a simple dict of key/value pairs into XML
    '''
    elem = Element(tag)
    for key, val in d.items():
        child = Element(key)
        child.text = str(val)
        elem.append(child)
        return elem

#Usage
s = { 'name': 'GOOG', 'shares': 100, 'price':490.1 }
e = dict_to_xml('stock', s)
print(e) # <Element 'stock' at 0x1004b64c8>
```

The result of this conversion is an Element instance. For I/O, it is easy to convert this to a byte string using the `tostring()` function in `xml.etree.ElementTree` . 
<br>
For example:

``` py
from xml.etree.ElementTree import tostring
print(tostring(e)) #b'<stock><price>490.1</price><shares>100</shares><name>GOOG</name></stock>'
```

If you want to attach attributes to an element, use its `set()` method:
``` py
>>> e.set('_id','1234')
>>> tostring(e)
b'<stock _id="1234"><price>490.1</price><shares>100</shares><name>GOOG</name></stock>'
>>>
```
If the order of the elements matters, consider making an OrderedDict instead of a normal dictionary.

##### Discussion
When creating XML, you might be inclined to just make strings instead. For example:
``` py
def dict_to_xml_str(tag, d):
    '''
    Turn a simple dict of key/value pairs into XML
    '''
    parts = ['<{}>'.format(tag)]
    for key, val in d.items():
        parts.append('<{0}>{1}</{0}>'.format(key,val))
        parts.append('</{}>'.format(tag))
        return ''.join(parts)
```
The problem is that you’re going to make a real mess for yourself if you try to do things manually. 
<br>
For example, what happens if the dictionary values contain special characters like this?

``` py
>>> d = { 'name' : '<spam>' }
>>> # String creation
>>> dict_to_xml_str('item',d)
'<item><name><spam></name></item>'
>>> # Proper XML creation
>>> e = dict_to_xml('item',d)
>>> tostring(e)
b'<item><name>&lt;spam&gt;</name></item>'
>>>
```
Notice how in the latter example, the characters __<__ and __>__ got replaced with `&lt;` and `&gt;` .

Just for reference, if you ever need to manually escape or unescape such characters, you can use the `escape()` and `unescape()` functions in `xml.sax.saxutils` .
<br>
For example:

``` py
>>> from xml.sax.saxutils import escape, unescape
>>> escape('<spam>')
'&lt;spam&gt;'
>>> unescape(_)
'<spam>'
>>>
```

- reasons why it’s a good idea to create Element instances instead of strings:
    - creating correct output
    - they can be more easily combined together to make a larger document
    - The resulting Element instances can also be processed in various ways without ever having to worry about parsing the XML text

## Parsing, Modifying, and Rewriting XML
##### Problem
You want to read an XML document, make changes to it, and then write it back out as XML.
##### Solution
The `xml.etree.ElementTree` module makes it easy to perform such tasks. Essentially, you start out by parsing the document in the usual way.
<br> For example, suppose you have a document named pred.xml that looks like this:

``` xml
<?xml version="1.0"?>
<stop>
    <id>14791</id>
    <nm>Clark &amp; Balmoral</nm>
    <sri>
        <rt>22</rt>
        <d>North Bound</d>
        <dd>North Bound</dd>
    </sri>
    <cr>22</cr>
    <pre>
        <pt>5 MIN</pt>
        <fd>Howard</fd>
        <v>1378</v>
        <rn>22</rn>
    </pre>
    <pre>
        <pt>15 MIN</pt>
        <fd>Howard</fd>
        <v>1867</v>
        <rn>22</rn>
    </pre>
</stop>
``` 
Here is an example of using ElementTree to read it and make changes to the structure:
``` py
from xml.etree.ElementTree import parse, Element
doc = parse('pred.xml')
root = doc.getroot()
print(root) # <Element 'stop' at 0x100770cb0>

# Remove a few elements
root.remove(root.find('sri'))
root.remove(root.find('cr'))
# Insert a new element after <nm>...</nm>

print(root.getchildren().index(root.find('nm')))    #1

e = Element('spam')
e.text = 'This is a test'
root.insert(2, e)

# Write back to a file
doc.write('newpred.xml', xml_declaration=True)
```
The result of these operations is a new XML file that looks like this:
``` xml
<?xml version='1.0' encoding='us-ascii'?>
<stop>
    <id>14791</id>
    <nm>Clark &amp; Balmoral</nm>
    <spam>This is a test</spam>
    <pre>
        <pt>5 MIN</pt>
        <fd>Howard</fd>
        <v>1378</v>
        <rn>22</rn>
    </pre>
    <pre>
        <pt>15 MIN</pt>
        <fd>Howard</fd>
        <v>1867</v>
        <rn>22</rn>
    </pre>
</stop>
```

##### Discussion

?> All modifications are generally made to the parent element, treating it as if it were a list.

For example : 
- If you remove an element, it is removed from its immediate parent using the parent’s `remove()` method.
- If you insert or append new elements, you also use `insert()` and `append()` methods on the parent.
- Elements can also be manipulated using indexing and slicing operations, such as element[i] or element[i:j] .

?> If you need to make new elements, use the Element class, as shown in this recipe’s solution

## Parsing XML Documents with Namespaces
##### Problem
You need to parse an XML document, but it’s using XML namespaces.
##### Solution
Consider a document that uses namespaces like this:
``` xml
<?xml version="1.0" encoding="utf-8"?>
<top>
    <author>David Beazley</author>
    <content>
        <html xmlns="http://www.w3.org/1999/xhtml">
        <head>
            <title>Hello World</title>
        </head>
        <body>
            <h1>Hello World!</h1>
        </body>
        </html>
    </content>
</top>
```
If you parse this document and try to perform the usual queries, you’ll find that it doesn’t work so easily because everything becomes incredibly verbose:

``` py
>>> # Some queries that work
>>> doc.findtext('author')
'David Beazley'
>>> doc.find('content')
<Element 'content' at 0x100776ec0>
>>> # A query involving a namespace (doesn't work)
>>> doc.find('content/html')
>>> # Works if fully qualified
>>> doc.find('content/{http://www.w3.org/1999/xhtml}html')
<Element '{http://www.w3.org/1999/xhtml}html' at 0x1007767e0>
>>> # Doesn't work
>>> doc.findtext('content/{http://www.w3.org/1999/xhtml}html/head/title')
>>> # Fully qualified
>>> doc.findtext('content/{http://www.w3.org/1999/xhtml}html/{http://www.w3.org/1999/xhtml}head/{http://www.w3.org/1999/xhtml}title')
'Hello World'
>>>
```
You can often simplify matters for yourself by wrapping namespace handling up into a utility class.
``` py
class XMLNamespaces:
    def __init__(self, **kwargs):
    self.namespaces = {}
for name, uri in kwargs.items():
    self.register(name, uri)

def register(self, name, uri):
    self.namespaces[name] = '{'+uri+'}'
def __call__(self, path):
    return path.format_map(self.namespaces)
```
To use this class, you do the following:
``` py
>>> ns = XMLNamespaces(html='http://www.w3.org/1999/xhtml')
>>> doc.find(ns('content/{html}html'))
<Element '{http://www.w3.org/1999/xhtml}html' at 0x1007767e0>
>>> doc.findtext(ns('content/{html}html/{html}head/{html}title'))
'Hello World'
>>>
```
##### Discussion
Unfortunately, there is no mechanism in the basic ElementTree parser to get further information about namespaces.
<br>
However, you can get a bit more information about the scope of namespace processing if you’re willing to use the `iterparse()` function instead.

For example:
``` py
>>> from xml.etree.ElementTree import iterparse
>>> for evt, elem in iterparse('ns2.xml', ('end', 'start-ns', 'end-ns')):
...     print(evt, elem)
...
end <Element 'author' at 0x7fe2f3fd46d8>
start-ns ('', 'http://www.w3.org/1999/xhtml')
end <Element '{http://www.w3.org/1999/xhtml}title' at 0x7fe2f3fd4868>
end <Element '{http://www.w3.org/1999/xhtml}head' at 0x7fe2f3fd4818>
end <Element '{http://www.w3.org/1999/xhtml}h1' at 0x7fe2f3fd4908>
end <Element '{http://www.w3.org/1999/xhtml}body' at 0x7fe2f3fd48b8>
end <Element '{http://www.w3.org/1999/xhtml}html' at 0x7fe2f3fd47c8>
end-ns None
end <Element 'content' at 0x7fe2f3fd4728>
end <Element 'top' at 0x7fe2f3fd4688>
>>> elem    # This is the topmost element
<Element 'top' at 0x7fe2f3fd4688>
>>>>
```
?> As a final note, if the text you are parsing makes use of namespaces in addition to other advanced XML features, you’re really better off using the `lxml` library instead of `ElementTree` . For instance, `lxml` provides better support for validating documents against a __DTD__, more complete __XPath__ support, and other advanced XML features.

## Interacting with a Relational Database
##### Problem
You need to select, insert, or delete rows in a relational database.
##### Solution
A standard way to represent rows of data in Python is as a sequence of tuples.
<br>
For example:
```
stocks = [
        ('GOOG', 100, 490.1),
        ('AAPL', 50, 545.75),
        ('FB', 150, 7.45),
        ('HPQ', 75, 33.2),
]
```
Given data in this form, it is relatively straightforward to interact with a relational database using Python’s standard database API.

To illustrate, you can use the `sqlite3` module that comes with Python. If you are using a different database (e.g., MySql, Postgres, or ODBC), you’ll have to install a third-party module to support it. However, the underlying programming interface will be virtually the same, if not identical.

The first step is to connect to the database. Typically, you execute a `connect()` function, supplying parameters such as the name of the __database__, __hostname__, __username__, __password__, and other details as needed. For example:

``` py
>>> import sqlite3
>>> db = sqlite3.connect('database.db')
>>>
>>> c = db.cursor() # To do anything with the data, you next create a cursor. Once you have a cursor, you can start executing SQL queries.
>>> c.execute('create table portfolio (symbol text, shares integer, price real)')
<sqlite3.Cursor object at 0x10067a730>
>>> db.commit()
>>>
```
To insert a sequence of rows into the data, use a statement like this:

```py
>>> c.executemany('insert into portfolio values (?,?,?)', stocks)
<sqlite3.Cursor object at 0x10067a730>
>>> db.commit()
>>>
```
To perform a query, use a statement such as this:

```py
>>> for row in db.execute('select * from portfolio'):
...     print(row)
...
('GOOG', 100, 490.1)
('AAPL', 50, 545.75)
('FB', 150, 7.45)
('HPQ', 75, 33.2)
>>>
```

If you want to perform queries that accept user-supplied input parameters, make sure you escape the parameters using `?` like this:
``` py
>>> min_price = 100
>>> for row in db.execute('select * from portfolio where price >= ?',(min_price,)):
...     print(row)
...
('GOOG', 100, 490.1)
('AAPL', 50, 545.75)
>>>
```
##### Discussion
?> There are still some tricky details you’ll need to sort out on a case-by-case basis.

- One complication is the mapping of data from the database into Python types.
    - For entries such as dates, it is most common to use `datetime` instances from the `datetime` module, or possibly system `timestamps` , as used in the `time` module.- For numerical data, especially financial data involving decimals, numbers may be represented as Decimal instances from the `decimal` module.

Unfortunately, the exact mapping varies by database backend so you’ll have to read the associated documentation.

- Another extremely critical complication concerns the formation of SQL statement strings. You should never use Python string formatting operators (e.g., % ) or the `.format()` method to create such strings.

!> If the values provided to such formatting operators are derived from user input, this opens up your program to an SQL-injection attack.

?> The special `?` wildcard in queries instructs the database backend to use its own string substitution mechanism, which (hopefully) will do it safely.

- Sadly, there is some inconsistency across database backends with respect to the wildcard. Many modules use `?` or `%s` , while others may use a different symbol, such as `:0` or `:1` , to refer to parameters. Again, you’ll have to consult the documentation for the database module you’re using.

?> The paramstyle attribute of a database module also contains information about the quoting style.

If you’re doing something more complicated, it may make sense to use a higher-level interface, such as that provided by an object-relational mapper. Libraries
such as `SQLAlchemy` allow database tables to be described as Python classes and for database operations to be carried out while hiding most of the underlying SQL.

## Decoding and Encoding Hexadecimal Digits
##### Problem
You need to decode a string of hexadecimal digits into a byte string or encode a byte string as hex.
##### Solution
If you simply need to decode or encode a raw string of hex digits, use the `binascii` module. For example:
``` py
>>> # Initial byte string
>>> s = b'hello'
>>> # Encode as hex
>>> import binascii
>>> h = binascii.b2a_hex(s)
>>> h
b'68656c6c6f'
>>> # Decode back to bytes
>>> binascii.a2b_hex(h)
b'hello'
>>>
``` 
Similar functionality can also be found in the `base64` module. For example:
``` py
>>> import base64
>>> h = base64.b16encode(s)
>>> h
b'68656C6C6F'
>>> base64.b16decode(h)
b'hello'
>>>
``` 
##### Discussion
The main difference between the two techniques is in case folding .

!> The `base64.b16decode()` and `base64.b16encode()` functions only operate with uppercase hexadecimal letters, whereas the functions in `binascii` work with either case.

!> It’s also important to note that the output produced by the encoding functions is always a byte string. To coerce it to Unicode for output, you may need to add an extra decoding step.

For example:
``` py
>>> h = base64.b16encode(s)
>>> print(h)
b'68656C6C6F'
>>> print(h.decode('ascii'))
68656C6C6F
>>>
```
When decoding hex digits, the `b16decode()` and `a2b_hex()` functions accept either bytes or unicode strings. However, those strings must only contain __ASCII-encoded__
hexadecimal digits.

## Decoding and Encoding Base64
##### Problem
You need to decode or encode binary data using Base64 encoding.
##### Solution
The `base64` module has two functions— `b64encode()` and `b64decode()` —that do exactly what you want.

For example:
``` py
>>> # Some byte data
>>> s = b'hello'
>>> import base64
>>> # Encode as Base64
>>> a = base64.b64encode(s)
>>> a
b'aGVsbG8='
>>> # Decode from Base64
>>> base64.b64decode(a)
b'hello'
>>>
```
##### Discussion
Base64 encoding is only meant to be used on byte-oriented data such as byte strings and byte arrays.
Moreover, the output of the encoding process is always a byte string.

If you are mixing __Base64-encoded__ data with __Unicode__ text, you may have to perform an extra decoding step. For example:
``` py
>>> a = base64.b64encode(s).decode('ascii')
>>> a
'aGVsbG8='
>>>
```
When decoding Base64, both byte strings and Unicode text strings can be supplied.However, Unicode strings can only contain ASCII characters.

## Reading and Writing Binary Arrays of Structures
##### Problem
You want to read or write data encoded as a binary array of uniform structures into Python tuples.
##### Solution
To work with binary data, use the `struct` module. Here is an example of code that writes a list of Python tuples out to a binary file, encoding each tuple as a structure using `struct` :
``` py
from struct import Struct
def write_records(records, format, f):
    '''
    Write a sequence of tuples to a binary file of structures.
    '''
    record_struct = Struct(format)

    for r in records:
        f.write(record_struct.pack(*r))
        # Example
if __name__ == '__main__':
    records = [ (1, 2.3, 4.5),(6, 7.8, 9.0),(12, 13.4, 56.7) ]
with open('data.b', 'wb') as f:
    write_records(records, '<idd', f)
```
- There are several approaches for reading this file back into a list of tuples: 
    - If you’re going to read the file incrementally in chunks, you can write code such as this:
    ``` py
    from struct import Struct
    def read_records(format, f):
        record_struct = Struct(format)
        chunks = iter(lambda: f.read(record_struct.size), b'')
        return (record_struct.unpack(chunk) for chunk in chunks)
    # Example
    if __name__ == '__main__':
        with open('data.b','rb') as f:
            for rec in read_records('<idd', f):
                # Process rec
                ...
    ```

    - If you want to read the file entirely into a byte string with a single read and convert it piece by piece, you can write the following:
    ``` py
    from struct import Struct
    def unpack_records(format, data):
        record_struct = Struct(format)
        return (record_struct.unpack_from(data, offset) for offset in range(0, len(data), record_struct.size))
    # Example
    if __name__ == '__main__':
        with open('data.b', 'rb') as f:
            data = f.read()
            for rec in unpack_records('<idd', data):
                # Process rec
                ...
    ```


##### Discussion
For programs that must encode and decode binary data, it is common to use the struct `module`. To declare a new structure, simply create an instance of `Struct` such as:
``` py
# Little endian 32-bit integer, two double precision floats
record_struct = Struct('<idd')
```
- Structures are always defined using a set of structure codes such as i , d , f , and so forth [see the Python documentation]. These codes correspond to specific binary data types such as 32-bit integers, 64-bit floats, 32-bit floats, and so forth.

- The `<` in the first character specifies the byte ordering. In this example, it is indicating “little endian.” Change the character to `>` for big endian or `!` for network byte order.

- The `size` attribute contains the size of the structure in bytes, which is useful to have in I/O operations.

- `pack()` and `unpack()` methods are used to pack and unpack data.
For example:
``` py
>>> from struct import Struct
>>> record_struct = Struct('<idd')
>>> record_struct.size
20
>>> record_struct.pack(1, 2.0, 3.0)
b'\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00@\x00\x00\x00\x00\x00\x00\x08@'
>>> record_struct.unpack(_)
(1, 2.0, 3.0)
>>>
```

Sometimes you’ll see the `pack()` and `unpack()` operations called as module-level functions, as in the following:
``` py
>>> import struct
>>> struct.pack('<idd', 1, 2.0, 3.0)
b'\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00@\x00\x00\x00\x00\x00\x00\x08@'
>>> struct.unpack('<idd', _)
(1, 2.0, 3.0)
>>>
```
?> This works, but feels less elegant than creating a single Struct instance—especially if the same structure appears in multiple places in your code.
By creating a Struct instance, the format code is only specified once and all of the useful operations are grouped together nicely. This certainly makes it easier to maintain your code if you need to fiddle with the structure code (as you only have to change it in one place).

The code for reading binary structures involves a number of interesting, yet elegant programming idioms :

In the `read_records()` function, `iter()` is being used to make an iterator that returns fixed-sized chunks.This iterator repeatedly calls a user-supplied callable (e.g., lambda: f.read(record_struct.size) ) until it returns a specified value (e.g., b ), at which point iteration stops.

For example:
``` py
>>> f = open('data.b', 'rb')
>>> chunks = iter(lambda: f.read(20), b'')
>>> chunks
<callable_iterator object at 0x10069e6d0>
>>> for chk in chunks:
...     print(chk)
...
b'\x01\x00\x00\x00ffffff\x02@\x00\x00\x00\x00\x00\x00\x12@'
b'\x06\x00\x00\x00333333\x1f@\x00\x00\x00\x00\x00\x00"@'
b'\x0c\x00\x00\x00\xcd\xcc\xcc\xcc\xcc\xcc*@\x9a\x99\x99\x99\x99YL@'
>>>
```
?> One reason for creating an iterable is that it nicely allows records to be created using a generator comprehension, as shown in the solution.

If you didn’t use this approach, the code might look like this:
``` py
def read_records(format, f):
    record_struct = Struct(format)
    while True:
        chk = f.read(record_struct.size)
        if chk == b'':
            break
        yield record_struct.unpack(chk)
    return records
```

In the `unpack_records()` function, a different approach using the `unpack_from()` method is used. `unpack_from()` is a useful method for extracting binary data from a
larger binary array, because it does so without making any temporary objects or memory copies. You just give it a byte string (or any array) along with a byte offset, and it will unpack fields directly from that location.

If you used `unpack()` instead of `unpack_from()` , you would need to modify the code to make a lot of small slices and offset calculations.

For example:
``` py
def unpack_records(format, data):
    record_struct = Struct(format)
    return (record_struct.unpack(data[offset:offset + record_struct.size])for offset in range(0, len(data), record_struct.size))
```

In addition to being more complicated to read, this version also requires a lot more work, as it performs various offset calculations, copies data, and makes small slice objects.

?> If you’re going to be unpacking a lot of structures from a large byte string you’ve already read, `unpack_from()` is a more elegant approach.

?> Unpacking records is one place where you might want to use namedtuple objects from the `collections` module. This allows you to set attribute names on the returned tuples.

For example:
``` py
from collections import namedtuple
Record = namedtuple('Record', ['kind','x','y'])
with open('data.p', 'rb') as f:
    records = (Record(*r) for r in read_records('<idd', f))
    for r in records:
        print(r.kind, r.x, r.y)
```

?> If you’re writing a program that needs to work with a large amount of binary data, you may be better off using a library such as `numpy` .

For example, instead of reading a binary into a list of tuples, you could read it into a structured array, like this:
``` py
>>> import numpy as np
>>> f = open('data.b', 'rb')
>>> records = np.fromfile(f, dtype='<i,<d,<d')
>>> records
array([(1, 2.3, 4.5), (6, 7.8, 9.0), (12, 13.4, 56.7)],
dtype=[('f0', '<i4'), ('f1', '<f8'), ('f2', '<f8')])
>>> records[0]
(1, 2.3, 4.5)
>>> records[1]
(6, 7.8, 9.0)
>>>
```

?> Last, but not least, if you’re faced with the task of reading binary data in some known file format (i.e., image formats, shape files, HDF5, etc.), check to see if a Python module already exists for it. There’s no reason to reinvent the wheel if you don’t have to.
## Reading Nested and Variable-Sized Binary Structures
##### Problem
You need to read complicated binary-encoded data that contains a collection of nested and/or variable-sized records. Such data might include images, video, shapefiles, and so on.
##### Solution
The `struct` module can be used to decode and encode almost any kind of binary data structure.

To illustrate the kind of data in question here, suppose you have this Python data structure representing a collection of points that make up a series of polygons:
``` py
polys = [
    [ (1.0, 2.5), (3.5, 4.0), (2.5, 1.5) ],
    [ (7.0, 1.2), (5.1, 3.0), (0.5, 7.5), (0.8, 9.0) ],
    [ (3.4, 6.3), (1.2, 0.5), (4.6, 9.2) ],
]
```
Now suppose this data was to be encoded into a binary file where the file started with the following header:
<table>
    <thead>
        <tr style="background : black ; color : white">
            <th>
                Byte
            </th>
            <th>
                Type
            </th>
            <th>
                Description
            </th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>0</td>
            <td>int</td>
            <td>File code (0x1234, little endian)</td>
        </tr>
        <tr>
            <td>4</td>
            <td>double</td>
            <td>Minimum x (little endian)</td>
        </tr>
        <tr>
            <td>12</td>
            <td>double</td>
            <td>Minimum y (little endian)</td>
        </tr>
        <tr>
            <td>20</td>
            <td>double</td>
            <td>Maximum x (little endian)</td>
        </tr>
        <tr>
            <td>28</td>
            <td>double</td>
            <td>Maximum y (little endian)</td>
        </tr>
        <tr>
            <td>36</td>
            <td>int</td>
            <td>Number of polygons (little endian)</td>
        </tr>
    </tbody>
</table>  
Following the header, a series of polygon records follow, each encoded as follows:

<br>
<br>

<table>
    <thead>
        <tr style="background : black ; color : white">
            <th>
                Byte
            </th>
            <th>
                Type
            </th>
            <th>
                Description
            </th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>0</td>
            <td>int</td>
            <td>Record length including length (N bytes)</td>
        </tr>
        <tr>
            <td>4-N</td>
            <td>Points</td>
            <td>Pairs of (X,Y) coords as doubles</td>
        </tr>
    </tbody>
</table>
  
To write this file, you can use Python code like this:
``` py
import struct
import itertools
def write_polys(filename, polys):
    # Determine bounding box
    flattened = list(itertools.chain(*polys))
    min_x = min(x for x, y in flattened)
    max_x = max(x for x, y in flattened)
    min_y = min(y for x, y in flattened)
    max_y = max(y for x, y in flattened)
    with open(filename, 'wb') as f:
        f.write(struct.pack('<iddddi',
        0x1234,
        min_x, min_y,
        max_x, max_y,
        len(polys)))
        for poly in polys:
            size = len(poly) * struct.calcsize('<dd')
            f.write(struct.pack('<i', size+4))
            for pt in poly:
                f.write(struct.pack('<dd', *pt))
                # Call it with our polygon data
                write_polys('polys.bin', polys)
```
To read the resulting data back, you can write very similar looking code using the `struct.unpack()` function, reversing the operations performed during writing.

For example:
``` py
import struct
def read_polys(filename):
    with open(filename, 'rb') as f:
        # Read the header
        header = f.read(40)
        file_code, min_x, min_y, max_x, max_y, num_polys = \
            struct.unpack('<iddddi', header)

        polys = []
        for n in range(num_polys):
            pbytes, = struct.unpack('<i', f.read(4))
            poly = []
            for m in range(pbytes // 16):
                pt = struct.unpack('<dd', f.read(16))
                poly.append(pt)
            polys.append(poly)
        return polys
```
If code like this is used to process a real datafile, it can quickly become even messier. Thus, it’s an obvious candidate for an alternative solution that might simplify some of the steps and free the programmer to focus on more important matters.

In the remainder of this recipe, a rather advanced solution for interpreting binary data will be built up in pieces. The goal will be to allow a programmer to provide a high-level specification of the file format, and to simply have the details of reading and unpacking all of the data worked out under the covers.

!> As a forewarning, the code that follows may be the most advanced example in this entire documentation, utilizing various object-oriented programming and metaprogramming techniques. Be sure to carefully read the discussion section as well as cross-references to other recipes.

First, when reading binary data, it is common for the file to contain headers and other data structures. Although the `struct` module can unpack this data into a tuple, another way to represent such information is through the use of a class. 

Here’s some code that allows just that:
``` py
import struct
class StructField:
    '''
    Descriptor representing a simple structure field
    '''
    def __init__(self, format, offset):
        self.format = format
        self.offset = offset
    def __get__(self, instance, cls):
        if instance is None:
            return self
        else:
            r = struct.unpack_from(self.format,instance._buffer, self.offset)
            return r[0] if len(r) == 1 else r
class Structure:
    def __init__(self, bytedata):
        self._buffer = memoryview(bytedata)
```
This code uses a descriptor to represent each structure field. Each descriptor contains a struct-compatible format code along with a byte offset into an underlying memory
buffer. 

In the `__get__()` method, the `struct.unpack_from()` function is used to unpack a value from the buffer without having to make extra slices or copies.
The __Structure__ class just serves as a base class that accepts some byte data and stores it as the underlying memory buffer used by the __StructField__ descriptor.

Using this code, you can now define a structure as a high-level class that mirrors the information found in the tables that described the expected file format.

For example:

``` py
class PolyHeader(Structure):
    file_code = StructField('<i', 0)
    min_x = StructField('<d', 4)
    min_y = StructField('<d', 12)
    max_x = StructField('<d', 20)
    max_y = StructField('<d', 28)
    num_polys = StructField('<i', 36)
```
Here is an example of using this class to read the header from the polygon data written earlier:
``` py
>>> f = open('polys.bin', 'rb')
>>> phead = PolyHeader(f.read(40))
>>> phead.file_code == 0x1234
True
>>> phead.min_x
0.5
>>> phead.min_y
0.5
>>> phead.max_x
7.0
>>> phead.max_y
9.2
>>> phead.num_polys
3
>>>
```
- This is interesting, but there are a number of annoyances with this approach.
    - Even though you get the convenience of a class-like interface, the code is rather verbose and requires the user to specify a lot of low-level detail (e.g., repeated uses of StructField , specification of offsets, etc.).
    - The resulting class is also missing common conveniences such as providing a way to compute the total size of the structure.

?> Any time you are faced with class definitions that are overly verbose like this, you might consider the use of a class __decorator__ or __metaclass__. One of the features of a __metaclass__ is that it can be used to fill in a lot of low-level implementation details, taking that burden off of the user. 

As an example, consider this __metaclass__ and slight reformulation of the Structure class:
``` py
class StructureMeta(type):
    '''
    Metaclass that automatically creates StructField descriptors
    '''
    def __init__(self, clsname, bases, clsdict):
        fields = getattr(self, '_fields_', [])
        byte_order = ''
        offset = 0
        for format, fieldname in fields:
            if format.startswith(('<','>','!','@')):
                byte_order = format[0]
                format = format[1:]
                format = byte_order + format
                setattr(self, fieldname, StructField(format, offset))
                offset += struct.calcsize(format)
                setattr(self, 'struct_size', offset)
class Structure(metaclass=StructureMeta):
    def __init__(self, bytedata):
        self._buffer = bytedata

    @classmethod
    def from_file(cls, f):
        return cls(f.read(cls.struct_size))
```
Using this new Structure class, you can now write a structure definition like this:
``` py
class PolyHeader(Structure):
    _fields_ = [
        ('<i', 'file_code'),
        ('d', 'min_x'),
        ('d', 'min_y'),
        ('d', 'max_x'),
        ('d', 'max_y'),
        ('i', 'num_polys')
    ]
```
As you can see, the specification is a lot less verbose. The added `from_file()` class method also makes it easier to read the data from a file without knowing any details about the size or structure of the data.

For example:
``` py
>>> f = open('polys.bin', 'rb')
>>> phead = PolyHeader.from_file(f)
>>> phead.file_code == 0x1234
True
>>> phead.min_x
0.5
>>> phead.min_y
0.5
>>> phead.max_x
7.0
>>> phead.max_y
9.2
>>> phead.num_polys
3
>>>
```
Once you introduce a metaclass into the mix, you can build more intelligence into it.

For example, suppose you want to support nested binary structures. Here’s a reformulation of the metaclass along with a new supporting descriptor that allows it:

``` py
class NestedStruct:
    '''
    Descriptor representing a nested structure
    '''
    def __init__(self, name, struct_type, offset):
        self.name = name
        self.struct_type = struct_type
        self.offset = offset
    def __get__(self, instance, cls):
        if instance is None:
            return self
        else:
            data = instance._buffer[self.offset:self.offset+self.struct_type.struct_size]
            result = self.struct_type(data)

            # Save resulting structure back on instance to avoid
            # further recomputation of this step
            setattr(instance, self.name, result)
            return result

class StructureMeta(type):
    '''
    Metaclass that automatically creates StructField descriptors
    '''
    def __init__(self, clsname, bases, clsdict):
        fields = getattr(self, '_fields_', [])
        byte_order = ''
        offset = 0
        for format, fieldname in fields:
            if isinstance(format, StructureMeta):
                setattr(self, fieldname,NestedStruct(fieldname, format, offset))
                offset += format.struct_size
            else:
                if format.startswith(('<','>','!','@')):
                    byte_order = format[0]
                    format = format[1:]
                format = byte_order + format
                setattr(self, fieldname, StructField(format, offset))
                offset += struct.calcsize(format)
        setattr(self, 'struct_size', offset)
```
In this code, the NestedStruct descriptor is used to overlay another structure definition over a region of memory. It does this by taking a slice of the original memory buffer and using it to instantiate the given structure type. Since the underlying memory buffer was initialized as a memoryview, this slicing does not incur any extra memory copies.Instead, it’s just an overlay on the original memory.

Moreover, to avoid repeated instantiations, the descriptor then stores the resulting inner structure object on the instance.

Using this new formulation, you can start to write code like this:
``` py
class Point(Structure):
    _fields_ = [
        ('<d', 'x'),
        ('d', 'y')
    ]
class PolyHeader(Structure):
    _fields_ = [
        ('<i', 'file_code'),
        (Point, 'min'), # nested struct
        (Point, 'max'), # nested struct
        ('i', 'num_polys')
    ]
```
Amazingly, it will all still work as you expect. For example:
``` py
>>> f = open('polys.bin', 'rb')
>>> phead = PolyHeader.from_file(f)
>>> phead.file_code == 0x1234
True
>>> phead.min # Nested structure
<__main__.Point object at 0x1006a48d0>
>>> phead.min.x
0.5
>>> phead.min.y
0.5
>>> phead.max.x
7.0
>>> phead.max.y
9.2
>>> phead.num_polys
3
>>>
```

At this point, a framework for dealing with fixed-sized records has been developed, but what about the variable-sized components?

For example, the remainder of the polygon files contain sections of variable size.One way to handle this is to write a class that simply represents a chunk of binary data
along with a utility function for interpreting the contents in different ways :
``` py
class SizedRecord:
    def __init__(self, bytedata):
        self._buffer = memoryview(bytedata)

    @classmethod
    def from_file(cls, f, size_fmt, includes_size=True):
        sz_nbytes = struct.calcsize(size_fmt)
        sz_bytes = f.read(sz_nbytes)
        sz, = struct.unpack(size_fmt, sz_bytes)
        buf = f.read(sz - includes_size * sz_nbytes)
        return cls(buf)
    def iter_as(self, code):
        if isinstance(code, str):
            s = struct.Struct(code)
            for off in range(0, len(self._buffer), s.size):
                yield s.unpack_from(self._buffer, off)
        elif isinstance(code, StructureMeta):
            size = code.struct_size
            for off in range(0, len(self._buffer), size):
                data = self._buffer[off:off+size]
                yield code(data)
```
The __SizedRecord.from_file()__ class method is a utility for reading a size-prefixed chunk of data from a file, which is common in many file formats. As input, it accepts a structure format code containing the encoding of the size, which is expected to be in bytes.

The optional __includes_size__ argument specifies whether the number of bytes includes the size header or not.

Here’s an example of how you would use this code to read the individual polygons in the polygon file:
``` py
>>> f = open('polys.bin', 'rb')
>>> phead = PolyHeader.from_file(f)
>>> phead.num_polys
3
>>> polydata = [ SizedRecord.from_file(f, '<i') for n in range(phead.num_polys) ]
>>> polydata
[<__main__.SizedRecord object at 0x1006a4d50>,
<__main__.SizedRecord object at 0x1006a4f50>,
<__main__.SizedRecord object at 0x10070da90>]
>>>
```

As shown, the contents of the SizedRecord instances have not yet been interpreted. To do that, use the `iter_as()` method, which accepts a structure format code or Structure class as input. This gives you a lot of flexibility in how to interpret the data. 

For example:
``` py
>>> for n, poly in enumerate(polydata):
...     print('Polygon', n)
... for p in poly.iter_as('<dd'):
...     print(p)
...
Polygon 0
(1.0, 2.5)
(3.5, 4.0)
(2.5, 1.5)
Polygon 1
(7.0, 1.2)
(5.1, 3.0)
(0.5, 7.5)
(0.8, 9.0)
Polygon 2
(3.4, 6.3)
(1.2, 0.5)
(4.6, 9.2)


>>> for n, poly in enumerate(polydata):
...     print('Polygon', n)
...     for p in poly.iter_as(Point):
...         print(p.x, p.y)
...
Polygon 0
1.0 2.5
3.5 4.0
2.5 1.5
Polygon 1
7.0 1.2
5.1 3.0
0.5 7.5
0.8 9.0
Polygon 2
3.4 6.3
1.2 0.5
4.6 9.2
>>>
```

Putting all of this together, here’s an alternative formulation of the `read_polys()` function:
``` py
class Point(Structure):
    _fields_ = [
        ('<d', 'x'),
        ('d', 'y')
        ]
class PolyHeader(Structure):
    _fields_ = [
        ('<i', 'file_code'),
        (Point, 'min'),
        (Point, 'max'),
        ('i', 'num_polys')
        ]

def read_polys(filename):
    polys = []
    with open(filename, 'rb') as f:
        phead = PolyHeader.from_file(f)
        for n in range(phead.num_polys):
            rec = SizedRecord.from_file(f, '<i')
            poly = [ (p.x, p.y) for p in rec.iter_as(Point) ]
            polys.append(poly)
    return polys
```

##### Discussion
This recipe provides a practical application of various advanced programming techniques, including __descriptors__, __lazy evaluation__, __metaclasses__, __class variables__, and __memoryviews__.

?> A major feature of the implementation is that it is strongly based on the idea of lazyunpacking.

When an instance of Structure is created, the `__init__()` merely creates a memoryview of the supplied byte data and does nothing else.Specifically, no unpacking or other structure-related operations take place at this time.

?> One motivation for taking this approach is that you might only be interested in a few specific parts of a binary record. Rather than unpacking the whole file, only the parts that are actually accessed will be unpacked.

To implement the lazy unpacking and packing of values, the StructField descriptor class is used. Each attribute the user lists in _fields_ gets converted to a Struct Field descriptor that stores the associated structure format code and byte offset into the stored buffer.

The StructureMeta metaclass is what creates these descriptors automatically when various structure classes are defined. The main reason for using a metaclass is to make it extremely easy for a user to specify a structure format with a high-level description without worrying about low-level details.

One subtle aspect of the StructureMeta metaclass is that it makes byte order sticky. That is, if any attribute specifies a byte order ( `<` for little endian or `>` for big endian), that ordering is applied to all fields that follow. This helps avoid extra typing, but also makes it possible to switch in the middle of a definition.

For example, you might have something more complicated, such as this:
``` py
class ShapeFile(Structure):
    _fields_ = [ ('>i', 'file_code'), # Big endian
                ('20s', 'unused'),
                ('i', 'file_length'), # Little endian
                ('<i', 'version'),
                ('i', 'shape_type'),
                ('d', 'min_x'),
                ('d', 'min_y'),
                ('d', 'max_x'),
                ('d', 'max_y'),
                ('d', 'min_z'),
                ('d', 'max_z'),
                ('d', 'min_m'),
                ('d', 'max_m') ]
```
As noted, the use of a `memoryview()` in the solution serves a useful role in avoiding memory copies.
When structures start to nest, `memoryviews` can be used to overlay different parts of the structure definition on the same region of memory. This aspect of
the solution is subtle, but it concerns the slicing behavior of a memoryview versus a normal byte array. If you slice a byte string or byte array, you usually get a copy of the data. Not so with a memoryview—slices simply overlay the existing memory. Thus, this approach is more efficient.

The source code for Python’s `ctypes` library may also be of interest, due to its similar support for defining data structures, nesting of data structures, and similar
functionality.

## Summarizing Data and Performing Statistics
##### Problem
You need to crunch through large datasets and generate summaries or other kinds of statistics.
##### Solution
For any kind of data analysis involving statistics, time series, and other related techniques, you should look at the `Pandas` library.

To give you a taste, here’s an example of using Pandas to analyze the City of Chicago rat and rodent database. At the time of this writing, it’s a CSV file with about 74,000 entries:
``` py
>>> import pandas
>>> # Read a CSV file, skipping last line
>>> rats = pandas.read_csv('rats.csv', skip_footer=1)
>>> rats
<class 'pandas.core.frame.DataFrame'>
Int64Index: 74055 entries, 0 to 74054
Data columns:
Creation Date                       74055 non-null values
Status                              74055 non-null values
Completion Date                     72154 non-null values
Service Request Number              74055 non-null values
Type of Service Request             74055 non-null values
Number of Premises Baited           65804 non-null values
Number of Premises with Garbage     65600 non-null values
Number of Premises with Rats        65752 non-null values
Current Activity                    66041 non-null values
Most Recent Action                  66023 non-null values
Street Address                      74055 non-null values
ZIP Code                            73584 non-null values
X Coordinate                        74043 non-null values
Y Coordinate                        74043 non-null values
Ward                                74044 non-null values
Police District                     74044 non-null values
Community Area                      74044 non-null values
Latitude                            74043 non-null values
Longitude                           74043 non-null values
Location                            74043 non-null values
dtypes: float64(11), object(9)
>>> # Investigate range of values for a certain field
>>> rats['Current Activity'].unique()
array([nan, Dispatch Crew, Request Sanitation Inspector], dtype=object)
>>> # Filter the data
>>> crew_dispatched = rats[rats['Current Activity'] == 'Dispatch Crew']
>>> len(crew_dispatched)
65676
>>>
>>> # Find 10 most rat-infested ZIP codes in Chicago
>>> crew_dispatched['ZIP Code'].value_counts()[:10]
60647   3837
60618   3530
60614   3284
60629   3251
60636   2801
60657   2465
60641   2238
60609   2206
60651   2152
60632   2071
>>>
>>> # Group by completion date
>>> dates = crew_dispatched.groupby('Completion Date')
<pandas.core.groupby.DataFrameGroupBy object at 0x10d0a2a10>
>>> len(dates)
472
>>>
>>> # Determine counts on each day
>>> date_counts = dates.size()
>>> date_counts[0:10]
Completion Date
01/03/2011      4
01/03/2012      125
01/04/2011      54
01/04/2012      38
01/05/2011      78
01/05/2012      100
01/06/2011      100
01/06/2012      58
01/07/2011      1
01/09/2012      12
>>>
>>> # Sort the counts
>>> date_counts.sort()
>>> date_counts[-10:]
Completion Date
10/12/2012      313
10/21/2011      314
09/20/2011      316
10/26/2011      319
02/22/2011      325
10/26/2012      333
03/17/2011      336
10/13/2011      378
10/14/2011      391
10/07/2011      457
>>>
```

##### Discussion
`Pandas` is a large library that has more features than can be described here.  However, if you need to analyze large datasets, group data, perform statistics, or other similar tasks, it’s definitely worth a look.

Python for Data Analysis by Wes McKinney (O’Reilly) also contains much more information.