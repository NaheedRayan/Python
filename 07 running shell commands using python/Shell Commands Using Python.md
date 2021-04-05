#  Executing Shell commands using python



## 1. Older Methods


- os.system
    - Execute the command (a string) in a subshell.
    - On unix the return value is the exit status of the process.
    - On windows the return value is what returned by the system shell after running command.

- os.open
    - Open a pipe to or from command being executed.
    - The return value is an open file object connected to the pipe , which can be read or written depending on the mode 'r'(default) or 'w'.
    - The ``close`` method returns None if the subprocess exited successfully,or the subprocess's return code if there was an error.

- os.spawn*
    - Execute the program path in a new process.
    - If mode is ``P_NOWAIT`` , this function returns the process id of the new process.
    - If mode is ``p_WAIT`` , returns the process's exit code if it exits normally,or -signal,where signal is the signal that killed the process.



Example:
```python
import os
os.system('ls')
```

<br>

```python
import os
print(os.popen('ls').read())
```
<br>

```python
import os
os.spawn(os.P_NOWAIT , "/usr/sh",'ls')
```


## 1. Subprocess Module

Main API

- run(...)
    - Runs a command, waits for it to compelete, then returns a ``CompletedProcess`` instance.(added in python 3.5)

- Popen(...)
    - A class for flexibly executing a command in a new process

<br>

## Exploring subprocess

<br>

## 1. run command

    import subprocess
    print(subprocess.run('ls'))

Output:

    CompletedProcess(args='ls', returncode=0)

## 2. running a python file
```python
Input:

print(subprocess.run(["python3" , "test.py"]))

Output:

Hello world
CompletedProcess(args=['python3', 'test.py'], returncode=0)
```

<br>

```python

Input:

print(subprocess.run(["python3" , "test.py"] , stdout = subprocess.PIPE , stderr = subprocess.PIPE))

Output:

CompletedProcess(args=['python3', 'test.py'], returncode=0, stdout=b'Hello world\n', stderr=b'')
```

## 3. Python 3.7 and later capture_output function is given

- If ``capture_output`` is true , stdout and stderr will be automatically captured.
- ``stdout= subprocess.PIPE`` and ``stderr= subprocess.PIPE`` is automatically set.



```python

Input:

print(subprocess.run(["python3" , "test.py"] , capture_output = True))

Output:

CompletedProcess(args=['python3', 'test.py'], returncode=0, stdout=b'Hello world\n', stderr=b'')

```

<br>

So its more convinient.

## 4. run command by passing a single string

For this ``shell``= True should be set.


```python

Input:

print(subprocess.run("python3 test.py" ,capture_output=True ,shell=True ))

Output:

CompletedProcess(args='python3 test.py', returncode=0, stdout=b'Hello world\n', stderr=b'')

```


## Security Considerations before using shell=True.

    Unlike some other popen functions, this implementation will never implicitly call a system shell. This means that all characters, including shell metacharacters, can safely be passed to child processes. If the shell is invoked explicitly, via shell=True, it is the application’s responsibility to ensure that all whitespace and metacharacters are quoted appropriately to avoid shell injection vulnerabilities.

When using shell=True, the shlex.quote() function can be used to properly escape whitespace and shell metacharacters in strings that are going to be used to construct shell commands.


```python
Input:

user_input = "a.txt ; pwd"
command = "cat {}".format(user_input)
subprocess.run(command, shell=True, capture_output=True)

Output:

CompletedProcess(args='cat a.txt ; pwd', returncode=0, stdout=b'/home/nikhil/Videos/shell\n', stderr=b'cat: a.txt: No such file or directory\n')
```

## 4. run command and pass input

    input param
    stdin param


In [25]:

    subprocess.run(["python3", "test.py"], capture_output=True, input="abc\ndef".encode())

Out[25]:

    CompletedProcess(args=['python3', 'test.py'], returncode=2, stdout=b'', stderr=b"python3: can't open file 'test.py': [Errno 2] No such file or directory\n")

In [26]:

    subprocess.run(["python3", "test.py"], capture_output=True, input="abc\ndef", 
               universal_newlines=True)

Out[26]:

    CompletedProcess(args=['python3', 'test.py'], returncode=2, stdout='', stderr="python3: can't open file 'test.py': [Errno 2] No such file or directory\n")

In [27]:

    subprocess.run(["python3", "test.py"], capture_output=True, input="abc\ndef", text=True)

Out[27]:

    CompletedProcess(args=['python3', 'test.py'], returncode=2, stdout='', stderr="python3: can't open file 'test.py': [Errno 2] No such file or directory\n")

In [28]:

    !cat a.txt

    cat: a.txt: No such file or directory

In [29]:

    subprocess.run(["python3", "test.py"], capture_output=True, stdin=open("a.txt", 'r'))

---------------------------------------------------------------------------
    FileNotFoundError                         Traceback (most recent call last)
    <ipython-input-29-214507e646cc> in <module>
     
    
## 5. run command with timeout

Set timeout parameter.
In [30]:

    subprocess.run(["sleep", "5"], timeout=3)

```
---------------------------------------------------------------------------
TimeoutExpired                            Traceback (most recent call last)
<ipython-input-30-0309493381cb> in <module>
----> 1 subprocess.run(["sleep", "5"], timeout=3)

/usr/lib/python3.7/subprocess.py in run(input, capture_output, timeout, check, *popenargs, **kwargs)
    488     with Popen(*popenargs, **kwargs) as process:
    489         try:
--> 490             stdout, stderr = process.communicate(input, timeout=timeout)
    491         except TimeoutExpired as exc:
    492             process.kill()

/usr/lib/python3.7/subprocess.py in communicate(self, input, timeout)
    962 
    963             try:
--> 964                 stdout, stderr = self._communicate(input, endtime, timeout)
    965             except KeyboardInterrupt:
    966                 # https://bugs.python.org/issue25942

/usr/lib/python3.7/subprocess.py in _communicate(self, input, endtime, orig_timeout)
   1739                             self._fileobj2output[key.fileobj].append(data)
   1740 
-> 1741             self.wait(timeout=self._remaining_time(endtime))
   1742 
   1743             # All data exchanged.  Translate lists into strings.

/usr/lib/python3.7/subprocess.py in wait(self, timeout)
   1017             endtime = _time() + timeout
   1018         try:
-> 1019             return self._wait(timeout=timeout)
   1020         except KeyboardInterrupt:
   1021             # https://bugs.python.org/issue25942

/usr/lib/python3.7/subprocess.py in _wait(self, timeout)
   1643                     remaining = self._remaining_time(endtime)
   1644                     if remaining <= 0:
-> 1645                         raise TimeoutExpired(self.args, timeout)
   1646                     delay = min(delay * 2, remaining, .05)
   1647                     time.sleep(delay)

TimeoutExpired: Command '['sleep', '5']' timed out after 2.9998896930001138 seconds
```

In [31]:

    subprocess.run(["sleep", "3"], timeout=5)

Out[31]:

    CompletedProcess(args=['sleep', '3'], returncode=0)

## 6. run command and throw error if fail
In [32]:

    subprocess.run(["rm", "xyz"])

Out[32]:

    CompletedProcess(args=['rm', 'xyz'], returncode=1)

In [33]:

    try:
        subprocess.run(["rm", "xyz"], check=True)
    except subprocess.CalledProcessError:
        print("failed")

    failed



## 6. My code for running scrits with input and timelimit

```python
import subprocess


try:
    ans = subprocess.run(["python3" , "test.py"] , capture_output=True , input = "45\n55".encode() , timeout=2 )
    print(ans.stdout.decode())
except :
    print('failed')
```



