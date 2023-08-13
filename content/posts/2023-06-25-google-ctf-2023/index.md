---
title: "Google CTF 2023 Writeup"
date: 2023-06-26T00:00:00
description: The Google CTF 2023 challenge writeup. 1 solved web challenge.
tags:
  - ctf
showTableOfContents: true
showCover: false
---

A CTF organized by [Google](https://g.co/ctf). I only solved 1 web challenge but learnt a lot of things for other categories.

Google uploaded [the complete solution writeups](https://github.com/google/google-ctf/tree/master/2023) immediately after the challenge is over. This was great since I always wanted to read the author's writeup or thoughts for the solution. Hopefully other CTF events can do the same.


## web/under-construction

* solved
* description: We were building a web app but the new CEO wants it remade in php.
* solves: 466

The challenge consists with 2 applications: `flask` and `php` apps. The page has a signup & login where the CEO can view the flag in the `php` app but users can only signup in the `flask` app. Each user can signup with different tier and only CEO can sign up with `GOLD` tier.

In the signup route in `python`, we can see the validation
```python
def signup_post():
    raw_request = request.get_data()
    username = request.form.get('username')
    password = request.form.get('password')
    tier = models.Tier(request.form.get('tier'))
    # ...
    if(tier == models.Tier.GOLD):
        flash('GOLD tier only allowed for the CEO')
        return redirect(url_for('authorized.signup'))
    # ...
    requests.post(f"http://{PHP_HOST}:1337/account_migrator.php", 
        headers={"token": TOKEN, "content-type": request.headers.get("content-type")}, data=raw_request)
```

Once the user is validated and inserted to the database. It is also to the `php` app via the special API token to the endpoint. However, the migration request body is the raw data (`data=raw_request`) instead of the validated request. This is where we can bypass the validation.

In the `php` migrator endpoint, the request is only validated as a no empty string for `tier`.

```php
if (!isset($_POST['username']) || !isset($_POST['password']) || !isset($_POST['tier'])) {
	http_response_code(400);
	exit();
}

if (!is_string($_POST['username']) || !is_string($_POST['password']) || !is_string($_POST['tier'])) {
	http_response_code(400);
	exit();
}

insertUser($_POST['username'], $_POST['password'], $_POST['tier']);
```

The `php` app does not validate. With how `php` and `flask` read the body, they can be reading different value. In `flask`, the first occurrence wins, in `php`, the last occurrence wins. So we can construct our request body to be like this to bypass the validation.

```javascript
username=aliu&password=dev&tier=blue&tier=gold
```

And signin to the `php` app with the new user, then we can see the flag.

{{< figure
    src="under-construction.png"
    alt="A screenshot of the under-construction challenge with the flag after login."
>}}

## misc/papapapa

* unsolved
* description: Is this image really just white?
* solves: 128

This is the challenge I spent my most of time on. The challenge is only a white jpg. There is nothing else and when opened, it is only a white image. I used `strings` and `exiftool` but they showed nothing. I tried looking at the hex code values, it looks like there are some values. I can see sections of `00` and some random values. This tells me something is hidden within the jpeg. 

This is where I was stuck. I started looking into how could someone include a flag into the jpg image. There are a lot of steganography tools. Steganography is the practice of concealing information within another message or physical object to avoid detection.

I found 2 things `binwalk` and `stegsolve`

`binwalk`: Binwalk is a tool for searching a given binary image for embedded files and executable code. Installed by `brew install binwalk`.

`stegsolve`: Able to see different alpha channels and grayscale of the image. 

Installed with this repository for Mac users
```bash
git clone https://github.com/tex2e/stegsolve-macos
javac stegsolve/*.java
jar cvfm stegsolve.jar META-INF/MANIFEST.MF -C stegsolve/*.class

# usage
java -jar stegsolve.jar
```

However, these 2 tools did not show any flag for me. The image seems like just only want to show the white part of the image.

### LSB Steganography

> This is a technique of hiding message inside an image. LSB stands for Least Significant Bit. The idea is that if we change the last bit value of a pixel, there won’t be much visible change in the color. For example, 0 is black. Changing the value to 1 won’t make much of a difference since it is still black, just a lighter shade.

I wrote a python scrip to loop through the last bit of the image and try to convert them into message but failed. At this point, I stopped since I don't think I could "discover" any more information from the image. But I did come up with a short steps of what to do next time if I see a image challenge again

Image related steps
* Check the header for binary format, maybe the header is corrupted
* Try `strings` for hidden message
* Try `exiftool` for metadata
* Try `binwalk` for extracting hidden file within the file
* Try `https://www.tineye.com/` to reverse image search
* Try `stegsolve` to see different alpha channels or other image operations

### papapa solution

[See author\'s solution](https://github.com/google/google-ctf/tree/master/2023/misc-papapapa/solution), it looks like a simple solve but pretty amazing challenge.

## Conclusion

This was a cool CTF since it was hosted by Google and the challenges were pretty interesting. For jpg challenge it was new to me so I was learning a lot just by searching the category. Hopefully next time I will be able to solve similar challenges.
