---
title: "Intigriti 0522 XSS Challenge"
date: 2022-05-29T17:47:58-07:00
description: Solving Intigriti 0522 XSS Challenge
images:
  - images/0222-challenge/solved.png
tags:
  - javascript
  - xss
  - intigriti
type: post
---


## Intro

Intigriti is a hacking and bug bounty platform. They host a monthly XSS challenge.

See
* Challenge: https://challenge-0522.intigriti.io/challenge/challenge.html


## My Solution

The page is a simple view with 3 content tabs at the top. User can select which tab they want to see. Among different view, the `page` query parameter changes from 1 to 3.

Let's take a look at the develop console. We can see a log statement was fired.

```js
var pages = {
    1: ...,
    2: ...,
    3: ...,
    4: ...
  };

  var pl = $.query.get('page');
  if(pages[pl] != undefined){
    console.log(pages);
    document.getElementById("root").innerHTML = pages['4']+filterXSS(pages[pl]);
  }else{
    document.location.search = "?page=1"
  }
```

The page will load from `pages` string depends on which `page` the query parameter. If the page is found, then it will be filtered and injected to the `root` element. Otherwise, the user will be directed to the `1` (first) page.

### 💉 Dissect the code


To start, the `if` statment stood out to me. Because if we need to inject some code, we will need to inject _something_ to the `pages` object. One of the way we can do is prototype pollution. We can pollute the object to be `[pl]` property and bypass the `undefined` check.

```js
pages[pl] != undefined
```


Prototype pollution needs some help. Let's how the libraries loaded in the page can help us. First, how the page parameter is parsed from the URL. It seems to be using `jQuery`.

```js
$.query.get('page');
```

`jQuery` is loaded at the top of the `head` element along with something called `jQuery.query` plugin. The plugin will help operation of getting and modifying the query paramters. However, this library has client side prototype client pollution vulnerability. [CVE-2021-20083](https://nvd.nist.gov/vuln/detail/CVE-2021-20083). It helps us to inject any property we want to any object. 

Now, that we have a possible way to bypass the `if` check, we still need to bypass the `filterXSS` function call. The method is from `js-xss` library which help to sanitize the user input in case of any XSS injection. 

Luckily for us, it also has an use case where we can use the prototype pollution to our advantages. Turns out it has `whiteList` option. Although we cannot inject to the call option, we can pollute all object to have an `whiteList` property.


[Both of the use case are documented here](https://github.com/BlackFan/client-side-prototype-pollution). A very helpful resource when I was researching prototype pollution.

### 💨 Pollute the code

`jQuery.query` pollution PoC: inject prototype in the search parameter: `?__proto__[test]=test`. 


**Step 1**

We need to bypass the `if` check for any pages. Let's have an undefined page called `a`, ie. `?page=a`. We also need to pollute the prototype with `a` property. Let's inject to the query parameter as well. `&__proto__[a]=pollution`

`?page=a&__proto__[a]=pollution`

Now we can see the html is showing whatever we inejct.

{{< figure
    src="/images/2022-05-29/step-1.png"
    alt="A screenshot of pollution to the html."
>}}

**Step 2**

Now we need to bypass `filterXSS`. We can inject the whitelist option from the search parameter now. `&__proto__[whiteList][img][0]=onerror&__proto__[whiteList][img][1]=src`. This tells the `js-xss` to allow the `image` tag with `src` and `onerror` attributes. 

One of the common way to inject xss is `<img src onerror=alert(1)>`

Let's if we can do that. `?page=a&__proto__[a]=<img src onerror=alert(1)>&__proto__[whiteList][img][0]=onerror&__proto__[whiteList][img][1]=src`

{{< figure
    src="/images/2022-05-29/step-2.png"
    alt="A screenshot of an incomplete xss. Text shows: '<img src onerror'"
>}}

The `alert(1)` did not show up, even though we bypass the filter. We should've got a popup. Turns out `=` is a reserved word in URL. It parses `alert(1)` as some other value. So the `a` property only has `<img src onerror` part.

**Step 3**

Encode the `=` to URL component. The encoded value is `%3d`. The poopup shows up. 🎉 Now we can finally show the `document.domain`.

{{< figure
    src="/images/2022-05-29/step-3.png"
    alt="A screenshot of alert document domain popup"
>}}



## Thoughts

This was a fun challenge. I think the difficully was rihgt now something for XSS begineer like me, not too difficult, also not too easy. It reuiqres 2 libraries to do prototype polllution and finally maybe a small step to encode the value for the xss to work.

## Things I learned

- Learning about Javascript's special properties `__proto__` & `prototype`
- How to submit solution to Intigiti

## Resources

- [JavaScript prototype pollution attack in NodeJS by Olivier Arteau](https://github.com/HoLyVieR/prototype-pollution-nsec18/blob/master/paper/JavaScript_prototype_pollution_attack_in_NodeJS.pdf)
- [Client side prototype pollution repository](https://github.com/BlackFan/client-side-prototype-pollution)
