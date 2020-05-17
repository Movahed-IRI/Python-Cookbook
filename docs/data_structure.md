## Unpacking a sequence into seprate variables

You have an N-element tuple or sequence that you whould like to unpack into a collection of N variables.

###### solution

any sequnce (or iterable) can be unpacked into variables using a simple assignement operation. The only requirement is that the number of variables and structure match the sequence.For example

```` python
>>> p = (4,5)
>>> x ,y = p 
>>> x
4
````
###### discussion

Unpacking actually work with any object that happens to be iterable , not just tuples or lists . This includes strings , files , iterators , and generators.For example :

```` python
>>> s = "HELLO"
>>> a,b,c,d,e = s
>>> a
H
````

when unpacking you may sometimes want to discard certain values. Python has no  special syntax for this , but you can often just pick a throwaway variable name for it. For example : 
```` python
>>> data = ['ACME' , 's0' , '91.1' ,(2012,12,21)]
>>> _ , share , price , _ = data
>>> share
s0
````

!> However , make sure that the variable name you pick isn't being used for something else already.

!> If there is a mismatch in the number of elements , you'll get an error.

## Unpacking elements from iterables of arbitary length

###### problem
You need to unpack N elements from an iterable , but the iterable may be longer than N elements , causing a "Too many values to unpack" exception.

###### solution
python "star expressions" can be used to address this problem.For example , you're going to drop the first and last item , and only average the rest of them.

````` python
 def drop_first_last(grades):
    first, *middle , last = grades
    return avg(middle)
`````
Suppose you have user records that consist of a name and email address , followed by an arbitary number of phone numbers . You could unpack the records like this :

`````  python
>>> record = ('dave', 'dave@example.com' , '773-555-12' , '123-123')
>>> name , email , *phone_numbers = record
>>> phone_numbers
['773-555-12','123-123']
`````

!> it's worth nothing that the phone_numbers variable will always be a list , regardless of how many phone numbers are unpacked (including none).

other example as follow :

````` python
>>> *trailing , current = [10,8,7,1,9,5,10,3]
>>> trailing
[10,8,7,1,9,5,10]
>>> current
3
`````

It's worth nothing that the star syntax can be especially useful when iterating over a sequence of tuples of varying length.

````` phyton
records = [
    ('foo',1,2),
    ('bar','hello'),
    ('foo',3,4),
]
def do_foo(x,y):
    print('foo' , x , y)

def do_bar(s):
    print('bar' , s)

for tag,*args in records :
    if tag == 'foo':
        do_foo(*args)
    elif tag == 'bar':
        do_bar(*args)
`````

Star unpacking can also be useful when combined with certain kinds of string processing operations , such as splitting. For example :

````` python
>>> line = 'nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false'
>>> uname , *fields , homedir , sh = line.split(':')
>>> uname
'nobody'
`````
Sometimes you might want to unpack values and throw them away . You can't just specify a bare * when unpacking .

One could imagine writing functions that perform such splitting in order to carry out some kind of clever recursive algorithm. For example :

````` phyton
>>> def sum(items):
    head , *tail = items
    return head + sum(tail) if tail else head
>>> items = [1,10,7,4,5,9]
>>> sum(items)
36
`````

!> However , be aware that recursion really isn't a strong Python feature due to the inherent recursion limit. Thus , this last example might be nothing more than an academic curiosity in practice

## Keeping the Last N Items

##### problem 

you want to keep a limited history of the last few items seen during iteration or during some other king of processing

##### solution

Keeping a limited history is a perfect use for collection.deque . For example , the following code performs a simple text match on a sequence of lines and yields the matching line along with previous N lines of context when found:

````` python
from collections import deque

def search(lines , pattern , history = 5):
    previous_lines = deque(maxlen=history)
    for line in lines:
        if pattern in line:
            yield line , previous_lines
        previous_lines.append(line)

#example use on a file
if __name__ == '__main__' :
    with open('./somefile.txt') as f:
        for line , prevlines in  search(f,'python' , 5):
            for pline in prevlines:
                print(pline , end='')
                print('-'*20)
`````

##### Discussion

When writing code to search for items , it is comman to use a generator function involving yield , as shown in this recipe's solution. This decouples the process of searching from the code that uses the results.

Using deque(maxlen=N) creates a fixed-sized queue . when new items are added and the queue is full , the oldest item is automatically removed

?>More generally , a deque can be used whenever you need a simple queue structure . If you don't give it a maximum size . you get an unbounded queue that you append and pop items on either end.

!> Adding or popping items from either end of a queue has O(1) complexity. This is unlike a list where inserting or removing items from the front of the list is O(N).

## Finding the Largest or Smallest N Items

##### Problem

you want to make a list of the largest or smallest N items in a collection.

##### Solution

The heapq module has two functions -nlargest() and nsmallest()- that do exactly what you want for example

````` python
import heapq
nums = [1,8,2,23,7,-4,18,23,42,37,2]
print(heapq.nlargest(3,nums)) #prints [42,37,23]
print(heapq.nsmallest(3,nums)) #prints [-4,1,2]
`````

Both function also accept a key parameter that allow them to be used with more complicated data structures . For example:

````` python
import heapq

portfolio = [
    {'name' : "IBM" , 'shares' : 100 , 'price' : 91.1},
    {'name' : 'AAPL' , 'shares' : 50 , 'price' : 543.22},
    {'name' : 'FB' , 'shares' : 200 , 'price' : 21.09},
    {'name' : 'HPQ' , 'shares' : 35 , 'price' : 31.75}
]
cheap = heapq.nsmallest(2 , portfolio , key=lambda s: s['price'])
expensive = heapq.nlargest(2 , portfolio , key=lambda s: s['price'])
`````

?> Underneath the covers , they work by first converting the data into a list where items are ordered as a heap

````` python
>>> nums = [1,8,-4,42,23]
>>> import heapq
>>> heap = list(nums)
>>> heapq.heapify(heap)
>>> heap
[-4,1,8,42,23]
`````

     More-over , smallest item can easily found using the heapq.heaopop() method , which pops off the first item and replaces it with the next smallest item

If you are simply trying to find the the single smallest or largest item , it is faster to use ````` min() ````` and ````` max() `````

If N is about the same size as the collection itself , it is usually faster to sort it first and take a slice (i.e, use ````` sorted(items)[:N] ````` or ````` sorted(items)[-N:] ````` )

## Implementing a Priority Queue

##### Problem
You want to implement a queue that sorts items by a given priority and always return the item with the highest priority on each pop operation.

````` python
import heapq

class PriorityQueue:
    def __init__(self):
        self._queue = []
        self._index = 0
    def push(self , item , priority):
        heapq.heappush(self._queue,(-proprity , self._index , item))
        self._index += 1
    def pop(self) :
        return heapq.heappop(self._queue)[-1]
`````
Here is an example of how it might be used:

````` python 
>>> class Item:
    def __init__(self , name):
        self.name = name
    def __repr__(self):
        return 'Item({!r})'.format(self.name)

>>> q = PriorityQueue()
>>> q.push(Item('foo') , 1)
>>> q.push(Item('bar') , 5)
>>> q.push(Item('spam') , 4)
>>> q.push(Item('grok') , 1)
>>> q.pop()
Item('bar')
>>> q.pop()
Item('spam')
>>> q.pop()
Item('foo')
>>> q.pop()
Item('grok')
`````
!> Observe how  the two items with  the same priority(foo and grok) were returned in the same order in which they were inserted into the queue

     Moreover  , since the push and pop operations have O(log N) complexity where N in the number of items in the heap, they are fairly efficient even for fairly large value of N.

!> The priority value is negated to get the queue to sort items from highest priority to lowest priority.

!> If you make(priority , item) tuples , they can be compared as long as the priorities are different . However . if two with equal priorities are compared , the comparision fails . For Example:

````` python
>>> a = (1,Item('foo'))
>>> b = (5,Item('bar'))
>>> a < b
True
>>> c = (1,Item('grok'))
>>> a < c
Traceback (most recent call last):
  File "<stdin>" , line 1 , in <module>
TypeError: unorderable types: Item() < Item()
>>>
`````

!> By introducing the extra index and making ````` (priority, index, item) ````` tuples , you avoid this problem entirely since no two tuples will ever have tha same value for index.

## Mapping Keys to Multiple Values in a Dictionary

##### Problem
You want to make a dictionary that maps keys to more than one value (a so-called
“multidict”).

##### Solution
A dictionary is a mapping where each key is mapped to a single value. If you want to map keys to multiple values, you need to store the multiple values in another container such as a list or set. 
The choice of whether or not to use lists or sets depends on intended use.
- Use a list if you want to preserve the insertion order of the items.
- Use a set if you want to eliminate
duplicates (and don’t care about the order).

To easily construct such dictionaries, you can use ````` defaultdict ````` in the collections module.
A feature of ````` defaultdict ````` is that it automatically initializes the first value so
you can simply focus on adding items. For example:
````` python
from collections import defaultdict

d = defaultdict(list)
d['a'].append(1)
d['a'].append(2)
d['b'].append(4)
`````
````` python
from collections import defaultdict

d = defaultdict(set)
d['a'].add(1)
d['a'].add(2)
d['b'].add(4)
`````

!> One caution with defaultdict is that it will automatically create dictionary entries for keys accessed later on (even if they aren’t currently found in the dictionary). If you don’t
want this behavior, you might use ````` setdefault() ````` on an ordinary dictionary instead. For example:
````` python
d = {} # A regular dictionary
d.setdefault('a', []).append(1)
d.setdefault('a', []).append(2)
d.setdefault('b', []).append(4)
````` 
!> However, many programmers find setdefault() to be a little unnatural—not to mention the fact that it always creates a new instance of the initial value on each invocation
(the empty list [] in the example).

?> This reciepe is strongly related to the problem of grouping records together in data processing problems.

## Keeping Dictionaries in Order
##### Problem
You want to create a dictionary, and you also want to control the order of items when iterating or serializing.

##### Solution
To control the order of items in a dictionary, you can use an OrderedDict from the collections module. It exactly preserves the original insertion order of data when iterating. For example:
````` python
from collections import OrderedDict
d = OrderedDict()
d['foo'] = 1
d['bar'] = 2
d['spam'] = 3
d['grok'] = 4
# Outputs "foo 1", "bar 2", "spam 3", "grok 4"
for key in d:
print(key, d[key])
`````
?> An OrderedDict can be particularly useful when you want to build a mapping that you may want to later serialize or encode into a different format.

!> Be aware that the size of an OrderedDict is more than twice as large as a normal dictionary due to the extra linked list that’s created.

## Calculating with Dictionaries

##### Problem
You want to perform various calculations (e.g., minimum value, maximum value, sorting, etc.) on a dictionary of data.

##### Solution
Consider a dictionary that maps stock names to prices:
````` python 
prices = {
'ACME': 45.23,
'AAPL': 612.78,
'IBM': 205.55,
'HPQ': 37.20,
'FB': 10.75
}
`````

In order to perform useful calculations on the dictionary contents, it is often useful to invert the keys and values of the dictionary using ````` zip() ````` . For example, here is how to find the minimum and maximum price and stock name:
````` python
min_price = min(zip(prices.values(), prices.keys()))
# min_price is (10.75, 'FB')
max_price = max(zip(prices.values(), prices.keys()))
# max_price is (612.78, 'AAPL')
`````
!> When doing these calculations, be aware that zip() creates an iterator that can only be consumed once.

You can get the key corresponding to the min or max value if you supply a key function to min() and max() . For example:
````` python
min(prices, key=lambda k: prices[k]) # Returns 'FB'
max(prices, key=lambda k: prices[k]) # Returns 'AAPL'
`````
However, to get the minimum value, you’ll need to perform an extra lookup step. For example:
````` 
min_value = prices[min(prices, key=lambda k: prices[k])]
`````
?> The solution involving zip() solves the problem by “inverting” the dictionary into a sequence of (value, key) pairs

It should be noted that in calculations involving (value, key) pairs, the key will be used to determine the result in instances where multiple entries happen to have the same value.

## Finding Commonalities in Two Dictionaries
##### Problem
You have two dictionaries and want to find out what they might have in common (same keys, same values, etc.).
##### Solution
Consider two dictionaries:
`````
a = {
'x' : 1,
'y' : 2,
'z' : 3
}
b = {
'w' : 10,
'x' : 11,
'y' : 2
}
`````
To find out what the two dictionaries have in common, simply perform common set operations using the ````` keys() ````` or ````` items() ````` methods. For example:
`````
# Find keys in common
a.keys() & b.keys()
# { 'x', 'y' }
# Find keys in a that are not in b
a.keys() - b.keys()
# { 'z' }
# Find (key,value) pairs in common
a.items() & b.items() # { ('y', 2) }
`````
alter or filter dictionary contents. For example, suppose you want to make a new dictionary with selected keys removed. Here
is some sample code using a dictionary comprehension:
`````
# Make a new dictionary with certain keys removed
c = {key:a[key] for key in a.keys() - {'z', 'w'}}
# c is {'x': 1, 'y': 2}
`````

!> Although similar, the values() method of a dictionary does not support the set oper‐
ations described in this recipe. In part, this is due to the fact that unlike keys, the items
contained in a values view aren’t guaranteed to be unique. This alone makes certain set
operations of questionable utility. However, if you must perform such calculations, they
can be accomplished by simply converting the values to a set first.

## Removing Duplicates from a Sequence while Maintaining Order
###### Problem
You want to eliminate the duplicate values in a sequence, but preserve the order of the remaining items.
###### Solution
If the values in the sequence are hashable, the problem can be easily solved using a set and a generator. For example:

`````
def dedupe(items):
seen = set()
for item in items:
if item not in seen:
yield item
seen.add(item)
`````
Here is an example of how to use your function:
`````
>>> a = [1, 5, 2, 1, 9, 1, 5, 10]
>>> list(dedupe(a))
[1, 5, 2, 9, 10]
>>>
`````
!> This only works if the items in the sequence are hashable.
If you are trying to eliminate duplicates in a sequence of unhashable types (such as dicts), you can make a slight
change to this recipe, as follows:
`````
def dedupe(items, key=None):
    seen = set()
    for item in items:
        val = item if key is None else key(item)
        if val not in seen:
            yield item
            seen.add(val)
`````
Here, the purpose of the key argument is to specify a function that converts sequence items into a hashable type for the purposes of duplicate detection. Here’s how it works:
`````
>>> a = [ {'x':1, 'y':2}, {'x':1, 'y':3}, {'x':1, 'y':2}, {'x':2, 'y':4}]
>>> list(dedupe(a, key=lambda d: (d['x'],d['y'])))
[{'x': 1, 'y': 2}, {'x': 1, 'y': 3}, {'x': 2, 'y': 4}]
>>> list(dedupe(a, key=lambda d: d['x']))
[{'x': 1, 'y': 2}, {'x': 2, 'y': 4}]
>>>
`````
?> This latter solution also works nicely if you want to eliminate duplicates based on the value of a single field or attribute or a larger data structure.

##### discussion

If all you want to do is eliminate duplicates, it is often easy enough to make a set. For example:
`````
>>>
[1,
>>>
{1,
>>>
a
5, 2, 1, 9, 1, 5, 10]
set(a)
2, 10, 5, 9}
`````
!> However, this approach doesn’t preserve any kind of ordering

##### Naming a Slice

In general, built-in ````` slice() ````` creates a slice object that can be used anywhere a slice is allowed. For example:
````` python
items = [0, 1, 2, 3, 4, 5, 6]
a = slice(2, 4)
items[2:4]
#[2 ,3]
items[a]
#[2 ,3]
items[a] = [10,11]
items
# [0, 1, 10, 11, 4, 5, 6]
del items[a]
items
# [0, 1, 4, 5, 6]
`````
If you have a slice instance s , you can get more information about it by looking at its ````` s.start ````` , ````` s.stop ````` , and `````s.step ````` attributes, respectively.

?> In addition, you can map a slice onto a sequence of a specific size by using its indices(size) method. This returns a tuple (start, stop, step) where all values have
been suitably limited to fit within bounds (as to avoid IndexError exceptions when
indexing). For example:
`````
>>> s = 'HelloWorld'
>>> a = slice(5,10, 2)
>>> for i in range(*a.indices(len(s))):
>>> print(s[i])
...
...
W
r
d
`````

## Determining the Most Frequently Occurring Items in a Sequence
###### Problem
You have a sequence of items, and you’d like to determine the most frequently occurring items in the sequence.
###### Solution
The collections.Counter class is designed for just such a problem. It even comes with a handy most_common() method that will give you the answer.

````` python
words = [
'look', 'into', 'my', 'eyes', 'look', 'into', 'my', 'eyes',
'the', 'eyes', 'the', 'eyes', 'the', 'eyes', 'not', 'around', 'the',
'eyes', "don't", 'look', 'around', 'the', 'eyes', 'look', 'into',
'my', 'eyes', "you're", 'under'
]
from collections import Counter
word_counts = Counter(words)
top_three = word_counts.most_common(3)
print(top_three)
# Outputs [('eyes', 8), ('the', 5), ('look', 4)]
`````

?> A little-known feature of Counter instances is that they can be easily combined using various mathematical operations Needless to say, Counter objects are a tremendously useful tool for almost any kind of problem where you need to tabulate and count data. You should prefer this over manually written solutions involving dictionaries.

## Sorting a List of Dictionaries by a Common Key
##### Problem
You have a list of dictionaries and you would like to sort the entries according to one or more of the dictionary values.
##### Solution
Sorting this type of structure is easy using the operator module’s ````` itemgetter ````` function.
````` python
rows = [
    {'fname': 'Brian', 'lname': 'Jones', 'uid': 1003},
    {'fname': 'David', 'lname': 'Beazley', 'uid': 1002},
    {'fname': 'John', 'lname': 'Cleese', 'uid': 1001},
    {'fname': 'Big', 'lname': 'Jones', 'uid': 1004}
]
from operator import itemgetter
rows_by_fname = sorted(rows, key=itemgetter('fname'))
print(rows_by_fname)
`````
The preceding code would output the following:
````` python
[{'fname': 'Big', 'uid': 1004, 'lname': 'Jones'},
{'fname': 'Brian', 'uid': 1003, 'lname': 'Jones'},
{'fname': 'David', 'uid': 1002, 'lname': 'Beazley'},
{'fname': 'John', 'uid': 1001, 'lname': 'Cleese'}]
`````

The ````` itemgetter() ````` function can also accept multiple keys.
##### discussion
````` sorted() ````` function, accepts a key‐
word argument key . This argument is expected to be a callable that accepts a single item from rows as input and returns a value that will be used as the basis for sorting. The
itemgetter() function creates just such a callable.

?> The ````` operator.itemgetter() ````` function takes as arguments the lookup indices used to extract the desired values from the records in rows . It can be a dictionary key name, a
numeric list element, or any value that can be fed to an object’s \_\___getitem\_\_()__ method.

The functionality of ````` itemgetter() ````` is sometimes replaced by lambda expressions. For example:
`````
rows_by_fname = sorted(rows, key=lambda r: r['fname'])
rows_by_lfname = sorted(rows, key=lambda r: (r['lname'],r['fname']))
`````
?> This solution often works just fine. However, the solution involving ````` itemgetter() ````` typically runs a bit faster.

## Sorting Objects Without Native Comparison Support

##### Problem
You want to sort objects of the same class, but they don’t natively support comparison operations.
##### Solution
The built-in `````sorted() ````` function takes a key argument that can be passed a callable that will return some value in the object that sorted will use to compare the objects.
`````
>>> sorted(users, key=lambda u: u.user_id)
`````
Instead of using lambda , an alternative approach is to use ````` operator.attrgetter() ````` :
````` python
from operator import attrgetter
sorted(users, key=attrgetter('user_id'))
`````
?> ````` attrgetter() ````` is often a tad bit faster and also has the added feature of allowing multiple fields to be extracted simultaneously.

## Grouping Records Together Based on a Field
##### Problem
You have a sequence of dictionaries or instances and you want to iterate over the data in groups based on the value of a particular field, such as date.

##### Solution
The ````` itertools.groupby() ````` function is particularly useful for grouping data together like this.

````` python
rows = [
    {'address': '5412 N CLARK', 'date': '07/01/2012'},
    {'address': '5148 N CLARK', 'date': '07/04/2012'},
    {'address': '5800 E 58TH', 'date': '07/02/2012'},
    {'address': '2122 N CLARK', 'date': '07/03/2012'},
    {'address': '5645 N RAVENSWOOD', 'date': '07/02/2012'},
    {'address': '1060 W ADDISON', 'date': '07/02/2012'},
    {'address': '4801 N BROADWAY', 'date': '07/01/2012'},
    {'address': '1039 W GRANVILLE', 'date': '07/04/2012'},
]
from operator import itemgetter
from itertools import groupby
# Sort by the desired field first
rows.sort(key=itemgetter('date'))
# Iterate in groups
for date, items in groupby(rows, key=itemgetter('date')):
    print(date)
    for i in items:
        print('     ', i)
`````
This produces the following output:
`````
07/01/2012
    {'date': '07/01/2012', 'address': '5412 N CLARK'}
    {'date': '07/01/2012', 'address': '4801 N BROADWAY'}
07/02/2012
    {'date': '07/02/2012', 'address': '5800 E 58TH'}
    {'date': '07/02/2012', 'address': '5645 N RAVENSWOOD'}
    {'date': '07/02/2012', 'address': '1060 W ADDISON'}
07/03/2012
    {'date': '07/03/2012', 'address': '2122 N CLARK'}
07/04/2012
    {'date': '07/04/2012', 'address': '5148 N CLARK'}
    {'date': '07/04/2012', 'address': '1039 W GRANVILLE'}
`````

If your goal is to simply group the data together by dates into a large data structure that allows random access, you may have better luck using ````` defaultdict() ````` to build a
multidict

## Filtering Sequence Elements
##### Problem
You have data inside of a sequence, and need to extract values or reduce the sequence using some criteria.
##### Solution
`````
>>> mylist = [1, 4, -5, 10, -7, 2, 3, -1]
>>> [n for n in mylist if n > 0]
[1, 4, 10, 2, 3]
>>> [n for n in mylist if n < 0]
[-5, -7, -1]
>>>
`````
to produce the filtered values iteratively. For example:
`````
>>> pos = (n for n in mylist if n > 0)
>>> pos
<generator object <genexpr> at 0x1006a0eb0>
>>> for x in pos:
    print(x)
4
10
2
3
>>>
`````
?> Suppose that the filtering process involves exceptionhandling or some other complicated detail. For this, put the filtering code into its own function and use the built-in ````` filter() `````function. For example:
`````
values = ['1', '2', '-3', '-', '4', 'N/A', '5']
def is_int(val):
    try:
        x = int(val)
        return True
    except ValueError:
        return False
ivals = list(filter(is_int, values))
print(ivals)
# Outputs ['1', '2', '-3', '4', '5']
`````
!> ````` filter() ````` creates an iterator, so if you want to create a list of results, make sure you also
use `````list() ````` as shown.

One variation on filtering involves replacing the values that don’t meet the criteria with a new value instead of discarding them. For example, perhaps instead of just finding
positive values, you want to also clip bad values to fit within a specified range. This is often easily accomplished by moving the filter criterion into a conditional expression
like this:
````` python
>>> mylist = [1, 4, -5, 10, -7, 2, 3, -1]
>>> clip_neg = [n if n > 0 else 0 for n in mylist]
>>> clip_neg
[1,4, 0, 10, 0, 2, 3, 0]
>>> clip_pos = [n if n < 0 else 0 for n in mylist]
>>> clip_pos
[0, 0, -5, 0, -7, 0, 0, -1]
>>>
`````
Another notable filtering tool is itertools.compress() , which takes an iterable and an accompanying Boolean selector sequence as input. As output, it gives you all of the
items in the iterable where the corresponding element in the selector is True 

!> Like `````filter()````` , `````compress()````` normally returns an iterator. Thus, you need to use `````list()````` to turn the results into a list if desired.

## Extracting a Subset of a Dictionary
##### Problem
You want to make a dictionary that is a subset of another dictionary.
##### Solution
This is easily accomplished using a dictionary comprehension. For example:
`````
prices = {
'ACME': 45.23,
'AAPL': 612.78,
'IBM': 205.55,
'HPQ': 37.20,
'FB': 10.75
}
# Make a dictionary of all prices over 200
p1 = { key:value for key, value in prices.items() if value > 200 }
# Make a dictionary of tech stocks
tech_names = { 'AAPL', 'IBM', 'HPQ', 'MSFT' }
p2 = { key:value for key,value in prices.items() if key in tech_names }
`````
##### Discussion
?> However, the dictionary comprehension solution is a bit clearer and actually runs quite a bit faster (over twice as fast when tested on the prices dictionary used in the follows example).
`````
p1 = dict((key, value) for key, value in prices.items() if value > 200)
`````

## Mapping Names to Sequence Elements
##### Problem
You’d also like to be less dependent on position in the structure, by accessing the elements by name.
##### Solution
````` collections.namedtuple() ````` provides these benefits, while adding minimal overhead over using a normal tuple object. ````` collections.namedtuple() ````` is actually a factory
method that returns a subclass of the standard Python tuple type. You feed it a type name, and the fields it should have, and it returns a class that you can instantiate, passing in values for the fields you’ve defined, and so on. For example:
````` python
>>> from collections import namedtuple
>>> Subscriber = namedtuple('Subscriber', ['addr', 'joined'])
>>> sub = Subscriber('jonesy@example.com', '2012-10-19')
>>> sub
Subscriber(addr='jonesy@example.com', joined='2012-10-19')
>>> sub.addr
'jonesy@example.com'
`````
?> Although an instance of a namedtuple looks like a normal class instance, it is interchangeable with a tuple and supports all of the usual tuple operations such as indexing and unpacking.

?> One possible use of a namedtuple is as a replacement for a dictionary, which requires more space to store. Thus, if you are building large data structures involving dictionaries,
use of a namedtuple will be more efficient.

!> However, be aware that unlike a dictionary, a namedtuple is immutable.If you need to change any of the attributes, it can be done using the ````` _replace() ````` method of a namedtuple instance, which makes an entirely new namedtuple with specified values replaced.

it should be noted that if your goal is to define an efficient data structure where you will be changing various instance attributes, using namedtuple is not your best choice . instead, consider defining a class using ````` \_\_slots\_\_ ````` instead


## Transforming and Reducing Data at the Same Time
##### Problem
You need to execute a reduction function (e.g., `````sum()````` , `````min()````` , `````max()````` ), but first need to transform or filter the data.
##### Solution
A very elegant way to combine a data reduction and a transformation is to use a generator-expression argument
````` python
# Determine if any .py files exist in a directory
import os
files = os.listdir('dirname')
if any(name.endswith('.py') for name in files):
print('There be python!')
else:
print('Sorry, no python.')
# Output a tuple as CSV
s = ('ACME', 50, 123.45)
print(','.join(str(x) for x in s))
# Data reduction across fields of a data structure
portfolio = [
{'name':'GOOG', 'shares': 50},
{'name':'YHOO', 'shares': 75},
{'name':'AOL', 'shares': 20},
{'name':'SCOX', 'shares': 65}
]
min_shares = min(s['shares'] for s in portfolio)
`````
`````
s = sum((x * x for x in nums)) # Pass generator-expr as argument
s = sum(x * x for x in nums) # More elegant syntax
`````
Using a generator argument is often a more efficient and elegant approach than first creating a temporary list. For example, if you didn’t use a generator expression, you might consider this alternative implementation:
`````
nums = [1, 2, 3, 4, 5]
s = sum([x * x for x in nums])
`````
Certain reduction functions such as `````min()````` and `````max()````` accept a key argument that might be useful in situations where you might be inclined to use a generator.

## Combining Multiple Mappings into a Single Mapping
##### Problem
You have multiple dictionaries or mappings that you want to logically combine into a single mapping to perform certain operations, such as looking up values or checking for the existence of keys.
##### discussion
````` python 
a = {'x': 1, 'z': 3 }
b = {'y': 2, 'z': 4 }
from collections import ChainMap
c = ChainMap(a,b)
print(c['x'])
# Outputs 1 (from a)
print(c['y'])
# Outputs 2 (from b)
print(c['z'])
# Outputs 3 (from a)
`````
A ChainMap takes multiple mappings and makes them logically appear as one. However, the mappings are not literally merged together.
!> If there are duplicate keys, the values from the first mapping get used.

!> Operations that mutate the mapping always affect the first mapping listed.

````` python
>>> values = ChainMap()
>>> values['x'] = 1
>>> values = values.new_child()  # Add a new mapping
>>> values['x'] = 2
>>> values = values.new_child() # Add a new mapping
>>> values['x'] = 3
>>> values
ChainMap({'x': 3}, {'x': 2}, {'x': 1})
>>> values['x']
3
>>> values = values.parents # Discard last mapping
>>> values['x']
2
>>> values = values.parents # Discard last mapping
>>> values['x']
1
>>> values
ChainMap({'x': 1})
>>>
`````
As an alternative to ChainMap , you might consider merging dictionaries together using the `````update()````` method , but it requires you to make a completely separate dictionary object (or destructively alter one of the existing dictionaries). Also, if any of the original dictionaries mutate, the changes don’t get reflected in the merged dictionary. For example:
`````
>>> a['x'] = 13
>>> merged['x']
1
`````
A ChainMap uses the original dictionaries, so it doesn’t have this behavior. For example:
`````
>>> a = {'x': 1, 'z': 3 }
>>> b = {'y': 2, 'z': 4 }
>>> merged = ChainMap(a, b)
>>> merged['x']
1
>>> a['x'] = 42
>>> merged['x'] # Notice change to merged dicts
42
>>>
`````