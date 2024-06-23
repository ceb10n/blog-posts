# Understanding FastAPI: The Basics

* [Linkedin](https://www.linkedin.com/pulse/understanding-fastapi-basics-rafael-de-oliveira-marques-xfvaf)
* [dev.to/ceb10n](https://dev.to/ceb10n/understanding-fastapi-the-basics-246j)
* [github.com/ceb10n](https://github.com/ceb10n/blog-posts/tree/master/understanding-fastapi-the-basics)
* [ceb10n.medium.com](https://ceb10n.medium.com/understanding-fastapi-the-basics-14221665f742)

I've been working with [FastAPI](https://fastapi.tiangolo.com/) for a while now, and I decided to start digging a little deeper on it's internals.

Let's start from the beggining:

## What is FastAPI?

As the docs says:

> FastAPI is a modern, fast (high-performance), web framework for building APIs with Python based on standard Python type hints.

So here we have a good idea about what is FastAPI: A web framework for building APIs.

But it's not a framework built from scratch, it's a framework that was built on top of another framework: [Starlette](https://www.starlette.io/).

## And what is Starlette?

![Starlette's logo](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/r9ciuazaaxe2mi7exndx.png)

If we go to starlette's website, we'll find that:

> Starlette is a lightweight ASGI framework/toolkit, which is ideal for building async web services in Python.

## And what is ASGI?

ASGI, or Asynchronous Server Gateway Interface is a specification that proposes an interface between web servers and python applications.

When we are running our fastapi application, we're using an ASGI server that will forward the request to our app.

Some of the most well-known asgi servers are:

- [Uvicorn](https://www.uvicorn.org/)
- [Hypercorn](https://hypercorn.readthedocs.io/en/latest/index.html)
- [Daphne](https://github.com/django/daphne)

## Let's recap

FastAPI is a modern webframework written in python that is built on top Starlette, which in turn is a lightweight ASGI framework that needs an ASGI server to run.

## Let's create a basic ASGI application

So if we want to understand how FastAPI works, we must start from the beginning: a simple asgi application:

```python
async def app(scope, receive, send):
    await send({
        "type": "http.response.start",
        "status": 200,
        "headers": [
            [b"content-type", b"text/plain"],
        ],
    })
    await send({
        "type": "http.response.body",
        "body": b"Hello, World!",
    })
```

Can you imagine that all features that FastAPI offers you, all middlewares and error handling, OpenAPI docs, etc., starts from this?

That's how ASGI is specified: a single asynchronous callable that takes a dict and two asynchronous callables as parameters.

And it works, without any webframework installed! Now let's check if it's really working:

First, install an ASGI server:

```shell
pip install uvicorn
```

And create a python file called app with the code above.

Let's run it and see if it's working:

```shell
uvicorn app:app
```

Now enter `http://localhost:8000` in your browser:

```shell
INFO:     Started server process [4808]
INFO:     Waiting for application startup.
INFO:     ASGI 'lifespan' protocol appears unsupported.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     127.0.0.1:64045 - "GET / HTTP/1.1" 200 OK
```

Let's change to hypercorn to see if we are not tied to uvicorn somehow:

```shell
pip install hypercorn
```

And we can see it still works:

```shell
hypercorn app:app
[2024-06-22 19:00:55 -0300] [13008] [WARNING] ASGI Framework Lifespan error, continuing without Lifespan support
[2024-06-22 19:00:55 -0300] [13008] [INFO] Running on http://127.0.0.1:8000 (CTRL + C to quit)
```

![Browse's output](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2ohyc133xvqxlh2ivaf6.jpg)

## Creating the simplest FastAPI clone ever

Now that we know what's beneath FastAPI, we can start playing a little bit and create the smallest, simpler ASGI framework ever:

```
class SimplestFrameworkEver:
    async def __call__(self, scope, receive, send):
        await send({
        "type": "http.response.start",
        "status": 200,
        "headers": [
            [b"content-type", b"text/plain"],
        ],

        })
        await send({
            "type": "http.response.body",
            "body": b"Hello, World!",
        })


app = SimplestFrameworkEver()
```

And you can still run it like before, with:

```shell
hypercorn app:app
```

or

```shell
uvicorn app:app
```

## Next steps

Now that we can understand the basics on which FastAPI is made, we can move on to see:

* how Starllete works
* how FastAPI extends Starllete

Stay tuned for the next posts ;)
