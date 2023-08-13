---
title: "CORS Guide Part 1"
date: 2021-06-19T12:16:50-07:00
description: Learn what is CORS and how it works with examples.
slug: cors-guide-1
tags:
  - web
  - security
---

## Introduction

Recently Instagram started to set `cross-origin-resource-policy` in their image's response header. This is not strictly CORS but it complements CORS. If you are a web developer, you will see CORS errors at some point.

I thought to myself to write a blog series about CORS and how to handle them. The actual purpose of this blog series was twofold:

1. Solidify my own understanding of CORS
1. Helping others to also understand the purposes of CORS

Security can be confusing. It’s taken me longer than I care to admit to really understand the things I’ll be discussing here (and even then I’ll likely have missed a lot of important nuances). With that said: what am I planning on covering?

1. What is CORS?
1. Why do we need it?
1. How to fix CORS?
1. Understanding HTTP security

...and a lot more in between.

---

## What is CORS?

💡Let's see from [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).
> CORS stands for "Cross-Origin Resource Sharing" is an HTTP-header based mechanism that allows a server to indicate any other origins (domain, scheme, or port) than its own from which a browser should permit loading of resources.

To simplify, it is a security measure that the **browser** implements to deny JavaScript access to the `XMLHttpRequest` or `fetch` response.

## Why do we need it?

Cross-origin requests are useful or even necessary to many websites. Why? Cost and performance are major factors. If you can host your public resources on someone's machine, it can save a lot of money. This is why Content Delivery Network (CDN) exists. It saves you operational cost and improves your user experiences. A win-win for everyone.

By using `<script />` or `<img />` with a different domain, they are cross-origin network access requests. Even using Google's font stylesheet `<link rel="stylesheet" href="…">` is a cross-origin request. [Web fonts rely on CORS to work](https://www.w3.org/TR/css-fonts-3/#font-fetching-requirements).

If it's common, why do browsers block it? Let's say they do not block cross-origin requests. Now every site can fetch anything from the internet and see the response. What would happen? 👀

**Case 1**

Alice 👩 is visiting a website that has been compromised using their company's network. The website can send requests to all the internal sites in Alice's network. The attackers can send `XMLHttpRequest`s to those internal sites and send data back to their servers. You may say it is unlikely that the hackers know the internal sites. But the probability is not zero. And there are some common names that malicious attacker can guess, ie. `git`, `jira`, `api`, `dashboard`, etc.

**Case 2**

Let's say the attackers do not know the internal sites' addresses/names. They can still scan the `localhost` with all the available ports. If you have any local development setup, they are now exposed.

**Server's POV**

If the first 2 cases don't convince you, let's think of it from resource owner's perspective. Do you want other people to send requests to your server and process the response? The simple answer is "No". The response should only be seen by trusted parties. The attackers could be impersonating your site as you and steal information from your users.

&nbsp;

Even if you trust the site, it does not take a lot to compromise a site if it's well planned, the bad actors could use third party scripts or ads that they are using. There are many other ways: xss, server misconfiguration, bad serialization, etc. In addition, Alice could be social engineered to open a malicious site.

Without CORS, it opens up many potential attacks. It’s an important part of the web security model.
These are all the scenarios that I can think of why CORS is needed. This is definitely not an exhaustive list, but hopefully enough to convince you that CORS is a good security measure and browsers are doing everyone a favor. 👍

## How does it work?

To see how it works, let's look at the error together and analyze the CORS behaviors.
```
Access to fetch at 'http://localhost:5000/' from origin 'http://localhost:3000' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

When a server has been configured to allow CORS, some special headers will be included. Browsers can use these headers to determine whether or not an `XMLHttpRequest` or `fetch` call should continue or fail. The primary header is [Access-Control-Allow-Origin](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#access-control-allow-origin).

For example: to only allow `example.com` with `https` protocol.
```
Access-Control-Allow-Origin: https://example.com
```
Or if you want to allow everyone, you can set a `*` wildcard character.
```
Access-Control-Allow-Origin: *
```
The error is saying that the response does not have the CORS header, therefore, browsers block the response. If you navigate to the network tab in the dev tool and see the request. You can see the request is completed but the response is blocked by browser. By default, the server does not need to do anything if CORS isn't needed.

**💡 What is an origin? [MDN](https://developer.mozilla.org/en-US/docs/Glossary/Origin)**

> Web content's origin is defined by the `scheme` (protocol), `host` (domain), and `port` of the URL used to access it. Two objects have the same origin only when the scheme, host, and port all match.

Quick summary:
* `https://example.com` and `https://example.com/api` are the same origin.
* `https://EXAMPLE.com` and `https://example.com:443` are the same origin because `https` default port is 443 and host is case-insensitive in URL context.
* `https://example.com` and `http://example.com` are not the same because the scheme is different.
* `https://example.com` and `https://app.example.com` are not the same because the host is different as it's part of the host.
* `https://example.com` and `https://example.com:8080` are not the same because the port is different.

What would happen outside browsers? You can try to use `curl` or any other API client, the response will not be blocked. Since there is no way to determine the request's `origin`. These headers would mean nothing to them. CORS cannot and should not apply by these clients.

## Conclusion

* CORS is a browser security measure that denies response access if the response says so.
* It is the **response** that is blocked, not the request. Since servers has to return the "allowed" origin in the response. Browsers block the response by default or if there is a mismatch.
* The special headers are useless outside browsers.

Next blog I will talk about how to "fix" the CORS error.
