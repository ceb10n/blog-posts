# Understanding Uvicorn: The basics

* [Linkedin](https://www.linkedin.com/pulse/understanding-uvicorn-basics-rafael-de-oliveira-marques-oycvf)
* [dev.to/ceb10n](https://dev.to/ceb10n/understanding-uvicorn-4gi8)

Last year I wrote a series of articles explaining how [FastAPI](https://github.com/fastapi/fastapi) works. It was a rewarding experience, since I was able to both share my knowledge and learn more about it.

So I've decided to create another series of articles, and this time talking about another piece of FastAPI development: ASGI servers, and mainly focusing on 🦄 [uvicorn](https://github.com/encode/uvicorn) due to the evident synergy between both projects 😎.

## 🤔 So, what is uvicorn?

![uvicorn-logo](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xxnbw0i7ew5h3aka4wyc.png)

As the project says in a very direct and clear way:

> Uvicorn is an ASGI web server implementation for Python.

So, following the same approach as the FastAPI articles, we'll try to understand some concepts and libs before moving on.

As the creators of the 🦄 uvicorn says, it is an ASGI *web server* implementation.

So, let's take a small look at what is a web server.

## 💻 What is a web server?

<figure>
<a title="Ade56facc, CC BY-SA 4.0 &lt;https://creativecommons.org/licenses/by-sa/4.0&gt;, via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File:Web_server_serving_static_content.png"><img width="256" alt="Web / HTTP server serving static content only (from file system)" src="https://upload.wikimedia.org/wikipedia/commons/1/1e/Web_server_serving_static_content.png?20211117135837"></a>
  <figcaption>Ade56facc, CC BY-SA 4.0 <https://creativecommons.org/licenses/by-sa/4.0>, via Wikimedia Commons</figcaption>
</figure>

From the name, we can try to infer that this is a server used on the web, right? But let's look at some definitions to understand it better.

At [Mozilla Developer Network](https://developer.mozilla.org/en-US/), we can find the definition in the [What is a web server?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_web_server) document:

> The term web server can refer to hardware or software, or both of them working together...
> ...On the software side, a web server includes several parts that control how web users access hosted files. At a minimum, this is an HTTP server. An HTTP server is software that understands URLs (web addresses) and HTTP (the protocol your browser uses to view webpages). An HTTP server can be accessed through the domain names of the websites it stores, and it delivers the content of these hosted websites to the end user's device.

So, going back to 🦄 `uvicorn`, as it is an ASGI web server, we can begin to imagine that maybe it is a server that handles HTTP and implements the [ASGI](https://asgi.readthedocs.io/en/latest/) specification, right?

## 📜 And what is HTTP?

[HTTP](https://en.wikipedia.org/wiki/HTTP) stands for HyperText Transfer Protocol, and as the name says, it's a Protocol to transfer HyperText. Wow, mind blowing, right? 🤯

Since a Protocol is a set of rules for comunication, we can now understand that HTTP is a set of rules of communication for transfering HyperText. And since we can say in a summarized way that HyperText is a text that references hyperlinks, we can now understand that:

Uvicorn is a software that enables two or more computers to communicate and transfer text throughout a set a rules defined in the [HTTP](https://datatracker.ietf.org/doc/html/rfc2068) Protocol.

⚠️ I know, I know. Uvicorn supports WebSockets too. But let's stick with the basics for now. 😊

## 💾 Creating your first socket

We just saw that 🦄 `uvicorn` is a software that enables communication between computers using some standards, etc.

The way it enables the communication is throughout sockets, since uvicorn understands HTTP, and HTTP uses TCP/IP, which commonly uses sockets for the communication.

Let's start creating our first socket that will simply echo the received message:

```python
import socket

print("Creating server")
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server:
    server.bind((socket.gethostname(), 12345))
    print(f"Listening at {socket.gethostname()}:12345")

    server.listen()

    print("Waiting for connection")
    connection, adress = server.accept()

    while True:
        data = connection.recv(1024)
        print(f"Received: {data.decode()}")
        connection.sendall(data)
```

What we've just created is a server that will keep echoing forever the messages that were received:

```shell
Creating server
Listening at ceb10n:12345
Waiting for connection
Received: My first socket connection :D
```

Now, we need to create a client socket to connect to our server and see the connection in action:

```python
import socket

print("Creating client")
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    print(f"Connecting to {socket.gethostname()}:12345")
    s.connect((socket.gethostname(), 12345))
    while True:
        s.sendall(b"My first socket connection :D")
        data = s.recv(1024)
        print(f"Received {data!r}")
```

And the result will be:

```shell
Creating client
Connecting to ceb10n:12345
Received b'My first socket connection :D'
```

But what we just did right here?

We created a socket specifying the parameters [socket.AF_INET](https://docs.python.org/3/library/socket.html#socket.AF_INET) and [socket.SOCK_STREAM](https://docs.python.org/3/library/socket.html#socket.SOCK_STREAM).

* [socket.AF_INET](https://docs.python.org/3/library/socket.html#socket.AF_INET): It's the address family of our socket. In this case, our socket will use the [IPv4](https://en.wikipedia.org/wiki/IPv4).
* [socket.SOCK_STREAM](https://docs.python.org/3/library/socket.html#socket.SOCK_STREAM): It's the socket type. This way, our socket will use the TCP protocol (the first step for creating our HTTP server)

The next part of our server is binding the our socket with a host and port.

Then we need to start accepting a connection. We do this by enabling the socket to accept connections with [listen](https://docs.python.org/3/library/socket.html#socket.socket.listen). After our socket is enabled, we can use the [accept](https://docs.python.org/3/library/socket.html#socket.socket.accept) method to  wait for a connection.

After that, we start a loop to keep receiving chunks of data forever. The `recv` method receives the `1024` parameter, specifying that we want to receive blocks of 1024 bytes of data. You can test it by changing 1024 to only one or two. If you do that, you'll start seeing the following:

```shell
Creating server
Listening at ceb10n:12345
Waiting for connection
Received: My
```

Notice that if the message is bigger than what we've specified there, we'll lose all the remaining bytes of the message.

And probably that's the most basic socket server you can create. On the client side, it's almost identical, except we'll be sending data with `sendall`, that receives the message we want to send as a parameter.

Now we start figuring out the amazing work that the uvicorn team (and the teams from the other dependencies) have done! Just imagine starting from this simple socket server and transforming it in the amazing server that is uvicorn! 🤯

I would suggest you all to consider 💸 sponsoring the project and the main maintainer, 🇧🇷 [Marcelo Trylesinski](https://github.com/Kludex)! Just visit it's 🦄 [official repository](https://github.com/encode/uvicorn) and start supporting them!

Well, I think that's all for this article. Stay tuned for the next ones 😎.