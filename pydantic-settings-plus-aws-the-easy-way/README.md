# Pydantic Settings + AWS the easy way

[Pydantic Settings](https://github.com/pydantic/pydantic-settings) is a python library that extends [🚀 Pydantic](https://github.com/pydantic/pydantic) for dealing with settings management.

If you've never heard of pydantic, please, check [it's docs page](https://docs.pydantic.dev/latest/) to see how easy and powerfull it is. In my opinion, it is one of the best python libraries I have come across in the last few months/years. Congrats [Samuel Colvin](https://github.com/samuelcolvin)!

## ⚙️ Using Pydantic Settings

To use `pydantic-settings`, we first need to install it:

```shell
pip install pydantic
pip install pydantic-settings
```

Now let's imagine that we have a [FastAPI](https://fastapi.tiangolo.com/) microservice that needs to load some configuration.

Let's say we depend on some information to work properly:

* Environment that is available in the environment variables
* Database host that is available in an AWS Parameter Store
* Database authentication info that is available in an AWS Secrets Manager

What we would need to do is something like:

```python
import json

import boto3

from pydantic_settings import BaseSettings, SettingsConfigDict


class AppSettings(BaseSettings):
    model_config = SettingsConfigDict(env_prefix="APP_")

    environment: str  # will be loaded by environment variable APP_ENVIRONMENT
    host: str  # need to pass it when creating AppSettings :(
    username: str  # need to pass it when creating AppSettings :(
    password: str  # need to pass it when creating AppSettings :(


ssm_client = boto3.client("boto3")
secrets_client = boto3.client("secretsmanager")

# need to get the secret before creating our settings object
secret_response = secrets_client.get_secret_value(SecretId="my/secrets")
secret_string = secret_response.get("SecretString")
secret = json.loads(secret_string)

# need to get the parameter store before creating our settings object
parameter = ssm_client.get_parameter(Name="/my/ssm/parameter")
host = parameter.get("Parameter").get("Value")


settings = AppSettings(host=host, **secret)
```

Don't get me wrong, it's still great. But it would be better if we had a pydantic settings source to do that work for us!

Obs: You can check `pydantic-settings` [📖 docs](https://docs.pydantic.dev/latest/concepts/pydantic_settings/#other-settings-source) to see all settings sources that it offers.

## ☁️ Pydantic Settings AWS


![Pydantic Settings + AWS](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ge68rjritodq5ffi2r8n.png)

I recently started creating a `pydantic-settings` extension to deal with settings that lives at [AWS](https://aws.amazon.com/). It called [Pydantic Settings AWS](https://ceb10n.github.io/pydantic-settings-aws/) and you can see the source code at [github.com/ceb10n/pydantic-settings-aws](https://github.com/ceb10n/pydantic-settings-aws)

### 🏃 Gettings started

If you still don't have boto3 or pydantic-settings installed:

```shell
pip install boto3
pip install pydantic-settings
```

And then you can install pydantic-settings-aws

```shell
pip install pydantic-settings-aws
```

Now let's get back to our example.

We have a basic settings class that needs to load data from Environmet variables, [parameter store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) and [secrets manager](https://aws.amazon.com/secrets-manager/).

`pydantic-settings-aws` offers a class called `AWSBaseSettings`, that gives you the ability to deal with all that data sources.

```python
from typing import Annotated

from pydantic_settings import SettingsConfigDict
from pydantic_settings_aws import AWSBaseSettings


class AppSettings(AWSBaseSettings):
    model_config = SettingsConfigDict(
        secrets_name="my/secrets",
        env_prefix="APP_"
    )

    environment: str
    host: Annotated[str, {"service": "ssm", "ssm": "/my/ssm/parameter"}]
    username: Annotated[str, {"service": "secrets"}]
    password: Annotated[str, {"service": "secrets"}]


settings = AppSettings()
```

With `AWSBaseSettings`, all you need to do is using `typing.Annotated` to inform what service you want to use, and other things, like the secret and parameter store names.

### 🙊 Settings with Secrets Manager only

If you simply want to use Secrets Manager, you can use the `SecretsManagerBaseSettings`.

Since we are using only Secrets Manager as a data source, we don't need to specify anything, except the secrets name:

```python
from pydantic_settings import SettingsConfigDict
from pydantic_settings_aws import SecretsManagerBaseSettings


class AppSettings(SecretsManagerBaseSettings):
    model_config = SettingsConfigDict(
        secrets_name="my/secrets"
    )

    username: str
    password: str

settings = AppSettings()
# username='my-username' password='my-super-secret-password'
```

### 🔒 Settings with Parameter Store only

Let's say you use Parameter Store to save a lot of you apps configuration, like queue names, urls, etc.

You can leverage the `ParameterStoreBaseSettings` class to load all data from different parameter stores.

```python

from pydantic_settings_aws import ParameterStoreBaseSettings


class AppSettings(ParameterStoreBaseSettings):

    dev_base_endpoint: str
    database_host: str
    database_port: str
```

And thats it. `ParameterStoreBaseSettings` will try to create a `boto3` client with your local configuration, and then will try to get the value from a parameter store with the same name as your field.

But it's not uncommon for the names of our parameter stores to not match the names of our variables. In theses cases, you can use `typing.Annotated`:

```python
from typing import Annotated

from pydantic_settings_aws import ParameterStoreBaseSettings


class AppSettings(ParameterStoreBaseSettings):

    dev_base_endpoint: Annotated[str, "/payments/endpoint"]
    database_host: Annotated[str, "/databases/mongodb/payments/dbhost"]
    database_port: Annotated[str, "/databases/mongodb/payments/dbport"]
```

## 📜 Docs and source code

You can take a look at `pydantic-settings-aws` documentations at [ceb10n.github.io/pydantic-settings-aws](https://ceb10n.github.io/pydantic-settings-aws/].

The project is hosted at github: [github.com/ceb10n/pydantic-settings-aws](https://github.com/ceb10n/pydantic-settings-aws). It's still an ongoing project. Feel free to open an issue, make a PR, etc. 🤗
