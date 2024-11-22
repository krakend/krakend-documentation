---
lastmod: 2024-11-20
date: 2024-11-20
linktitle: Intro to AuthZ and AuthN
title: Authorization and Authentication (authZ + authN)
description: Overview of the options available to integrate authentication and authorization services with KrakenD API Gateway for secure access control to your APIs
weight: -100
menu:
  community_current:
    parent: "080 Authentication & Authorization"
meta:
  #since:
  #source: https://github.com/krakend/krakend-jose
  #namespace:
  #- auth/validator
  #scope:
  #- endpoint
  #log_prefix:
  #- "[ENDPOINT: /foo][JWTValidator]"
---
Authorization and authentication are fundamental to managing access to your APIs. While authentication verifies the identity of a user or service (e.g., *is this a valid user/pass?*), the authorization checks their access rights (e.g., *Should John see this resource?*).

KrakenD offers a versatile set of features enabling you to control who can access your APIs and what actions they are authorized to perform. Whether your use case involves simple client credentials or complex multi-provider authentication scenarios, KrakenD ensures your system is both secure and scalable.

**This introduction provides an overview of KrakenD's key authorization and authentication features.**

## How does auth change when you have a gateway?
When you work with a gateway, you add a piece of software **in the middle of the client-server communication**. Because it is in the middle, you have the opportunity to handle the authentication between the two sides separately and independently, adding more options to your setup.

![auth-overview.mmd diagram](/images/documentation/diagrams/auth-overview.mmd.svg)

The possible scenarios you can have are:

| Auth Scenarios |   |
|---|---|
| **No auth**: Public endpoints. No authentication is needed to use them. | ![auth-overview-no-auth.mmd diagram](/images/documentation/diagrams/auth-overview-no-auth.mmd.svg) |
| **Delegated auth**: The gateway does nothing other than forwarding the auth headers to the backend | ![auth-overview-forward.mmd diagram](/images/documentation/diagrams/auth-overview-forward.mmd.svg) |
| **Clients auth**: The client is authenticated, but the communication between the gateway and the service is open | ![auth-overview-client.mmd diagram](/images/documentation/diagrams/auth-overview-client.mmd.svg) |
| **Clients and gateway auth**: Both the client and the gateway use authentication, they can be completely different. | ![auth-overview-all.mmd diagram](/images/documentation/diagrams/auth-overview-all.mmd.svg) |
| **Gateway auth**: Public access for clients, but auth between the gateway and the services | ![auth-overview-backend.mmd diagram](/images/documentation/diagrams/auth-overview-backend.mmd.svg) |


It is important to realize that since Authorization and Authentication are independent of each other, you can **mix authentication methods** and add new options on a platform that does not support them. You can enable **machine-to-machine** between the gateway and the services and other methods with the clients. A few use cases of different auth implementations could be:

- Your platform does not have any authentication today, but you want to open it protected to the world
- You have a classic monolith with user/password/session, and you want to modernize it with JWT
- Your internal service requires a hardcoded user and password, but you would like to use granular JWT tokens to publish its functionality.
- You want to implement different authentication methods between the users and the gateway (e.g, publish with API keys, connect to services with mTLS)
- etc.

You can make many possible combinations, and the independence of these two sides gives you a lot of power. KrakenD provides multiple tools to get this job done.

## Authentication (AuthN)
Authentication takes care of **recognizing** legitimate users. Generally, you will delegate the authentication to an Identity Provider (for short, **IdP**). The IdP can be your existing monolithic application that checks a user and password or a more fancy IdP using industry standards, whether it's SaaS or self-hosted (e.g., Auth0, Azure AD, Google Firebase, Keycloak...). The IdP takes care of the authentication methods you want to support. E.g., Password-based User Authentication, One-time Password (OTP), Single Sign-On (SSO), Biometric/Passwordless Authentication, etc. The IdP will then generate a signed token that KrakenD can use to authorize the user.

Aside from tokens, you can also use a certificate to authenticate to your gateway or the gateway to connect to a backend service. Both scenarios are supported on [Mutual TLS (mTLS)](/docs/authorization/mutual-authentication/).

In addition, the Enterprise Edition adds other methods like [Google Cloud service-to-service](/docs/enterprise/authentication/gcloud/) or more "classic" (and less secure) authentication methods, such as [API-key authentication](/docs/enterprise/authentication/api-keys/), [Basic Authentication](/docs/enterprise/authentication/basic-authentication/), or even [Microsoft's NTLM (NT Lan Manager) Authentication](/docs/enterprise/authentication/ntlm/).

## Authorization (AuthZ)
Authorization determines what authenticated users or systems are **permitted** to do. KrakenD provides flexible options for fine-grained access control. The most common method is Token-based authorization (see below). In addition to JWT, you can also use [OAuth2 Client Credentials](/docs/enterprise/authorization/client-credentials/) for machine-to-machine communication. KrakenD verifies the client's credentials with this flow and enforces scope-based access controls.

KrakenD's ability to integrate with multiple identity providers simultaneously ({{< badge >}}Enterprise{{< /badge >}}) offers great flexibility. Whether catering to different user groups, migrating, multi-tenant, or implementing complex organizational structures, KrakenD adapts to your needs.

## Token-based authorization
Today, most applications use JSON Web Tokens (JWTs), a widely used method for securing APIs. They are our first recommended way to limit the access of end-users to your API. KrakenD does not issue the JWT tokens (as we said, this is the job of the identity provider) but validates them to ensure their authenticity and can check claims, such as audience, issuer, and expiration, to name a few examples.

Read the [JWT Overview](/docs/authorization/jwt-overview/) for an introduction to token-based authorization in KrakenD. When working with tokens, there are different things you can do:

- [JWT Signing](/docs/authorization/jwt-signing/): If you don't yet have an Identity Provider, you can make your classic login controller return a JSON response emulating a token and let KrakenD sign it with secure cryptography. This enables your system to provide trusted, verifiable tokens for authenticated clients even when you have not transitioned your users to an identity service. 
- [JWT Validation](/docs/authorization/jwt-validation/): This is the core functionality, where a user sends a token generated by an Identity Provider, and KrakenD checks for its validity and any required permissions to access the resource.
- [JWK Key Caching](/docs/authorization/jwk-caching/): As identity providers don't change their signing private keys very often, it is recommended that you cache the public key in the gateway to avoid unnecessary internal calls to fetch public keys, improving performance without sacrificing security.

Apart from that, you can also apply business rules to incoming tokens using [CEL](/docs/endpoints/common-expression-language-cel/) or [Security Policies](/docs/enterprise/security-policies/), or [revoke valid tokens](/docs/authorization/revoking-tokens/) to handle scenarios like compromised credentials or access revocation. The [Revoke Server](/docs/enterprise/authentication/revoke-server/) centralizes and manages token blocklists, ensuring invalidated tokens cannot be reused.

If you have [multiple identity providers](/docs/enterprise/authentication/multiple-identity-providers/) ({{< badge >}}Enterprise{{< /badge >}} feature), you can let KrakenD work with all of them simultaneously.

## Role-Based Access Control (RBAC) and Attribute-Based Access Control (ABAC)
It is important to know who can access a resource and decide if a user has a specific role or attribute that grants them access. The different components of authz in KrakenD allow you to set custom rules validated on runtime during **requests, responses, and token validation** itself.

From issuer, audience, or [role validation](/docs/authorization/jwt-validation/#roles) in JWT tokens to [custom policies](/docs/enterprise/security-policies/) with Security Policies ({{< badge >}}Enterprise{{< /badge >}}), you can control in the gateway itself who sees what and offload this responsibility from your services.

The policies allow you to implement all sorts of validations and user access control, from parameter compliance, to **RBAC** (Role-Based Access Control) and **ABAC** (Attribute-Based Access Control) strategies.

## Integration with External Identity Providers

Because KrakenD adopted security standards, you can use it **with any provider**: we still need to find a single major provider that doesn't follow the JSON Web Encryption (RFC 7516), JSON Web Signature (RFC 7515), or JSON Web Token (RFC 7519) specifications.

That being said, it means that you have native integration to validate tokens, manage user sessions, and enforce rules with SaaS providers and self-hosted tools. Some examples are:

- Keycloak
- Gluu
- Auth0
- Okta
- Google
- Azure AD
- OneLogin
- PingIdentity
- and many, many more.

## Do I need to expose my IdP?
A common question we get is whether **it is necessary to publish my IDP to the Internet**? Not really.

If you have a self-hosted IdP, you can expose it to the Internet, but you can also place it behind the gateway and publicly enable the routes that users will use. For instance, you could have a setup like this:

![auth-overview-architecture.mmd diagram](/images/documentation/diagrams/auth-overview-architecture.mmd.svg)

In this scenario, the `/login` endpoint passes through KrakenD, allowing you to set rate limiting or any other protection mechanism you consider to secure even more your IdP. The client stores locally the token for the next interactions, and send the bearer to access resources through KrakenD. The IdP is not contacted again until the token is expired (when the client will refresh it).