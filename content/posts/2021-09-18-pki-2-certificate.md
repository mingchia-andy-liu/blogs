---
title: "Public Key Infrastructure Part 2 - Certificate"
date: 2021-09-18T11:01:03-07:00
description: PKI serie part 1. Explain what is a encryption and decryption, and how to use them.
images:
  - images/lock.jpeg
slug: pki-2
tags:
  - pki
  - cryptography
type: post
---

## Introduction

Last blog, I described how we can use cryptography to securely communicate between 2 people. In this blog, I want to discuss how we can use the same pattern and apply it to computers. How do 1 computer securely talk to another computer over the internet?

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
1. Collision resistant: it should be computationally infeasible for 2 messages have the same hash.

**NOTE**: the message content are usually in plaintext along with the message digest.

#### Authentication

> 💡 A Message Authentication Code (MAC) is the similar to a message digests but the hash function also takes another shared secret parameter.

A valid MAC can be generated only by the entities who know the shared secrets. The hash from these type of hash function is known as a MAC. With MAC, both entities need to know the shared secret. Assuming only trusted entities know the shared secret, then recipient can trust the message.


> 💡 A digital signature is an encrypted message digest using asymmetric key pairs.

The digital signatures are designed to prove the authenticity of the message. Given the property of asymmetric cryptography, the recipient know only the entity with private key can encrypt the digest to the matching signature.

&nbsp;

### Certificates

In last blog, I mentioned people use services like public key service to look up someone else's public key. But how would one computer to know what to search for when it receive a message.

The validity of the authentication of a digital signature only if we know the public key's owner and identity. We can't know the identity unless we know the public key's owner. Having a central service for looking up the public key's true identity is hard and waste of resource. We need a better solution.


We use a digital certificate for the public key.
