# Understanding FastAPI: How OpenAPI works

* [Linkedin](https://www.linkedin.com/pulse/understanding-fastapi-how-openapi-works-rafael-de-oliveira-marques-ynmpf)
* [dev.to/ceb10n](https://dev.to/ceb10n/understanding-fastapi-how-openapi-works-4b6j)
* [github.com/ceb10n](https://github.com/ceb10n/blog-posts/tree/master/understanding-fastapi-how-openapi-works)

We already saw [what is ASGI](https://dev.to/ceb10n/understanding-fastapi-the-basics-246j), [how starlette works](https://dev.to/ceb10n/understanding-fastapi-how-starlette-works-43i1) and [how FastAPI extends starlatte](https://dev.to/ceb10n/understanding-fastapi-how-fastapi-works-37od). Now it's time to see how FastAPI offers one of the greatest features (in my opinion): Out of the box API docs with [Swagger](https://swagger.io/) and [Redoc](https://redocly.com).

## 💭 What is OpenAPI?

If we go to the [OpenAPI's repository](https://github.com/OAI/OpenAPI-Specification), we'll see that:

> The OpenAPI Specification (OAS) defines a standard, programming language-agnostic interface description for HTTP APIs. This allows both humans and computers to discover and understand the capabilities of a service without requiring access to source code, additional documentation, or inspection of network traffic. When properly defined via OpenAPI, a consumer can understand and interact with the remote service with a minimal amount of implementation logic. Similar to what interface descriptions have done for lower-level programming, the OpenAPI Specification removes guesswork in calling a service.

So the OpenAPI is simply a document that tells you how you can define and implement an API following a standard, making it easier for other developers to understand and consume.

If you have ever worked with any type of systems integration, you know how easy or extremely difficult your life can be, depending on the quality of the integration documentation.

And here is where FastAPI ✨ shines: It leverages [Pydantic](https://docs.pydantic.dev) powerful data validation to offer out of the box JSON Schema and OpenAPI specs via Swagger and Redoc.

## How FastAPI offers JSON Schema and OpenAPI?

![OpenAPI logo](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fnzwdb72uw40s7g8l6kp.png)

FastAPI will give you for free OpenAPI docs with both Swagger and Redoc.

The JSON Schema is offered using Pydantic, and if you are using FastAPI, you probably already know it.

### 🚀 Pydantic and JSON Schema

![pydantic logo](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3i24csifpyoanmmmlznk.png)

See how easy is to generate a JSON Schema with Pydantic:

```python
from pydantic import BaseModel, Field, TypeAdapter


class User(BaseModel):

    id: str = Field(
        ...,
        title="User ID",
        description="User's unique identification"
    )

    name: str = Field(
        ...,
        title="Name",
        description="User's name",
        min_length=5,
        max_length=50
    )


type_adapter = TypeAdapter(User)
```

Now if we call `TypeAdapter`'s `json_schema` method, we'll get a valid JSON Schema compliant with OpenAPI Specification v3.1.0:

```json
{
    "properties": {
        "id": {
            "description": "User's unique identification",
            "title": "User ID",
            "type": "string"
        },
        "name": {
            "description": "User's name",
            "maxLength": 50,
            "minLength": 5,
            "title": "Name",
            "type": "string"
        }
    },
    "required": [
        "id",
        "name"
    ],
    "title": "User",
    "type": "object"
}
```

### ⚙️ FastAPI documentation setup

When we create a FastAPI instance, it will setup all you need to create the docs.

The 🧙 _magic_ will happen inside `setup` function, that is called inside `__init__`.

If we look at `setup` function, we'll see that `FastAPI` creates for us an `openapi` route unless we set the `openapi_url=None`.

```python
if self.openapi_url:
    urls = (server_data.get("url") for server_data in self.servers)
    server_urls = {url for url in urls if url}

    async def openapi(req: Request) -> JSONResponse:
        root_path = req.scope.get("root_path", "").rstrip("/")
        if root_path not in server_urls:
            if root_path and self.root_path_in_servers:
                self.servers.insert(0, {"url": root_path})
                server_urls.add(root_path)
        return JSONResponse(self.openapi())

    self.add_route(self.openapi_url, openapi, include_in_schema=False)
```

FastAPI is setting up here the `/openapi.json` route, that will call the `get_openapi` function. It will return all the Schema from our project, or more precisely, an `OpenAPI` object defined inside openapi module.

So the order of events is:

When you create your app, FastAPI will add the routes:
  * /openapi.json
  * /docs
  * /redoc

if you don't remove it explicitly.

The `/openapi.json` route will return a `JSONResponse` containing all needed data from your routes when called.

The `/docs` and `/redoc` routes will basically return an HTML that loads the swagger and redoc respectively.

### 📑 Minimal example

If we create a minimal application:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def hello():
    return {"Hello": "world :D"}
```

We can see our swagger docs calling `/docs`:

![FastAPI swagger docs](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2kv6mxgeepth2g18wpd6.jpg)

And redoc docs calling `/redoc`:

![FastAPI redoc docs](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/cfq8gfuv68jxhwxjl1lk.jpg)

## 📰 Setting up Redoc

![Redoc logo](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ut8f8da7ip02uytwg46n.png)

We have seen that redoc will be available for free when you create a FastAPI project.

What is happening here is that FastAPI will define a redoc url by default:

```python
if self.openapi_url and self.redoc_url:
    async def redoc_html(req: Request) -> HTMLResponse:
        root_path = req.scope.get("root_path", "").rstrip("/")
        openapi_url = root_path + self.openapi_url
        return get_redoc_html(
            openapi_url=openapi_url, title=f"{self.title} - ReDoc"
        )

    self.add_route(self.redoc_url, redoc_html, include_in_schema=False)
```

Since both `openapi_url` and `redoc_url` has default values of `/openapi.json` and `/redoc` respectively, redoc will be served at `/redoc` by default.

If you want to remove redoc from your project, you just need to set `redoc_url=False`:

```python
app = FastAPI(redoc_url=None)
```

If you want to remove OpenAPI docs too, you just need to set `openapi_url` to `None`:

```python
app = FastAPI(openapi_url=None)
```

## 📃 Setting up Swagger

![Swagger Logo](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ve2onioesgvbirj81bi4.png)

`Swagger` will work the same way as `redoc`. The path `/docs` will be added automaticaly if don't pass any value to `docs_url`. If you inform a new path, swagger will be served in this new url:

```python
if self.openapi_url and self.docs_url:

    async def swagger_ui_html(req: Request) -> HTMLResponse:
        root_path = req.scope.get("root_path", "").rstrip("/")
        openapi_url = root_path + self.openapi_url
        oauth2_redirect_url = self.swagger_ui_oauth2_redirect_url
        if oauth2_redirect_url:
            oauth2_redirect_url = root_path + oauth2_redirect_url
        return get_swagger_ui_html(
            openapi_url=openapi_url,
            title=f"{self.title} - Swagger UI",
            oauth2_redirect_url=oauth2_redirect_url,
            init_oauth=self.swagger_ui_init_oauth,
            swagger_ui_parameters=self.swagger_ui_parameters,
        )

    self.add_route(self.docs_url, swagger_ui_html, include_in_schema=False)
```

As we can see, the `setup` function will add the swagger ui url to our app unless we define `docs_url=None`.

👋 and that's all for today. See you in the next article 🤗
