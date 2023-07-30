---
title: "Learn Shell Expansion"
date: 2022-05-21T09:30:00-07:00
description: Learn shell expansion
image: images/2022-05-21/og.png
tags:
  - shell
type: post
---

## Intro

I stumbled upon this bash code and couldn't figure out what it does. I'm not a bash expert and do very little on bash scripts. My main usage is for-loop and if statements. Both of them I will have to google the correct syntax every time.

```bash
#/bin/bash

msg="needs to be set"
: ${var:?$msg}
```


## 💉 Dissect the code

From this simple 2 lines of code, I only understood 3 little things. 

* `=` variable assignment
* `$msg` is the variable usage
* `${}` some sort of expansion

Things I didn't understand

* `:` at the start of line
* `:?` weird syntax after a variable

## 📗 Learning

`:` was not easy to search since lots of other syntax also involves `:`. Let's see what it is.

Turns out `:` is actually a null command in bash. [Manual](https://www.gnu.org/software/bash/manual/bash.html#Bourne-Shell-Builtins)

> Do nothing beyond expanding arguments and performing redirections. The return status is zero. 

So it does nothing and always returns true. But if there are arguments it will expand them. This is a very interesting use case. Let's see the arguments then. What does the expansion do?

`${var:?$msg}` looks very similar to a normal parameter with braces. After the parameter name, it includes `:?` which puzzles me. I've never saw this before. 

Shell parameter expansion has no much more.

[Manual](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html)

Bash can test if a parameter is unset or `null`. It has various forms to detect and output differently. It is combined with `:` and an additional operator.


Some basic forms:
* `${parameter:-word}`: If parameter is unset or null, the expansion of word is substituted. Otherwise, the value of parameter is substituted.
* `${parameter:=word}`: If parameter is unset or null, the expansion of word is assigned to parameter. The value of parameter is then substituted. Positional parameters and special parameters may not be assigned to in this way.
* `${parameter:?word}`: If parameter is null or unset, the expansion of word (or a message to that effect if word is not present) is written to the standard error and the shell, if it is not interactive, exits. Otherwise, the value of parameter is substituted.
* `${parameter:+word}`: If parameter is null or unset, nothing is substituted, otherwise the expansion of word is substituted.

There are many other forms and tricks, please check out the manual for more!


**Back to problem**

`${var:?$msg}` is a clear way to detect if `var` variable is set or not. If it is not, then it will write the error message to the stderr. What a cool way to check required arguments instead of using `if` statements. It also exits the shell which prevents variables being used.

However, why does it need to have `:` null command?

It does not have a functional point for this expansion. But let's see this one

```bash
: ${var:=$msg}
```

It will become a shell command! Then since `needs to be set` or `needs` in particular is not a command, we will see an error message `needs : command not found`. 

This is not ideal. Although `var` will be substituted, it is not desirable to have an error and continue. 

## Things I learned


* `:?` shell expansion syntax logging to error is unset or null.
* `:-` shell expansion syntax for default value if unset or null.
* `:` is a null command in Bash.
