# Part 3
## Sending and receiving objects in Python

<br>



Sockets Tutorial with Python 3 part 3 - sending and receiving Python Objects with sockets



Welcome to part 3 of the sockets tutorial with Python. We've learned how to send and receive string data via sockets, and now I want to talk about is pickles. Not the food, but the serialization technique in Python.

In Python, everything is an object, and all of your objects can be serialized with Pickle. Serialization is the conversion of your object to bytes.

...and we send bytes with sockets. This means that you can communicate between your python programs both locally, or remotely, via sockets, using pickle. So now, literally anything...functions, a giant dictionary, some arrays, a TensorFlow model...etc can be sent back and forth between your programs! Let's see a quick example of that before I close out this tutorial.

So first, quickly, just in case you don't know about pickles, let's convert a pickle to a byte string:


```python
>>> import pickle
>>> d = {1:"hi", 2: "there"}
>>> msg = pickle.dumps(d)
>>> msg
b'\x80\x03}q\x00(K\x01X\x02\x00\x00\x00hiq\x01K\x02X\x05\x00\x00\x00thereq\x02u.'
```
Now, that's our msg, we just send it. When we get it, we can read it with loads.

```python
>>> recd = pickle.loads(msg)
>>> recd
{1: 'hi', 2: 'there'}
```

Okay, let's put it together and send that. Our server.py will have the following code for the message to send:


```python
    d = {1:"hi", 2: "there"}
    msg = pickle.dumps(d)
    msg = bytes(f"{len(msg):<{HEADERSIZE}}", 'utf-8')+msg
    print(msg)
    clientsocket.send(msg)
```

Making the full script (with a lot removed from before since we're just trying to illustrate sending objects):
server.py

```python
import socket
import time
import pickle


HEADERSIZE = 10

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind((socket.gethostname(), 1243))
s.listen(5)

while True:
    # now our endpoint knows about the OTHER endpoint.
    clientsocket, address = s.accept()
    print(f"Connection from {address} has been established.")

    d = {1:"hi", 2: "there"}
    msg = pickle.dumps(d)
    msg = bytes(f"{len(msg):<{HEADERSIZE}}", 'utf-8')+msg
    print(msg)
    clientsocket.send(msg)
```

In the above case as well, we're sending a header that counts in bytes, rather than characters, which makes more sense in either case, really. Now, our client needs to handle this. In line with this change, we change our starting message to (both initally, then after the message is complete):

```python
   full_msg = b''
```

Then, we can remove the decoding bits, so we just build our message as:

```python
       full_msg += msg
```

Then, we wait for the full message, then convert from bytes to object with pickel.loads():

```python
       if len(full_msg)-HEADERSIZE == msglen:
            print("full msg recvd")
            print(full_msg[HEADERSIZE:])
            print(pickle.loads(full_msg[HEADERSIZE:]))

            new_msg = True
            full_msg = b""
```

## Full client.py

```python
import socket
import pickle

HEADERSIZE = 10

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((socket.gethostname(), 1243))

while True:
    full_msg = b''
    new_msg = True
    while True:
        msg = s.recv(16)
        if new_msg:
            print("new msg len:",msg[:HEADERSIZE])
            msglen = int(msg[:HEADERSIZE])
            new_msg = False

        print(f"full message length: {msglen}")

        full_msg += msg

        print(len(full_msg))

        if len(full_msg)-HEADERSIZE == msglen:
            print("full msg recvd")
            print(full_msg[HEADERSIZE:])
            print(pickle.loads(full_msg[HEADERSIZE:]))
            new_msg = True
            full_msg = b""
```

Running these, I get:

```python
 python3 server.py
Connection from ('192.168.86.25', 50373) has been established.
b'33        \x80\x03}q\x00(K\x01X\x02\x00\x00\x00hiq\x01K\x02X\x05\x00\x00\x00thereq\x02u.'
```

And on the client:

```python
$ python3 client.py
new msg len: b'33        '
full message length: 33
16
full message length: 33
32
full message length: 33
43
full msg recvd
b'\x80\x03}q\x00(K\x01X\x02\x00\x00\x00hiq\x01K\x02X\x05\x00\x00\x00thereq\x02u.'
{1: 'hi', 2: 'there'}

```

<br>

# The end