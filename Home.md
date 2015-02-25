## How it works

Secret data you provide is encrypted, and only people you specify can access it. Sharelock restricts access to your data by requiring recipients to authenticate themselves first. If their proven identity matches your expectations, they will be allowed to view the secret. You can specify people who should have access to the secret using their e-mail addresses, e-mail address domains, or Twitter handles. 

![sharelockio](https://cloud.githubusercontent.com/assets/822369/6075236/8073adf6-ad83-11e4-9dc0-9006b61c3934.png)

Sharelock service does not store your secret data. Your secret data is stored in an encrypted and signed form in a rather long URL we call a "sharelock". It is up to you how you transmit the sharelock URL to your intended recipients: you can send it by e-mail, tweet it, or publish it in The New York Times. Regardless who intercepts your sharelock in transmission, only the people you specified will be able to access the secret within it. 


# Getting started

## Hosting

Sharelock service is an HTTP service that exposes both web APIs and the web application UI. The service sets up an HTTP listener. To make sure your deployment is secure, you must host it behind an HTTPS terminating proxy, for example in Heroku or Windows Azure Web Sites. 

## Auth0 application

Sharelock service uses [Auth0](https://auth0.com) to authenticate recipients of a sharelock. You must create an account with Auth0, and register an application. In that process, you will be given three pieces of information, which you will need to host your own Sharelock service:

* Auth0 domain (e.g. *example27.auth0.com*)  
* Auth0 client ID
* Auth0 client secret

Furthermore, your application must be configured to enable the following *connections* (connections represent identity providers):

* Facebook  
* GitHub  
* Google-OAuth2  
* Twitter  
* Windows Live  
* Yahoo  

Lastly, in your application settings at Auth0, remember to specify the callback URL of your Sharelock service. The specific callback URL depends on where you have decided to host your service. For example, it may look like this:

```
http://localhost:3000/callback,https://sharelock.io/callback
```

NOTE: the *http://localhost:3000/callback* URL is useful during development. 

## 

Starting the service is simple: 

```
node server.js
```

The service requires several environment variables to be set (all of them are required unless specified otherwise):


| Environment variable | Value |
| ------------- | ----------- |
| AUTH0_DOMAIN | The Auth0 domain obtained from [Auth0](https://auth0.com) |
| AUTH0_CLIENT_ID | The Auth0 client ID obtained from [Auth0](https://auth0.com) |
| AUTH0_CLIENT_SECRET| The Auth0 client secret obtained from [Auth0](https://auth0.com) |
| AUTH0_CALLBACK | The callback URL specified when creating the [Auth0](https://auth0.com) application. In production it may look like *https://sharelock.io/callback*. In development it may be *http://localhost:3000/callback*. |
| SIGNATURE_KEY_1 | A secret key that will be used to sign your sharelock. You can generate one with `openssl rand 32 -hex`. |
| ENCRYPTION_KEY_1 | A secret key that will be used to encrypt your sharelock. You can generate one with `openssl rand 32 -hex`. |
| CURRENT_KEY | Specify `1`. More on this below. |
| COOKIE_SECRET | A secret key to protect web application UI HTTP cookies with. You can generate one with `openssl rand 32 -hex`. |
| FORCE_HTTPS | Specify 1 to redirect all HTTP requests to corresponding HTTPS endpoints. It is recommended to use `1` for production deployment. The value of `0` is useful during development when there is no HTTPS terminating proxy. |
| CURRENT_KEY | Specify `1`. More on this below. |
| GA_PROPERTY_ID | Optional. Specify the Google Analytics property ID to hook up Google Analytics to the web UI. |
| PORT | Optional. Specify the port to listen on. Specific hosting environments like Heroku or Windows Azure Web Sites will provide this envronment property for you. If not specified (e.g. during development), the default value is *3000*. |

During development, you can provide all these environment variables through the `.env` file at the root of your enlistment.

## Key management

Sharelocks created by the service are signed and encrypted with a pair of keys (one for signing, another for encryption). These need to be 256 bit keys and are best generated with:

```
openssl rand 32 -hex
```

The Sharelock service supports decrypting and signature verification with any number of historical signing and encryption key pairs. This allows you to revoke a specific key pair in case it had been compromised. Each signing and encryption key pair is specified with a pair of environment variables: 

```
SIGNATURE_KEY_{N}={hex_encoded_signature_key}
ENCRYPTION_KEY_{N}={hex_encoded_encruption_key}
```

Where *{N}* is an arbitrary literal (but we recommend using integers for brevity).

At any point in time, Sharelock service uses *one* of these key pairs to sign and encrypt newly created sharelocks. This key pair is specified with the environment variable: 

```
CURRENT_KEY={N}
```
