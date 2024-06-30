# Understanding FastAPI: How FastAPI works

At this point we've seen[ how ASGI servers and our applications talk to each other](https://dev.to/ceb10n/understanding-fastapi-the-basics-246j) and how [Starllete, the foundation of FastAPI works](https://dev.to/ceb10n/understanding-fastapi-how-starlette-works-43i1).

Now it's time to take a closer look on how [FastAPI](https://fastapi.tiangolo.com/) extends [Starllete](https://www.starlette.io/).

## FastAPI, a Starllete app

First of all, to understand how FastAPI works, the are two main sources of information:

* [FastAPI's source code](https://github.com/tiangolo/fastapi)
* [FastAPI's documentation](https://fastapi.tiangolo.com/)

So make sure you clone Sebastián's repository and start looking at it.

FastAPI's first entrypoint is `FastAPI` class, that lives under fastapi/applications.py.

Since we are studying an ASGI framework, we can expect that FastAPI is a callable that receives `scope`, `receive` and `send`, like any other ASGI app:

```python
async def __call__(self, scope: Scope, receive: Receive, send: Send) -> None:
    if self.root_path:
        scope["root_path"] = self.root_path
    await super().__call__(scope, receive, send)
```

We can see that not only FastAPI has a `__call__` function as we expected, but it delegates to Starlette the request.

## Difference between FastAPI and Starlette when initializing

If FastAPI extends Starlette, it will likely add some functionality during initialization.

When we look at [FastAPI](https://fastapi.tiangolo.com/reference/fastapi/#fastapi.FastAPI)'s `__init__` function, we can see two main things:

* It add routes to OpenAPI docs on `setup` function
* It sets the `Router` to `APIRouter`

`setup` function will add one of the coolest features of FastAPI: It will add a free OpenAPI documentation to our project with [Swagger](https://swagger.io/) and [Redoc](https://redocly.com/redoc).

The `APIRouter` will be where all your path operations live. Either you add your route directly with your FastAPI app ou creating an `APIRouter`, all routes will be included in FastAPI's router.

In this post we'll take a better look at FastAPI's routers and routes. We'll leave OpenAPI to a next post.

## Request lifecycle


![HTTP Request](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/z83vfznqlsqs9jg0m3jn.png)

Since FastAPI is a Starlette app with extra features, we can assume that a request lifecycle using FastAPI will be almost equal to Starlette's request lifecycle.

On the [previous post](https://dev.to/ceb10n/understanding-fastapi-how-starlette-works-43i1), we talked about how a request will be handled. The chain of middlewares will be something like:

```
-> ServerErrorMiddleware
    -> Other Middlewares
        -> ExceptionMiddleware
            -> Router
```

When working with FastAPI we can see the it is overriding Starlette's `Router` with it's own `APIRouter`.

That said, when can see that FastAPI is still relying on Starlette's lifecycle, but it prefers to handle the requests its own way.

So with FastAPI, you'll have:

```
-> FastAPI App
  -> Starlette's App
    -> Starlette's ServerErrorMiddleware
      -> Starlette's ExceptionMiddleware
        -> FastAPI's APIRouter (and Router, since it don't override Router's __call__)
```

## FastAPI routers and routes

When we are creating a FastAPI app, there are two main ways to add a route:

Adding a route directly with FastAPI's instance:

```python
app = FastAPI()

@app.get("/{name}")
async def hi(name: str):
    return {"hi": name}
```

Or using `APIRouter` that is used typically in larger apps:

```python
app = FastAPI()
router = APIRouter(prefix="/v1")

@router.get("/compliments/{name}")
async def hi1(name: str):
    return {"hi": name}

app.include_router(router)
```

Since we are trying to understand how FastAPI works, lets see what is happening when we use `@app.{verb}`:

```python
    def get(
        self,
        path: Annotated[
            str,
            Doc("... # docs here"),
        ],
        *,
        ... # other args here
    ) -> Callable[[DecoratedCallable], DecoratedCallable]:
        return self.router.get(
            path,
            ... # code continues
        )
```

What we can see here is that `FastAPI.{get,put,post,etc}` are simply decorators that will include the path to APIRouter.

What about `FastAPI.include_router`?

FastAPI's function `include_router` will simply call it's own APIRouter's `include_router`, that basically will iterate through all routes included in your APIRouter and add the route.

```python
    # FastAPI include_router

    def include_router(
        self,
        router: Annotated[routing.APIRouter, Doc("The `APIRouter` to include.")],
        *,
        ... # other args
    ) -> None:
        self.router.include_router(
            router,
            ... # other args
        )

    # APIRouter include_router

    def include_router(
        self,
        router: Annotated["APIRouter", Doc("The `APIRouter` to include.")],
        ... # other args
    ) -> None:
        for route in router.routes:
            if isinstance(route, APIRoute):
                ... # some logic here

                self.add_api_route(
                    prefix + route.path,
                    route.endpoint,
                    ... # other args
                )
```

Looking at `APIRouter.include_router` we can see that it handles other type of routes, like Starlette's routes, `APIWebSocketRoute`, etc...

## And when my route function gets called?

When we receive a request, Starlette's Router will be called, since APIRouter don't override `__call__`. If it finds a matching route, it will call the route's handle function.

`handle` belongs to `Route` too, since it is not overwritten as well. What `APIRoute` does is setting Route's `app` to Starlette's function `request_response`, receiving `APIRoute`'s `get_route_handler` as a parameter.

```python
class APIRoute(routing.Route):
    def __init__(
        self,
        path: str,
        endpoint: Callable[..., Any],
        *,
        ... # other args
    ) -> None:
        ... # some logic here
        self.app = request_response(self.get_route_handler())
```

`get_route_handler` returns the `get_request_handler` function. It's here that we start to see the "translation" of Starlette's request to a FastAPI route with dependants, pydantic models, etc.

It will run the `run_endpoint_function` function. And here is where our route function is being called with all the resolved dependencies, pydantic models, etc.

```python
def get_request_handler(
    ... # args
) -> Callable[[Request], Coroutine[Any, Any, Response]]:
    # logic here

    async def app(request: Request) -> Response:
        response: Union[Response, None] = None
        async with AsyncExitStack() as file_stack:
            # logic here
            errors: List[Any] = []
            async with AsyncExitStack() as async_exit_stack:
                # logic here
                if not errors:
                    raw_response = await run_endpoint_function(
                        dependant=dependant, values=values, is_coroutine=is_coroutine
                    )
```

Pretty cool the see how the framework you are using handles your code right?

In the next post, we'll take a look where and when FastAPI handles your API documentation.
