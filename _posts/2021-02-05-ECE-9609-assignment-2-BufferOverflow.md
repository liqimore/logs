---
layout: post
title: "ECE9609 - assignment 2, PicoCTF buffer overflow challenges"
description: "ECE9609 assignment 1."
date: 2021-02-05
tags: [uwo, ECE9609]
categories: [WesternU, hacking]
comments: true
---

# Assignmnet 2

Starting from assignment 2, I'll using a kali linux VM to do the homework.  

---

* table of contents
{:toc .toc}


## Question 1 - File Permissions

Following the steps:  

> Daddy told me about cool MD5 hash collision today.
> I wanna do something like that too!
>
> ssh col@pwnable.kr -p2222 (pw:guest)

Login pwnable's server through port 2222:  

<img src="/assets/images/202102/image-20210202172730531.png" alt="image-20210202172730531"  />

Checking the file permissions:  

<img src="/assets/images/202102/image-20210202172846806.png" alt="image-20210202172846806"  />

The answers to the questions:  

1. Which user owns the file `col`?

   `col_pwn` owns the file col.

2. Which files in this directory can the users in the group `col` read from?

   `col` and `col.c` can be read by the users in group `col`. Current user is in the group `col` , so the current user can `r-x`, which are reading and executing to the file. `col.c` has been set that everyone is readable(`r--`). 

3. What does the [SUID](https://www.linuxnix.com/suid-set-suid-linuxunix/) flag do?

   `SUID` is a type of special permissions, files with `SUID` permission could be excuted as if the owner is excuting it. `s` indicates the file has `SUID` permission.  

   In my opinion, `SUID` is normally used by the command, for example: while a none root user change its password, he need to use command `passwd`, which has `SUID` permission to modify the password file(`/etc/passwd`).

   <img src="/assets/images/202102/image-20210202175109643.png" alt="image-20210202175109643"  />

4. What exactly does `-r-sr-x---` tell us about the file `col`? Be sure to explain who is allowed to do what.

   in `-r-sr-x---`, the first `-` tells us it is a file. 

   **For the owner of the file**:  `r-s` is the permissions the owner has, means `read` and `SUID`, allow other user execute it with the owner's permission.  

   **For the users in group** `col`: they have `r-x` permission, which is `read` and `excute`. 

   **For current logged in user col**: The current user is in the group `col`, so he can `read` and `excute` the file. 

   **For other users(root not included)**: no permission allowed

The following pictures I found at http://www.csit.parkland.edu/~smauney/csc128/permissions_and_links.html and https://pamirwebhost.com/check-linux-file-permissions-with-ls/ are very helpful with leaning the linux file permissions:  

<img src="/assets/images/202102/image-20210202182208298.png" alt="image-20210202182208298"  />

<img src="/assets/images/202102/image-20210202182304797.png" alt="image-20210202182304797"  />



## Question 2 - Basics of C

Steps:  

Create the source file by using `vi flag.c` and paste the code into the file:  

![image-20210202181036605](/assets/images/202102/image-20210202181036605.png)

Then save the file (`:wq`):  

![image-20210202181124322](/assets/images/202102/image-20210202181124322.png)

Compile the source code by `gcc flag.c -O ex_flag`, compile and save the binary to `ex_flag` and finally, execute it:  

![image-20210202181518796](/assets/images/202102/image-20210202181518796.png)

So for the questions:  

1. What is the flag?

   The program outputs `Here's your HINT`, and the `flag = HINT`. 

2. What command(s) did you use to compile and run this program?

   I used `gcc` to compile the source code. After compiling it, the output file will automatically has the execute permission, so I execute using `./theFileName`. 



## Question 3 - Basics of Computer Memory

**What I understand about memory**: 

1. in a X86 32-bit system, every process has a 32-bit long virtual address and it has an offset compared to the physical address.

2. in a X86 32-bit system, virtual address space is allocated to two part, kernel space and user space. Kernel space normally takes up to 1G space at high address, the rest is user space which is 3G.

3. Each process has the following memory space:  

   ![image-20210203012733285](/assets/images/202102/image-20210203012733285.png)

   *Source: https://www.cnblogs.com/chaozhu/p/6377485.html* 

   On linux system, we can take a look at the memory map:  

   ![image-20210203013259642](/assets/images/202102/image-20210203013259642.png)

And at the bottum of the memory is stack:  

![image-20210203013354359](/assets/images/202102/image-20210203013354359.png)



**What I understand about endianness**:

Endianness defines how a operating system read memory, from high address to low address or the oppsite order. 

Answers to the questions:

1. What is the number 3735928559 in hexadecimal form?

   **0xDEADBEEF** (I used an online converter)

2. Suppose this number was stored as an integer (i.e., `int` type) in little-endian format at memory address **0x12345678**. Fill in the following memory map showing where each byte is stored. If the value is unknown/not relevant, leave it as `0x??`.

   First of all, **little-endian** means the **last byte** of the data is stored at the **lowest address**. In terms of `0xDEADBEEF`, the least significant is `EF `(the last). Because the number is stored at memory address `0x12345678`, the start of the memory is `0x12345678 `(lowest address). So, `EF` should be placed at address `0x12345678` . 

   ```c
   Address     | Value
   -------------------------
   ...         |
   0x12345674  | 0x??
   0x12345675  | 0x??
   0x12345676  | 0x??
   0x12345677  | 0x??
   0x12345678  | 0xEF
   0x12345679  | 0xBE
   0x1234567a  | 0xAD
   0x1234567b  | 0xDE
   ```

   

## Question 4 - Collision Challenge

First login the server:  

`ssh col@pwnable.kr -p2222`

After login, I checked the directory using `ls`, I find that there's a file called flag, it's possible where the flag is. My mission is to access the file. Then I checked flag file's permission using `ls -l` .

![image-20210203022031664](/assets/images/202102/image-20210203022031664.png)

So, flag is owned by col_pwn. Before I tried other stuff, I checked if my current user is on the sudoer list using `sudo cat flag`.

![image-20210203022229551](/assets/images/202102/image-20210203022229551.png)

But I'm not a sudoer. I have a new idea while dealing with sudo, on January 26, 2021, someone found sudo has a buffer overflow bug, which is exploitable by any user on the machine. So, I look this up on Google and find this article https://www.sudo.ws/alerts/unescape_overflow.html. Here's a test to check if the machine has the bug:  

>  To test whether your version of sudo is vulnerable, the following command can be used:
>
> ```
> sudoedit -s /
> ```
>
> A **vulnerable** version of sudo will *either* prompt for a password or display an error similar to:
>
> ```
> sudoedit: /: not a regular file
> ```
>
> A **patched** version of sudo will simply display a usage statement, for example:
>
> ```
> usage: sudoedit [-AknS] [-a type] [-C num] [-c class] [-D directory] [-g group]
>                 [-h host] [-p prompt] [-R directory] [-T timeout] [-u user]
>                 file ...
> ```
>
> If the sudoers plugin has been patched but the sudo front-end has not, the following error will be displayed:
>
> ```text
> sudoedit: invalid mode flags from sudo front end: 0x20002
> ```

Here's what i find: 

![image-20210203030309869](/assets/images/202102/image-20210203030309869.png)

**And the OS has been patched.** 

Next, I opened col.c to see the code using `vi col.c` :

![image-20210203024316057](/assets/images/202102/image-20210203024316057.png)

So, the question became how to make `0x21DD09EC` equal to `check_password(INPUT)`. 

After reviewing the `check_password()` function, it takes the char input, force convert to int, copy it to the location of pointer `ip`, then add them together like numbers. From the notice, I know the input is 20-bytes long. And one int takes up 4 bytes, so it's 20/4=5 numbers I need know. 

Now I convert `0x21DD09EC` to decimal(`568134124`) to figure out the 5 numbers.  However, 568134124 cannot be divided by 5. 

![image-20210203031433211](/assets/images/202102/image-20210203031433211.png)

So, I just take the integer part `113626824` and mutiply it with 4 (I also could set the first 4 number as 113626825, the rest is the same) , `113626824 * 4 = 454507296`. Now I have 4 numbers, the rest value would be the fifth number. `568134124 - 454507296 = 113626828`. Check if this is correct, `113626824 * 4 + 113626828 = 568134124`. 

Now I have the 5 numbers: `113626824  -  113626824  -  113626824  -  113626824  -  113626828`

In hex format: `0x06C5CEC8 0x06C5CEC8 0x06C5CEC8 0x06C5CEC8 0x06C5CECC`

Finally, we input the numbers like the course website(https://whisperlab.org/introduction-to-hacking/lectures/collision):  

**./col $'\x06\xc5\xce\xc8\x06\xc5\xce\xc8\x06\xc5\xce\xc8\x06\xc5\xce\xc8\x06\xc5\xce\xcc'**

Surprisingly, the answer is wrong:  

![image-20210203033338471](/assets/images/202102/image-20210203033338471.png)

I stucked here, so I googled. In this article https://0xrick.github.io/pwn/collision/#Exploitation, it says that the system is little-endian, I'm not sure where this is come from. I did not find anything related to this. So, I use a python program to check if the system is little-endian. 

`python -c "import sys;print(sys.byteorder=='little')"`

And it is little-endian: 

![image-20210203034220293](/assets/images/202102/image-20210203034220293.png)

So, I need to reverse the input and try again:

**./col $'\xc8\xce\xc5\x06\xc8\xce\xc5\x06\xc8\xce\xc5\x06\xc8\xce\xc5\x06\xcc\xce\xc5\x06'**

Eventually, I have the flag:  

![image-20210203034418244](/assets/images/202102/image-20210203034418244.png)

Flag: **daddy! I just managed to create a hash collision :)**



## Question 5 - bof Challenge

The challenge asks me to download 2 files, so I did. And it seems `bof` is a executable file, so I give it excute permission and run it:  

![image-20210203034841156](/assets/images/202102/image-20210203034841156.png)

Ok, so `nc pwnable.kr 9000` is running the same program:  

![image-20210203035007785](/assets/images/202102/image-20210203035007785.png)

The next step is to analysis the source code, I use vim to see the code, `vi bof.c`.

![image-20210203143421150](/assets/images/202102/image-20210203143421150.png)

so, `main()` calls `func()` then exit, the logic is in function `func()`. It has an integer input `key`, then compare the key with `0xcafebabe`. **Before comparison, the program requires a std input from keyboard and save it to a 32 bytes char variable `overflowme`, so it's possible to overflow variable `overflowme` and write the data (`0xcafebabe`) to the address where variable `key` is.**  This will make the varibale `key` equal to `0xcafebabe` and calls `system("/bin/sh")` then enter a new bash. (`gets()` function in C language does not validate the memory, it just wirte everything it gets to the memory address)

I'll try to get the exploit on my local kali linux and then use it on pawnable.kr's server. This time, I'll only use gdb to debug the program, but I could also use a gun program on kali linux to debug it. In order to debug c program with GDB, I need to re-compile `bof.c` with flag  `gcc -g3 -m32 bof.c -o new_bof` . `-g3` means compile full debug information to the binary(It's ok to use g3, because it's a small program, while debug complex program, -g is much preferred). `-m32` means compile it to 32-bit binary. 

But it did not work:  

![image-20210203150530376](/assets/images/202102/image-20210203150530376.png)

It seems that I don't have `stdio.h` header, and this is impossible on a linux machine with gcc. So, I googled the error:

![image-20210203150731575](/assets/images/202102/image-20210203150731575.png)

https://stackoverflow.com/questions/54082459/fatal-error-bits-libc-header-start-h-no-such-file-or-directory-while-compili/54082790

ok, I need to install 32-bit library manully. **Kali linux use apt to manage the packages**, so I only need to `apt-get install gcc-multilib` to install mulilib. 

![image-20210203151015221](/assets/images/202102/image-20210203151015221.png)

Then run gcc to compile it again, and it worked just fine:

![image-20210203151052880](/assets/images/202102/image-20210203151052880.png)

 Great! GDB is not installed too:

![image-20210203151346415](/assets/images/202102/image-20210203151346415.png)

After install GDB, I can finally debug the program:  

![image-20210203151452780](/assets/images/202102/image-20210203151452780.png)

Then I use `run` to start the program and try something random just to play with it.

![image-20210203151914488](/assets/images/202102/image-20210203151914488.png)

I take a look at the source code, I understand I need to set a breakpoint at the comparison to check the value in `key`.  (it's just partial code in the picture)

![image-20210203152416042](/assets/images/202102/image-20210203152416042.png)

Now run the program, I put a long string to overflow it:

![image-20210203153153702](/assets/images/202102/image-20210203153153702.png)

So, the key's value is replaced with `0xffffd180`, I tried again without overflow:

![image-20210203153222389](/assets/images/202102/image-20210203153222389.png)

Now I know the buffer overflow could work on this program. So, I use an online tool genrate a string(https://wiremask.eu/tools/buffer-overflow-pattern-generator/) which is very easy to use on buffer overflow. What I used here is `Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag`

I could use the string to find where is target place.

![image-20210203154029685](/assets/images/202102/image-20210203154029685.png)

I know it overflows the varibable at the start of `Ab` . Next is to find the content of variable `key`, then I know the place to overflow key's data. 

![image-20210203154931846](/assets/images/202102/image-20210203154931846.png)

I learned to print the key as a sequence of 4 characters on the class tutorial notes. 

Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6A|**KEY**|b7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag

The bold words are where key is. Now I can set any value I want to key. 

So, the expoit string is `Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6A + \xbe\xba\xfe\xca`. 

Since it doesn't matter what content is in the overflow variable, so I replace the long string with one char `q`.

Now I have the overflow string, so I'll use a python script to help me input it:

```
(python -c "print 'q'*52 +'\xbe\xba\xfe\xca'"; cat) | nc pwnable.kr 9000
```

This pythin script will input the data to pwnable.kr's 9000 port. It's like running a local program `./bof`. The difference is piped the data to a remote server.  

![image-20210203162452643](/assets/images/202102/image-20210203162452643.png)

Finally, I have the user `col`'s  shell and use it to get the flag.

And the flag is `daddy, I just pwned a buFFer :)`


