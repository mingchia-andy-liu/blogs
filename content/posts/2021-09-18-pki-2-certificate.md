---
title: "Public Key Infrastructure Part 2 - Certificate"
date: 2021-09-18T11:01:03-07:00
description: PKI serie part 2. What is a certificate.
images:
  - images/passport.jpg
slug: pki-2
tags:
  - pki
  - cryptography
type: post
---

{{< figure
    src="/images/passport.jpg"
    alt="A calendar."
>}}
Photo by [Kyrie kim](https://unsplash.com/@convertkit) on [Unsplash](https://unsplash.com/)

## Introduction


[Last blog]({{< ref "/posts/2021-09-06-pki-1-cryptography.md" >}}), I described how we can use cryptography to securely communicate between 2 people. In this blog, I want to discuss how we can use the same pattern and apply it to computers. How do 1 computer securely talk to another computer over the internet?

To recap the cryptography blog, if 2 people want to securely communicate to each other, they can achieve with asymmetric cryptography. Each person sign with their private key with a fix message and the recipient's public key. It ensures the message isn't altered in transit and only the recipient can decode the message.

&nbsp;

### Message Digest & Digital Signature

There are actual terms for what the above usage are. They are called "**message digest**" and "**digital signature**". I didn't use them to avoid confusion. Let's see what they are

#### Integrity

> 💡 A message digest is a fixed size numeric representation of the contents of a message, computed by a hash function.

The digests are designed to protect the "integrity" of the message content. Any altercation of any part of the message will change the result of hash function. A hash function is a algorithm that takes an arbitrary length input data to an output of fixed length. The output is message digest, also known hash values, checksums, fingerprints, or simply hashes.

There are a few requirements of a good hash function:
1. Deterministic: always generate the same hash given the same input.
1. One-way: it should be computationally infeasible to take the output of a hash function and reconstruct its input.
1. Collision resistant: it should be computationally infeasible for 2 messages to have the same hash.

**NOTE**: the message content are usually in plaintext along with the message digest.

#### Authentication

> 💡 A Message Authentication Code (MAC) is the similar to a message digests but the hash function also takes another shared secret parameter.

A valid MAC can be generated only by the entities who know the shared secrets. The hash from these type of hash function is known as a MAC. With MAC, both entities need to know the shared secret. Assuming only trusted entities know the shared secret, then recipient can trust the message.


> 💡 A digital signature is an encrypted message digest using asymmetric key pairs.

The digital signatures are designed to prove the authenticity of the message. Given the property of asymmetric cryptography, the recipient know only the entity with private key can encrypt the digest to the matching signature.

&nbsp;

### Certificates

A certificate is like a passport for computers and servers. It contains the name of the identity, the issuer, the valid date range, what can the identity do, etc.

Some similarities:
* subject -> the passport holder's name.
* issuer -> the government.
* signature -> the passport id.


When a client connects to a server, the server present a certificate for the client to verify that it is what it claims to be. 


**HOW? use the public key**

The certificate contains issuer's information. The client can use the issuer's public key to verify the signature. Basically to decryt signature using public key. If it can be decrypted and match the information, then the client can TRUST the certificate is issued by the issuer's private key.


My blog's certificate:

{{< figure
    src="/images/blog-certificate.png"
    alt="A screenshot of the blog.aliu.dev certificate in a terminal window."
>}}

It has a lot stuff. The overall gist is that it is a certificate for "blog.aliu.dev" and issued by Cloudflare. It has the signature at the bottom signed by Cloudflare's private. And if you trust Cloudflare, then you can trust me, "blog.aliu.dev".


Obviously it has way more stuff than I described in above. It also has a public key for my "blog.aliu.dev" domain. If my domain ever signs a certificate, other people can use this public key for verification.

&nbsp;

_UPDATE:_
* [PKI - Part 1]({{< ref "/posts/2021-09-06-pki-1-cryptography.md" >}})
* [PKI - Part 3]({{< ref "/posts/2022-01-09-pki-3-ca.md" >}})