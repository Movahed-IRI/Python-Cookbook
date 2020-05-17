## Starting and Stopping Threads
##### Problem
You want to create and destroy threads for concurrent execution of code.
##### Solution
The `threading` library can be used to execute any Python callable in its own thread. To do this, you create a Thread instance and supply the callable that you wish to executecas a target.

Here is a simple example:
``` py
# Code to execute in an independent thread
import time
def countdown(n):
    while n > 0:
        print('T-minus', n)
        n -= 1
        time.sleep(5)

# Create and launch a thread
from threading import Thread
t = Thread(target=countdown, args=(10,))
t.start()
```
When you create a thread instance, it doesn’t start executing until you invoke its `start()` method (which invokes the target function with the arguments you supplied).

Threads are executed in their own system-level thread (e.g., a POSIX thread or Windows threads) that is fully managed by the host operating system. Once started, threads run independently until the target function returns.

You can query a thread instance to see if it’s still running:
``` py
if t.is_alive():
    print('Still running')
else:
    print('Completed')
```
You can also request to join with a thread, which waits for it to terminate:
``` py
t.join()
```
The interpreter remains running until all threads terminate. For long-running threads or background tasks that run forever, you should consider making the thread daemonic.

For example:
``` py
t = Thread(target=countdown, args=(10,), daemon=True)
t.start()
```
Daemonic threads can’t be joined. However, they are destroyed automatically when the main thread terminates.

Beyond the two operations shown, there aren’t many other things you can do with threads. For example, there are no operations to terminate a thread, signal a thread,adjust its scheduling, or perform any other high-level operations. If you want these features, you need to build them yourself.If you want to be able to terminate threads, the thread must be programmed to poll for exit at selected points.

For example, you might put your thread in a class such as this:
``` py
class CountdownTask:
    def __init__(self):
        self._running = True
    def terminate(self):
        self._running = False
    def run(self, n):
        while self._running and n > 0:
            print('T-minus', n)
            n -= 1
            time.sleep(5)
cyc = CountdownTask()
t = Thread(target=c.run, args=(10,))
t.start()
...
c.terminate() # Signal termination
t.join() # Wait for actual termination (if needed)
‍‍‍```
Polling for thread termination can be tricky to coordinate if threads perform blocking operations such as I/O. For example, a thread blocked indefinitely on an I/O operation may never return to check if it’s been killed. To correctly deal with this case, you’ll need to carefully program thread to utilize timeout loops.

For example:
``` py
class IOTask:
    def terminate(self):
        self._running = False
    def run(self, sock):
        # sock is a socket
        sock.settimeout(5) # Set timeout period
        while self._running:
            # Perform a blocking I/O operation w/ timeout
            try:
                data = sock.recv(8192)
                break
            except socket.timeout:
                continue
            # Continued processing
        ...
        # Terminated
        return
```
##### Discussion
Due to a __global interpreter lock__ (GIL), Python threads are restricted to an execution model that only allows one thread to execute in the interpreter at any given time.

For this reason, Python threads should generally not be used for computationally intensive tasks where you are trying to achieve parallelism on multiple CPUs.They are much better suited for I/O handling and handling concurrent execution in code that performs blocking operations (e.g., waiting for I/O, waiting for results from a database, etc.).

Sometimes you will see threads defined via inheritance from the `Thread` class. For example:
``` py
from threading import Thread
class CountdownThread(Thread):
    def __init__(self, n):
        super().__init__()
        self.n = 0
    def run(self):
        while self.n > 0:
            print('T-minus', self.n)
            self.n -= 1
            time.sleep(5)

c = CountdownThread(5)
c.start()
```
Although this works, it introduces an extra dependency between the code and the threading library. That is, you can only use the resulting code in the context of threads, whereas the technique shown earlier involves writing code with no explicit dependency on threading . By freeing your code of such dependencies, it becomes usable in other contexts that may or may not involve threads.

For instance, you might be able to execute your code in a separate process using the `multiprocessing` module using code like this:
``` py
import multiprocessing
c = CountdownTask(5)
p = multiprocessing.Process(target=c.run)
p.start()
...
```
Again, this only works if the CountdownTask class has been written in a manner that is neutral to the actual means of concurrency (threads, processes, etc.).

## Determining If a Thread Has Started
##### Problem
You’ve launched a thread, but want to know when it actually starts running.
##### Solution
A key feature of threads is that they execute independently and nondeterministically.This can present a tricky synchronization problem if other threads in the program need to know if a thread has reached a certain point in its execution before carrying out further operations.To solve such problems, use the Event object from the `threading` library.

Event instances are similar to a “sticky” flag that allows threads to wait for something to happen.
 - Initially, an event is set to 0.
 - If the event is unset and a thread waits on the event, it will block (i.e., go to sleep) until the event gets set.
 - A thread that sets the event will wake up all of the threads that happen to be waiting (if any).
 - If a thread waits on an event that has already been set, it merely moves on, continuing to execute.

Here is some sample code that uses an Event to coordinate the startup of a thread:
``` py
from threading import Thread, Event
import time
# Code to execute in an independent thread
def countdown(n, started_evt):
    print('countdown starting')
    started_evt.set()
    while n > 0:
        print('T-minus', n)
        n -= 1
        time.sleep(5)

# Create the event object that will be used to signal startup
started_evt = Event()

# Launch the thread and pass the startup event
print('Launching countdown')
t = Thread(target=countdown, args=(10,started_evt))
t.start()

# Wait for the thread to start
started_evt.wait()
print('countdown is running')
```
When you run this code, the “countdown is running” message will always appear after the “countdown starting” message. This is coordinated by the event that makes the main thread wait until the countdown() function has first printed the startup message.
##### Discussion
Event objects are best used for one-time events. That is, you create an event, threads wait for the event to be set, and once set, the Event is discarded.

Although it is possible to clear an event using its `clear()` method, safely clearing an event and waiting for it to be set again is tricky to coordinate, and can lead to missed events, deadlock, or other problems (in particular, you can’t guarantee that a request to clear an event after setting it will execute before a released thread cycles back to wait on the event again).

If a thread is going to repeatedly signal an event over and over, you’re probably better off using a Condition object instead. For example, this code implements a periodic timer that other threads can monitor to see whenever the timer expires:
``` py
import threading
import time
class PeriodicTimer:
    def __init__(self, interval):
        self._interval = interval
        self._flag = 0
        self._cv = threading.Condition()
    def start(self):
        t = threading.Thread(target=self.run)
        t.daemon = True
        t.start()
    def run(self):
        '''
        Run the timer and notify waiting threads after each interval
        '''
        while True:
            time.sleep(self._interval)
            with self._cv:
                self._flag ^= 1
                self._cv.notify_all()
    def wait_for_tick(self):
        '''
        Wait for the next tick of the timer
        '''
        with self._cv:
            last_flag = self._flag
            while last_flag == self._flag:
                self._cv.wait()

# Example use of the timer
ptimer = PeriodicTimer(5)
ptimer.start()
# Two threads that synchronize on the timer
def countdown(nticks):
    while nticks > 0:
        ptimer.wait_for_tick()
        print('T-minus', nticks)
        nticks -= 1
def countup(last):
    n = 0
    while n < last:
        ptimer.wait_for_tick()
        print('Counting', n)
        n += 1

threading.Thread(target=countdown, args=(10,)).start()
threading.Thread(target=countup, args=(5,)).start()
```
?> A critical feature of Event objects is that they wake all waiting threads. If you are writing a program where you only want to wake up a single waiting thread, it is probably better to use a `Semaphore` or Condition object instead.

For example, consider this code involving semaphores:
``` py
# Worker thread
def worker(n, sema):
    # Wait to be signaled
    sema.acquire()
    # Do some work
    print('Working', n)

# Create some threads
sema = threading.Semaphore(0)
nworkers = 10
for n in range(nworkers):
    t = threading.Thread(target=worker, args=(n, sema,))
    t.start()
```
If you run this, a pool of threads will start, but nothing happens because they’re all blocked waiting to acquire the `semaphore`.

Each time the `semaphore` is released, only one worker will wake up and run.

For example:
``` py
>>> sema.release()
Working 0
>>> sema.release()
Working 1
>>>
```
Writing code that involves a lot of tricky synchronization between threads is likely to make your head explode. A more sane approach is to thread threads as communicating tasks using __queues__ or as __actors__. Queues are described in the next recipe.

## Communicating Between Threads
##### Problem
You have multiple threads in your program and you want to safely communicate or exchange data between them.
##### Solution
Perhaps the safest way to send data from one thread to another is to use a Queue from the `queue` library. To do this, you create a `Queue` instance that is shared by the `threads`.

__Threads__ then use `put()` or `get()` operations to add or remove items from the __queue__.
For example:
``` py
from queue import Queue
from threading import Thread
# A thread that produces data
def producer(out_q):
    while True:
        # Produce some data
        ...
        out_q.put(data)
        # A thread that consumes data
def consumer(in_q):
    while True:
        # Get some data
        data = in_q.get()
        # Process the data
        ...
# Create the shared queue and launch both threads
q = Queue()
t1 = Thread(target=consumer, args=(q,))
t2 = Thread(target=producer, args=(q,))
t1.start()
t2.start()
```
Queue instances already have all of the required locking, so they can be safely shared by as many threads as you wish.

When using queues, it can be somewhat tricky to coordinate the shutdown of the producer and consumer. A common solution to this problem is to rely on a special sentinel value, which when placed in the queue, causes consumers to terminate.

For example:
``` py
from queue import Queue
from threading import Thread

# Object that signals shutdown
_sentinel = object()

# A thread that produces data
def producer(out_q):
    while running:
        # Produce some data
        ...
        out_q.put(data)
    # Put the sentinel on the queue to indicate completion
    out_q.put(_sentinel)

# A thread that consumes data
def consumer(in_q):
    while True:
        # Get some data
        data = in_q.get()

        # Check for termination
        if data is _sentinel:
            in_q.put(_sentinel)
            break

        # Process the data
        ...
```
A subtle feature of this example is that the consumer, upon receiving the special sentinel value, immediately places it back onto the queue. This propagates the sentinel to other consumers threads that might be listening on the same queue—thus shutting them all down one after the other.

Although queues are the most common thread communication mechanism, you can build your own data structures as long as you add the required locking and synchronization. The most common way to do this is to wrap your data structures with a condition variable.

For example, here is how you might build a thread-safe priority queue::
``` py
import heapq
import threading
class PriorityQueue:
    def __init__(self):
        self._queue = []
        self._count = 0
        self._cv = threading.Condition()
    def put(self, item, priority):
        with self._cv:
            heapq.heappush(self._queue, (-priority, self._count, item))
            self._count += 1
            self._cv.notify()
    def get(self):
        with self._cv:
            while len(self._queue) == 0:
                self._cv.wait()
            return heapq.heappop(self._queue)[-1]
```
Thread communication with a queue is a one-way and nondeterministic process. In general, there is no way to know when the receiving thread has actually received a message and worked on it. However, `Queue` objects do provide some basic completion features, as illustrated by the __task_done()__ and __join()__ methods in this example:
``` py
from queue import Queue
from threading import Thread
# A thread that produces data
def producer(out_q):
    while running:
        # Produce some data
        ...
        out_q.put(data)

# A thread that consumes data
def consumer(in_q):
    while True:
        # Get some data
        data = in_q.get()
        # Process the data
        ...
        # Indicate completion
        in_q.task_done()

# Create the shared queue and launch both threads
q = Queue()
t1 = Thread(target=consumer, args=(q,))
t2 = Thread(target=producer, args=(q,))
t1.start()
t2.start()

# Wait for all produced items to be consumed
q.join()
```
If a thread needs to know immediately when a consumer thread has processed a particular item of data, you should pair the sent data with an Event object that allows the producer to monitor its progress.

For example:
``` py
from queue import Queue
from threading import Thread, Event
# A thread that produces data
def producer(out_q):
    while running:
        # Produce some data
        ...
        # Make an (data, event) pair and hand it to the consumer
        evt = Event()
        out_q.put((data, evt))
        ...
        # Wait for the consumer to process the item
        evt.wait()

# A thread that consumes data
def consumer(in_q):
while True:
    # Get some data
    data, evt = in_q.get()
    # Process the data
    ...
    # Indicate completion
    evt.set()
```
##### Discussion
Writing threaded programs based on simple queuing is often a good way to maintain sanity. If you can break everything down to simple thread-safe queuing, you’ll find that you don’t need to litter your program with locks and other low-level synchronization. Also, communicating with queues often leads to designs that can be scaled up to other kinds of message-based communication patterns later on. For instance, you might be able to split your program into multiple processes, or even a distributed system, without changing much of its underlying queuing architecture.

One caution with thread queues is that putting an item in a queue doesn’t make a copy of the item. Thus, communication actually involves passing an object reference between threads. If you are concerned about shared state, it may make sense to only pass immutable data structures (e.g., integers, strings, or tuples) or to make deep copies of the queued items.

For example:
```py
from queue import Queue
from threading import Thread
import copy
# A thread that produces data
def producer(out_q):
    while True:
        # Produce some data
        ...
        out_q.put(copy.deepcopy(data))
        # A thread that consumes data
def consumer(in_q):
    while True:
        # Get some data
        data = in_q.get()
        # Process the data
        ...
```
Queue objects provide a few additional features that may prove to be useful in certain contexts. If you create a Queue with an optional size, such as `Queue(N)` , it places a limit on the number of items that can be enqueued before the `put()` blocks the producer.

Adding an upper bound to a queue might make sense if there is mismatch in speed between a producer and consumer.For instance, if a producer is generating items at a much faster rate than they can be consumed. On the other hand, making a queue block when it’s full can also have an unintended cascading effect throughout your program, possibly causing it to deadlock or run poorly. In general, the problem of “flow control” between communicating threads is a much harder problem than it seems. If you ever find yourself trying to fix a problem by fiddling with queue sizes, it could be an indicator of a fragile design or some other inherent scaling problem.

Both the `get()` and `put()` methods support nonblocking and timeouts.

For example:
``` py
import queue
q = queue.Queue()
try:
    data = q.get(block=False)
except queue.Empty:
...
try:
    q.put(item, block=False)
except queue.Full:
...
try:
    data = q.get(timeout=5.0)
except queue.Empty:
...
```
Both of these options can be used to avoid the problem of just blocking indefinitely on a particular queuing operation.For example, a nonblocking `put()` could be used with a fixed-sized queue to implement different kinds of handling code for when a queue is full.

For example, issuing a log message and discarding:
``` py
def producer(q):
    ...
    try:
        q.put(item, block=False)
    except queue.Full:
        log.warning('queued item %r discarded!', item)
```
A timeout is useful if you’re trying to make consumer threads periodically give up on operations such as `q.get()` so that they can check things such as a termination flag.
``` py
_running = True
def consumer(q):
    while _running:
        try:
            item = q.get(timeout=5.0)
            # Process item
            ...
        except queue.Empty:
            pass
```
Lastly, there are utility methods `q.qsize()` , `q.full()` , `q.empty()` that can tell you the current size and status of the queue. !> However, be aware that all of these are unreliable in a multithreaded environment. For example, a call to `q.empty()` might tell you that the queue is empty, but in the time that has elapsed since making the call, another thread could have added an item to the queue. Frankly, it’s best to write your code not to rely on such functions.

## Locking Critical Sections
##### Problem
Your program uses threads and you want to lock critical sections of code to avoid race conditions.
##### Solution
To make mutable objects safe to use by multiple threads, use __Lock objects__ in the `threading` library, as shown here:
``` py
import threading
class SharedCounter:
    '''
    A counter object that can be shared by multiple threads.
    '''
    def __init__(self, initial_value = 0):
        self._value = initial_value
        self._value_lock = threading.Lock()
    def incr(self,delta=1):
        '''
        Increment the counter with locking
        '''
        with self._value_lock:
             self._value += delta
    def decr(self,delta=1):
        '''
        Decrement the counter with locking
        '''
        with self._value_lock:
             self._value -= delta
```
?> A Lock guarantees mutual exclusion when used with the with statement—that is, only one thread is allowed to execute the block of statements under the with statement at a time. The with statement acquires the lock for the duration of the indented statements
and releases the lock when control flow exits the indented block.
##### Discussion
Thread scheduling is inherently nondeterministic. Because of this, failure to use locks in threaded programs can result in randomly corrupted data and bizarre behavior known as a “race condition.” To avoid this, locks should always be used whenever shared mutable state is accessed by multiple threads.

In older Python code, it is common to see locks explicitly acquired and released. For example, in this variant of the last example:
``` py
import threading
class SharedCounter:
    '''
    A counter object that can be shared by multiple threads.
    '''
    def __init__(self, initial_value = 0):
        self._value = initial_value
        self._value_lock = threading.Lock()
    def incr(self,delta=1):
        '''
        Increment the counter with locking
        '''
        self._value_lock.acquire()
        self._value += delta
        self._value_lock.release()
    def decr(self,delta=1):
        '''
        Decrement the counter with locking
        '''
        self._value_lock.acquire()
        self._value -= delta
        self._value_lock.release()
```
The with statement is more elegant and less prone to error—especially in situations where a programmer might forget to call the `release()` method or if a program happens to raise an exception while holding a lock (the with statement guarantees that locks are
always released in both cases).

To avoid the potential for deadlock, programs that use locks should be written in a way such that each thread is only allowed to acquire one lock at a time. If this is not possible, you may need to introduce more advanced deadlock avoidance into your program, as
described in next Recipe.

In the threading library, you’ll find other synchronization primitives, such as __RLock__ and __Semaphore__ objects. As a general rule of thumb, these are more special purpose and should not be used for simple locking of mutable state. An __RLock__ or __re-entrant lock__ object is a lock that can be acquired multiple times by the same thread. It is primarily used to implement code based locking or synchronization based on a construct known as a “monitor.” With this kind of locking, only one thread is allowed to use an entire
function or the methods of a class while the lock is held.

For example, you could implement the SharedCounter class like this:
``` py
import threading
class SharedCounter:
    '''
    A counter object that can be shared by multiple threads.
    '''
    _lock = threading.RLock()
    def __init__(self, initial_value = 0):
        self._value = initial_value
    def incr(self,delta=1):
        '''
        Increment the counter with locking
        '''
        with SharedCounter._lock:
            self._value += delta
    def decr(self,delta=1):
        '''
        Decrement the counter with locking
        '''
        with SharedCounter._lock:
            self.incr(-delta)
```
In this variant of the code, there is just a single class-level lock shared by all instances of the class. Instead of the lock being tied to the per-instance mutable state, the lock is meant to synchronize the methods of the class. Specifically, this lock ensures that only one thread is allowed to be using the methods of the class at once. However, unlike a standard lock, it is OK for methods that already have the lock to call other methods that also use the lock (e.g., see the `decr()` method). One feature of this implementation is that only one lock is created, regardless of how many counter instances are created. Thus, it is much more memory-efficient in situations where there are a large number of counters.

However, a possible downside is that it may cause more lock contention in programs that use a large number of threads and make frequent counter updates.

A __Semaphore__ object is a synchronization primitive based on a shared counter. If the counter is nonzero, the with statement decrements the count and a thread is allowed to proceed. The counter is incremented upon the conclusion of the with block. If the
counter is zero, progress is blocked until the counter is incremented by another thread.

Although a semaphore can be used in the same manner as a standard Lock , the added complexity in implementation negatively impacts performance. Instead of simple locking, __Semaphore__ objects are more useful for applications involving signaling between threads or throttling.

For example, if you want to limit the amount of concurrency in a part of code, you might use a semaphore, as follows:
``` py
from threading import Semaphore
import urllib.request
# At most, five threads allowed to run at once
_fetch_url_sema = Semaphore(5)
def fetch_url(url):
    with _fetch_url_sema:
        return urllib.request.urlopen(url)
```
If you’re interested in the underlying theory and implementation of thread synchronization primitives, consult almost any textbook on operating systems.
## Locking with Deadlock Avoidance
##### Problem
You’re writing a multithreaded program where threads need to acquire more than one lock at a time while avoiding deadlock.
##### Solution
In multithreaded programs, a common source of deadlock is due to threads that attempt to acquire multiple locks at once. For instance, if a thread acquires the first lock, butthen blocks trying to acquire the second lock, that thread can potentially block the progress of other threads and make the program freeze.

One solution to deadlock avoidance is to assign each lock in the program a unique number, and to enforce an ordering rule that only allows multiple locks to be acquired in ascending order. This is surprisingly easy to implement using a context manager as follows:
``` py
import threading
from contextlib import contextmanager

# Thread-local state to stored information on locks already acquired
_local = threading.local()

@contextmanager
def acquire(*locks):

    # Sort locks by object identifier
    locks = sorted(locks, key=lambda x: id(x))

    # Make sure lock order of previously acquired locks is not violated
    acquired = getattr(_local,'acquired',[])
    if acquired and max(id(lock) for lock in acquired) >= id(locks[0]):
        raise RuntimeError('Lock Order Violation')

    # Acquire all of the locks
    acquired.extend(locks)
    _local.acquired = acquired
    try:
        for lock in locks:
            lock.acquire()
        yield
    finally:
        # Release locks in reverse order of acquisition
        for lock in reversed(locks):
            lock.release()
        del acquired[-len(locks):]
```
To use this context manager, you simply allocate lock objects in the normal way, but use the `acquire()` function whenever you want to work with one or more locks. For example:
``` py
import threading
x_lock = threading.Lock()
y_lock = threading.Lock()
def thread_1():
    while True:
        with acquire(x_lock, y_lock):
            print('Thread-1')
def thread_2():
    while True:
        with acquire(y_lock, x_lock):
            print('Thread-2')

t1 = threading.Thread(target=thread_1)
t1.daemon = True
t1.start()
t2 = threading.Thread(target=thread_2)
t2.daemon = True
t2.start()
```
?> If you run this program, you’ll find that it happily runs forever without  deadlock—eventhough the acquisition of locks is specified in a different order in each function. The key to this recipe lies in the first statement that sorts the locks according to object identifier. By sorting the locks, they always get acquired in a consistent order regardless of how the user might have provided them to acquire() .

The solution uses thread-local storage to solve a subtle problem with detecting potential deadlock if multiple acquire() operations are nested.

For example, suppose you wrote the code like this:
``` py
import threading
x_lock = threading.Lock()
y_lock = threading.Lock()
def thread_1():
    while True:
        with acquire(x_lock):
            with acquire(y_lock):
                print('Thread-1')
def thread_2():
    while True:
        with acquire(y_lock):
            with acquire(x_lock):
                print('Thread-2')

t1 = threading.Thread(target=thread_1)
t1.daemon = True
t1.start()
t2 = threading.Thread(target=thread_2)
t2.daemon = True
t2.start()
```
If you run this version of the program, one of the threads will crash with an exception such as this:
``` py
Exception in thread Thread-1:
Traceback (most recent call last):
    File "/usr/local/lib/python3.3/threading.py", line 639, in _bootstrap_inner
        self.run()
    File "/usr/local/lib/python3.3/threading.py", line 596, in run
        self._target(*self._args, **self._kwargs)
    File "deadlock.py", line 49, in thread_1
        with acquire(y_lock):
    File "/usr/local/lib/python3.3/contextlib.py", line 48, in __enter__
        return next(self.gen)
    File "deadlock.py", line 15, in acquire
        raise RuntimeError("Lock Order Violation")
RuntimeError: Lock Order Violation
>>>
```
This crash is caused by the fact that each thread remembers the locks it has already acquired. The `acquire()` function checks the list of previously acquired locks and enforces the ordering constraint that previously acquired locks must have an object ID that is less than the new locks being acquired.
##### Discussion
The issue of deadlock is a well-known problem with programs involving threads (as
well as a common subject in textbooks on operating systems). As a rule of thumb, as
long as you can ensure that threads can hold only one lock at a time, your program will be deadlock free.However, once multiple locks are being acquired at the same time, all bets are off.

Detecting and recovering from deadlock is an extremely tricky problem with few elegant solutions. For example, a common deadlock detection and recovery scheme involves the use of a watchdog timer. As threads run, they periodically reset the timer, and as long as everything is running smoothly, all is well. However, if the program deadlocks, the watchdog timer will eventually expire. At that point, the program “recovers” by killing and then restarting itself.

Deadlock avoidance is a different strategy where locking operations are carried out in
a manner that simply does not allow the program to enter a deadlocked state. The solution in which locks are always acquired in strict order of ascending object ID can
be mathematically proven to avoid deadlock, although the proof is left as an exercise to the reader (the gist of it is that by acquiring locks in a purely increasing order, you can’t get cyclic locking dependencies, which are a necessary condition for deadlock to occur).

As a final example, a classic thread deadlock problem is the so-called “dining philosopher’s problem.” In this problem, five philosophers sit around a table on which there are five bowls of rice and five chopsticks. Each philosopher represents an independent thread and each chopstick represents a lock. In the problem, philosophers either sit and think or they eat rice. However, in order to eat rice, a philosopher needs two chopsticks.Unfortunately, if all of the philosophers reach over and grab the chopstick to their left, they’ll all just sit there with one stick and eventually starve to death. It’s a gruesome scene.

Using the solution, here is a simple deadlock free implementation of the dining philosopher’s problem:
``` py
import threading
# The philosopher thread
def philosopher(left, right):
    while True:
        with acquire(left,right):
            print(threading.currentThread(), 'eating')

# The chopsticks (represented by locks)
NSTICKS = 5
chopsticks = [threading.Lock() for n in range(NSTICKS)]

# Create all of the philosophers
for n in range(NSTICKS):
    t = threading.Thread(target=philosopher,
                         args=(chopsticks[n],chopsticks[(n+1) % NSTICKS]))
    t.start()
```
?> Last, but not least, it should be noted that in order to avoid deadlock, all locking operations must be carried out using our `acquire()` function. If some fragment of code decided to acquire a lock directly, then the deadlock avoidance algorithm wouldn’t work.

## Storing Thread-Specific State
##### Problem
You need to store state that’s specific to the currently executing thread and not visible to other threads.
##### Solution
Sometimes in multithreaded programs, you need to store data that is only specific to the currently executing thread. To do this, create a thread-local storage object using
`threading.local()` . Attributes stored and read on this object are only visible to the executing thread and no others.

As an interesting practical example of using thread-local storage, consider the LazyConnection context-manager class. Here is a slightly modified version that safely works with multiple threads:
``` py
from socket import socket, AF_INET, SOCK_STREAM
import threading
class LazyConnection:
    def __init__(self, address, family=AF_INET, type=SOCK_STREAM):
        self.address = address
        self.family = AF_INET
        self.type = SOCK_STREAM
        self.local = threading.local()
    def __enter__(self):
        if hasattr(self.local, 'sock'):
            raise RuntimeError('Already connected')
        self.local.sock = socket(self.family, self.type)
        self.local.sock.connect(self.address)
        return self.local.sock
    def __exit__(self, exc_ty, exc_val, tb):
        self.local.sock.close()
        del self.local.sock
```
In this code, carefully observe the use of the __self.local__ attribute. It is  nitialized as an instance of `threading.local()` . The other methods then manipulate a socket that’s stored as self.local.sock . This is enough to make it possible to safely use an instance of LazyConnection in multiple threads. For example:
``` py
from functools import partial
def test(conn):
    with conn as s:
        s.send(b'GET /index.html HTTP/1.0\r\n')
        s.send(b'Host: www.python.org\r\n')
        s.send(b'\r\n')
        resp = b''.join(iter(partial(s.recv, 8192), b''))
    print('Got {} bytes'.format(len(resp)))

if __name__ == '__main__':
    conn = LazyConnection(('www.python.org', 80))

    t1 = threading.Thread(target=test, args=(conn,))
    t2 = threading.Thread(target=test, args=(conn,))
    t1.start()
    t2.start()
    t1.join()
    t2.join()
```
The reason it works is that each thread actually creates its own dedicated socket connection (stored as self.local.sock ). Thus, when the different threads perform socket operations, they don’t interfere with one another as they are being performed on different sockets.
##### Discussion
> Creating and manipulating thread-specific state is not a problem that often arises in most programs. However, when it does, it commonly involves situations where an object being used by multiple threads needs to manipulate some kind of dedicated system resource, such as a socket or file. You can’t just have a single socket object shared by everyone because chaos would ensue if multiple threads ever started reading and writing on it at the same time. Thread-local storage fixes this by making such resources only visible in the thread where they’re being used.

?> In this recipe, the use of `threading.local()` makes the LazyConnection class support one connection per thread, as opposed to one connection for the entire process. It’s a subtle but interesting distinction.

?> Under the covers, an instance of `threading.local()` maintains a separate instance dictionary for each thread. All of the usual instance operations of getting, setting, and deleting values just manipulate the per-thread dictionary. The fact that each  thread uses a separate dictionary is what provides the isolation of data.
## Creating a Thread Pool
##### Problem
You want to create a pool of worker threads for serving clients or performing other kinds of work.
##### Solution
The `concurrent.futures` library has a ThreadPoolExecutor class that can be used for
this purpose. Here is an example of a simple TCP server that uses a thread-pool to serve clients:
``` py
from socket import AF_INET, SOCK_STREAM, socket
from concurrent.futures import ThreadPoolExecutor
def echo_client(sock, client_addr):
    '''
    Handle a client connection
    '''
    print('Got connection from', client_addr)
    while True:
        msg = sock.recv(65536)
        if not msg:
            break
        sock.sendall(msg)
    print('Client closed connection')
    sock.close()
def echo_server(addr):
    pool = ThreadPoolExecutor(128)
    sock = socket(AF_INET, SOCK_STREAM)
    sock.bind(addr)
    sock.listen(5)
    while True:
        client_sock, client_addr = sock.accept()
        pool.submit(echo_client, client_sock, client_addr)
echo_server(('',15000))
```
If you want to manually create your own thread pool, it’s usually easy enough to do it
using a Queue . Here is a slightly different, but manual implementation of the same code:
``` py
from socket import socket, AF_INET, SOCK_STREAM
from threading import Thread
from queue import Queue
def echo_client(q):
    '''
    Handle a client connection
    '''
    sock, client_addr = q.get()
    print('Got connection from', client_addr)
    while True:
        msg = sock.recv(65536)
        if not msg:
            break
        sock.sendall(msg)
    print('Client closed connection')
    sock.close()
def echo_server(addr, nworkers):

    # Launch the client workers
    q = Queue()
    for n in range(nworkers):
        t = Thread(target=echo_client, args=(q,))
        t.daemon = True
        t.start()

# Run the server
sock = socket(AF_INET, SOCK_STREAM)
sock.bind(addr)
sock.listen(5)
while True:
    client_sock, client_addr = sock.accept()
    q.put((client_sock, client_addr))
    echo_server(('',15000), 128)
```
> One advantage of using __ThreadPoolExecutor__ over a manual implementation is that it makes it easier for the submitter to receive results from the called function.

For example, you could write code like this:
``` py
from concurrent.futures import ThreadPoolExecutor
import urllib.request
def fetch_url(url):
    u = urllib.request.urlopen(url)
    data = u.read()
    return data

pool = ThreadPoolExecutor(10)

# Submit work to the pool
a = pool.submit(fetch_url, 'http://www.python.org')
b = pool.submit(fetch_url, 'http://www.pypy.org')

# Get the results back
x = a.result()
y = b.result()
```
The result objects in the example handle all of the blocking and coordination needed to get data back from the worker thread. Specifically, the operation a.result() blocks
until the corresponding function has been executed by the pool and returned a value.
##### Discussion
Generally, you should avoid writing programs that allow unlimited growth in the number of threads. 

For example, take a look at the following server:
``` py
from threading import Thread
from socket import socket, AF_INET, SOCK_STREAM
def echo_client(sock, client_addr):
    '''
    Handle a client connection
    '''
    print('Got connection from', client_addr)
    while True:
        msg = sock.recv(65536)
        if not msg:
            break
        sock.sendall(msg)
    print('Client closed connection')
    sock.close()
def echo_server(addr, nworkers):
    # Run the server
    sock = socket(AF_INET, SOCK_STREAM)
    sock.bind(addr)
    sock.listen(5)
    while True:
        client_sock, client_addr = sock.accept()
        t = Thread(target=echo_client, args=(client_sock, client_addr))
        t.daemon = True
        t.start()

echo_server(('',15000))
```
!> Although this works, it doesn’t prevent some asynchronous hipster from launching an
attack on the server that makes it create so many threads that your program runs out of resources and crashes (thus further demonstrating the “evils” of using threads). By
using a pre-initialized thread pool, you can carefully put an upper limit on the amount of supported concurrency.

You might be concerned with the effect of creating a large number of threads. However,
modern systems should have no trouble creating pools of a few thousand threads.
Moreover, having a thousand threads just sitting around waiting for work isn’t going to have much, if any, impact on the performance of other code (a sleeping thread does just that—nothing at all). Of course, if all of those threads wake up at the same time and start hammering on the CPU, that’s a different story—especially in light of the __Global Interpreter Lock__ (GIL). Generally, you only want to use thread pools for I/O-bound processing.

One possible concern with creating large thread pools might be memory use. For example, if you create 2,000 threads on OS X, the system shows the Python process using up more than 9 GB of virtual memory. However, this is actually somewhat misleading. When creating a thread, the operating system reserves a region of virtual memory to hold the thread’s execution stack (often as large as 8 MB). Only a small fragment of this memory is actually mapped to real memory, though. Thus, if you look a bit closer, you might find the Python process is using far less real memory (e.g., for 2,000 threads, only 70 MB of real memory is used, not 9 GB). If the size of the virtual memory is a concern, you can dial it down using the `threading.stack_size()` function. For example:
``` py
import threading
threading.stack_size(65536)
```
If you add this call and repeat the experiment of creating 2,000 threads, you’ll find that the Python process is now only using about 210 MB of virtual memory, although the
amount of real memory in use remains about the same. Note that the thread stack size
must be at least 32,768 bytes, and is usually restricted to be a multiple of the system memory page size (4096, 8192, etc.).
## Performing Simple Parallel Programming
##### Problem
You have a program that performs a lot of CPU-intensive work, and you want to make
it run faster by having it take advantage of multiple CPUs.
##### Solution
The `concurrent.futures` library provides a ProcessPoolExecutor class that can be used to execute computationally intensive functions in a separately running instance of the Python interpreter. However, in order to use it, you first need to have some computationally intensive work.

Let’s illustrate with a simple yet practical example. Suppose you have an entire directory of gzip-compressed Apache web server logs:
```
logs/
    20120701.log.gz
    20120702.log.gz
    20120703.log.gz
    20120704.log.gz
    20120705.log.gz
    20120706.log.gz
    ...
```
Further suppose each log file contains lines like this:
```
124.115.6.12 - - [10/Jul/2012:00:18:50 -0500] "GET /robots.txt ..." 200 71
210.212.209.67 - - [10/Jul/2012:00:18:51 -0500] "GET /ply/ ..." 200 11875
210.212.209.67 - - [10/Jul/2012:00:18:51 -0500] "GET /favicon.ico ..." 404 369
61.135.216.105 - - [10/Jul/2012:00:20:04 -0500] "GET /blog/atom.xml ..." 304 -
...
```
Here is a simple script that takes this data and identifies all hosts that have accessed the robots.txt file:
```py
# findrobots.py
import gzip
import io
import glob
def find_robots(filename):
    '''
    Find all of the hosts that access robots.txt in a single log file
    '''
    robots = set()
    with gzip.open(filename) as f:
        for line in io.TextIOWrapper(f,encoding='ascii'):
            fields = line.split()
            if fields[6] == '/robots.txt':
                robots.add(fields[0])
    return robots
def find_all_robots(logdir):
    '''
    Find all hosts across and entire sequence of files
    '''
    files = glob.glob(logdir+'/*.log.gz')
    all_robots = set()
    for robots in map(find_robots, files):
        all_robots.update(robots)
    return all_robots

if __name__ == '__main__':
    robots = find_all_robots('logs')
    for ipaddr in robots:
        print(ipaddr)
```
The preceding program is written in the commonly used map-reduce style. The function __find_robots()__ is mapped across a collection of filenames and the results are combined into a single result (the all_robots set in the find_all_robots() function).

Now, suppose you want to modify this program to use multiple CPUs. It turns out to be easy—simply replace the `map()` operation with a similar operation carried out on a process pool from the `concurrent.futures` library.

Here is a slightly modified version of the code:
``` py
# findrobots.py
import gzip
import io
import glob
from concurrent import futures
def find_robots(filename):
    '''
    Find all of the hosts that access robots.txt in a single log file
    '''
    robots = set()
    with gzip.open(filename) as f:
        for line in io.TextIOWrapper(f,encoding='ascii'):
            fields = line.split()
            if fields[6] == '/robots.txt':
                robots.add(fields[0])
    return robots

def find_all_robots(logdir):
    '''
    Find all hosts across and entire sequence of files
    '''
    files = glob.glob(logdir+'/*.log.gz')
    all_robots = set()
    with futures.ProcessPoolExecutor() as pool:
        for robots in pool.map(find_robots, files):
            all_robots.update(robots)
    return all_robots

if __name__ == '__main__':
    robots = find_all_robots('logs')
    for ipaddr in robots:
        print(ipaddr)
```
?> With this modification, the script produces the same result but runs about 3.5 times faster on our quad-core machine. The actual performance will vary according to the number of CPUs available on your machine.
##### Discussion
Typical usage of a __ProcessPoolExecutor__ is as follows:
``` py
from concurrent.futures import ProcessPoolExecutor
with ProcessPoolExecutor() as pool:
    ...
    do work in parallel using pool
    ...
```
Under the covers, a __ProcessPoolExecutor__ creates N independent running Python interpreters where N is the number of available CPUs detected on the system. You can change the number of processes created by supplying an optional argument to __ProcessPoolExecutor(N)__ . The pool runs until the last statement in the with block is executed, at which point the process pool is shut down. However, the program will wait until all submitted work has been processed.Work to be submitted to a pool must be defined in a function.

There are two methods for submission.
- If you are are trying to parallelize a list comprehension or a `map()` operation, you use `pool.map()` :
``` py
# A function that performs a lot of work
def work(x):
    ...
    return result
# Nonparallel code
results = map(work, data)
# Parallel implementation
with ProcessPoolExecutor() as pool:
    results = pool.map(work, data)
```
- Alternatively, you can manually submit single tasks using the `pool.submit()` method:
``` py
# Some function
def work(x):
    ...
    return result

with ProcessPoolExecutor() as pool:
    ...
    # Example of submitting work to the pool
    future_result = pool.submit(work, arg)
    # Obtaining the result (blocks until done)
    r = future_result.result()
    ...
```
If you manually submit a job, the result is an instance of __Future__ . To obtain the actual result, you call its `result()` method. This blocks until the result is computed and returned by the pool.

Instead of blocking, you can also arrange to have a callback function triggered upon completion instead. For example:
``` py
def when_done(r):
    print('Got:', r.result())

with ProcessPoolExecutor() as pool:
    future_result = pool.submit(work, arg)
    future_result.add_done_callback(when_done)
```
?> The user-supplied callback function receives an instance of __Future__ that must be used to obtain the actual result (i.e., by calling its result() method).

> - Although process pools can be easy to use, there are a number of important considerations to be made in designing larger programs. In no particular order:
    - This technique for parallelization only works well for problems that can be trivially decomposed into independent parts.
    - Work must be submitted in the form of simple functions. Parallel execution of instance methods, closures, or other kinds of constructs are not supported.
    - Function arguments and return values must be compatible with __pickle__ . Work is carried out in a separate interpreter using interprocess communication. Thus, data exchanged between interpreters has to be serialized.
    - Functions submitted for work should not maintain persistent state or have side
    effects. With the exception of simple things such as logging, you don’t really have any control over the behavior of child processes once started. Thus, to preserve your sanity, it is probably best to keep things simple and carry out work in pure-functions that don’t alter their environment.
    - Process pools are created by calling the `fork()` system call on Unix. This makes a clone of the Python interpreter, including all of the program state at the time of the fork. On Windows, an independent copy of the interpreter that does not clone state is launched. The actual forking process does not occur until the first `pool.map()` or `pool.submit()` method is called.
    - Great care should be made when combining process pools and programs that use
    threads. In particular, you should probably create and launch process pools prior
    to the creation of any threads (e.g., create the pool in the main thread at program startup).
## Dealing with the GIL (and How to Stop Worrying About It)
##### Problem
You’ve heard about the __Global Interpreter Lock__ (GIL), and are worried that it might be affecting the performance of your multithreaded program.
##### Solution
Although Python fully supports thread programming, parts of the C implementation of the interpreter are not entirely thread safe to a level of allowing fully concurrent execution. In fact, the interpreter is protected by a so-called Global Interpreter Lock (GIL) that only allows one Python thread to execute at any given time. The most noticeable effect of the GIL is that multithreaded Python programs are not able to fully take advantage of multiple CPU cores (e.g., a computationally intensive application using more than one thread only runs on a single CPU).

Before discussing common GIL workarounds, it is important to emphasize that the GIL tends to only affect programs that are heavily CPU bound (i.e., dominated by computation). If your program is mostly doing I/O, such as network communication, threads are often a sensible choice because they’re mostly going to spend their time sitting around waiting. In fact, you can create thousands of Python threads with barely a concern. Modern operating systems have no trouble running with that many threads, so it’s simply not something you should worry much about.

For CPU-bound programs, you really need to study the nature of the computation being performed. For instance, careful choice of the underlying algorithm may produce a far greater speedup than trying to parallelize an unoptimal algorithm with threads. Similarly, given that Python is interpreted, you might get a far greater speedup simply by moving performance-critical code into a C extension module. Extensions such as `NumPy` are also highly effective at speeding up certain kinds of calculations involving array data. Last, but not least, you might investigate alternative implementations, such as `PyPy`, which features optimizations such as a JIT compiler. 

It’s also worth noting that threads are not necessarily used exclusively for Performance. A CPU-bound program might be using threads to manage a graphical user interface, a network connection, or provide some other kind of service. In this case, the GIL can actually present more of a problem, since code that holds it for an excessively long period will cause annoying stalls in the non-CPU-bound threads. In fact, a poorly written C extension can actually make this problem worse, even though the computation part of the code might run faster than before.
Having said all of this, there are two common strategies for working around the limitations of the GIL. First, if you are working entirely in Python, you can use the `multiprocessing` module to create a process pool and use it like a co-processor.

For example, suppose you have the following thread code:
``` py
# Performs a large calculation (CPU bound)
def some_work(args):
    ...
    return result
# A thread that calls the above function
def some_thread():
    while True:
        ...
        r = some_work(args)
        ...
```
Here’s how you would modify the code to use a pool:
``` py
# Processing pool (see below for initiazation)
pool = None
# Performs a large calculation (CPU bound)
def some_work(args):
    ...
    return result
# A thread that calls the above function
def some_thread():
    while True:
        ...
        r = pool.apply(some_work, (args))
        ...
# Initiaze the pool
if __name__ == '__main__':
    import multiprocessing
    pool = multiprocessing.Pool()
```
This example with a pool works around the GIL using a neat trick. Whenever a thread wants to perform CPU-intensive work, it hands the work to the pool. The pool, in turn, hands the work to a separate Python interpreter running in a different process. While the thread is waiting for the result, it releases the GIL. Moreover, because the calculation is being performed in a separate interpreter, it’s no longer bound by the restrictions of the GIL. On a multicore system, you’ll find that this technique easily allows you to take advantage of all the CPUs.

The second strategy for working around the GIL is to focus on C extension programming. The general idea is to move computationally intensive tasks to C, independent of Python, and have the C code release the GIL while it’s working. This is done by inserting special macros into the C code like this:
``` c
#include "Python.h"
...
PyObject *pyfunc(PyObject *self, PyObject *args) {
    ...
    Py_BEGIN_ALLOW_THREADS
    // Threaded C code
    ...
    Py_END_ALLOW_THREADS
    ...
}
If you are using other tools to access C, such as the `ctypes` library or `Cython` , you may not need to do anything. For example, ctypes releases the GIL when calling into C by default.
##### Discussion
Many programmers, when faced with thread performance problems, are quick to blame the GIL for all of their ills. However, doing so is shortsighted and naive. Just as a real world example, mysterious “stalls” in a multithreaded network program might be caused by something entirely different (e.g., a stalled DNS lookup) rather than anything related to the GIL. The bottom line is that you really need to study your code to know if the GIL is an issue or not. Again, realize that the GIL is mostly concerned with CPU-bound processing, not I/O.

If you are going to use a process pool as a workaround, be aware that doing so  involves data serialization and communication with a different Python interpreter. For this to work, the operation to be performed needs to be contained within a Python function defined by the def statement (i.e., no lambdas, closures, callable instances, etc.), and the function arguments and return value must be compatible with pickle . Also, the amount of work to be performed must be sufficiently large to make up for the extra communication overhead.

Another subtle aspect of pools is that mixing threads and process pools together can be a good way to make your head explode. If you are going to use both of these features together, it is often best to create the process pool as a singleton at program startup, prior to the creation of any threads. Threads will then use the same process pool for all of their computationally intensive work.

For C extensions, the most important feature is maintaining isolation from the Python interpreter process. That is, if you’re going to offload work from Python to C, you  need to make sure the C code operates independently of Python. This means using no Python data structures and making no calls to Python’s C API. Another consideration is that you want to make sure your C extension does enough work to make it all worthwhile. That is, it’s much better if the extension can perform millions of calculations as opposed to just a few small calculations. 

Needless to say, these solutions to working around the GIL don’t apply to all possible
problems. For instance, certain kinds of applications don’t work well if separated into multiple processes, nor may you want to code parts in C. For these kinds of applications, you may have to come up with your own solution (e.g., multiple processes accessing shared memory regions, multiple interpreters running in the same process, etc.). Alternatively, you might look at some other implementations of the interpreter, such as `PyPy`.
## Defining an Actor Task
##### Problem
You’d like to define tasks with behavior similar to “actors” in the so-called “actor model.”
##### Solution
The “__actor model__” is one of the oldest and most simple approaches to concurrency and distributed computing. In fact, its underlying simplicity is part of its appeal. In a nutshell, an actor is a concurrently executing task that simply acts upon messages sent to it. In response to these messages, it may decide to send further messages to other actors.Communication with actors is one way and asynchronous. Thus, the sender of a message does not know when a message actually gets delivered, nor does it receive a response or acknowledgment that the message has been processed. 

__Actors__ are straightforward to define using a combination of a `thread` and a `queue`. 

For example:
``` py
from queue import Queue
from threading import Thread, Event
# Sentinel used for shutdown
class ActorExit(Exception):
    pass
class Actor:
    def __init__(self):
        self._mailbox = Queue()
    def send(self, msg):
        '''
        Send a message to the actor
        '''
        self._mailbox.put(msg)
    def recv(self):
        '''
        Receive an incoming message
        '''
        msg = self._mailbox.get()
        if msg is ActorExit:
            raise ActorExit()
        return msg
    def close(self):
        '''
        Close the actor, thus shutting it down
        '''
        self.send(ActorExit)
    def start(self):
        '''
        Start concurrent execution
        '''
        self._terminated = Event()
        t = Thread(target=self._bootstrap)
        t.daemon = True
        t.start()
    def _bootstrap(self):
        try:
            self.run()
        except ActorExit:
            pass
        finally:
            self._terminated.set()
    def join(self):
        self._terminated.wait()
    def run(self):
        '''
        Run method to be implemented by the user
        '''
        while True:
            msg = self.recv()
# Sample ActorTask
class PrintActor(Actor):
    def run(self):
        while True:
            msg = self.recv()
            print('Got:', msg)
# Sample use
p = PrintActor()
p.start()
p.send('Hello')
p.send('World')
p.close()
p.join()
```
In this example, Actor instances are things that you simply send a message to using their `send()` method. Under the covers, this places the message on a queue and hands it off to an internal thread that runs to process the received messages. The `close()` method is programmed to shut down the actor by placing a special sentinel value ( ActorExit ) on the queue.

Users define new actors by inheriting from `Actor` and redefining the `run()` method to implement their custom processing. The usage of the ActorExit exception is such that user-defined code can be programmed to catch the termination request and handle it if appropriate (the exception is raised by the `get()` method and propagated).

If you relax the requirement of concurrent and asynchronous message delivery, actor-like objects can also be minimally defined by generators.

For example:
``` py
def print_actor():
    while True:
        try:
            msg = yield
            # Get a message
            print('Got:', msg)
        except GeneratorExit:
            print('Actor terminating')
# Sample use
p = print_actor()
next(p)
# Advance to the yield (ready to receive)
p.send('Hello')
p.send('World')
p.close()
```
##### Discussion
Part of the appeal of actors is their underlying simplicity. In practice, there is just one core operation, `send()` . Plus, the general concept of a “message” in actor-based systems is something that can be expanded in many different directions.

For example, you could pass tagged messages in the form of tuples and have actors take different courses of action like this:
``` py
class TaggedActor(Actor):
    def run(self):
        while True:
            tag, *payload = self.recv()
            getattr(self,'do_'+tag)(*payload)
    # Methods correponding to different message tags
    def do_A(self, x):
        print('Running A', x)
    def do_B(self, x, y):
        print('Running B', x, y)
# Example
a = TaggedActor()
a.start()
a.send(('A', 1)) # Invokes do_A(1)
a.send(('B', 2, 3)) # Invokes do_B(2,3)
```
As another example, here is a variation of an actor that allows arbitrary functions to be executed in a worker and results to be communicated back using a special Result object:
``` py
from threading import Event
class Result:
    def __init__(self):
        self._evt = Event()
        self._result = None
    def set_result(self, value):
        self._result = value
        self._evt.set()
    def result(self):
        self._evt.wait()
        return self._result
class Worker(Actor):
    def submit(self, func, *args, **kwargs):
        r = Result()
        self.send((func, args, kwargs, r))
        return r
    def run(self):
        while True:
            func, args, kwargs, r = self.recv()
            r.set_result(func(*args, **kwargs))
# Example use
worker = Worker()
worker.start()
r = worker.submit(pow, 2, 3)
print(r.result())
```
> Last, but not least, the concept of “sending” a task a message is something that can be scaled up into systems involving multiple processes or even large distributed systems.For example, the `send()` method of an actor-like object could be programmed to transmit data on a socket connection or deliver it via some kind of messaging infrastructure (e.g., AMQP, ZMQ, etc.).

## Implementing Publish/Subscribe Messaging
##### Problem
You have a program based on communicating threads and want them to implement publish/subscribe messaging.
##### Solution
To implement publish/subscribe messaging, you typically introduce a separate “ex‐change” or “gateway” object that acts as an intermediary for all messages. That is, instead of directly sending a message from one task to another, a message is sent to the exchange and it delivers it to one or more attached tasks. Here is one example of a very simple exchange implementation:
``` py
from collections import defaultdict
class Exchange:
    def __init__(self):
        self._subscribers = set()
    def attach(self, task):
        self._subscribers.add(task)
    def detach(self, task):
        self._subscribers.remove(task)
    def send(self, msg):
        for subscriber in self._subscribers:
            subscriber.send(msg)
# Dictionary of all created exchanges
_exchanges = defaultdict(Exchange)
# Return the Exchange instance associated with a given name
def get_exchange(name):
    return _exchanges[name]
```
An exchange is really nothing more than an object that keeps a set of active subscribers and provides methods for attaching, detaching, and sending messages. Each exchange is identified by a name, and the get_exchange() function simply returns the Exchange instance associated with a given name.
Here is a simple example that shows how to use an exchange:
``` py
# Example of a task. Any object with a send() method
class Task:
    ...
    def send(self, msg):
        ...

task_a = Task()
task_b = Task()
# Example of getting an exchange
exc = get_exchange('name')
# Examples of subscribing tasks to it
exc.attach(task_a)
exc.attach(task_b)
# Example of sending messages
exc.send('msg1')
exc.send('msg2')
# Example of unsubscribing
exc.detach(task_a)
exc.detach(task_b)
```
Although there are many different variations on this theme, the overall idea is the same. Messages will be delivered to an exchange and the exchange will deliver them to attached subscribers.
##### Discussion
The concept of tasks or threads sending messages to one another (often via queues) is easy to implement and quite popular. However, the benefits of using a public/subscribe (pub/sub) model instead are often overlooked.

- First, the use of an exchange can simplify much of the plumbing involved in setting up communicating threads. Instead of trying to wire threads together across multiple program modules, you only worry about connecting them to a known exchange. In some sense, this is similar to how the logging library works. In practice, it can make it easier to decouple various tasks in the program.

- Second, the ability of the exchange to broadcast messages to multiple subscribers opens up new communication patterns. For example, you could implement systems with redundant tasks, broadcasting, or fan-out. You could also build debugging and diagnostic tools that attach themselves to exchanges as ordinary subscribers. For example, here is a simple diagnostic class that would display sent messages:
``` py
class DisplayMessages:
    def __init__(self):
        self.count = 0
    def send(self, msg):
        self.count += 1
        print('msg[{}]: {!r}'.format(self.count, msg))

exc = get_exchange('name')
d = DisplayMessages()
exc.attach(d)
```
?> Last, but not least, a notable aspect of the implementation is that it works with a variety of task-like objects. For example, the receivers of a message could be __actors__, __coroutines__, __network connections__, or just about anything that implements a proper `send()` method.

One potentially problematic aspect of an exchange concerns the proper attachment and detachment of subscribers. In order to properly manage resources, every subscriber that attaches must eventually detach. This leads to a programming model similar to this:
``` py
exc = get_exchange('name')
exc.attach(some_task)
try:
    ...
finally:
    exc.detach(some_task)
```
In some sense, this is similar to the usage of files, locks, and similar objects. Experience has shown that it is quite easy to forget the final `detach()` step. To simplify this, you might consider the use of the context-management protocol.

For example, adding a subscribe() method to the exchange like this:
``` py
from contextlib import contextmanager
from collections import defaultdict
class Exchange:
    def __init__(self):
        self._subscribers = set()
    def attach(self, task):
        self._subscribers.add(task)
    def detach(self, task):
        self._subscribers.remove(task)
    @contextmanager
    def subscribe(self, *tasks):
        for task in tasks:
            self.attach(task)
        try:
            yield
        finally:
            for task in tasks:
                self.detach(task)
    def send(self, msg):
        for subscriber in self._subscribers:
            subscriber.send(msg)
# Dictionary of all created exchanges
_exchanges = defaultdict(Exchange)
# Return the Exchange instance associated with a given name
def get_exchange(name):
    return _exchanges[name]
# Example of using the subscribe() method
exc = get_exchange('name')
with exc.subscribe(task_a, task_b):
    ...
    exc.send('msg1')
    exc.send('msg2')
    ...
# task_a and task_b detached here
```
Finally, it should be noted that there are numerous possible extensions to the exchange idea. For example, exchanges could implement an entire collection of message channels or apply pattern matching rules to exchange names. Exchanges can also be extended into distributed computing applications (e.g., routing messages to tasks on different machines, etc.).
## Using Generators As an Alternative to Threads
##### Problem
You want to implement concurrency using generators (coroutines) as an alternative to system threads. This is sometimes known as user-level threading or green threading.
##### Solution
To implement your own concurrency using generators, you first need a fundamental insight concerning generator functions and the yield statement. Specifically, the fundamental behavior of yield is that it causes a generator to suspend its execution. By suspending execution, it is possible to write a scheduler that treats generators as a kind of “task” and alternates their execution using a kind of cooperative task switching.

To illustrate this idea, consider the following two generator functions using a simple yield :
```py
# Two simple generator functions
def countdown(n):
    while n > 0:
        print('T-minus', n)
        yield
        n -= 1
    print('Blastoff!')
def countup(n):
    x = 0
    while x < n:
        print('Counting up', x)
        yield
        x += 1
```
These functions probably look a bit funny using yield all by itself. However, consider the following code that implements a simple task scheduler:
``` py
from collections import deque
class TaskScheduler:
    def __init__(self):
        self._task_queue = deque()
    def new_task(self, task):
        '''
        Admit a newly started task to the scheduler
        '''
        self._task_queue.append(task)
    def run(self):
        '''
        Run until there are no more tasks
        '''
        while self._task_queue:
            task = self._task_queue.popleft()
            try:
                # Run until the next yield statement
                next(task)
                self._task_queue.append(task)
            except StopIteration:
                # Generator is no longer executing
                pass
# Example use
sched = TaskScheduler()
sched.new_task(countdown(10))
sched.new_task(countdown(5))
sched.new_task(countup(15))
sched.run()
```
In this code, the TaskScheduler class runs a collection of generators in a round-robin manner—each one running until they reach a yield statement. For the sample, the output will be as follows:
```
T-minus 10
T-minus 5
Counting up 0
T-minus 9
T-minus 4
Counting up 1
T-minus 8
T-minus 3
Counting up 2
T-minus 7
T-minus 2
...
```
At this point, you’ve essentially implemented the tiny core of an “operating system” if you will. Generator functions are the tasks and the yield statement is how tasks signal that they want to suspend. The scheduler simply cycles over the tasks until none are left executing. In practice, you probably wouldn’t use generators to implement concurrency for something as simple as shown. Instead, you might use generators to replace the use of threads
when implementing actors or network servers.

The following code illustrates the use of generators to implement a thread-free version of actors:
``` py
from collections import deque
class ActorScheduler:
    def __init__(self):
        self._actors = { } # Mapping of names to actors
        self._msg_queue = deque() # Message queue
    def new_actor(self, name, actor):
        '''
        Admit a newly started actor to the scheduler and give it a name
        '''
        self._msg_queue.append((actor,None))
        self._actors[name] = actor
    def send(self, name, msg):
        '''
        Send a message to a named actor
        '''
        actor = self._actors.get(name)
        if actor:
            self._msg_queue.append((actor,msg))
    def run(self):
        '''
        Run as long as there are pending messages.
        '''
        while self._msg_queue:
            actor, msg = self._msg_queue.popleft()
        try:
            actor.send(msg)
        except StopIteration:
            pass
# Example use
if __name__ == '__main__':
    def printer():
        while True:
            msg = yield
            print('Got:', msg)
    def counter(sched):
        while True:
            # Receive the current
            n = yield
            if n == 0:
                break
            # Send to the printer
            sched.send('printer',
            # Send the next count to the counter task (recursive)
            sched.send('counter', n-1)
    sched = ActorScheduler()
    # Create the initial actors
    sched.new_actor('printer', printer())
    sched.new_actor('counter', counter(sched))
    # Send an initial message to the counter to initiate
    sched.send('counter', 10000)
    sched.run()
```
The execution of this code might take a bit of study, but the key is the queue of pending messages. Essentially, the scheduler runs as long as there are messages to deliver. A remarkable feature is that the counter generator sends messages to itself and ends up in a recursive cycle not bound by Python’s recursion limit.

Here is an advanced example showing the use of generators to implement a concurrent network application:
``` py
from collections import deque
from select import select
# This class represents a generic yield event in the scheduler
class YieldEvent:
    def handle_yield(self, sched, task):
        pass
    def handle_resume(self, sched, task):
        pass
# Task Scheduler
class Scheduler:
    def __init__(self):
        self._numtasks = 0  # total num of tasks
        self._ready = deque() # Tasks ready to run
        self._read_waiting = {} # Tasks  waiting to read
        self._write_waiting = {} # Tasks waiting to write
    # Poll for I/O events and restart waiting tasks
    def _iopoll(self):
        rset,wset,eset = select(self._read_waiting,
                                self._write_waiting,[])
        for r in rset:
            evt, task = self._read_waiting.pop(r)
            evt.handle_resume(self, task)
        for w in wset:
            evt, task = self._write_waiting.pop(w)
            evt.handle_resume(self, task)
    def new(self,task):
        '''
        Add a newly started task to the scheduler
        '''
        self._ready.append((task, None))
        self._numtasks += 1
    def add_ready(self, task, msg=None):
        '''
        Append an already started task to the ready queue.
        msg is what to send into the task when it resumes.
        '''
        self._ready.append((task, msg))
    # Add a task to the reading set
    def _read_wait(self, fileno, evt, task):
        self._read_waiting[fileno] = (evt, task)
    # Add a task to the write set
    def _write_wait(self, fileno, evt, task):
        self._write_waiting[fileno] = (evt, task)
    def run(self):
        '''
        Run the task scheduler until there are no tasks
        '''
        while self._numtasks:
            if not self._ready:
                self._iopoll()
            task, msg = self._ready.popleft()
            try:
                # Run the coroutine to the next yield
                r = task.send(msg)
                if isinstance(r, YieldEvent):
                    r.handle_yield(self, task)
                else:
                    raise RuntimeError('unrecognized yield event')
            except StopIteration:
                self._numtasks -= 1
# Example implementation of coroutine-based socket I/O
class ReadSocket(YieldEvent):
    def __init__(self, sock, nbytes):
        self.sock = sock
        self.nbytes = nbytes
    def handle_yield(self, sched, task):
        sched._read_wait(self.sock.fileno(), self, task)
    def handle_resume(self, sched, task):
        data = self.sock.recv(self.nbytes)
        sched.add_ready(task, data)

class WriteSocket(YieldEvent):
    def __init__(self, sock, data):
        self.sock = sock
        self.data = data
    def handle_yield(self, sched, task):
        sched._write_wait(self.sock.fileno(), self, task)
    def handle_resume(self, sched, task):
        nsent = self.sock.send(self.data)
        sched.add_ready(task, nsent)

class AcceptSocket(YieldEvent):
    def __init__(self, sock):
        self.sock = sock
    def handle_yield(self, sched, task):
        sched._read_wait(self.sock.fileno(), self, task)
    def handle_resume(self, sched, task):
        r = self.sock.accept()
        sched.add_ready(task, r)
# Wrapper around a socket object for use with yield
class Socket(object):
    def __init__(self, sock):
        self._sock = sock
    def recv(self, maxbytes):
        return ReadSocket(self._sock, maxbytes)
    def send(self, data):
        return WriteSocket(self._sock, data)
    def accept(self):
        return AcceptSocket(self._sock)
    def __getattr__(self, name):
        return getattr(self._sock, name)

if __name__ == '__main__':
    from socket import socket, AF_INET, SOCK_STREAM
    import time
# Example of a function involving generators. This should
# be called using line = yield from readline(sock)
def readline(sock):
    chars = []
    while True:
        c = yield sock.recv(1)
        if not c:
            break
        chars.append(c)
        if c == b'\n':
            break
    return b''.join(chars)
# Echo server using generators
class EchoServer:
    def __init__(self,addr,sched):
        self.sched = sched
lect() . Essentially, it just exposes the underlying file descriptor of the socket used by
        sched.new(self.server_loop(addr))

    def server_loop(self,addr):
        s = Socket(socket(AF_INET,SOCK_STREAM))
        s.bind(addr)
        s.listen(5)
        while True:
            c,a = yield s.accept()
            print('Got connection from ', a)
            self.sched.new(self.client_handler(Socket(c)))

    def client_handler(self,client):
        while True:
            line = yield from readline(client)
        if not line:
            break
        line = b'GOT:' + line
        while line:
            nsent = yield client.send(line)
            line = line[nsent:]
        client.close()
        print('Client closed')

sched = Scheduler()
EchoServer(('',16000),sched)
sched.run()
```
This code will undoubtedly require a certain amount of careful study. However, it is essentially implementing a small operating system. There is a queue of tasks ready to run and there are waiting areas for tasks sleeping for I/O. Much of the scheduler involves moving tasks between the ready queue and the I/O waiting area.
##### Discussion
When building generator-based concurrency frameworks, it is most common to work with the more general form of yield :
``` py
def some_generator():
    ...
    result = yield data
    ...
```
Functions that use yield in this manner are more generally referred to as “coroutines.”

Within a scheduler, the yield statement gets handled in a loop as follows:
``` py
f = some_generator()    
# Initial result. Is None to start since nothing has been computed
result = None
while True:
    try:
        data = f.send(result)
        result = ... do some calculation ...
    except StopIteration:
        break
```
The logic concerning the result is a bit convoluted. However, the value passed to `send()` defines what gets returned when the yield statement wakes back up. So, if a yield is going to return a result in response to data that was previously yielded, it gets returned on the next `send()` operation. If a generator function has just started, sending in a value of __None__ simply makes it advance to the first yield statement.

In addition to sending in values, it is also possible to execute a `close()` method on a generator. This causes a silent __GeneratorExit__ exception to be raised at the yield statement, which stops execution. If desired, a generator can catch this exception and perform cleanup actions.

It’s also possible to use the `throw()` method of a generator to raise an arbitrary execution at the yield statement. A task scheduler might use this to communicate errors into running generators.

The yield from statement used in the last example is used to implement coroutines that serve as subroutines or procedures to be called from other generators.Essentially, control transparently transfers to the new function. Unlike normal generators, a function that is called using yield from can return a value that becomes the result of the yield from statement. More information about yield from can be found in PEP 380.

Finally, if programming with generators, it is important to stress that there are some major limitations. In particular, you get none of the benefits that threads provide. For instance, if you execute any code that is CPU bound or which blocks for I/O, it will suspend the entire task scheduler until the completion of that operation. To work around this, your only real option is to delegate the operation to a separate thread or process where it can run independently. Another limitation is that most Python libraries have not been written to work well with generator-based threading. If you take this approach, you may find that you need to write replacements for many standard library functions.

As basic background on coroutines and the techniques utilized in this recipe, see PEP 342 and “A Curious Course on Coroutines and Concurrency”. PEP 3156 also has a modern take on asynchronous I/O involving coroutines. In practice, it is extremelyunlikely that you will write a low-level coroutine scheduler yourself. However, ideas surrounding coroutines are the basis for many popular libraries, including __gevent__, __greenlet__, __Stackless__ Python, and similar projects.

## Polling Multiple Thread Queues
##### Problem
You have a collection of thread queues, and you would like to be able to poll them for incoming items, much in the same way as you might poll a collection of network connections for incoming data.
##### Solution
A common solution to polling problems involves a little-known trick involving a hidden loopback network connection. Essentially, the idea is as follows: for each queue (or any object) that you want to poll, you create a pair of connected sockets. You then write on one of the sockets to signal the presence of data. The other sockect is then passed to `select()` or a similar function to poll for the arrival of data.

Here is some sample code that illustrates this idea:
``` py
import queue
import socket
import os
class PollableQueue(queue.Queue):
    def __init__(self):
        super().__init__()
            # Create a pair of connected sockets
            if os.name == 'posix':
                self._putsocket, self._getsocket = socket.socketpair()
            else:
                # Compatibility on non-POSIX systems
                server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                server.bind(('127.0.0.1', 0))
                server.listen(1)
                self._putsocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                self._putsocket.connect(server.getsockname())
                self._getsocket, _ = server.accept()
                server.close()
    def fileno(self):
        return self._getsocket.fileno()
    def put(self, item):
        super().put(item)
        self._putsocket.send(b'x')
    def get(self):
        self._getsocket.recv(1)
        return super().get()
```
In this code, a new kind of Queue instance is defined where there is an underlying pair of connected sockets.

The `socketpair()` function on Unix machines can establish such sockets easily. On Windows, you have to fake it using code similar to that shown (it looks a bit weird, but a server socket is created and a client immediately connects to it afterward). The normal `get()` and `put()` methods are then redefined slightly to perform a small bit of I/O on these sockets. The `put()` method writes a single byte of data to one of the sockets after putting data on the queue. The `get()` method reads a single byte of data from the other socket when removing an item from the queue.

The __fileno()__ method is what makes the queue pollable using a function such as `select()` . Essentially, it just exposes the underlying file descriptor of the socket used by the __get()__ function.

Here is an example of some code that defines a consumer which monitors multiple queues for incoming items:
``` py
import select
import threading
def consumer(queues):
    '''
    Consumer that reads data on multiple queues simultaneously
    '''
    while True:
        can_read, _, _ = select.select(queues,[],[])
        for r in can_read:
            item = r.get()
            print('Got:', item)

q1 = PollableQueue()
q2 = PollableQueue()
q3 = PollableQueue()
t = threading.Thread(target=consumer, args=([q1,q2,q3],))
t.daemon = True
t.start()
# Feed data to the queues
q1.put(1)
q2.put(10)
q3.put('hello')
q2.put(15)
...
```
If you try it, you’ll find that the consumer indeed receives all of the put items, regardless of which queues they are placed in.
##### Discussion
The problem of polling non-file-like objects, such as queues, is often a lot trickier than it looks. For instance, if you don’t use the socket technique shown, your only option is to write code that cycles through the queues and uses a timer, like this:
``` py
import time
def consumer(queues):
    while True:
        for q in queues:
            if not q.empty():
                item = q.get()
                print('Got:', item)
        # Sleep briefly to avoid 100% CPU
        time.sleep(0.01)
```
This might work for certain kinds of problems, but it’s clumsy and introduces other weird performance problems. For example, if new data is added to a queue, it won’t be detected for as long as 10 milliseconds (an eternity on a modern processor). You run into even further problems if the preceding polling is mixed with the polling of other objects, such as network sockets. For example, if you want to poll both sockets
and queues at the same time, you might have to use code like this:
``` py
import select
def event_loop(sockets, queues):
    while True:
        # polling with a timeout
        can_read, _, _ = select.select(sockets, [], [], 0.01)
        for r in can_read:
            handle_read(r)
        for q in queues:
            if not q.empty():
                item = q.get()
                print('Got:', item)
```
The solution shown solves a lot of these problems by simply putting queues on equal status with sockets. A single `select()` call can be used to poll for activity on both. It is not necessary to use timeouts or other time-based hacks to periodically check. Moreover, if data gets added to a queue, the consumer will be notified almost instantaneously. Although there is a tiny amount of overhead associated with the underlying I/O, it often is worth it to have better response time and simplified coding.

## Launching a Daemon Process on Unix
##### Problem
You would like to write a program that runs as a proper daemon process on Unix or Unix-like systems.
##### Solution
Creating a proper daemon process requires a precise sequence of system calls and careful attention to detail. The following code shows how to define a daemon process along with the ability to easily stop it once launched:
``` py
#!/usr/bin/env python3
# daemon.py
import os
import sys
import atexit
import signal
def daemonize(pidfile, *, stdin='/dev/null',
                          stdout='/dev/null',
                          stderr='/dev/null'):
    if os.path.exists(pidfile):
        raise RuntimeError('Already running')
    # First fork (detaches from parent)
    try:
        if os.fork() > 0:
            raise SystemExit(0) # Parent exit
    except OSError as e:
        raise RuntimeError('fork #1 failed.')
    os.chdir('/')
    os.umask(0)
    os.setsid()
    # Second fork (relinquish session leadership)
    try:
        if os.fork() > 0:
            raise SystemExit(0)
    except OSError as e:
        raise RuntimeError('fork #2 failed.')
    # Flush I/O buffers
    sys.stdout.flush()
    sys.stderr.flush()
    # Replace file descriptors for stdin, stdout, and stderr
    with open(stdin, 'rb', 0) as f:
        os.dup2(f.fileno(), sys.stdin.fileno())
    with open(stdout, 'ab', 0) as f:
        os.dup2(f.fileno(), sys.stdout.fileno())
    with open(stderr, 'ab', 0) as f:
        os.dup2(f.fileno(), sys.stderr.fileno())
    # Write the PID file
    with open(pidfile,'w') as f:
        print(os.getpid(),file=f)
    # Arrange to have the PID file removed on exit/signal
    atexit.register(lambda: os.remove(pidfile))
    # Signal handler for termination (required)
    def sigterm_handler(signo, frame):
        raise SystemExit(1)
    signal.signal(signal.SIGTERM, sigterm_handler)

def main():
    import time
    sys.stdout.write('Daemon started with pid {}\n'.format(os.getpid()))
    while True:
        sys.stdout.write('Daemon Alive! {}\n'.format(time.ctime()))
        time.sleep(10)

if __name__ == '__main__':
    PIDFILE = '/tmp/daemon.pid'
    if len(sys.argv) != 2:
        print('Usage: {} [start|stop]'.format(sys.argv[0]), file=sys.stderr)
        raise SystemExit(1)
    if sys.argv[1] == 'start':
        try:
            daemonize(PIDFILE,
                      stdout='/tmp/daemon.log',
                      stderr='/tmp/dameon.log')

        except RuntimeError as e:
            print(e, file=sys.stderr)
            raise SystemExit(1)
        main()
    elif sys.argv[1] == 'stop':
        if os.path.exists(PIDFILE):
            with open(PIDFILE) as f:
                os.kill(int(f.read()), signal.SIGTERM)
        else:
            print('Not running', file=sys.stderr)
            raise SystemExit(1)
    else:
        print('Unknown command {!r}'.format(sys.argv[1]), file=sys.stderr)
        raise SystemExit(1)
```
To launch the daemon, the user would use a command like this:
``` bash
daemon.py start
cat /tmp/daemon.pid
2882
tail -f /tmp/daemon.log
Daemon started with pid 2882
Daemon Alive! Fri Oct 12 13:45:37 2012
Daemon Alive! Fri Oct 12 13:45:47 2012
...
```
Daemon processes run entirely in the background, so the command returns immediately. However, you can view its associated pid file and log, as just shown. To stop the daemon, use:
```bash
daemon.py stop
```
##### Discussion
This recipe defines a function __daemonize()__ that should be called at program startup to make the program run as a daemon. The signature to __daemonize()__ is using keyword only arguments to make the purpose of the optional arguments more clear when used. This forces the user to use a call such as this:
``` py
daemonize('daemon.pid',
          stdin='/dev/null,
          stdout='/tmp/daemon.log',
          stderr='/tmp/daemon.log')
```
As opposed to a more cryptic call such as:
``` py
# Illegal. Must use keyword arguments
daemonize('daemon.pid',
          '/dev/null', '/tmp/daemon.log','/tmp/daemon.log')
```
The steps involved in creating a daemon are fairly cryptic, but the general idea is as
follows. 
- First, a daemon has to detach itself from its parent process. This is the purpose of the first `os.fork()` operation and immediate termination by the parent.

- After the child has been orphaned, the call to `os.setsid()` creates an entirely new
process session and sets the child as the leader. This also sets the child as the leader of a new process group and makes sure there is no controlling terminal. If this all sounds a bit too magical, it has to do with getting the daemon to detach properly from the terminal and making sure that things like signals don’t interfere with its operation.

- The calls to `os.chdir()` and `os.umask(0)` change the current working directory and reset the file mode mask. Changing the directory is usually a good idea so that the daemon is no longer working in the directory from which it was launched.

- The second call to `os.fork()` is by far the more mysterious operation here. This step makes the daemon process give up the ability to acquire a new controlling terminal and provides even more isolation (essentially, the daemon gives up its session leadership and thus no longer has the permission to open controlling terminals). Although you could probably omit this step, it’s typically recommended.

- Once the daemon process has been properly detached, it performs steps to reinitialize the standard I/O streams to point at files specified by the user. This part is actually somewhat tricky. References to file objects associated with the standard I/O streams are found in multiple places in the interpreter ( __sys.stdout__ , __sys.\_\_stdout\_\___ , etc.).

- Simply closing `sys.stdout` and reassigning it is not likely to work correctly, because there’s no way to know if it will fix all uses of __sys.stdout__ . Instead, a separate file object is opened, and the `os.dup2()` call is used to have it replace the file descriptor currently being used by sys.stdout .

- When this happens, the original file for sys.stdout will be closed and the new one takes its place. It must be emphasized that any file encoding or text handling already applied to the standard I/O streams will remain in place.

- A common practice with daemon processes is to write the process ID of the daemon in
a file for later use by other programs. The last part of the daemonize() function writes this file, but also arranges to have the file removed on program termination. 

- The `atexit.register()` function registers a function to execute when the Python interpreter terminates. The definition of a signal handler for __SIGTERM__ is also required for a graceful termination. The signal handler merely raises __SystemExit()__ and nothing more. This might look unnecessary, but without it, termination signals kill the interpreter without performing the cleanup actions registered with `atexit.register()` .An example of code that kills the daemon can be found in the handling of the stop command at the end of the program.

> More information about writing daemon processes can be found in Advanced Programming in the UNIX Environment, 2nd Edition, by W. Richard Stevens and Stephen A. Rago (Addison-Wesley, 2005). Although focused on C programming, all of the material is easily adapted to Python, since all of the required POSIX functions are available in the standard library.