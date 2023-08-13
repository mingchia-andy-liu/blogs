---
title: "CORS Guide Part 2"
date: 2021-06-25T19:45:15-07:00
description: Learn how to fix CORS with examples.
image: images/cors-error.png
slug: cors-guide-2
tags:
  - webdev
---

# Introduction

Welcome back. Last blog we talked about What CORS is and how to identify a CORS response. We understand why browsers have a same-origin policy and why it is important. In this blog, I would like to talk about how to fix the error.

To preface, the proper solution is to present the headers from the server. Ultimately, the server does not trust you enough to allow you to see the response.

To start with, the are a couple way to "fix" the symptom and you won't see the error anymore.
1. Set `no-cors` in mode*
1. Turn off CORS in the browser
1. Use a proxy server

## Method 1: Set `no-cors` in your request*

This is the most common mistake that a beginning developer can make! This method does **NOT** fix CORS.

Let's look at the error message again.

```
Access to fetch at 'http://localhost:5000/' from origin 'http://localhost:3000' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.
```

We already looked at the first part. Let's examine the second part.
> If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.

It sounds like if setting the request's mode to `no-cors` will do the trick. Let's try it.

```js
fetch('http://localhost:5000', { mode: 'no-cors' })
  .then((response) => {
    console.log('success.', response);
  })
  .catch((error) => {
    console.log('fail.', error)
  })
```
Let's run the code above. The request does not fail anymore. But something weird is happening, the response looks off.

{{< figure
    src="no-cors.png"
    alt="A screenshot of the chrome devtool with CORS response."
    position="center"
>}}

The response is status code is `0`, type is `opaque`, and ok is `false`. We don't see the error but we still do not see the response. [The definition of status code 0](https://fetch.spec.whatwg.org/#concept-network-error) is network error. It is not a standard HTTP error.

Why does this happen with `no-cors`? 🤔

The reason is that `no-cors` does not mean it ignores `cors` response. It means that you can perform requests to other origins, even if they don't set the required CORS headers. But you'll get an [opaque response](https://fetch.spec.whatwg.org/#concept-filtered-response-opaque).

This is the exact opposite of what we meant to do. With `no-cors`, JS cannot ever access the response object, even if the response has proper CORS headers. The only thing you will get is an opaque response.

If you find yourself setting `no-cors`, you are probably doing it wrong. It does not get rid of the error. It only hides it.

## Method 2: Turn off CORS in the browser

I mentioned that CORS response is blocked by the browser as one of the web security. The simple solution is turn off the security setting. Fair warning, I do not recommended this method because it is dangerous and only do at your own risk.

There are many guide on how to do it in **Chrome**, you turn on the `--disable-web-security` argument flag when you launch a new instance. For **Firefox**, there does not seem be an easy way to do it from a brief research. I am sure there is some way to bypass it in all the browsers.

However, this only fixes the problem for you. CORS responses are still blocked in other people's computers.

## Method 3: Use a proxy server

By the name of it, it is a server that proxy your requests and rely the response back. A proxy server can send the request outside browser, and response won't be blocked. If it's outside the browser, CORS error goes away. As long as the proxy server sets the header to a wild card `*` or to `yourdomain.com` back to you, otherwise we are back to step 1.

## Conclusion

With the above 3 methods, all of them are not the _correct_ solutions. Even though the proxy server can get the response, they are still not the best way to do it.

The proper solution is to include proper headers in the response. If you do not have control over the server, then you should contact server owner. If CORS is configured to exclude your domain, the proxy server might be the solution. However, please consider why the server is configured that way. Maybe it does not want other people to scan and process responses.

This sums up CORS in 2 very simple blog posts. I did not cover other advanced topics like: simple request vs non simple requests, preflight request, and other cross-origin headers. They will be in the next CORS blog.

For now, this should be enough for anyone wants to learn what is CORS and how to fix it. 😎
