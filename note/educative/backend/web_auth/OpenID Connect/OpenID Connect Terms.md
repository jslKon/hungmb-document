# OpenId Connect Terminologies

## Identity token

Identity token encodes user's authentication information.

ID token contains

Client receives identity token -> validate first. Client must validate following fields

| Key | Description                                                                     |
|-----|---------------------------------------------------------------------------------|
| iss | Client must validate that the issuer of this token is the Authorization Server. |
| aud | Client must validate that the token is meant for the client itself.             |
| exp | Client must validate that the token is not expired.                             |

Sample information in form of JSON

> {
"iss": "https://server.example.com",
"sub": "24400320",
"aud": "s6BhdRkqt3",
"nonce": "n-0S6_WzA2Mj",
"exp": 1311281970,
"iat": 1311280970,
"auth_time": 1311280969,
"acr": "urn:mace:incommon:iap:silver"
> }

| Key       | Description                                                                                                                                                                                                                                |
|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| iat       | The `iat` claim identifies the time at which the JWT was issued. It can determine the age of the JWT. Its value must be a number containing a NumericDate value.                                                                           |
| auth_time | Time when the End-User authentication occurred. Its value is a JSON number representing the number of seconds from 1970–01–01T0:0:0Z as measured in UTC until the date/time.                                                               |
| nonce     | ID token requests may come with a nonce request parameter to protect from replay attacks. When the request parameter is included, the server will embed a nonce claim in the issued ID token with the same value of the request parameter. |

## Scope

Client app must tell IP what user information that app is look for -> can be provided by scope field

4 type of scopes:

- Email
- Phone
- Profile
- Address

> **Important** scope openid is mandatory if client app need ID token

## Claims

Name/value key pair contain user info.
Claim by scope

| Scope   | Claims                                                                                                                                              |
|---------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| email   | email, email_verified                                                                                                                               |
| phone   | phone_number, phone_number_verified                                                                                                                 |
| profile | name, family_name, given_name, middle_name, nickname, preferred_username, profile picture, website, gender, birthdate, zoneinfo, locale, updated_at |
| address | address                                                                                                                                             |

## Endpoints

### Authorization endpoint

Endpoint returns the authorization code after the user authenticates and provides their consent to the authorization
server (called the Identity Provider in case of OpenID)

### Token endpoint

This endpoint is used to exchange an authorization token with an access token and identity token. The identity token
contains some basic user information

### Userinfo endpoint

The UserInfo endpoint is used by the client to request user profile information that was previously granted access to.

Following is a sample response from this endpoint:

HTTP/1.1 200 OK
Content-Type: application/json
{
"sub": "Joe",
"email": "Joe@dummy.net",
"email_verified": true,
"name": "Joe Frank"
}


