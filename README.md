easypool
========

Python module to easily create threadpools and serverpol with additional advanced functionality.

## Installation

Install from PyPi using [pip](http://www.pip-installer.org/en/latest/), a
package manager for Python.

    $ pip install easypool

Do not have pip installed? Try installing it, by running this from the command
line:

    $ curl https://raw.github.com/pypa/pip/master/contrib/get-pip.py | python

Or, you can [download the source code
(ZIP)](https://github.com/RobertLabonte/easypool/zipball/master "easypool
source code") for `easypool`, and then run:

    $ python setup.py install

You may need to run the above commands with `sudo`.

## Getting Started

easypool was created to make the creation of threadpools in Python very easy.  It
also supports the dynamic creation of threadpools by accepting common Python data
structures, including but not limited: integers, floats, lists, dictionaries, tuples,
strings and any other data structure that can be iterated.  easypool can link each
thread to the value of each item given to it (e.g. Inialize easypool with list
[a, b, c] and thread 0 is linked to a, thread 1 is linked to b and thread 1 is linked
to value c) the linked value can be sent as the first argument to the function that 
the thread is executing.  Link threads to server addresses and create a function 
that remotely executes commands on the server address. 

Simple threadpool example:
```python
from easypool import ThreadPool

threadpool = ThreadPool(3)
# foo(bar) becomes add_task(foo, bar)

# Keep adding tasks to the threadpool
threadpool.add_task(foo, bar)
threadpool.add_task(foo, bar)
threadpool.add_task(foo, bar)
threadpool.add_task(foo, bar)

# Wait for all tasks to copmplete
threadpool.wait_completion()
```

Server Pool example:
```python
from easypool import ThreadPool
import subprocess
import shlex
import threading
import string


server_list = ['127.0.0.1', '127.0.0.1', '127.0.0.1']
serverpool = ThreadPool(server_list, send_item=True)
threadlock = threading.RLock()

def get_uptime(server):
    ssh_cmd = "ssh " + str(server) + " 'uptime'"
    ssh_cmd_list = shlex.split(ssh_cmd)
    p = subprocess.Popen(ssh_cmd_list, shell=False, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    stdout, stderr = p.communicate()
    threadlock.acquire()
    print("Server: %s :: %s" % (server, stdout.rstrip()))
    threadlock.release()
    return

for i in range(6):
    serverpool.add_task(get_uptime)

serverpool.wait_completion()
```

## Advanced Options

send_item:   Acceptable values: True or False.  Threads are linked to the items of iterator given to 
             initialize easypool.  The item is sent as the first argument to the function being executed. 
max_pool:    Acceptable values: int or float.  Maximum size of the threadpool.  Example: If easypool is 
             initialized with a list containing 50 elements and max_pool is set to 20, the threadpool will 
             be shortened to only execute 20 functions at a time.  Default: Unlimited
min_pool:    Acceptable values: int or float.  Minimum size of the threadpool.  Example: If easypool is 
             initialized with a list containing 2 elements and min_pool is set to 5, the threadpool will 
             be expanded to execute 5 functions at a time.  Default: No minimum (easypool always creates a 
             threadpool of at least 1)
queue_type:  Options: 'fifo' or 'lifo'.  Default: 'fifo'.  'fifo' = First in first out; the first task 
             put into the queue is the first task executed from the queue.  'lifo' = Last in first out;
             the last task put into the queue is the first task executed from the queue.

Sample easypool initialization with a list:
```python
from easypool import ThreadPool

server_list = ['server1', 'server2', 'server3', 'server4', 'server5']
serverpool = ThreadPool(server_list, send_item=True, max_pool=5, min_pool=3, queue_type=lifo)
```

Serverpool with each server handling 2 tasks at a time:
```python
from easypool import ThreadPool

server_list = ['server1', 'server2', 'server3', 'server4', 'server5']
serverpool = ThreadPool(server_list, send_item=True, min_pool=(len(server_list)*2))
```
