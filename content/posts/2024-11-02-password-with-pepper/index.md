---
title: "Pepper in password"
date: 2024-11-02T22:43:06-07:00
slug: pepper-in-password
summary: Have you heard of pepper in best practice for password hashing?
tags:
  - security
  - cryptography
type: posts
coverCaption: Photo by <a href="https://unsplash.com/@joshuamphotography">Josh Massey</a> on <a href="https://unsplash.com/">Unsplash</a>
---


Today I stumbled upon a new term that I never heard of in best practice for password hashing, `pepper` 🫑. I've heard of salting that is adding a uniquely generate random value to password before its hash function. `Salt` is to prevent rainbow table attacks by adding enough space to the table size so it's impossible to look up. 

However, I've never heard or seen the term `pepper` being used in password hashing. I was very intrigued. It is not defined in the [RFC-4949](https://www.rfc-editor.org/info/rfc4949) where `salt` is defined.


## Pepper (cryptography)


The concept is similar to `salt`. It is a secret value added to the password before hash. Unlike `salt`, `peppers` must not be stored along side with the password hash, and the same pepper value can be used for different passwords. `Peppers` are usually stored in the application configuration securely, or in a secure vault that is not in the database. 

### Why is it useful?

Imagine each password is hashed as `hash(password + salt + pepper)`. Now, if the database is leaked, attackers get the `hashed` output and `salt` and can try brute-forcing them. But if there's an unknown pepper involved, they cannot directly brute force them. With the pepper, the attackers have to factor in `pepper`, making brute-forcing impossible unless they also have access to the application’s configuration.

It is an extra layer of defense, making it harder for anyone to crack the password.


There are also other considerations when employing `pepper` strategy:
* Additional storage complexity and ensure each service can connect to it securely.
* `pepper` rotation strategy should be considered. Any changes to the `pepper` value would invalidate all password hashes, multiple `pepper` should be considered.
* Each time a password change, `pepper` value should be the latest value per the policy.


Overall, this was a fun thing to learn about. `pepper` adds another layer of security, it also introduces **operational complexity** and is only effective when properly secured and isolated from the database.


### References
* [OWASP cheatsheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
* [IETF best practices for password hashing and storage](https://www.ietf.org/archive/id/draft-ietf-kitten-password-storage-04.html)
