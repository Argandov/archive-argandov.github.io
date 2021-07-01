---
title: "Tryhackme: Cicada 3301 Vol. 1"
date: 2021-05-13T10:20:20-05:00
draft: false
pager: true
authorbox: false
toc: true
categories: ["crypto","tryhackme"]
thumbnail: /writeups-reports/files/cicada/cicada1_thumbnail.PNG
---

A basic steganography and cryptography challenge room based on the Cicada 3301 challenges
<!--more-->

Link for the room: https://tryhackme.com/room/cicada3301vol1

---

![cicada_9.png](/writeups-reports/files/cicada/cicada_0.jpg)

There are a lot of [stories](https://www.telegraph.co.uk/technology/internet/10555088/Cicada-3301-update-the-baffling-internet-mystery-is-back.html) surrounding [cicada 3301](https://en.wikipedia.org/wiki/Cicada_3301), and this room was inspired by this mysterious crypto challenges.

Let's start:

---

Initially we are provided with 2 files:

![cicada_1.png](/writeups-reports/files/cicada/cicada_1.png)

## Analyzing 3301.wav

We used https://www.sonicvisualiser.org/ for analizing the 3301.wav file.

![cicada_2.png](/writeups-reports/files/cicada/cicada_2.png)

We can process the audio waves from it, and with a couple steps we retrieve:

![cicada_3.png](/writeups-reports/files/cicada/cicada_3.png)

Only a minor tweak to change the color so it's legible for QR scanners:

![cicada_4.png](/writeups-reports/files/cicada/cicada_4.png)

The QR code led us to a site with some information in it.

We decode the given passphrase/key from base 64 with the following commands:

```
[user@parrot/thm-cicada 17:58]$echo SG01Ul80X1A0NTVtaHA0NTMh | base64 -d

[user@parrot/thm-cicada 18:00]$echo Q2ljYWRh | base64 -d
```
After some testing, we used the Vigenere Cipher (https://www.dcode.fr/vigenere-cipher)

There were no results trying to decrypt, so I tried Encrypting it again with Vigenere, and the l33t password appeared!

## Analyzing welcome.jpg

##### 1.- Hidden information inside welcome.jpg:

For this file, we need to uncover some hidden data (Steganography). There is no information in the metadata, so we "unhide" a useful string using the passphrase given and the online tool https://futureboy.us/stegano/, which is itself a link again.

##### 2.- Embedded file inside welcome.jpg

There is another file embedded into our welcome.jpg file, and we can uncover it with:

![cicada_5.png](/writeups-reports/files/cicada/cicada_5.png)

Extracting this invitation.txt file:

![cicada_6.png](/writeups-reports/files/cicada/cicada_6.png)

## Cracking the hidden hash

Following the link given in the previous step, and downloading the next image in it, and trying to retrieve hidden information with "steghide" inside our new 85*.jpg file, the passphrase is now invalid 🤔

In Cicada 3301 challenges, the tool used was named outguess, so we can use it to test a new approach to this file, with the -r switch, which will retrieve hidden messages inside files:

![cicada_7.png](/writeups-reports/files/cicada/cicada_7.png)

We now have something to work with. (hash deliberately cut in image to avoid spoiling)

We also have now some instructions to decode a secret message in "a book". Let's see what we end up cracking; we still don't know what that "book" is.

Ran John the Ripper to try to crack the hash...

![cicada_8.png](/writeups-reports/files/cicada/cicada_8.png)

While we wait, let's try an online tool. No results at crackstation.net, so we move on to https://md5hashing.net/

![cicada_9.png](/writeups-reports/files/cicada/cicada_9.png)

(Image deliberately cut to avoid spoiling)

## Enough with decoding, let's encode!

Now, following the link, we arrive at this book:

![cicada_10.png](/writeups-reports/files/cicada/cicada_10.png)

…But we already know what to do with this, as our "out.txt" has some hints to proceed with this 🤔

The instructions are as follows:

"Use positive integers to go forward in the text use negative integers to go backwards in the text."

And the "codes" to proceed read as follows:

```
I:13:1
I:14:7
I:3:29
I:19:8

… And so on
```

This is a perfect time to practice some python, so we can make a script and "decode" our secret message (Code shared below):

![cicada_11.png](/writeups-reports/files/cicada/cicada_11.png)

Bingo! That's our link

![cicada_12.png](/writeups-reports/files/cicada/cicada_12.png)

Nope, I don't think that's it. Back to vim.

In case you want to try it this way, here's some explanation of this ~~~poorly written~~~ program:

>	I saved both the book and the "keys" of our "out.txt" file with some simple formatting (Some simple character substitutions) so it's easier to process with python:
>
>	The dots at the beginning of each paragraph ("13. The book…") of the book were changed with "~" so I don't have trouble splitting with python.
>
>	Also, the keys were saved in a new file, with the format "number:number", also to avoid splitting issues.

```with open('keysBook.txt','r') as codes:
    secret = ""
    for code in codes:
        if "/" in code:
            secret += "/"
        else:
            dig = code.split(":")
            code_index = dig[0]
            code_key = dig[1]

            with open('book.txt','r') as book:
                for line in book:

                    divided = line.split("~")
                    index = divided[0]
                    text = divided[1]
                    text = text.replace(" ","")
                    if index == code_index:
                        if int(code_key) < 0:
                            code_key = int(code_key) + 2
                            secret += line[int(code_key)]
                            break
                        try:
                            secret += text[int(code_key)-1]
                            break            
                        except IndexError:
                            break

print(secret)
```
![cicada_13.png](/writeups-reports/files/cicada/cicada_13.png)

The strings we needed to encode, with the instruction "Use positive integers to go forward in the text use negative integers to go backwards in the text.", the negative integers meant going from the start of the string, backwards, including the initial paragraph numbers. (So -2 in the paragraph "53.This shall regenerate" is 5, not "t").

This script was completely unnecessary because there were only 19 lines to "encode" and it could've been done perfectly fine with the naked eye, but it was a fun character-handling experiment.

This leads to the last piece of this Cicada 3301 puzzle, and with our final link open, and nothing else to retrieve from it, except for some plaintext.

Poor John the ripper is still trying to crack the hash.. I forgot it :)
