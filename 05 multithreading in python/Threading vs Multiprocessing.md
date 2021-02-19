# Threading vs Multiprocessing

Threading:
- A new thread is spawned within the existing process
- Starting a thread is faster than starting a process
- Memory is shared between all threads
- Mutexes often necessary to control access to shared data
- One GIL (Global Interpreter Lock) for all threads

Multiprocessing:
- A new process is started independent from the first process
- Starting a process is slower than starting a thread
- Memory is not shared between processes
- Mutexes not necessary (unless threading in the new process)
- One GIL (Global Interpreter Lock) for each process





## Examples

Threads.py

```python
from threading import Thread
import os
import math

def calc():
	for i in range(0, 4000000):
		math.sqrt(i)

threads = []

for i in range(os.cpu_count()):
	print('registering thread %d' % i)
	threads.append(Thread(target=calc))

for thread in threads:
	thread.start()

for thread in threads:
	thread.join()
```

<br>
<br>

processes.py

```python
from multiprocessing import Process
import os
import math

def calc():
	for i in range(0, 70000000):
		math.sqrt(i)

processes = []

for i in range(os.cpu_count()):
	print('registering process %d' % i)
	processes.append(Process(target=calc))

for process in processes:
	process.start()

for process in processes:
	process.join()
```
