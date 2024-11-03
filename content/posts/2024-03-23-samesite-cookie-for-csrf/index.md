---
title: "SameSite Cookie Attribute"
date: 2024-03-23T18:44:41-07:00
slug: samesite-cookie
summary: How can someone use SameSite cookie attribute to mitigate CSRF attacks?
tags:
  - web
  - security
  - xss
type: posts
showTableOfContents: true
weight: 1
---


There are many mitigation for CSRF attacks, both in frontend and backend have to implement layered defense to protect against it. Previously discussed CSP as a security layer for XSS protection. Is there something for CSRF? Can we add something to mitigate CSRF?


# Introducing SameSite Cookie Attribute

As the name suggests, a SameSite cookie is a cookie that is only sent under SameSite conditions. It is used by setting an attribute called `SameSite`, which can have three values:
1. None
2. Lax
3. Strict

`None` means that there is no restriction. The cookie does want the `SameSite` attribute.
`Strict` has the most restriction. It basically means that the cookie can be sent when the target is the same site.


