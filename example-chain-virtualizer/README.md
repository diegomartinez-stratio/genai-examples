# Example chain to show how to connect to Virtualizer from the chain

This is an example chain to show how to run queries in Virtualizer from a chain.

## Local deployment

We assume that you already have poetry installed. If not, you can install it from [here](https://python-poetry.org/docs/#installation).

Verify that you have a dependencies source in your `pyproject.toml` with the URL of a PyPi server providing the *Stratio GenAI Core* dependency, like the one in *Stratio GenAI Developer Proxy*.
Note that the URL below is just an example and you should add the correct URL for your case.

```toml
[[tool.poetry.source]]
name = "stratio-releases"
url = "https://genai-developer-proxy-loadbalancer.your-tenant-genai.yourdomain.com:8080/service/genai-api/v1/pypi/simple/"
priority = "supplemental"
```
You should also configure Poetry to use the CA of the cluster to verify the certificate of the
above configured repository (the CA of the cluster can be found in the zip you obtain from Gosec with your
certificates).

```
$ poetry config certificates.stratio-releases.cert /path/to/ca-cert.crt 
```

Then install the poetry environment:
```
$ poetry install
```

Set up the needed environment variables. You need to specify the Virtualizer server that the chain will connect to. This is normally specified in the deployment configuration of the chain when registering it in *Stratio GenAI API*. While developing locally, you run your chain in a standalone server which is started by running the the `main.py` script. This scripts obtains the Virtualizer URL from the `VIRTUALIZER_HOST` and `VIRTUALIZER_PORT` environment variables, so you should set it with correct value before starting the chain. Also, when accessing Virtualizer through the *Stratio GenAI Developer Proxy*, an extra variable `VIRTUALIZER_BASE_PATH` is needed.

You can create a file `env.sh` like the following (or use the [helper script](../README.md#extra-environment-variables)):

```bash
# Variables needed to access Virtualizer. These are used by the main.py that launches the chain locally :
export VIRTUALIZER_HOST="genai-developer-proxy-loadbalancer.your-tenant-genai.k8s.yourdonain.com"
export VIRTUALIZER_PORT=8080
# this variable is only needed if accessing Virtualizer via the GenAI developer proxy
export VIRTUALIZER_BASE_PATH="/service/virtualizer"

# variables needed to tell the VaulClient where to find the certificates so it does not need to
# actually access any Vault. You can obtain your certificates from your profile in Gosec
export VAULT_LOCAL_CLIENT_CERT="/path/to/cert.crt"
export VAULT_LOCAL_CLIENT_KEY="/path/to/private-key.key"
export VAULT_LOCAL_CA_CERTS="/path/to/ca-cert.crt"

# This is needed by the Virtualizer client used by the chain. Normally this variable is already
# defined when running inside genai-api, but for local development you need to provide it yourself.
# It should match the service name of the GenAI API that your GenAI developer proxy is configured to use
export GENAI_API_SERVICE_NAME="genai-api.s000001-genai"
```
and then source it (or [add to PyCharm](../README.md#running-from-pycharm))
```
$ source env.sh
```

Finally, you can now run the chain locally by calling the `main.py` script in the poetry environment (or [run from PyCharm](../README.md#running-from-pycharm)):
```
$ poetry run python virtualizer_chain/main.py
```

You can test your chain either via the swagger UI exposed by the local chain server, or with curl.
An example of request body for the invoke POST is the following:
```json
{
  "input": {
     "query": "SELECT 1 as id"
  },
  "config": {
    "metadata": {
      "__genai_state": {
        "client_auth_type": "mtls",
        "client_user_id": "your-user",
        "client_tenant": "your-tenant"
      }
    }
  }
}
```

The `"config"` key with the extra metadata about the user that has invoked the chain is normally added by GenAI API before passing the input to the chain, but while developing locally you should add it by hand. Note that the `client_user_id` should be the same user as the one in your certificates, otherwise the genai-developer-proxy will refuse to forward your query to the Virtualizer server (to make sure that you do not access any data for which you do not have permissions).


