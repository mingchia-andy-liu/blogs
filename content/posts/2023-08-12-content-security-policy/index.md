---
title: "Content Security Policy: Defense against XSS"
date: 2023-08-12T09:41:40-07:00
slug: content-security-policy
summary: How can someone use Content-Security-Policy to mitigate XSS attacks? This blog post goes in depth to explain what CSP can do
tags:
  - web
  - security
  - xss
type: posts
showTableOfContents: true
weight: 1
---

This is a blog post inspired by one of the [DiceCTF]({{< ref "posts/2023-02-11-dicectf-2023" >}}) challenges I did in February. The challenge involved Content-Security-Policy which I didn't know much about. After the event, I went on a deep dive into the topic. It turned out to be a very cool security mechanism developed to mitigate cross-site scripting (XSS) attacks.

[{{< icon "link" >}} My CSP demo site](https://csp.aliu.dev/). I will use this site throughout the blog. _Hosted on a serverless so it might take a while to load._

## What is CSP?

CSP is a defense-in-depth technique to help you mitigate cross-site scripting attacks. It is **not** meant to replace your existing security measures; it is an added layer of security.

In its core, servers can specify which origins are allowed to load scripts and other resources on a web page. This prevents malicious scripts from running, enhancing the security of the website.

To enable CSP, you need to configure your web server to return the `Content-Security-Policy` HTTP header. Sometimes you may see it enabled through `<meta>` tag but some features are not available if CSP is configured via `<meta>` tag.

A CSP example header that only allow scripts from its own domain and `google.com`:

```
Content-Security-Policy: script-src 'self' www.google.com
```

CSP is designed to be fully backward compatible. Legacy browsers that don’t support CSP, they ignore it. They are defaulting to the standard same-origin policy for web content.

CSP has been available in major browsers for a while now, including Safari. You can check its availability using [can I use](https://caniuse.com/?search=CSP). Some of the latest CSP 3 features are also available, even if the specification is still in draft.

## Why is CSP useful?

If you go to the link in my demo site: [dogs](https://csp.aliu.dev/dogs). You will see a list of dogs and a button that counts how many items there are in the list. 

Now, you can visit [dogs with XSS](https://csp.aliu.dev/xss-1). If you imagine the items are from user input and malicious users somehow escape your sanitizer. They can inject XSS into your page. What's even worse is that, if you look closer at the alert class. It is a `script` tag that loads from `api.google.com`. 🤔 What is happening, is google now hacked by someone else?

### JSONP endpoints

The XSS is injected by something called a JSONP endpoint. It is a technique for requesting data by loading a `<script>` element, which is an element intended to load ordinary JavaScript. It is vulnerable to the data source replacing the innocuous function call with malicious code.

More can be read from the [wiki](https://en.wikipedia.org/wiki/JSONP).


## Source of a script

Cross-site scripting attacks require the execution of a script. Therefore, we need to determine where the scripts are originating from. As you can see, attackers can even use an innocent JSONP endpoint to load malicious code.

In an HTML document, there are numerous ways to include a "script" for executing code or data. Some methods are safer than others. The goal of CSP is to minimize or block unsafe usage.

### `unsafe-inline`

**Inline scripts** are scripts from a `<script>` tag. This seems innocent but it actually poses a great danger. Imagine you are a browser engine and are parsing through the HTML document. If you encounter a `<script>` tag, how confident are you to trust the content of the script.

You should never implicitly trust it. 

Using the example below, the "origin" of the script is obvious. It has the same origin. Because browsers will request from the relative path. If a site has `'self'`, the script should be loaded and executed.

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

If a site has `'self'` and when the browser parses these "scripts", should they be executed? They could be from the developer or could be from the attackers. There is no way of knowing. Err on the side of caution, CSP disable inline scripts by default.


### `unsafe-eval`

[`eval()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/eval) allows JavaScript execution from a string. It is an enormous security risk. It is far too easy for a bad actor to run arbitrary code when you use `eval()`. 

Interestingly enough, `eval()` is no the only method for string to code execution, `setTimeout()` and `setInterval()` both accept a string argument for code execution, ie `setTimeout("console.log('hello world')")`.

{{< alert >}}
Never use `eval()` or equivalent!!
{{< /alert >}}


Now all these unsafe scripts are blocked, users can now know any script being executed is from a trusted domain. Everything is solved. No more XSS! Hooray. Not so fast


## Bring inline scripts back

If every inline script is blocked, most of the sites will not work. For inline scripts that cannot be refactored? Modern websites, we often load third party scripts for analytics, cookie handling, css animation, polyfill, and many other reasons. Google analytics is the most obvious example. What can we do now? We can enable `unsafe-inline`, but this reverts the powerful feature offered by CSP. 

CSP 2 spec added 2 new features that allowed inline scripts without allowing inline scripts: `nonce` and `hash`. Basically, if you want to execute an inline script, you can provide something to validate its origin. Then the browser can validate it before executing it. 

### `nonce`

In cryptography a `nonce` may be used to prevent replay attacks, where the attacker captures and replays a previously used message. A `nonce` is a randomly generated value and only used once for its purpose. 

```
Content-Security-Policy: script-src 'nonce-r4nd0m';
```

Now we can allow an inline `<script>` tag to execute by adding our random nonce value in the `nonce` attribute of the `script` tag:

```html
<script nonce="r4nd0m">
	doSomething(); // this line will execute
</script>
```

### `hash`


```html
<script>doSomething()</script>
```

```
Content-Security-Policy: script-src 'sha256-bnQkgwAfjTxnZSlFxZe1ogJadBHLnRuuL54WC+v+tMY='
```
This is the sha256 hash of the code `doSomething()`. 


You can go to [nonce](https://csp.aliu.dev/nonce) and [hash](https://csp.aliu.dev/hash) to see its effect. There won't be an alert popup. But the custom inline script still works and the button can still count.


## Practical example of CSP

How useful is CSP in production? Google did a search on this topic. [CSP Is Dead, Long Live CSP! (2016)](https://research.google/pubs/pub45542/).

> In this paper, we take a closer look at the practical benefits of adopting CSP and identify significant flaws in real-world deployments that result in bypasses in 94.72% of all distinct policies.
> ...
> In total, we find that 94.68% of policies that attempt to limit script execution are ineffective, and that 99.34% of hosts with CSP use policies that offer no benefit against XSS.

The problem with allowlist approach is that it is hard to maintain the list and people have no control over what is hosted on CDN. Anyone can host attack payload in S3 or static hosting sites. In addition, an attacker can inject a payload that leverages the JSONP endpoint to deliver malicious code. There is no way for developers to know the domain is being used as an attack because it is trusted in the policy. 

Going through to my demo site example in a practical sense: [twitter-1](https://csp.aliu.dev/twitter-1), [twitter-2](https://csp.aliu.dev/twitter-2), and [twitter-3](https://csp.aliu.dev/twitter-3). It can show how hard it is to integrate a third party feature ot a website. By adding a twitter timeline to a site, it requires adding their script but also adding an additional script to the CSP.

## Future of CSP

### `strict-dynamic`

This is the ultimate CSP to use. It allows you to build a chain of trust based on the script you already trusted. It is similar to how digital certificates work where they have to rely on a chain of trust. We have to start the trust at a point, and we can use either `nonce` or `hash`

With `strict-dynamic`, any `safe` or `unsafe-*` keywords and source allowlist are ignored, so attackers cannot even bypass CSP with JSONP endpoints. If you want to load any scripts from a different domain, what can we do now?

The following snippet is the best solution to load any third party scripts with `strict-dynamic`. No domain is allowed unless it is loaded via a trusted source. In this case, it is a trusted script because of the nonce in the tag. 

```html
<script nonce="r4nd0m">
  var scripts = [ 'https://example.org/foo.js', 'https://example.org/bar.js' ];
  scripts.forEach(function(scriptUrl) {
    var s = document.createElement('script');
    s.async = false; s.src = scriptUrl;
    document.head.appendChild(s);
  });
</script>
```


### SPA issues with `strict-dynamic`

* Difficult to use `nonce` because it has to be different each time, hard to do when your app is served statically.
* Difficult to use `hash` because `self` script is prohibited, the only way to load is via the above method to load any script. 

Overall, deploying strict CSP policy with SPAs is perfectly feasible, albeit with a bit more effort than you would expect.


## CSP beyond script code

So far, we've mainly discussed CSP as a secondary defense against XSS attacks. However, delving into CSP specifications reveals its extensive range of capabilities. This section will delve into the diverse types of content you can regulate using CSP.

#### Stylesheet, Image, Object, Frame, etc

The CSP header value is made up of one or more directives, multiple directives are separated with a semicolon `;`. One other direct that is familiar to other people is stylesheet. The `style-src` directive governs all aspects of CSS code, including `<style>` blocks, style attributes, and external style sheets. In the end, stylesheet is most likely less harmful than JavaScript. 

#### Reporting Mode

CSP offers report-only mode. In this mode, the browser processes the CSP policy and verifies its compatibility with all application features. If the browser detects a violation, it will send a POST request detailing the violation to an endpoint specified by you. It is a great way to fine tune the policy before flipping the switch. This is recommended if you are adopting CSP for an existing site. Start with report-only and only turn on once the violations are down to a minimum.


## Summary

CSP is a secondary security measure to protect websites against XSS attacks. However, allowlist approach is ineffective. It is too hard to manage and easily bypass-able. Use a strict CSP approach to have effective protection. 


Here are some resources:

- https://csp.withgoogle.com
- https://content-security-policy.com
- [CSP evaluator](https://csp-evaluator.withgoogle.com/)

