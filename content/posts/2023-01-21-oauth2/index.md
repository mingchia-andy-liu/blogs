---
title: "Basic of OAuth 2.0"
date: 2023-01-21T00:00:00
description: What is OAuth and how does it work?
tags:
  - OAuth
showTableOfContents: true
showCover: false
---

Recently, I completed Aaron Parecki's course on OAuth to learn how they work. I was fascinated by it. I started making notes to make sure I understand it better. I even gave a talk in my team at my work. It felt really good to learn how these protocol/framework work behind the scene. Here are my notes on OAuth.

## Motivation

{{< figure
    src="facebook.jpg"
    alt="A screenshot of Facebook asking user their email credentials."
>}}

Third applications used to ask user’s credentials of another application in order to get information about the user and their data. Because there was no way to do it. They (third party) promise they don’t store the credentials, and will handle them with utmost security around them. Can you really trust them?

ie, facebook will ask your Gmail credentials to find out whom of your contacts is also on Facebook. 

This is terrible way to even if you trust Facebook to not store the credentials

Sadly, this still exist with a lot of financial applications, ie Mint or Plaid. They ask for your 🏦 bank credentials to get your transaction history. It sounds crazy because they can access your account based on your behalf. It can have financial impact if the data is leaked. Personally, I would never trust another third party to handle any of my financial credentials. 

{{< figure
    src="mint.png"
    alt="A screenshot of Mint's FAQ page. First question is answering why they need customer's bank credentials."
>}}


## High level concepts

OAuth provides a secure way for a user to grant access to their resources on one site (the "resource server"), to another site (the "client"), without having to share their credentials. Instead, the user grants access by authenticating with the resource server and providing the client with a token that can be used to access the resources. It allows an application to request permission from a user to access their information on another application, without giving away their credentials.

{{< figure
    src="high-level.png"
    alt="A sequence diagram of how OAuth works."
>}}

1. `User` want to use the `Application`. But the `Application` needs some photos from `Google` in order to work.
2. `Application` asks `User` to login to Google and share the photo with them
3. `User` is redirected to Google 
4. `Google` asks about sharing photo permission with `Application`.
5. If permission is granted, the `Application` somehow has all the photos from `Google`.

### Terminology

**Roles**

1. User
2. Device (User agent)
3. Application (Client)
4. API (Resource server)
5. Authorization Server

These are the terms used in OAuth. Device can be any devices used by Users. This can be a browser, a phone, or even a TV. The API (Resource server) would be the any service that holds the data that the Application wants. Finally, the Authorization server, the server that will authorize the user to give permission to the Application.

**Application Types**

There are 2 types of Applications. Public clients and Confidential clients. Confidential clients are applications that are able to securely communicate with authorization servers, and keep any deployment secret safe. Public clients cannot keep secrets secure, such as applications running in a browser or on a mobile device.

- Public Client
  - Single Page Application
  - Mobile applications (extract string from binary files, or a string match to find the secret)
  - Internet of Things (ie. Smart TV)
- Confidential Client

**Redirect URIs**

It is a URI that is specified by the client when it requests access to a user's resources. Also known as "callback URI". The authorization Server will only redirect users to a registered URI. This is to prevent attackers tricking users to a malicious web sites. In addition, any http URI needs to be served with via HTTPS. This prevent tokens from being intercepted during the authorization process. Some authorization server will allow HTTP for `localhost` URI for development. Native applications can also register protocols for redirect. For example, `google://redirect-uri`. Then phone OS will redirect to the application if it sees any URI with this protocol.

**Client ID and Secret**

After registering your app, you will receive a client ID and optionally a client secret. Client ID and client secret are used to authenticate the client. The client ID is public information, and is used to build login URLs, or included in Javascript source code on a page. The client secret must be kept confidential. If a deployed app cannot keep the secret confidential, such as single-page Javascript apps or native apps, then the secret is not used, and ideally the service shouldn't issue a secret to these types of apps in the first place.

## OAuth Grant Types

OAuth specifies several grant types for different use cases, as well as a framework for creating new grant types. In OAuth 2.0, the term "grant type" refers to the way an application gets an access token. OAuth 2.0 defines several grant types, including the authorization code flow. OAuth 2.0 extensions can also define new grant types.


### Authorization Code

This type is for Applications running on a web server. Applications running on a web server where it can securely store the secret that is not available to the public. This means the application is able to use its client secret when communicating with the authorization server, which can help avoid many attack vectors.

{{< figure
    src="auth-code.jpg"
    alt="A sequence diagram of how authorization code work in OAuth."
>}}

When the user clicks on the link for redirect (step 2 and 3), the link will look like this.
{{< figure
    src="auth-code-2-3.png"
    alt="The URI the application will take the users to."
>}}

The URI the Auth server will redirect the user to. (step 4)
```bash
https://example.com/redirect?
  code=AUTH_CODE&
  state=XXXXXX

https://example.com/redirect?
  error=login_failed&
  state=XXXXXX
```

The URI the Application (web server) will request once it receives the callback from the users (step 5 and 6)
```bash
POST https://authorization-server.com/token
  grant_type=authorization_code
  code=AUTH_CODE
  redirect_uri=https://example.com/redirect
  client_id=CLIENT_ID
  client_secret=CLIENT_SECRET
  code_verifier=CODE_VERIFIER
  code=CODE
```

```json
{
    "token_type": "Bearer",
    "expires_in": 3600,
    "access_token": "dfs@LIQ@JDLKSJADIH",
    "scope": "photos"
}
```

### Device Code
The Device Code grant type is an extension of Authorization Code grant type. It separates the device that is getting the access token from the device that user device use to login. It is typically used by devices such as smart TVs, game consoles, or other Internet-connected devices that do not have a browser or are difficult to use with a browser. Those device will display a device code, and the user goes to the verification URI on another device and enters the user code. The flow is similar to Authorization Code with some minor differences.

{{< figure
    src="device-flow.jpg"
    alt="The URI the application will take the users to."
>}}


### Client Credentials

THe simplest grant type. There is no user involved. No user user means no browser, and then no redirect. Applications take its own credentials and exchange for access token. When access token is returned, there is no refresh token. Because refresh token is for applications to get a new access token without user involvement. If user already does not involve. There is no need for a refresh token

Why do we need this case? It is usually done for service-to-service communication. This is done when the application is **not** trying to access data on behalf of a user, it’s trying to access on behalf of itself. The application is **not** acting on behalf of any user.


```bash
POST https://authorization-server.com/token
  grant_type=client_credentials
  client_id=CLIENT_ID
  client_secret=CLIENT_SECRET
  scope=photos
```

### Refresh Token

A refresh token is a token that is issued by the authorization server along with an access token. The refresh token can be used by the client to obtain a new access token, once the original access token has expired. This allows the client to continue to access the user's resources without having to prompt the user for authorization again.

```bash
POST https:/authorization-server.com/token
  grant_type=authorization_code
  redirect_uri=https://example.com/redirect
  client_id=CLIENT_ID
  client_secret=CLIENT_SECRET
  code_verifier=CODE_VERIFIER
  code=CODE

// { refresh_token: xxx }

POST https:/authorization-server.com/token
    grant_type=refresh_token
    client_id=CLIENT_ID
    refresh_token=xxx
```


### Legacy 🚨 Implicit Flow

The Implicit flow is a simplified version of Authorization Code that is typically used by client-side applications, ie SPA. Unlike the Authorization Code flow, the Implicit flow does not involve the exchange of an authorization code for an access token. Instead, the access token is returned to the client immediately in the URI, along with an optional ID token. It is consider a security risk since there is no way to verify the user will receive it securely.  

### Legacy 🚨 Password Grant

OAuth Password Grant Type is a way to get an access token given a username and password. The user enter their credentials to the third party applications and then the applications will request resource on behalf of the user. Sounds familiar? This is the exactly the same flow before OAuth framework. This is considered a security risk since users should not, again should NOT, enter their credentials to third party applications. 

---

### PKCE
Proof Key for Code Exchange (https://tools.ietf.org/html/rfc7636). PKCE is an extension to the Authorization Code to prevent CSRF and authorization code injection attacks. PKCE is not a replacement for a client secret, and PKCE is recommended even if a client is using a client secret.

#### OpenID Connect
An extension of OAuth. It is about application learning information about user. OAuth is about application getting access to the API. Simply by including a `openid` scope in the request, the authorization server will include a id token for exchanging the user information. 

- is always a JWT
- read by the application (client). It is not communicated/transmitted to somewhere else.
- Access token is read by the API, should be opaque to application


## Conclusion

This was a really fun and interesting topic for me to learn. OAuth is everywhere and I happen to use it all the time. It never occurred to me how they work. Thanks for this, I have a way better understanding how OAuth work and what are the differences between each grant type. 


## Resources
* [The Nuts and Bolts of OAuth](https://www.udemy.com/course/oauth-2-simplified/?referralCode=B04F59AED67B8DA74FA7) by [Aaron Parecki](https://aaronparecki.com/)
