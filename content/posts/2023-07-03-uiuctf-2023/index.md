---
title: "UIUCTF 2023 Writeup"
date: 2023-07-03T17:56:34-07:00
description: The UIUCTF 2023 Writeup. One crypto challenge. 
tags:
  - ctf
showTableOfContents: true
showCover: false
---

UIUCTF was organized by [SIGPwny](https://2023.uiuc.tf/). The UI of the challenge was really cool. An old Windows look but it's based on the standard CTF template. It was impressive to see their work to make the UI memorable.


## crypto/Three-Time Pad

* solved
* description: "We've been monitoring our adversaries' communication channels, but they encrypt their data with XOR one-time pads! However, we hear rumors that they're reusing the pads...\n\nEnclosed are three encrypted messages. Our mole overheard the plaintext of message 2. Given this information, can you break the enemy's encryption and get the plaintext of the other messages?"
* solves: 390


We have 3 encrypted messages and 1 known plaintext for message 2. The description says it is re-using the key for the XOR one time pad. So we can use the known pair to extract the key and decrypt the other 2 messages.

We first convert the binary to hex and the message to hex. 

```python
from Crypto.Util.number import *

def xor(a, b):
    if len(a) > len(b):
        return '%x' % (int(a[:len(b)],16)^int(b,16))
    else:
        return '%x' % (int(a,16)^int(b[:len(a)],16))

c1 = "14F5F95B4A252948A8AEF177D6C92D82E3016362BD7463F41F40A00AD9E0CCAD911B959EF8DFAD5F1CC4481ECB64"
c2 = "06E2F65A4C256D0BA8ADA164CECD329CAE436069F83476E91757E91BD4A4CCE2C60A8F9AAC8CB14210D55253CD787C0F6A"
c3 = "03F9EA574C267249B2B1EF5D91CD3C99904A3F75873871E94157DF0FCBB5D1EAB94F9386"

p2 = "printed on flammable material so that spies could"
p2hex = "7072696e746564206f6e20666c616d6d61626c65206d6174657269616c20736f207468617420737069657320636f756c64"

k = xor(p2hex, c2)

p1h = xor(c1, k)
p3h = xor(c3, k)

byte_string = bytes.fromhex(p1h)
p1 = byte_string.decode("ASCII")
byte_string = bytes.fromhex(p3h)
p3 = byte_string.decode("ASCII")

print(p1)
print(p3)
```

Then we get the flag.
```
before computers, one-time pads were sometimes
uiuctf{burn_3ach_k3y_aft3r_us1ng_1t}
```

---

At the start, I misread the description and through p2 is the keywords used for the XOR pad. I learned a new technique: Crib dragging. Crib dragging is a [known plain text attack](https://en.wikipedia.org/wiki/Known-plaintext_attack) where if you can guess or know part of a plain text message you can begin to decrypt other encrypted messages by dragging a XOR of the known plain text over two or more messages.

It was interesting to learn that you can XOR back with known words to start guessing the complete message. Although this was not relevant to the actual solution, it's useful for future challenges. [cribdrag](http://cribdrag.com/) is something I found online to visually represent the attack.
