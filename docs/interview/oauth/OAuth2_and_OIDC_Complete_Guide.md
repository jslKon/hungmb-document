# OAuth 2.0 and OpenID Connect (OIDC) Complete Guide

## Table of Contents

1. [Introduction](#introduction)
2. [OAuth 2.0 Fundamentals](#oauth-20-fundamentals)
3. [OAuth 2.0 Grant Types](#oauth-20-grant-types)
4. [OpenID Connect (OIDC)](#openid-connect-oidc)
5. [Security Considerations](#security-considerations)
6. [Implementation Examples](#implementation-examples)
7. [Best Practices](#best-practices)
8. [Common Vulnerabilities](#common-vulnerabilities)
9. [Tools and Libraries](#tools-and-libraries)

## Introduction

### What is OAuth 2.0?

OAuth 2.0 is an authorization framework that enables applications to obtain limited access to user
accounts on an HTTP service. It works by delegating user authentication to the service that hosts
the user account and authorizing third-party applications to access that account.

**Key Concepts:**

- **Authorization**: The process of granting access to resources
- **Delegation**: Users can grant access to their resources without sharing credentials
- **Scopes**: Define the level of access granted to the application
- **Tokens**: Bearer tokens used to access protected resources

### What is OpenID Connect (OIDC)?

OpenID Connect is an identity layer built on top of OAuth 2.0 that provides authentication and user
information. While OAuth 2.0 handles authorization, OIDC adds authentication capabilities.

**Key Benefits:**

- Single sign-on (SSO) across applications
- Standardized user information
- Built-in security features
- Interoperability between different systems

## OAuth 2.0 Fundamentals

### Core Components

#### 1. **Resource Owner (User)**

- The entity that owns the protected resources
- Can grant access to their resources

#### 2. **Client Application**

- The application requesting access to protected resources
- Can be web applications, mobile apps, or single-page applications

#### 3. **Authorization Server**

- Issues access tokens after successful authentication
- Handles user consent and authorization
- Examples: Google OAuth, Facebook OAuth, Auth0

#### 4. **Resource Server**

- Hosts protected resources
- Accepts and validates access tokens
- Examples: API endpoints, protected data

#### 5. **User Agent**

- The browser or application that handles the OAuth flow
- Manages redirects and user interactions

### OAuth 2.0 Flow Overview

```
┌─────────────┐    ┌──────────────┐    ┌─────────────┐
│   Client    │    │ Authorization│    │   Resource  │
│ Application │    │    Server    │    │    Owner    │
└─────────────┘    └──────────────┘    └─────────────┘
       │                   │                   │
       │ 1. Request        │                   │
       │ Authorization     │                   │
       │──────────────────▶│                   │
       │                   │                   │
       │                   │ 2. Authenticate  │
       │                   │ User & Get       │
       │                   │ Consent          │
       │                   │◀─────────────────▶│
       │                   │                   │
       │ 3. Authorization │                   │
       │ Code             │                   │
       │◀─────────────────│                   │
       │                   │                   │
       │ 4. Exchange Code │                   │
       │ for Access Token │                   │
       │──────────────────▶│                   │
       │                   │                   │
       │ 5. Access Token  │                   │
       │◀─────────────────│                   │
       │                   │                   │
       │ 6. Access        │                   │
       │ Protected        │                   │
       │ Resource         │                   │
       │──────────────────────────────────────▶│
       │                   │                   │
       │ 7. Protected     │                   │
       │ Resource         │                   │
       │◀─────────────────────────────────────│
```

## OAuth 2.0 Grant Types

### 1. **Authorization Code Grant (Most Secure)**

**Use Case:** Web applications with backend servers

**Flow:**

1. Client redirects user to authorization server
2. User authenticates and consents
3. Authorization server returns authorization code
4. Client exchanges code for access token
5. Client uses access token to access resources

**Advantages:**

- Most secure grant type
- Access tokens never exposed to user agent
- Refresh token support
- PKCE support for public clients

**Example Request:**

```http
GET /oauth/authorize?
  response_type=code&
  client_id=your_client_id&
  redirect_uri=https://your-app.com/callback&
  scope=read:user&
  state=random_state_string
```

### 2. **Implicit Grant (Legacy)**

**Use Case:** Single-page applications (SPAs) - **DEPRECATED**

**Flow:**

1. Client redirects user to authorization server
2. User authenticates and consents
3. Authorization server returns access token directly
4. Client uses access token to access resources

**Security Issues:**

- Access token exposed in URL
- No refresh token support
- Vulnerable to token theft

### 3. **Client Credentials Grant**

**Use Case:** Machine-to-machine (M2M) communication

**Flow:**

1. Client authenticates with client ID and secret
2. Authorization server validates credentials
3. Authorization server returns access token
4. Client uses access token to access resources

**Example:**

```http
POST /oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&
client_id=your_client_id&
client_secret=your_client_secret&
scope=api:read
```

### 4. **Resource Owner Password Credentials Grant**

**Use Case:** Legacy applications migrating to OAuth 2.0

**Flow:**

1. Client collects username and password
2. Client sends credentials to authorization server
3. Authorization server validates credentials
4. Authorization server returns access token

**Security Issues:**

- Client handles user credentials
- Credentials transmitted over network
- Should only be used for trusted first-party applications

### 5. **Refresh Token Grant**

**Use Case:** Obtaining new access tokens when they expire

**Flow:**

1. Client sends refresh token to authorization server
2. Authorization server validates refresh token
3. Authorization server returns new access token (and optionally new refresh token)

**Example:**

```http
POST /oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&
refresh_token=your_refresh_token&
client_id=your_client_id&
client_secret=your_client_secret
```

## OpenID Connect (OIDC)

### What OIDC Adds to OAuth 2.0

OIDC extends OAuth 2.0 with:

- **ID Token**: JWT containing user identity information
- **UserInfo Endpoint**: Standardized way to get user information
- **Standardized Scopes**: `openid`, `profile`, `email`, etc.
- **Discovery**: Dynamic client configuration

### OIDC Flows

#### 1. **Authorization Code Flow with OIDC**

**Additional Steps:**

1. Include `openid` scope
2. Receive ID token along with access token
3. Validate ID token signature and claims
4. Extract user information from ID token

**Example Request:**

```http
GET /oauth/authorize?
  response_type=code&
  client_id=your_client_id&
  redirect_uri=https://your-app.com/callback&
  scope=openid profile email&
  state=random_state_string&
  nonce=random_nonce_string
```

#### 2. **Implicit Flow with OIDC**

**Additional Steps:**

1. Include `openid` scope
2. Receive ID token along with access token
3. Validate ID token

#### 3. **Hybrid Flow**

**Combines:**

- Authorization code flow
- Implicit flow
- Returns both authorization code and tokens

### ID Token Structure

ID tokens are JWTs with standard claims:

```json
{
  "iss": "https://auth.example.com",
  "sub": "1234567890",
  "aud": "your_client_id",
  "exp": 1516239022,
  "iat": 1516235422,
  "nonce": "random_nonce_string",
  "name": "John Doe",
  "email": "john@example.com",
  "picture": "https://example.com/john.jpg"
}
```

**Required Claims:**

- `iss`: Issuer (authorization server)
- `sub`: Subject (user ID)
- `aud`: Audience (client ID)
- `exp`: Expiration time
- `iat`: Issued at time

**Optional Claims:**

- `name`: User's full name
- `email`: User's email address
- `picture`: User's profile picture
- `nonce`: Client-provided nonce for replay protection

### UserInfo Endpoint

After receiving an ID token, clients can get additional user information:

```http
GET /userinfo
Authorization: Bearer your_access_token
```

**Response:**

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "given_name": "John",
  "family_name": "Doe",
  "email": "john@example.com",
  "email_verified": true,
  "picture": "https://example.com/john.jpg"
}
```

## Security Considerations

### 1. **Token Security**

**Access Tokens:**

- Use short expiration times (15-60 minutes)
- Store securely (HTTPS only, secure cookies)
- Validate on every request
- Use appropriate scopes

**Refresh Tokens:**

- Store securely (encrypted at rest)
- Rotate regularly
- Revoke when compromised
- Use secure transmission

**ID Tokens:**

- Validate signature and claims
- Check expiration and audience
- Verify nonce for replay protection

### 2. **Client Security**

**Public Clients (SPAs, Mobile Apps):**

- Use PKCE (Proof Key for Code Exchange)
- Implement state parameter
- Use HTTPS for all communications
- Store tokens securely

**Confidential Clients (Web Apps with Backend):**

- Keep client secrets secure
- Use backend storage for tokens
- Implement proper session management
- Use secure redirect URIs

### 3. **Authorization Server Security**

**Best Practices:**

- Use HTTPS for all endpoints
- Implement rate limiting
- Validate redirect URIs
- Use secure session management
- Implement proper logging and monitoring

### 4. **Common Attack Vectors**

**Authorization Code Interception:**

- Use PKCE for public clients
- Validate redirect URIs strictly
- Use state parameter for CSRF protection

**Token Theft:**

- Use short-lived access tokens
- Implement token binding
- Use secure storage mechanisms
- Monitor for suspicious activity

**Replay Attacks:**

- Use nonce values
- Implement proper timestamp validation
- Use one-time tokens where possible

## Implementation Examples

### 1. **Node.js/Express OAuth Client**

```javascript
const express = require('express');
const axios = require('axios');
const app = express();

const CLIENT_ID = 'your_client_id';
const CLIENT_SECRET = 'your_client_secret';
const REDIRECT_URI = 'https://your-app.com/callback';
const AUTH_URL = 'https://auth.example.com/oauth/authorize';
const TOKEN_URL = 'https://auth.example.com/oauth/token';

app.get('/login', (req, res) => {
  const authUrl = `${AUTH_URL}?` +
      `response_type=code&` +
      `client_id=${CLIENT_ID}&` +
      `redirect_uri=${REDIRECT_URI}&` +
      `scope=openid profile email&` +
      `state=${generateRandomString()}` +
      `nonce=${generateRandomString()}`;

  res.redirect(authUrl);
});

app.get('/callback', async (req, res) => {
  const {code, state} = req.query;

  try {
    const tokenResponse = await axios.post(TOKEN_URL, {
      grant_type: 'authorization_code',
      code,
      client_id: CLIENT_ID,
      client_secret: CLIENT_SECRET,
      redirect_uri: REDIRECT_URI
    });

    const {access_token, id_token, refresh_token} = tokenResponse.data;

    // Store tokens securely
    // Validate ID token
    // Redirect to dashboard

    res.redirect('/dashboard');
  } catch (error) {
    res.status(500).send('Authentication failed');
  }
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### 2. **Python Flask OAuth Client**

```python
from flask import Flask, request, redirect, session
import requests
import jwt
import secrets

app = Flask(__name__)
app.secret_key = 'your-secret-key'

CLIENT_ID = 'your_client_id'
CLIENT_SECRET = 'your_client_secret'
REDIRECT_URI = 'https://your-app.com/callback'
AUTH_URL = 'https://auth.example.com/oauth/authorize'
TOKEN_URL = 'https://auth.example.com/oauth/token'

@app.route('/login')
def login():
    state = secrets.token_urlsafe(32)
    nonce = secrets.token_urlsafe(32)
    
    session['state'] = state
    session['nonce'] = nonce
    
    auth_url = f"{AUTH_URL}?" + \
               f"response_type=code&" + \
               f"client_id={CLIENT_ID}&" + \
               f"redirect_uri={REDIRECT_URI}&" + \
               f"scope=openid profile email&" + \
               f"state={state}&" + \
               f"nonce={nonce}"
    
    return redirect(auth_url)

@app.route('/callback')
def callback():
    code = request.args.get('code')
    state = request.args.get('state')
    
    if state != session.get('state'):
        return 'Invalid state parameter', 400
    
    # Exchange code for tokens
    token_data = {
        'grant_type': 'authorization_code',
        'code': code,
        'client_id': CLIENT_ID,
        'client_secret': CLIENT_SECRET,
        'redirect_uri': REDIRECT_URI
    }
    
    response = requests.post(TOKEN_URL, data=token_data)
    tokens = response.json()
    
    # Store tokens securely
    session['access_token'] = tokens['access_token']
    session['id_token'] = tokens['id_token']
    
    return redirect('/dashboard')

if __name__ == '__main__':
    app.run(debug=True)
```

### 3. **Java Spring Boot OAuth Client**

```java

@RestController
public class OAuthController {

  @Value("${oauth.client.id}")
  private String clientId;

  @Value("${oauth.client.secret}")
  private String clientSecret;

  @Value("${oauth.redirect.uri}")
  private String redirectUri;

  @Value("${oauth.auth.url}")
  private String authUrl;

  @Value("${oauth.token.url}")
  private String tokenUrl;

  @GetMapping("/login")
  public ResponseEntity<String> login(HttpSession session) {
    String state = generateRandomString();
    String nonce = generateRandomString();

    session.setAttribute("state", state);
    session.setAttribute("nonce", nonce);

    String authUrlWithParams = authUrl + "?" +
        "response_type=code&" +
        "client_id=" + clientId + "&" +
        "redirect_uri=" + redirectUri + "&" +
        "scope=openid profile email&" +
        "state=" + state + "&" +
        "nonce=" + nonce;

    return ResponseEntity.ok()
        .header("Location", authUrlWithParams)
        .build();
  }

  @GetMapping("/callback")
  public ResponseEntity<String> callback(
      @RequestParam String code,
      @RequestParam String state,
      HttpSession session) {

    String sessionState = (String) session.getAttribute("state");
    if (!state.equals(sessionState)) {
      return ResponseEntity.badRequest().body("Invalid state parameter");
    }

    // Exchange code for tokens
    TokenRequest tokenRequest = new TokenRequest();
    tokenRequest.setGrantType("authorization_code");
    tokenRequest.setCode(code);
    tokenRequest.setClientId(clientId);
    tokenRequest.setClientSecret(clientSecret);
    tokenRequest.setRedirectUri(redirectUri);

    // Make HTTP request to token endpoint
    // Store tokens securely
    // Redirect to dashboard

    return ResponseEntity.ok("Authentication successful");
  }

  private String generateRandomString() {
    return UUID.randomUUID().toString();
  }
}
```

## Best Practices

### 1. **Client Implementation**

- **Use HTTPS**: Always use HTTPS for all OAuth communications
- **Validate Tokens**: Validate all tokens on the server side
- **Secure Storage**: Store tokens securely (encrypted at rest)
- **Token Rotation**: Implement refresh token rotation
- **Error Handling**: Handle OAuth errors gracefully
- **Logging**: Log authentication events for security monitoring

### 2. **Server Implementation**

- **Scope Validation**: Validate requested scopes against allowed scopes
- **Rate Limiting**: Implement rate limiting for all endpoints
- **Audit Logging**: Log all authorization and token events
- **Token Binding**: Implement token binding for enhanced security
- **Session Management**: Use secure session management
- **CORS**: Configure CORS properly for web applications

### 3. **Security Headers**

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
```

### 4. **Token Validation**

```javascript
// Validate ID token
function validateIdToken(idToken, clientId, issuer) {
  try {
    const decoded = jwt.verify(idToken, publicKey, {
      algorithms: ['RS256'],
      audience: clientId,
      issuer: issuer,
      clockTolerance: 30
    });

    // Validate nonce if present
    if (decoded.nonce !== expectedNonce) {
      throw new Error('Invalid nonce');
    }

    return decoded;
  } catch (error) {
    throw new Error('Invalid ID token');
  }
}
```

## Common Vulnerabilities

### 1. **Open Redirects**

**Problem:** Unvalidated redirect URIs can lead to phishing attacks

**Solution:** Whitelist allowed redirect URIs

```javascript
const allowedRedirects = [
  'https://your-app.com/callback',
  'https://your-app.com/auth/callback'
];

function validateRedirectUri(uri) {
  return allowedRedirects.includes(uri);
}
```

### 2. **CSRF Attacks**

**Problem:** Attackers can trick users into authorizing malicious applications

**Solution:** Use state parameter and validate it

```javascript
// Generate state parameter
const state = generateRandomString();
session.state = state;

// Validate state parameter
if (req.query.state !== session.state) {
  return res.status(400).send('Invalid state parameter');
}
```

### 3. **Token Storage Vulnerabilities**

**Problem:** Storing tokens in localStorage or insecure cookies

**Solution:** Use secure storage mechanisms

```javascript
// Secure cookie storage
res.cookie('access_token', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict',
  maxAge: 15 * 60 * 1000 // 15 minutes
});

// Secure session storage
req.session.accessToken = token;
```

### 4. **Scope Escalation**

**Problem:** Clients requesting more scopes than needed

**Solution:** Implement scope validation and user consent

```javascript
const requestedScopes = req.query.scope.split(' ');
const allowedScopes = ['openid', 'profile', 'email'];

const validScopes = requestedScopes.filter(scope =>
    allowedScopes.includes(scope)
);

if (validScopes.length !== requestedScopes.length) {
  return res.status(400).send('Invalid scope requested');
}
```

## Tools and Libraries

### 1. **OAuth 2.0 Libraries**

**Node.js:**

- `passport-oauth2`: OAuth 2.0 strategy for Passport
- `simple-oauth2`: Simple OAuth 2.0 client
- `oauth4webapi`: Modern OAuth 2.0 client

**Python:**

- `authlib`: Comprehensive OAuth and OIDC library
- `requests-oauthlib`: OAuth 1.0 and 2.0 for requests
- `python-jose`: JWT handling for OIDC

**Java:**

- Spring Security OAuth: OAuth 2.0 support
- Nimbus JOSE+JWT: JWT handling
- Pac4j: Authentication and authorization library

**C#/.NET:**

- `Microsoft.AspNetCore.Authentication.OpenIdConnect`
- `IdentityModel`: OAuth 2.0 and OIDC client
- `System.IdentityModel.Tokens.Jwt`: JWT handling

### 2. **Testing Tools**

- **Postman**: API testing with OAuth 2.0 support
- **OAuth 2.0 Playground**: Interactive OAuth testing
- **JWT.io**: JWT debugging and validation
- **OAuth 2.0 Test Suite**: Automated security testing

### 3. **Development Tools**

- **Auth0**: OAuth 2.0 and OIDC provider
- **Keycloak**: Open-source identity and access management
- **Okta**: Enterprise identity platform
- **Firebase Auth**: Google's authentication service

## Conclusion

OAuth 2.0 and OpenID Connect provide robust, secure, and standardized ways to handle authentication
and authorization in modern applications. By following the best practices outlined in this guide,
developers can implement secure authentication systems that protect both users and applications.

**Key Takeaways:**

1. **Use Authorization Code Flow** for web applications
2. **Implement PKCE** for public clients
3. **Validate all tokens** on the server side
4. **Store tokens securely** using appropriate mechanisms
5. **Follow security best practices** to prevent common vulnerabilities
6. **Use OIDC** when you need user identity information
7. **Implement proper error handling** and logging
8. **Regularly review and update** security measures

Remember that security is an ongoing process, and staying informed about the latest threats and best
practices is crucial for maintaining secure OAuth 2.0 and OIDC implementations.
