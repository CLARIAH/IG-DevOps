# Authentication

## Summary

CLARIAH provides an OpenID-based authentication service at https://authentication.clariah.nl which can be used to add authentication to your application.

This service is based on [Satosa](https://github.com/IdentityPython/SATOSA). Satosa  is a proxy which sits between applications (service providers or "relying parties") and identity providers (that is, organizations where people can log in). The applications behind authentication.clariah.nl are not known to the identity providers: from the point of view of the identity provider, the application (service provider) which is trying to log in is authentication.clariah.nl. That authentication.clariah.nl is doing that on behalf of another application is of no concern to the identity provider.

By the same token, the various identity providers connnected to authentication.clariah.nl are not directly known to the applications. From the point of view of the application, its identity provider is authentication.clariah.nl. The application does not know that authentication.clariah.nl redirects its authentication request to another (the "real") identity provider, which is chosen by the user by means of the discovery service [discovery.clariah.nl](https://discovery.clariah.nl).

discovery.clariah.nl contains all of the identity providers registered at CLARIN (see "Participating Identity Federations" at CLARINs [Service Provider Federation](https://www.clarin.eu/content/service-provider-federation)). This basically means: all European academic institutions. In addition to this CLARIN-provided collection of identity providers, [Beeld en Geluid](https://www.beeldengeluid.nl/) is also available as an identity provider in discovery.clariah.nl.


## How to register your service at authentication.clariah.nl

To use the OpenID-provider at authentication.clariah.nl, your service needs to be registered. This is a manual process at the moment, handled by the Concern Infrastructure department at HuC-DI.

The OpenID client in your application must have three options configured: `ClientID`, `ClientSecret` and `RedirectURI`. To make this OpenID client known to authentication.clariah.nl, it needs information about it in the following JSON format, in a file called `<ClientID>.json`:

```json
{
  "client_name": "<human-readable string>",
  "client_secret": "<random string>",
  "redirect_uris": [
    "<URL at your application to redirect to after succesful login>"
  ]
}
```

The file should contain at least the `ClientSecret` in the key `client_secret` and the `RedirectURI` in the key `redirect_uris` (an array, as there can be more than one). Depending on the OpenID implementation on the client, more configuration options may be necessary.

HuC-DI Concern Infrastructure can add this JSON file to authentication.clariah.nl and if everything works correctly a federated login flow will be active from then on.

### TODO
concise description of federated login flow

describe probable need for whitelisting on the basis of attributes delivered by the identity provider

more detailed description of configuration of a specific OpenID client