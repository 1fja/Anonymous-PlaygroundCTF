# Anonymous Playground - Tryhackme

(kinda) Technical write-up about the Anonymous Playground CTF

## Recon:
I've started with a default reconnaissance using nmap:

nmap -p- -sC -sV 10.67.156.126 --min-rate 10000

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3c:7c:be:e6:8f:1e:dc:f9:67:65:9d:1f:2c:c3:43:bf (RSA)
|   256 2d:75:45:2f:b5:e0:f3:ff:76:db:05:aa:14:ba:f9:a5 (ECDSA)
|_  256 e8:9e:c8:c7:e3:bf:66:6a:93:3a:bf:41:7f:ae:6c:c1 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/zYdHuAKjP
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Proving Grounds
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel


### As we can see, we found a SSH port and a web port, but in robots.txt we can see a weird directory popping out

And if we go to the website, we'll see the main page and 'operatives' tab. But it's kinda of irrelevant now

### Acessing the weird directory, we get a page saying 'Acess Denied'. But no worries! We can bypass that

By acessing the Developer Tools in our browser and going to "Local Storage", we'll se the cookie that manage our permissions, but just by changing the value from 'denied' to 'granted', we've bypassed the permission thing


## We came across a very very wierd looking cryptographic string, but no matter what we do... We can't decrypt with normal online tools. What do we do now?

Going to the tryhackme page, we'll se a hint for our first flag and the hint is:

"zA = a"

With that hint, we'll create a python script that can decode that weird string. The python script I've used is this one, feel free to use:

```python
import string

alphabet = string.ascii_lowercase

def decode_pair(pair):
    a, b = pair.lower()

    value = (alphabet.index(a) + 1) + (alphabet.index(b) + 1)

    value = ((value - 1) % 26) + 1

    return alphabet[value - 1]


cipher = "hEzAdCfHzAhAiJzAeIaDjBcBhHgAzAfHfN" #remember to change this string

result = ""

for i in range(0, len(cipher), 2):
    result += decode_pair(cipher[i:i+2])

print(result)
```

**(What the code does):**
***Each letter is converted to its alphabet position (a = 1, b = 2, ..., z = 26). The two values are then added together, and modulo 26 is used to keep the result within the alphabet.***
The modulo operation also makes the alphabet wrap around. For example:

z = 26
a = 1

26 + 1 = 27
27 % 26 = 1

1 = a


After decrypting this string, we'll get the user and the password. We can use it in the SSH port!

## SSH

In magna directory, we can get our first flag.
But inside of it, we can spot 2 files, one that is a note from spooky and the other one is "hacktheworld"

In the note, he says that he's working his skills in C, creating a type of R.E and malware thing, but he also says that we can break his file called hacktheworld

For that, we can use the tools that we have in the machine, which will be radare2 and gdb

So...

## Binary exploitation

This part takes a lot of time if you're not into binary exploitation.

But, first of all, we need to verify what the code of this file says, so, we'll be using a R.E ***(reverse engineering)*** tool of our preference

Once you're in the C file's code, we can see that in the main fuction, we have a character limit of 64 bytes
and in another function called 'call_bash', we can see that this fucntion calls the system and put /bin/bash, which will lead to our Privilege escalation

So now, we'll try to "break" the file, to identify the buffer capacity and get 'Segmentation Fault'

The file breaks after 72 bytes, 
