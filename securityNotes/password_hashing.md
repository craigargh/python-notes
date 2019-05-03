# Password Hashing

- Why is password security important?
- What is a hash?
- How does a system check your password?
  - Example of MD5 hashing
- Dictionary attack/Rainbow table
- Salt
- Example of PBKDF2


Imagine that you want to set up an online shoe shop. [IS THIS THE BEST USE CASE TO ILLUSTRATE THESE CONCEPTS THROUGHOUT THE CHAPTER?] 

How do you keep your customer and staff passwords secure?

Passwords are very valuable pieces of information to hackers. With a password a hacker can access a user's account with very little effort. Users also tend to use the same password across multiple websites, meaning that a password stolen from one site can be used to access accounts on other sites.

Leaked databases are one way that hackers can gain access to user passwords. 

Hashing is a technique used to protect sensitive user data, like passwords, so that the data is protected even if a database is leaked to hackers.

## What is a hash?

A hashing algorithm takes a value, messes it up a bit so that it is unrecognisable, and returns a hashed value. 

```text
password --> hashing algorithm --> hashed password
```

*MD5* is an example of a hashing algorithm. It takes a value, processes it to make it unrecognisable, and then returns the hashed value.

The `hashlib` standard Python library has an `md5` algorithm.

```python
from hashlib import md5

hash = md5(bytes('hello')).hexdigest()

print(hash)
```


> A quick warning in case you skip reading the rest of this chapter: MD5 is not a very secure hashing algorithm (I'll explain this more in a bit). Please don't use it in any projects that contain real user data.  

There is a differece between hashing and encryption. With encryption you are able to convert the hashed data back to its original value using a private key. With hashing you cannot convert the data back to its original value.

How are hashes used when a user logs into a system?

The password entered on the form is ran through the same hashing algorithm. The result is then compared to the stored hash. If the hashes are the same then the password is correct. If the hashes are different then the password is incorrect.


## Raindow Tables



## Salt
