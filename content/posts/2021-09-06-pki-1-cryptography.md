---
title: "Public Key Infrastructure Part 1 - Cryptography"
date: 2021-09-06T11:25:37-07:00
description: PKI serie part 1. Explain what is a encryption and decryption, and how to use them.
images:
  - images/lock.jpeg
slug: pki-1
tags:
  - pki
  - cryptography
type: post
---

{{< figure
    src="/images/lock.jpeg"
    alt="3 locks on a table."
>}}

Photo credit: [FLY:D](https://unsplash.com/photos/ZNOxwCEj5mw) on [Unsplash](https://unsplash.com/)

## Introduction

In my [openssl certificate command blog]({{< relref "/posts/2021-06-12-openssl-notes.md" >}}), I listed out common commands for dealing with certificates using openssl. I was doing some TLS for my work and had to learn the PKI. I wished I had learned it sooner. Security as a concept has always scared me because I always thought you need strong math skills. Although it is true, understanding the concept is not hard.

The goal for me is to write a blog series about PKI and solidify my own understanding. By doing so, hopefully I can help others understand the power and importance of PKI. This is the first part and I will be separating the topics into small digestible blogs.

## What is it?

There is no single definition of a Public Key Infrastructure, but a PKI is build on top a lot of other concept, cryptography, certificates, CA, RA, etc. To illustrate the concept, I have to start with the basics and I find it easier to use them in examples. Explaining it in a way that will tell how it works and why it is needed. As is it's the standard in all CS concept blogs, the old Alice and Bob tale. I will try to make it fun and exciting to read.

&nbsp;

## Act 1

_Alice_ 👩 and _Bob_ 🧑 are in love with each other and they live far away from each other. The only way they can talk to each other is by mail. The evil mail man _Chad_ 😈 is jealous of them, so he is trying to break the their relationship.

If they write each other letters, _Chad_ can easily open it up, read it, and seal it in a new envelope as if nothing has happened. When they receive the letters, they can't know if _Chad_ has tampered with it.

### Scene 1 🍿 - Cryptography

To solve this problem, _Alice_ and _Bob_ turn their hope to cryptography. They want to encrypt their letters so that other people can read it but cannot understand it. The steps are as follow:

1. The sender converts the plaintext message to ciphertext. This step is called **encryption**
1. The ciphertext is transmitted to the receiver
1. The receiver converts the ciphertext back to a plaintext message. The step is called **decryption**

Cryptography techniques are usually composed with **key(s)** and **mathematical algorithm(s)**. I am not a math expert, I don't understand the implementation details. All I care about is the end goal, one can encrypt a message with a key and someone else can decrypt only if they also have the same key. The encryption should be 1-way encryption, meaning it is impossible or nearly impossible to revert the encryption process without the key.

The key should be held privately and securely. Otherwise, if someone else can steal the key, the encryption is compromised.

&nbsp;

### Scene 2 🍿🍿 - Symmetric Cryptography

One type of algorithm is **"symmetric cryptography"**. The principal idea is that the same key is used to encrypt and decrypt the message. The end parties both have the same secret key.

{{< figure
    src="/images/symmetric-key.png"
    alt="symmetric cryptography algorithm"
>}}

Back to the love story, _Alice_ and _Bob_ agree on a key and start communicating. Now _Chad_ cannot understand the letter even if he can secretly view it. Does this solve their problem? Yes, it does!

⚠️ **But** how do they agree on the key? Can they send their key through letters again? Not really, because there is a chance that _Chad_ bribe the mailman and steal the key. One way is to solve this is that _Alice_ and _Bob_ can meet up and agree on a key.

&nbsp;

### Scene 3 🍿🍿🍿 - Asymmetric Cryptography

We have a new problem now: exchange the key in a secure manner.

Exchanging key physically is an inconvenient and not practical way of exchanging the key. Sometimes it is physically impossible. They need a secure way to exchange the key. They can't establish secure communication without a key. This is the chicken and egg problem.

To safely communicate an encryption key, the other type of algorithm is **"asymmetric cryptography"**. The principal idea being that you generate two keys:

1. A public key
1. A private key

As the name suggests, the public key can be distributed and shared to the public (everyone). The private key is kept securely locally. If someone gets the hold of the public, they can't do anything with it. One cool thing about asymmetric crypto is that a message encrypted by public key can only be decrypted by the private key, vise versa.

{{< figure
    src="/images/asymmetric-key.png"
    alt="asymmetric cryptography algorithm"
>}}


How does having 2 keys solve the problem? Now _Alice_ and _Bob_ never have to meet up, they can just send their own public keys to each other, then only the other person can decrypt the messages. People can share their public keys on the internet, at the end of an email, anywhere. It is meant to be shared.

Let's go back to the love story. _Alice_ and _Bob_ create their own key pairs. Now, let's try again.

1. _Alice_ encrypts the letter with _Bob_'s public key
1. She sends the ciphertext to _Bob_
1. _Bob_ receives the ciphertext and decrypts with his' private key.
1. <...>

Now, _Alice_ can be mathematically sure that only _Bob_ can decrypt the message, because she encrypted it with his public key. If _Chad_ steals the message, he cannot decrypt it. Perfect!

Problem solved? Almost. In a perfect world, this would work and nobody can understand what they are talking about. The message will all be garbage. In the real world, there is a very serious problem.


⚠️ How does _Bob_ know that the message is encrypted by _Alice_? He can only be sure that the message is encrypted using his public key. But this can be done by anyone. The public key is shared. Imagine _Chad_ impersonating _Alice_. He tosses out her encrypted letter and encrypts a bad message to _Bob_. Their relationship can be in jeopardy.

&nbsp;

### Scene 4 🍿🍿🍿🍿 - Asymmetric Cryptography Part 2

The best way to solve this is to use 2 keys to encrypt the message. Let's refresh about how the asymmetric keys work. If a message is encrypted by public key, only private key can decrypt it, **vise versa**. Someone can encrypt using the private key and everyone else can decrypt the message with the public key.

On the surface, this seems useless because everyone can decrypt and the message will never be secure. If a message can be _decrypted_ by my public key, it means that I am the owner of the private key. Mathematically proven that I am the owner of the key pair. This can prove the identity of the sender.

Let's go back to the story again.

1. _Alice_ encrypt the first couple words with her private key
1. _Alice_ encrypts the letter and the encryption from step 1. with _Bob_'s public key
1. <... Sends the letter ...>
1. _Bob_ decrypts the letter with his own private key.
1. _Bob_ also decrypts the encryption using _Alice_'s public key.
1. If the decrypted result matches up with the first couple words, then _Bob_ can be certain the message is sent from _Alice_.

{{< figure
    src="/images/asymmetric-key-2.png"
    alt="asymmetric cryptography algorithm - message"
>}}

After all this trouble, _Alice_ and _Bob_ can now talk in secret and they live happily ever after.

&nbsp;

## Summary

In this blog, I have described symmetric and asymmetric cryptography, why they are needed, and their use cases. Please note that I am **not** saying that symmetric cryptography is bad, because we can't share the key easily. I am only describing it as not the right tool for the job (establishing a secure connection). In fact, most of the messages exchanged over the internet are using symmetric cryptography once we have established a secure connection because it is much faster.

Cryptography is an amazing and simple concept. The actual math is complicated, for sure, but I don’t need to understand it to appreciate its value.

I have dismissed where people would store the public key and others can be ensured that they have not been tampered with. By publishing the keys online, we would have to do it by an insecure measure. Like the scene 2's problem. They have services like this [keyserver.ubuntu.com](http://keyserver.ubuntu.com:11371/) or [keyserver.pgp.com](https://keyserver.pgp.com/vkd/GetWelcomeScreen.event).

In the next blog, I will explain how we can take this concept and apply to computers instead of humans.
