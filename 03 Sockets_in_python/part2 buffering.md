# Part2
## Buffering

<br>

Welcome to part 2 of the sockets tutorial with Python. In the previous tutorial, we learned how we could send and receive data using sockets, but then we illustrated the problem that can arise when our communication exceeds our buffer size. In this tutorial, we'll talk about overcoming this!

As mentioned before, there are a few logical ways that you could handle for this, but one common way is by starting all messages with a header that contains the length of the message that is going to come. The next challenge is normalizing this header in some way. You might consider using some series of characters, or some format, but then you run the risk of people accidentally, or purposefully, mimicking this formatting. Instead, you can go with a fixed-length header, where the first n bytes of data will be the header data, which will include the length of the message to come. Once we've received that length of data, we know any following information will be a new message, where we need to grab the header and continue repeating this process.

So what we need to do now is choose some truly maximal message size. Say, 1,000,000,000 bytes. Right, there's almost no circumstance where someone would attempt anything even close to this via our chat app, so this will be fine. That number is 10 bytes (10 chars). In python, how might we represent any number as 10 characters? We can use string formatting! Yay basics! Since this is a lesser-used functionality, see more here: [format examples](https://docs.python.org/3/library/string.html#format-examples), which you will see examples like:

```python
#Aligning the text and specifying a width:

>>>
>>> '{:<30}'.format('left aligned')
'left aligned                  '
>>> '{:>30}'.format('right aligned')
'                 right aligned'
>>> '{:^30}'.format('centered')
'           centered           '
>>> '{:*^30}'.format('centered')  # use '*' as a fill char
'***********centered***********'
```

In this case, you can see various examples where there are 30 characters used every time, but you can do various alignments. While this is mainly used to make text-based GUIs pretty, we can also use this for our purposes, like:

```python
f'{len("your message here!"):<10}'
```

In the above case, this will produce the length of our message using 10 characters.

```python
>>> f'{len("your message here!"):<10}'
'18  
```

All we do now is just pre-pend all of our messages with this, then we can convert the first 10 chars of new messages to an int to know how much more is a part of a unique message. To do this, we'll start in our server script:

server.py

```python
import socket

HEADERSIZE = 10

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind((socket.gethostname(), 1241))
s.listen(5)

while True:
    # now our endpoint knows about the OTHER endpoint.
    clientsocket, address = s.accept()
    print(f"Connection from {address} has been established.")

    msg = "Welcome to the server!"
    msg = f"{len(msg):<{HEADERSIZE}}"+msg

    clientsocket.send(bytes(msg,"utf-8"))
```

So now our messages will have a header of 10 characters/bytes that will contain the length of the message, which our client use to inform it when the end of the message is received. Let's work on the client.py next:

client.py

```python
import socket

HEADERSIZE = 10

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((socket.gethostname(), 1241))

while True:
    full_msg = ''
    new_msg = True
    while True:
        msg = s.recv(16)
        if new_msg:
            print("new msg len:",msg[:HEADERSIZE])
            msglen = int(msg[:HEADERSIZE])
            new_msg = False

        print(f"full message length: {msglen}")

        full_msg += msg.decode("utf-8")

        print(len(full_msg))


        if len(full_msg)-HEADERSIZE == msglen:
            print("full msg recvd")
            print(full_msg[HEADERSIZE:])
            new_msg = True
```

This one is a bit more involved, but nothing too crazy here. I increased out buffer to 16 bytes. 8 wouldnt even be enough to read the header, so that would have been a problem, and you would probably never have a buffer as small as these anyway. We're just doing it for example. So, we start off in a state where the next bit of data we get is a new_msg.

If the message is a new_msg, then the first thing we do is parse the header, which we already know is a fixed-length of 10 characters. From here, we parse the message length. Then, we continue to build the full_msg, until that var is the size of msglen + our HEADERSIZE. Once this happens, we print out the full message.

Going from this to some sort of streaming API is quite simple. Let's do an example where the server just streams out something simple, like the current time.

To do this, we just add the following to the end:

```python
    while True:
        time.sleep(3)
        msg = f"The time is {time.time()}"
        msg = f"{len(msg):<{HEADERSIZE}}"+msg

        print(msg)

        clientsocket.send(bytes(msg,"utf-8"))
```
Our full server.py is now:

```python
import socket
import time


HEADERSIZE = 10

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind((socket.gethostname(), 1243))
s.listen(5)

while True:
    # now our endpoint knows about the OTHER endpoint.
    clientsocket, address = s.accept()
    print(f"Connection from {address} has been established.")

    msg = "Welcome to the server!"
    msg = f"{len(msg):<{HEADERSIZE}}"+msg

    clientsocket.send(bytes(msg,"utf-8"))

    while True:
        time.sleep(3)
        msg = f"The time is {time.time()}"
        msg = f"{len(msg):<{HEADERSIZE}}"+msg

        print(msg)

        clientsocket.send(bytes(msg,"utf-8"))
```

Now, nothing changes for our client, except for the preparation that we will accept new messages after the first, so we need to reset the full_msg var, so our full client.py becomes:

```python
import socket

HEADERSIZE = 10

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((socket.gethostname(), 1243))

while True:
    full_msg = ''
    new_msg = True
    while True:
        msg = s.recv(16)
        if new_msg:
            print("new msg len:",msg[:HEADERSIZE])
            msglen = int(msg[:HEADERSIZE])
            new_msg = False

        print(f"full message length: {msglen}")

        full_msg += msg.decode("utf-8")

        print(len(full_msg))


        if len(full_msg)-HEADERSIZE == msglen:
            print("full msg recvd")
            print(full_msg[HEADERSIZE:])
            new_msg = True
            full_msg = ""
```

Now, run these two things, and you should see the server outputting something like:

```cmd
28        The time is 1552068299.01783
30        The time is 1552068302.0181189
30        The time is 1552068305.0206459
29        The time is 1552068308.021842
29        The time is 1552068311.024837
29        The time is 1552068314.025016
28        The time is 1552068317.02619
29        The time is 1552068320.026504
29        The time is 1552068323.031633
30        The time is 1552068326.0359411
29        The time is 1552068329.039903
29        The time is 1552068332.040124
30        The time is 1552068335.0402749
27        The time is 1552068338.0437
29        The time is 1552068341.043971
```

And the client:

```cmd
new msg len: b'27        '
full message length: 27
16
full message length: 27
32
full message length: 27
37
full msg recvd
The time is 1552068338.0437
new msg len: b'29        '
full message length: 29
16
full message length: 29
32
full message length: 29
39
full msg recvd
The time is 1552068341.043971

```


<br>

# The end
