# OAuth Terminologies

## Resource owner

The resource owner is the owner of the resource that is being accessed

When you log in to PicsArt App using your Facebook account, you are granting access to PicsArt to access your images. In
this case, you are the resource owner

## Client

The client is an application that accesses protected resources on behalf of the resource owner

## Resource server

The server that is hosting the protected resources and is capable of accepting and responding to requests by clients
using access tokens. In our example, Facebook is the resource server

## Authorization server

The server which issues access tokens after successfully authenticating a client and resource owner is called
authorization server

In our example, the Facebook server was issuing the access token. Normally, there is a separate server that does this
task.

## Authorization grant

An authorization grant is a mechanism by which a client obtains an access token from the authorization server

## Access token

Access tokens are credentials used to access protected resources

## Scope

Scope defines the permissions of a token. It defines what resources can be accessed using a given access token. In our
example, the client app wants to access only images, so the images are scope