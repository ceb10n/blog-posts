# Understanding FastAPI: How Starlette works

* [Linkedin](https://www.linkedin.com/pulse/understanding-fastapi-how-starlette-works-rafael-de-oliveira-marques-juflf)
* [dev.to/ceb10n](https://dev.to/ceb10n/understanding-fastapi-how-starlette-works-43i1)
* [github.com/ceb10n](https://github.com/ceb10n/blog-posts/tree/master/understanding-fastapi-how-starlette-works)
* [ceb10n.medium.com](https://ceb10n.medium.com/understanding-fastapi-how-starlette-works-d518a3d22222)

We've seen in the [last post](https://dev.to/ceb10n/understanding-fastapi-the-basics-246j) that FastAPI is built on top of Starllete. We also saw that [Starlette](https://www.starlette.io/) is a lightweight ASGI framework/toolkit.

Now, to start undertanding how FastAPI works, let's see what Starlette has to offer, how he deals with our HTTP requests, etc.

⚠️ _Note: To work with starlette, you'll need at least python 3.8, so check which version you have installed on your computer before continuing_

## Hello world Starlette

Let's recreate that simple hello world from the previous post using Starllete:

```python
from starlette.applications import Starlette
from starlette.responses import PlainTextResponse
from starlette.routing import Route


async def hello(request):
    return PlainTextResponse("Hello, World!")


app = Starlette(routes=[
    Route('/', hello),
])

```

Can you see the similarities with the example in the previous post? Let's remember it:

```python
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

In both cases we have a class (`SimplestFrameworkEver` or `Starlette`), we create an instance of this class and pass it to the ASGI server to deal with it.

And according with [ASGI's specification](https://asgi.readthedocs.io/en/latest/index.html), an ASGI must expose a a single, `asynchronous callable` who receives a dictionary named scope and two other `async callables` named `receive` and `send` as parameters.

So if we are passing a Starlette object to the ASGI server, **it MUST be** a class that has a `__call__` method implemented. Let's open Starlette's source code to check if it's true.

If we go to its [official repository](https://github.com/encode/starlette/), we can find Starllete class inside [starlette/applications.py](https://github.com/encode/starlette/blob/master/starlette/applications.py):

```python
async def __call__(self, scope: Scope, receive: Receive, send: Send) -> None:
    scope["app"] = self
    if self.middleware_stack is None:
        self.middleware_stack = self.build_middleware_stack()
    await self.middleware_stack(scope, receive, send)
```

So we can see here that Starlette is a _"callable"_ class, and apparently has a list of middlewares that will be executed each request. Simple as that, right? Yes and no 🤣

Things are never as simple as they seem, and we'll need to see what is a `middleware_stack`

## ASGIApp

What we've seen this far is: `Starlette` is a callable that receives a scope, a receive and a send parameters. Which is exactly what and ASGI application is supposed to be. And what is a `middleware_stack`?

```python
self.middleware_stack: ASGIApp | None = None
```

`middleware_stack` is an `ASGIApp`. But if you look in Starlette's types, ASGIApp is just an ASGI callable:

```python
ASGIApp = typing.Callable[[Scope, Receive, Send], typing.Awaitable[None]]
```

If we take a look at `Starlette.build_middleware_stack`, we'll se a strange piece of code:

```python
middleware = (
    [Middleware(ServerErrorMiddleware, handler=error_handler, debug=debug)]
    + self.user_middleware
    + [
        Middleware(
            ExceptionMiddleware, handlers=exception_handlers, debug=debug
        )
    ]
)

app = self.router
for cls, args, kwargs in reversed(middleware):
    app = cls(app=app, *args, **kwargs)
return app
```

What is happening here is that starlette is creating sort of a [chain of responsability](https://refactoring.guru/design-patterns/chain-of-responsibility) of middlewares, and lastly our router / endpoint. Things will work like:

```
-> ServerErrorMiddleware
    -> Other Middlewares
        -> ExceptionMiddleware
            -> Router
```

Each ASGIApp will receive another ASGIApp as a dependency, and each one will call the next app when it gets called, till we reach `ExceptionMiddleware`, that will wrap our `Router` to deal with our exceptions.

The `Router` then will not receive any ASGIApp as a dependency. It will implement it's own app function (ASGI app) to match a route and execute our path operation function:

```python
async def app(self, scope: Scope, receive: Receive, send: Send) -> None:
    # ... previous code

    for route in self.routes:
        # Determine if any route matches the incoming scope,
        # and hand over to the matching route if found.
        match, child_scope = route.matches(scope)
        if match == Match.FULL:
            scope.update(child_scope)
            await route.handle(scope, receive, send)
            return
     
    # ... code continues
```

Now that we know the flow of a Starlette's request, we can create a simple middleware to log the resquest's path, and another one that logs that everything went ok after the response is sent:

```python
class LogRequestMiddleware:
    def __init__(self, app: ASGIApp):
        self.app = app

    async def __call__(self, scope: Scope, receive: Receive, send: Send):
        logging.info(f"-> received a request @ {scope['path']}")
        await self.app(scope, receive, send)


class LogResponseMiddleware:
    def __init__(self, app: ASGIApp) -> None:
        self.app = app

    async def __call__(self, scope: Scope, receive: Receive, send: Send):
        await self.app(scope, receive, send)
        logging.info("-> wow, we did it")


async def hello(request):
    logging.info("Great news, we got a request!")
    return PlainTextResponse("Hello, World!")


app = Starlette(
    routes=[
        Route('/', hello),
    ],
    middleware=[
        Middleware(LogRequestMiddleware),
        Middleware(LogResponseMiddleware)
    ]
)
```

And we'll get:

```shell
INFO:root:-> received a request @ /
INFO:root:Great news, we got a request!
INFO:     127.0.0.1:51770 - "GET / HTTP/1.1" 200 OK
INFO:root:-> wow, we did it
```

To learn more about Starlette's middlewares, you can read it's own documentation: [Middleware](https://www.starlette.io/middleware/). The people from the project made a great work documenting it.

## Routes and Router

And last but not least, after all the chain of middlewares, we'll get our Router beeing executed wrapped in an `ExceptionMiddleware`, so it can deal with our exceptions. `Router` will have a list of routes to deal with.

If the router finds a matching route, it will call the route's handle function. The handle function will call our endpoint, that is basically the function or class that you passed while creating the Starlette app:

```python
app = Starlette(
    routes=[
        Route('/', hello), # -> hello is the function that will be handled by Router's handle
    ],
```

Infact, a `Router` is an `ASGIApp` too, and you can dismiss all Starlette's middlewares by creating only a Router:

```python
app = Router(routes=[
    Route('/', hello)
])
```

## And what about FastAPI?

But Rafael, the title of your posts are saying Understanding FastAPI, but it's the second post and you keep writing stuff about ASGI specs, Starlette, etc.

But remember what I said: **FastAPI is Starlette**. It's a framework built on top of Starllete, and if you go to FastAPI's source code, you'll find this:

```python
class FastAPI(Starlette):

    def __init__(
    # ... code continues
```

All this time you were reading about `FastAPI`, but I was writing about the internals of FastAPI, the foundations that support FastAPI.

In the next post we'll look how FastAPI extends Starlette and ASGI specs to offer us `Pydantic` models, OpenAPI specs out of the box, etc.

Stay tuned ;)
