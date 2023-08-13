---
title: "HackTM CTF Quals 2023 Writeup"
date: 2023-03-18T15:21:43-07:00
description: The HackTM CTF Quals 2023 challenge writeup. 1 solved web challenge.
image: images/2023-03-18/hackTM-CTF.png
tags:
  - ctf
showTableOfContents: true
---

A CTF organized by [WreckTheLine](https://wrecktheline.com/). For the event, I did not solve much for the challenges, only able to solve 1 challenge. Here is a quick and short writeup, hopefully I can next time I can do more!

## web/blog

* solved

It is a simple php where users can sign up and post blogs. After user creates a blog, the blog is shown with a unique URL. After browsing the source code, we can find that the flag is copied to a specific location.

```dockerfile
COPY ./chal/flag.txt /02d92f5f-a58c-42b1-98c7-746bbda7abe9/flag.txt
```

To get the flag, we need to utilize php's `serialize` to do a Local File Inclusion. Once user is logged in, the user object is being read from the cookie without sanitization or validation. This is a entry point for us to include some unwanted code. Even the [php\'s manual](https://www.php.net/manual/en/function.unserialize.php) says do not use this method for sansitive data.

> **Warning** Do not pass untrusted user input to unserialize() regardless of the options value of allowed_classes. Unserialization can result in code being loaded and executed due to object instantiation and autoloading, and a malicious user may be able to exploit this. Use a safe, standard data interchange format such as JSON (via json_decode() and json_encode()) if you need to pass serialized data to the user. 

```php
<?php
include("util.php");
if (!isset($_COOKIE["user"])) {
    header("Location: /login.php");
    die();
} else {
    $user = unserialize(base64_decode($_COOKIE["user"]));
}
```

In the utils file, we can find that each user's profile picture is included in the `Profile` class which is a property in the `User` class. This is where we can manipulate the code a little bit.

```php
$picture = base64_encode(file_get_contents($this->picture_path));
```

Since we know exactly where the flag is, we can simply modify the cookie to load from flag path instead of the default. Then we can get the flag `HackTM{r3t__toString_1s_s0_fun_13c573f6}`.


## web/blog-revenge

* unsolved

This is exact same challenge but the flag is not leaked through the dockerfile. I couldn't find out how to solve this CTF and couldn't find the author or any other writeup about it. There are a lot places in the code with some questionable code but nothing seems to work. I do see people performing sql injection but nothing came from it.


## Conclusion

This was a short CTF but it was a fun one. Same "code" being used for different challenge is a pretty good concept. It can help people ease into the event. 

Here are things that I learned:
* Serialize/Unserialize in PHP is not safe for untrusted data
* [Local File Inclusion](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion).
