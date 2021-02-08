---
layout: post
title: "ECE9609 - assignment 1, PicoCTF easy challenges"
description: "ECE9609 assignment 1."
date: 2021-01-22
tags: [uwo, ECE9609]
categories: [WesternU, hacking]
comments: true
---

* table of contents
{:toc .toc}


# Web Exploitation
## dont-use-client-side
from the title of the challenge, I could know that the exploit is most likely on the client side, which is the web page of the challenge.   
After open the web page provided in the description, it's a simple login form asking for a credential. Since it's a web page, I open dev tool in Chrome to look at the source.   
**And then, I discoverd the following JS code**. The js is to valid the string input from the form. and valid the string by spliting the string.  So just append the string together would be the flag I need(*at the order of the spliting*).  
```javascript
function verify() {
    checkpass = document.getElementById("pass").value;
    split = 4;
    if (checkpass.substring(0, split) == 'pico') {
      if (checkpass.substring(split*6, split*7) == '723c') {
        if (checkpass.substring(split, split*2) == 'CTF{') {
         if (checkpass.substring(split*4, split*5) == 'ts_p') {
          if (checkpass.substring(split*3, split*4) == 'lien') {
            if (checkpass.substring(split*5, split*6) == 'lz_7') {
              if (checkpass.substring(split*2, split*3) == 'no_c') {
                if (checkpass.substring(split*7, split*8) == 'e}') {
                  alert("Password Verified")
                  }
                }
              }
      
            }
          }
        }
      }
    }
    else {
      alert("Incorrect password");
    }
    
  }
  ```
  the flag is `picoCTF{no_clients_plz_7723ce}`(the reassemble of the flag could be done by a simple program, but it's just very short string, I just glued it together by hand).   


## picobrowser
The description of the challenge tells that the web page can only be rendered by picobrowser. The webpage has a big green button with a `flag` text on it. Of course i click on it, and it reterns with this error:  

> You're not picobrowser! Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.141 Safari/537.36

The first step is check the html code. I found the button's html code but also a html a link `/flag`. So, I click on it.  
![flaglink](/assets/images/202101/Snipaste_2021-01-22_22-31-40.jpg)  
And it's the same page. I start wondering if the alert message isn't static. Then I opened the page with Safari, and it changed to this:  

> You're not picobrowser! Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.0.2 Safari/605.1.15

After seeing this, I understand that the web page actually valids the broswer information. In this case, by change the user agent from the http request will let the webpage thinks I'm using a different broswer. So, I use Postman to send to GET request with user agent `picobroswer`.  
![borswer](/assets/images/202101/Snipaste_2021-01-22_22-59-54.jpg)    

At the respon part of Postman, I click on preview, now I got the flag.  
![borswer1](/assets/images/202101/Snipaste_2021-01-22_23-02-10.jpg)    

The flag is `picoCTF{p1c0_s3cr3t_ag3nt_51414fa7}`.   

# Cryptography
## la cifra de
The challenge ask me to connect to a server, after connect to the server. I got this message:  
![msg](/assets/images/202101/Snipaste_2021-01-22_23-06-58.jpg)    

Clearly, I cannot read this. However, in the paragraph, the format of this string is quite like the flag `hgqqpohzCZK{m311a50_0x_a1rn3x3_h1ah3xf966878l}`. I know that picoCTF's flag has prefix of `picoCTF`. I start to think if `hgqqpohzCZK` has any connection to the prefix.   

First thing come to my mind is Caesar cipher, it's very common in movies. So, but the length of the `hgqqpohzCZK` is not match to the `picoCTF`. But I think the last three, `CZK` is `CTF`. Because this is the only capital letters in the string. But I didn't figure it out in the end.    

So, I used an online tool(www.boxentriq.com), the website has lots of code breaking tools including a text analysis tool(https://www.boxentriq.com/code-breaking/text-analysis) to check how the text are encoded.    

I put `hgqqpohzCZK` into it, it returns with a list of possible encription algothems in the order of possibility:    
1. Caesar Cipher
2. Polyalphabetic: Vigenère Cipher, Gronsfeld Cipher and Beaufort Cipher  

I already know caesar is not the one, then I start try one by one.     

The first one is Vigenère Cipher, using online tool(https://www.boxentriq.com/code-breaking/vigenere-cipher). I'm very lucky I get a hit with a key `flag`. The auto sover gave me the correct answer.     

![msg](/assets/images/202101/Snipaste_2021-01-22_23-33-17.jpg)    

Then i decoded `hgqqpohzCZK{m311a50_0x_a1rn3x3_h1ah3xf966878l}` with `flag` as key. But it's returns `bbfqjjwzWUZ{m311u50_0s_p1rh3s3_w1ab3su966878l}`, it not correct because i'm pretty sure that `CZK` means `CTF`.   

I got very confused. I tried to use the key to decode other paragraph and it was not working. So I put the whole article into the auto solver, each time, only one paragraph is correct.   

I did not know why but I put the paragraph with the flag into auto solver, and I get the flag. But the is not `flag` anymore, it's `gfla` now.    
![msg](/assets/images/202101/Snipaste_2021-01-22_23-44-08.jpg)     

Now I know what's going on, each paragraph has a different key to solve.  
And the flag is `halfpicoCTF{b311a50_0r_v1gn3r3_c1ph3ra966878a}`. CZK is actually means CTF.(** half must be deleted, picoCTF's has its format**. I tried twice because of this)     



## john_pollard
I have no idea how to solve this. I checked the hint at first, I know I need to submit the flag in this format `picoCTF{p,q}`, P and Q are used to generate the RSA key. If the flag not woking, swap P and Q.  

In terms of how RSA generates the key, I look up online from the same website I used before(https://www.boxentriq.com/code-breaking/rsa). Here's a screenshot from the website about RSA.   

![msg](/assets/images/202101/Snipaste_2021-01-23_00-04-56.jpg)    

The challenge is actually to get P and Q from a RAS certificate. In order to get P and Q, i need to know modulus`n` because `n = p * q`.    

After some search on google, I find out that the cert file already has modulus information in it and I could use `openssl` tool to check it out.  
By execute `openssl x509 -noout -text -in cert`, I have the modules number `n`. It's time to figured out p and q from n.    

![msg](/assets/images/202101/Snipaste_2021-01-23_00-31-59.jpg)    

The modulus is `4966306421059967`, now I need to find which two prime numbers could multiply to the modulus. I wrote a simple program to do this, however, it takes too long to run.   
```python
def isPrime(x):
    if (x == 2) or (x == 3):
        return True
    if (x % 6 != 1) and (x % 6 != 5):
        return False
    for i in range(5, int(x ** 0.5) + 1, 6):
        if (x % i == 0) or (x % (i + 2) == 0):
            return False
    return True


if __name__ == '__main__':
    modulus = 4966306421059967
    primeNumberP = 2
    primeNumberQ = 2

    while primeNumberQ < modulus:
        primeNumberQ = primeNumberQ + 1
        if isPrime(primeNumberQ):

            while primeNumberP < modulus:
                primeNumberP = primeNumberP + 1
                if isPrime(primeNumberP):
                    print("Now calculating --- " + str(primeNumberP) + "*" + str(primeNumberQ))
                    if primeNumberP*primeNumberQ == modulus:
                        print("P:" + str(primeNumberP) + "Q:" + str(primeNumberQ))
```

I don't know what to do next at the moment, then I look for answers to this challenge, and I find this article https://ctf.samsongama.com/ctf/crypto/picoctf19-john_pollard.html . FactorDB is introduced in the article. That's where I actually find the P and Q.  

![msg](/assets/images/202101/Snipaste_2021-01-23_01-20-00.jpg)     

So, the flag is `picoCTF{73176001,67867967}`.    

# Reverse Engineering
## vault-door-3
This is a relatively simple one, just reuse the `checkPassword()` method in the source code will reverse the password. Here's my code:   
```java
public static String decodePasswd(String password) {
        char[] buffer = new char[32];
        int i;
        for (i=0; i<8; i++) {
            buffer[i] = password.charAt(i);
        }
        for (; i<16; i++) {
            buffer[i] = password.charAt(23-i);
        }
        for (; i<32; i+=2) {
            buffer[i] = password.charAt(46-i);
        }
        for (i=31; i>=17; i-=2) {
            buffer[i] = password.charAt(i);
        }

        return new String(buffer);
    }

System.out.println(decodePasswd("jU5t_a_sna_3lpm18g947_u_4_m9r54f"));
```

The flag is `jU5t_a_s1mpl3_an4gr4m_4_u_79958f`.   

![msg](/assets/images/202101/Snipaste_2021-01-23_01-42-44.jpg)   


## vault-door-4
In this challenge, the `checkPassword()` method is not very long, after analysis the code, the table in the method is the flag. But it's encoded to ASCII. Here's the flag table:   
```java
byte[] myBytes = {
            106 , 85  , 53  , 116 , 95  , 52  , 95  , 98  ,
            0x55, 0x6e, 0x43, 0x68, 0x5f, 0x30, 0x66, 0x5f,
            0142, 0131, 0164, 063 , 0163, 0137, 070 , 0146,
            '4' , 'a' , '6' , 'c' , 'b' , 'f' , '3' , 'b' ,
        };
```

From the table, I know that the last line is likely the oringinal flag, the first like is the decimal number in ACSII table. So I only need to convert hexadecimal(second line) and octal(third line) to decimal.    

There are a lot of online tools, like ASCII,Hex,Binary,Decimal,Base64 converter (https://www.rapidtables.com/convert/number/ascii-hex-bin-dec-converter.html), I used this tools covert the table directly to the flag.   

First line: jU5t_4_b  
Second line: UnCh_0f_   
Third line: bYt3s_8f (Octal to decimal(98 89 116 51 115 95 56 102) to ASCII)   
Last line: 4a6cbf3b   

combined together:  `picoCTF{jU5t_4_bUnCh_0f_bYt3s_8f4a6cbf3b}`


# Forensics
## So Meta
The name tells everything. Every photo has metadata, the flag is possible in the metadata. Just upload the picture to an online tool(http://exif.regex.info/exif.cgi), the flag is in the PNG Artist metadata.   

![msg](/assets/images/202101/Snipaste_2021-01-23_02-11-19.jpg)      

So, the flag is `picoCTF{s0_m3ta_eb36bf44}`.   

## extensions
After downloading the txt file, this first thought is to open it with a text editor, and I got this:   

![msg](/assets/images/202101/Snipaste_2021-01-23_02-15-37.jpg)    

The first line says something `PNG`, think of the title `extensions`, this is probobably a PNG file. Then I change the extension to `.png`. And open it with `preview` photo viewer on MacOS.    

Here's the flag:  

![msg](/assets/images/202101/Snipaste_2021-01-23_02-18-03.jpg)    

`picoCTF{now_you_know_about_extensions}`    

# Summary
All of those challenges are very interesting. Every detail matters.

