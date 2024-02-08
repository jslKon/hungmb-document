## Cryptographic Key Management

## Will the secret keys rotate?

Definitely yes! It is not safe or advisable to use the same key for a long time. If someone accidentally gets ahold of
our application's key, then the attacker can send malicious requests to our server and we might not even know. If we
rotate our keys regularly then we decrease the window of unauthorized access that an attacker gets.

Now the question is, how often should we change the keys? There is no definite answer to this, as it depends on the
application. Some might change them every hour and others might keep the same key for a month. It depends on how many
tokens you generate and how vulnerable your application is to attacks.

## What would happen to old JWT once a key is changed?

Changing the secret key is easy, the hard part is validating the tokens that were signed by the previous key. We can’t
just invalidate the old tokens every time we change the key. There is a way to handle this problem. What if while
generating the token, we also mention the key that was used for signing inside the header of the token? When we receive
this token, we can check the header to see what key was used to sign it.

I know what you are wondering: if we write the key in the token itself, then an attacker can easily read our token and
will get to know about the key. Here comes the trick. We will not write the actual key in the token but only an
identifier. We will maintain a list of keyIds and corresponding keys in our memory or DB. When we receive a token, we
will fetch the keyId from the token. Then we will look up our memory or DB for the actual key.

The JWT header supports a kid parameter to hold a key identifier. It can take any value, as it is just used to identify
the actual key.

## What would happen if the private key is changed in asymmetric signing?

As we know that in the case of asymmetric signing, the private key is used to sign the token and it can be verified by
all the applications that have the public key. If we keep rotating the private key, then the public key will also
change. How will an application get to know which public key to use?

We can associate each public key with a keyId and provide this keyId in the kid parameter of the token header. But in
that case, each application will have to maintain a set of public ids for lookup. This is not a feasible option.

We have a JSON Web Key specification that offers a way to represent cryptographic keys in a JSON format. A JWK can be
included in a JWT token as a way to distribute the public key. We have two ways in which we can include the public keys
in our token header:

1. Directly embedding the public key
   As the name suggests, public keys are public. So even if we add them into the token, it is not a security threat, as
   everyone can easily access the public key.

However, we don’t add the public key directly into the token. We can convert the public key into JSON format using any
of the available libraries. A JSON object that represents a cryptographic key is called JWK.

A JWK can be included in the token header using jwk parameter.

2. Embedding a URL which contains the key
   We can maintain the complete list of all the public cases at a particular location. Then in the token header, we can
   provide the URL of this location along with the key identifier. There are two claims available which help us in doing
   this. The first is kid, which we have discussed earlier also. It will contain the key identifier. The second claim is
   jku. It contains the URL of the location where all the keys are stored.

In the next lesson, we will discuss the ways to hack a JWT.
