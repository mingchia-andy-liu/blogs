---
title: "Public Key Infrastructure Part 3 - Certificate Authority"
date: 2022-01-09T13:19:53-08:00
description: PKI serie part 3. Certificate authority.
slug: pki-3
tags:
  - pki
  - cryptography
showTableOfContents: true
coverCaption: Photo by [Angelo Abear](https://unsplash.com/@angeloabear) on [Unsplash](https://unsplash.com/)
---

_SERIES:_
* [PKI - Part 1]({{< ref "/posts/2021-09-06-pki-1-cryptography.md" >}})
* [PKI - Part 2]({{< ref "/posts/2021-09-18-pki-2-certificate.md" >}})

A certificate authority (CA) is an authority that issues certificates. It is a crucial part of PKI. Knowing the certificate is only half of the story.

A CA is the entity that can create and issue certificates. A "trusted" CA is an entity that you trust. PKI is all built on trust. If you trust the issuer of this certificate, then you trust the certificate. A CA is also a certificate itself which contains the public key for other to verify its issued certs.

From [last blog]({{< ref "/posts/2021-09-18-pki-2-certificate.md" >}}), users verify a certificate by using issuer's public key. But how can we trust the issuer's certificate (public key). You trust it by looking at its issuer. And so on. At some point, the chain will end and encounter a root certificate authority. 


A root CA is self-signed. This means that it is its own issuer. In PKI, any root CA is self-signed. Here is where is "trust chain" comes in. We/Computers over the internet have a list over trusted root CAs. These CAs are public CA that establish themselves as trusted CA. Their CA's private are most likely air-gapped and stored in a secure place. 


List of CA in your system.
* MacOS: in the keychain application.
* Linux: `/etc/ssl/certs`, `/etc/ssl/certs/ca-bundle`, etc.



## Intermediate CA

If some root CAs are trusted by default, it is dangerous if their private keys are leaked/stolen. Then they can impersonate as a trusted entity. So root CAs create many intermediate CAs. So they are **not** trusted by default. Any website is usually the third layer from the root CA.

And the servers should always send the certificate and any intermediate certificate that can build up to the root CA. 

{{< figure
    src="certificate-chain.png"
    alt="A screenshot of certificate chain."
>}}

Therefore, user can verify the entity certificate by the additional certificate from server, then verify intermediate one from its own system. If they all verified, then the trust chain is established. 


## Internal PKI

Internal PKI is people run on their own. Usually it is only within the network. Company IT team manages the CA and issue certificate for any service that's needed. Why do we need it? Web/Internet PKI is not designed nor needed for internal network case. If the services are within a network firewall, there is no point to get a public root CA to issue certificate(s) for internal usage. And it can also allow entities across public internet establish encrypted communication channels. In addition, there is cost for certificates from trusted public CA. 


* [Apple\'s PKI](https://www.apple.com/certificateauthority/)
* [Microsoft\'s trusted root program](https://docs.microsoft.com/en-us/security/trusted-root/participants-list)
* [Mozlia\'s trusted root program](https://wiki.mozilla.org/CA)
* Chrome does not have its own program. [It uses Mozlia\'s list](https://chromium.googlesource.com/chromiumos/docs/+/HEAD/ca_certs.md).

---

This is the final blog of the 3 parts PKI serie. I have learnt a lot by writing this serie and quite proud of what I've written. PKI is a really interesting and has much more than what I've covered. But for me, I think this is it. 🤠  till next time.