---
layout: post
title: "Cryptographic Identities for the World Wide Web"
date: 2017-12-30
excerpt: "Centrally controlled logical identities put control in hands of web services. Users need to maintain full control of their identities, especially those that affect their social and economic credibility. Cryptographic identities can help regain this control."
tags: [web 3.0, crypto, oauth, authentication, keygen, webcrypto]
comments: true
---

## Overview
Proof of Identity is a primary problem for any network protocol and the World Wide Web (HTTP) is no different. Introduction of the state management RFC in 1997<sup><a href="https://tools.ietf.org/html/rfc2109" target="_blank">[1]</a></sup> led to common use of Cookie headers to maintain a state of identity in HTTP sessions. These identities have been dominantly powered by logical authentication mechanisms. The world wide web has progressed far ahead in its adoption of identities that are authenticated by trusted delegates. These identities still aren't universal over the world wide web. While these identities inter-operate, there's isn't a single central authority that controls these identities. To remain aligned with the philosophy of the open web, there shouldn't be such a centralized entity. Here we discuss the possibility of extending cryptographic identities in the way they are being used in Distributed Ledger Technologies (popularly known as Blockchains), while keeping in mind the expected operational simplicity for an Internet user. Such identities would consistently keep the user in control and can be utilized by modern age applications to extend services with end-to-end encryption.

Considering the deprecation of the proposed `keygen` element as part of the HTML5 specification<sup><a href="https://developer.mozilla.org/en-US/docs/Web/HTML/Element/keygen#Browser_compatibility">[2]</a></sup>, we discuss a ground up approach for cryptographic signatures for authentication on the web.


## Cryptographic signatures for the Web
As opposed to conventional authentication that relies on username and password combinations, cryptographic authentication for the web can rely on messages signed by users with their secrets. The signing mechanism needs to fullfil the following two requirements.

1. Allow users to sign challenges posed by webpages to prove their identity.
2. Allow users to sign HTTP transactions that may occur through forms or XHR requests as a proof of authenticity.

### Approach
1. Users will first have to generate their keys using standardized algorithms asymmetric cryptography.
2. Web pages can challenge the user to sign a message with their secret/private key.
3. Web services receive the signed challenge, and parse the signature for a public key, and verify the signature against the same public key.

Challenges can be used by web pages for registertion, login or even transactions. Public keys can be substituted as usernames for services that do not require user friendly naming conventions. Services like social platforms can map usernames with public keys for a friendly authentication experience.

### Browser based interface
The standard HTML input field can be used as a base to build the signature component.

```
<input name="sign" type="signature" challenge="Hello Doctor Watson" />
```
On user action, the `signature` field generates a signature based on the form inputs filled by the user. The `challenge` attribute can be set by javascript or markup with information that needs to be signed.

### Verification by services
Web services can receive HTTP form requests or XHR requests to parse the signature element in plain text for verification.

```
sendTo=omtalk
amount=3.14159
sign=-----BEGIN PGP SIGNATURE-----\nVersion: OpenPGP.js v2.5.10\nComment:https://openpgpjs.org\n\nwsFcBAEBCAAQBQJaSPPeCRCYcC33miZm0gAAiksQANEZS3XjrWx2Ah7v8aqs\nZXl7zURE8HPQn/9cv7M2/2bV82+1Kpnzr1\nnPhigQS4w2a7o/29jcMyrW0Bh\nI8N0R76sF3jVKiEY4YY5WJHu4HRJWi\nA9roUJRYPpvJgPAsCQgDb8HZF0v4x\nus/KJrT38xKQiCzqYbQCacqkWv/JuHCS4UA2D0RvCNy/xJeb750xJE2quNv9\nhkjffwnH49R1tzOm9zDkXtoQYxPBmrKCW8iKDbRIDSTs15kVIlBsmUEISKQN\nRH6B30rZZXulsY0dJjE6ovzsuGbR/Em8TefMu5pOzE4jvflqOu3hsVIkx1ZE\nd1/YuuJmbBy8pFWaj5efnqpSgCcCSSCAquAfXu2arh7LMD2yIO+mYrhA2gEv\n+8Yd6CrkgTRzKCE8D22qylbmlppJRPIfAutspvKvZrETcfJTpzONVUiFCjxt\nMnU7t9mn+frItQ4fBrHqfJyfNfAppIWqqnPZvr0iUsO4v2+JzlkNx/OtFjr6\nx6msgK87MzGgVzNPoLAWo054IDIWMO7YVrg2sB6NpsbtDhXAmXytloLfjcCx\nBWrl9Cpm2SWkRADg8IAB8FPnqbARdwJtlSQgG6A5v7FXwKEwA41uVkAkVyZs\nuoDEY3yri1FZF1JPlDnKT8CrKYQijkj2BXHcvDBo66hvxbalNQZvzOb02W1r\nHlBb\n=b++B\n-----END PGP SIGNATURE-----
```
The example uses PGP signature considering its widespread use by email messaging systems. The signature can be substituted by RSA, DSA or EC signatures as well.

### Third-party authentication
Host services can procure signed contracts from users to obtain or delegate access to third-party services. Contracts shall be signed within scope of the host service. These contracts can then be forwarded to the third-party service for token generation.

Contracts are nothing but challenges that comply with a previously agreed upon structure. For example:

```
host: 0x8.in
delegate: twitter.com
token: -----BEGIN PGP SIGNATURE-----\nVersion: OpenPGP.js v2.5.10\nComment:https://openpgpjs.org\n\nwsFcBAEBCAAQB.....
```

## Security measures
Transactions signed with cryptographic signatures can seem permanent in nature.

1. Signed transactions need to be secured within the scope of a single host at all times.
2. Signed transactions should not be replayable even within the scope of a single host.

Additional two parameters are prepended to all information being signed by the user.

```
host: 0x8.in
nonce: AzkiyxbYbnskb68B-1514731815897
```

- host: The root domain used by HTTP requests.
- nonce: A random challenge previously agreed upon by the browser and the host. The challenge can have the same expiry as the browser session.

## Conclusion
The structure of identity and content ownership on the World Wide Web can be improved with the adoption of cryptographic proofs as a standard. The approach has proved fruitful for email signatures with PGP.

Moreover, cryptographic identities can provide the necessary foundation for end-to-end encryption (E2EE) required to ensure security and privacy of users of the world wide web. Further, the experimental Web Crypto API<sup><a href="https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API">[3]</a></sup> could be leveraged to standardize cryptographic algorithms paired with a browser based key store interface.

## References
1. [HTTP State Management Mechanism (rfc2109)](https://tools.ietf.org/html/rfc2109)
2. [Keygen browser compatibility](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/keygen#Browser_compatibility)
3. [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API)