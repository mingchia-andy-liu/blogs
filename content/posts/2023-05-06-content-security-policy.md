---
title: "Content Security Policy: Defense against XSS"
date: 2023-05-06T09:41:40-07:00
description: How can Content-Security-Policy help you mitigate XSS attacks?
images:
  - images/2023-03-18/hackTM-CTF.png
tags:
  - web
  - security
type: post
---

This is a blog post inspired by the DiceCTF I did in February. The challenge involved with Content-Security-Policy and I started learning what it was about. It is a actually a very cool security mechanism developed to mitigate cross-site scripting (XSS) attack.

## What is it?

## How does it work?


### `unsafe-inline`

Use the example below, the "origin" of the script is obvious. It is same origin. Because browser will request from the relative path. If a site has `'self'`, the script should be loaded and executed.

```html
<script src="/js/script.js"></script>
```

What about these 2 examples?

```html
<img src="x" onerror="console.log('should I be executed?')">
<script>
  console.log("what about me?");
</script>
```

If a site has `'self'` and the browser starts to parse these scripts, should they be executed? They could be from the developer or could be from the attackers. There is no way of knowning.


### `unsafe-eval`

## Bring inline scripts back

For inline scripts that cannot be refactored? Modern websites, we often load third party scripts for analytics, cookie handling, css animation, polyfill, and many other reasons. 

### `nonce`

In cryptography a `nonce` may be used to prevent replay attacks, where the attacker captures and replays a previosuly used message. A `nonce` is a randomly generated value and only used once for its purpose. 

```
Content-Security-Policy: script-src 'nonce-r4nd0m';
```

Now we can allow an inline `<script>` tag to execute by adding our random nonce value in the `nonce` attribute of the `script` tag:

```html
<script nonce="r4nd0m">
	doSomething();
</script>
```

### `hash`

### `strict-dynamic`

### Summary



