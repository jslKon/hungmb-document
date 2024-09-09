# Customer App Design

## Component design

## Use case diagram

## Sequence diagram

## Security solution

My solution is to protect the app by using OAuth 2.0 with Proof Key for Code Exchange (PKCE) (Since our application is a
public app)

The flows should be:

- The customer opens the Customer App and initiates a login request.
- The Customer App generates a code_verifier (a random string) and its corresponding code_challenge using a SHA-256
  hash.
- The Customer App redirects the user to the UserService (IDP) login page with the following parameters:
  ```
  response_type=code
  client_id=coffee-app
  redirect_uri=//Callback URL herer
  scope=openid profile
  code_challenge=[BASE64URL(SHA256(code_verifier))]
  code_challenge_method=S256
  ```
- UserService redirects the user back to the Customer App using the redirect_uri with the authorization code included in
  the query parameters (code=abc123).
- The Customer App sends a request to the UserService token endpoint to exchange the authorization code for an access
  token:
  POST /oauth/token
  Headers: Content-Type: application/x-www-form-urlencoded
  ``` json
  {
  "access_token": "",
  "token_type": "Bearer",
  "expires_in": ,
  "refresh_token": "",
  "scope": "openid profile"
  }
  ```
- The Customer App uses the access token to make authenticated requests to the APIGateway and other backend services.

## API endpoints

Get nearby shop endpoints

```
curl -X GET "https://{domain}/shops?lat=37.7749&lng=-122.4194" \
     -H "Authorization: Bearer your_jwt_token_here" \
     -H "Locale: en-US"
```

Get shop menu

```
curl -X GET "https://{domain}/shops/123/menu" \
     -H "Authorization: Bearer your_jwt_token_here" \
     -H "Locale: en-US"
```

Register new user endpoints

```
curl -X POST https://{domain}/register \
     -H "Content-Type: application/json" \
     -H "Locale: en-US" \
     -d '{"mobile": "", "name": "", "address": ""}'
```

Create new order endpoint

```
curl -X POST https://{domain}/orders \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer your_jwt_token_here" \
     -H "Locale: en-US" \
     -d '{"shopId": "123", "items": [{"itemId": "", "quantity": }]}'
```

Get order info

```
curl -X GET https://{domain}/orders/456 \
     -H "Authorization: Bearer your_jwt_token_here" \
     -H "Locale: en-US"
```

Get orders info by customer id

```
curl -X GET "https://{domain}/customers/{customerId}/orders" \
     -H "Authorization: Bearer your_jwt_token_here" \
     -H "Locale: en-US"
```

Confirm order

```
curl -X PUT https://{domain}/orders/456 \
     -H "Authorization: Bearer your_jwt_token_here" \
     -H "Locale: en-US"
```

Cancel an order

```
curl -X DELETE https://{domain}/orders/456 \
     -H "Authorization: Bearer your_jwt_token_here" \
     -H "Locale: en-US"
```