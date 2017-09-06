# Beginners Guides

The Agave REST APIs enable applications to create and manage digital laboratories that spans campuses, the cloud, and multiple data centers using a cohesive set of web-friendly interfaces. In order for an application to access user-related data through Agave's REST API, it must get the user’s authorization to access that data.

This tutorial is focused familiarizing you with the REST API through the use of the <a href="http://preview.agaveapi.co/tools/command-line-interface/" title="Agave Command Line Interface (CLI)">Agave Command Line Interface (CLI)</a> and the Unix `curl` command. We will show you how to:

* <a href="/documentation/beginners-guides/user-discovery/" title="User Discovery">Discover other users</a>
* <a href="/documentation/beginners-guides/system-discovery/" title="System Discovery">Browse and use multiple compute and storage systems</a>
* <a href="/documentation/beginners-guides/managing-data/" title="Managing Data">Browse, manage, and share data across systems</a>
* <a href="/documentation/beginners-guides/app-discovery/" title="App Discovery">Browse and search the app catalog</a>
* <a href="/documentation/beginners-guides/running-a-simulation/" title="Running a Simulation">Run a simulation</a>
* <a href="/documentation/beginners-guides/managing-metadata/" title="Managing Metadata">Search and manage metadata</a>
* <a href="/documentation/beginners-guides/share-results/" title="Share Results">Share your results</a>

The authorization flow we will use in this tutorial is the Client Credentials Flow. This flow is meant for situations like the command line, where the user implicitly trusts the client application and no browser is involved.

The data that we will retrieve will come from the Profiles, Systems, Files, Apps, Jobs, Metadata, and PostIts services, and will include information specific to the authenticated user. To customize this tutorial for your personal user account, please <a class="federated-login" href="#" title="Login" target="_blank">Login</a>. If you do not already have an account, you can create one <a href="https://user.iplantcollaborative.org" title="iPlant User Portal" target="_blank">here</a>.

The step by step instructions and sample data we will use in these lessons is available <a href="https://bitbucket.org/taccaci/agave-beginners-guide" title="Agave Beginner's Guide" target="_blank">here</a>.

## Setting Up Your Account

To use the REST APIs, the first thing you will need is a user account. If your organization is already using Agave to power its infrastructure, you can <a class="federated-login" href="#" title="Login" target="_blank">Login</a> with that account. If not, you can create a free <a href="https://user.iplantcollaborative.org" title="iPlant User Portal" target="_blank">iPlant user account</a> and take advantage of their world class cyberinfrastructure while kicking the tires.

## Getting Your Client API Keys

In order to authenticate and interact with the API, you will need to get a set of <a href="http://agaveapi.co/client-registration" title="API Keys">API keys</a>. This is a one-time action. If you already have your API keys, skip to the next section. If not, you can create your keys using the Clients service.

```shell
curl -sku "$API_USERNAME:$API_PASSWORD" -X POST -d "client_name=my_cli_app&description=Client app used for scripting up cool stuff" https://public.tenants.agaveapi.co/clients/v2
```
```plaintext
clients-create -S -v -N my_cli_app -D "Client app used for scripting up cool stuff"
```
> Note: the -S option will store the new API keys for future use so you don't need to manually enter then when you authenticate later.


The response to this call for our example user looks like this:

```javascript
{
    "message": "Client created successfully.",
    "result": {
        "callbackUrl": "",
        "consumerKey": "gTgpCecqtOc6Ao3GmZ_FecVSSV8a",
        "consumerSecret": "hZ_z3f4Hf3CcgvGoMix0aksN4BOD6",
        "description": "Client app used for scripting up cool stuff",
        "name": "my_cli_app",
        "tier": "Unlimited",
        "_links": {
            "self": {
                "href": "https://public.tenants.agaveapi.co/clients/v2/my_cli_app"
            },
            "subscriber": {
                "href": "https://public.tenants.agaveapi.co/profiles/v2/nryan"
            },
            "subscriptions": {
                "href": "https://public.tenants.agaveapi.co/clients/v2/my_cli_app/subscriptions/"
            }
        }
    },
    "status": "success",
    "version": "2.0.0-SNAPSHOT-rc3fad"
}
```

Creating your API keys is pretty straightforward and, as mentioned above, a one-time action. The one thing you should note from the above example is that, unlike the rest of the APIs, <strong>the Clients service requires HTTP BASIC authentication with your API username and password</strong> rather than an Authorization header with an access token. This discrepancy is intentional. Until you create your API keys, you cannot obtain an access token. By using BASIC auth, we avoid a chicken and egg problem.

## Obtain an Authentication Token

Using the API username, password, and keys from above, you can obtain an authentication token from the OAuth service.

```shell
curl -sku "hZ_z3f4Hf3CcgvGoMix0aksN4BOD6:gTgpCecqtOc6Ao3GmZ_FecVSSV8a" -X POST -d "grant_type=client_credentials&username=$API_USERNAME&password=$API_USERNAME&scope=PRODUCTION" -H "Content-Type:application/x-www-form-urlencoded" https://public.tenants.agaveapi.co/token
```

```plaintext
auth-tokens-create -S -v
```
> Note: the -S option will store the token for future use so you don't need to keep re-authenticating with every call.


The response to this call for our example user looks like this:

```javascript
{
    "access_token": "$ACCESS_TOKEN",
    "expires_in": 3058,
    "refresh_token": "$REFRESH_TOKEN",
    "token_type": "bearer"
}
```
