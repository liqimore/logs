---
layout: post
title: "ECE9609 - Notes for Sudo Heap-based Buffer Overflow (CVE-2021-3156)"
description: "Sudo Heap-based Buffer Overflow (CVE-2021-3156)"
date: 2021-03-20
tags: [hacking, ECE9609, uwo]
categories: [uwo]
comments: true
---

Sudo Heap-based Buffer Overflow (CVE-2021-3156)

* table of contents
{:toc .toc}

# Background

Common Vulnerabilities & Exposures, so-called CVE, is a dictionary of system vulnerabilities that has been disclosed to the public. Normally, it consists of CVE-ID, a description, and a list of references. Specifically speaking, the CVE-ID specifies the identity of a particular CVE, the description field explains the detail of this CVE, and the references list all reports from each department that found this CVE. In this presentation, we are going to explore the latest CVE, CVE-2021-3156. It reported that the command “sudoedit -s” and any command that ends with a single backslash character will mistakenly promote the user’s permission as well as the root. 

# Methodology

## 2.1 **Vulnerability abstract** 

According to what the report declared, if users who are not supposed to obtain the administrative-level permission find this vulnerability and successfully exploit it to elevate their privileges, they will be able to execute system commands which should only be executed by administrators or some particular users. take a one step further, it will then cause information leakage and malicious tampering. 

## 2.2 **Vulnerability analyzation** 

When applying sudo to run commands in "shell" mode, there are two options, which are -i and -s. sudo dash s option will set sudo's MODE_SHELL flag, whereas sudo -i will set both MODE_SHELL and MODE_login_shell flag. 

Here is the code segment from sudo.c's main function, it invokes parse_args function, which parses command line input into appropriate data structures. 

<img src="/assets/images/202103/1r18oADTLzjfvVQ65EtJY8QirUKFktxJxD61rCHfFLt3bl3ydLDtpxku4lmSx1GI-b5_9ghrinbjz2vZY0gMYN2M2Vu2JvfeZbDoiU23O8uC4Ft4K_mcC8D40ijbJi8-t5VXItz0" alt="img" style="zoom:80%;" />

The next code segment determines whether Mode_shell with -s or -i is activated. If one of them is chosen, it will rewrite argv by adding a backslash to the meta-characters. 

<img src="/assets/images/202103/AAg3lf5MpdlRbh2G0HRO1EN4TkACGASqCoRPtELjUtFgnfnJJg8jv-KzSGJkVziPTPKlikJ9gvEaIncobqEpnmOL54q6_DsToqXlJ8GjkXxm_tkdPhEUf9CAnwKKFUFf9LSzwnru" alt="img" style="zoom:80%;" />

Then, as the next code segment shows, it will invoke set_cmnd() function. The set_cmnd function will first calculate the size of input via strlen() function, then invokes mallc() function to allocate a buffer which is named user_args, each has a size of size. last, it will determine whether Model_shell is activated, if yes, it will connect command line input and store it into user_args. 

![img](/assets/images/202103/QnUj10UZoAFtV28mqivgruWVLowIe6QeGHxi1xvBN8ZQNNbuvSVyW7SWJUb0voJ2TAo44YuXr-yDETyfTOqK3Ca0llzjLfLznyokUhEbCyCYhcGMich_L0sJa95gPvshUsNtc2rG)

The next code segment is where it causes the problem. This is the code segment that set_cmnd() stores the command line input into user_args. For example, if we input sudo -s / 112233. According to the code, from[0] will be the backslash character, and from[1] is going to be the argument's null terminator. The null terminator is not a space character and it is not visible here. null terminator has it’s all bits set to zero and it is used to represent the end of a string of characters. As it fulfills all requirements here, the array variable "from" is going to be incremented and point to the null terminator now. Then, as next from[0], this null terminator is going to be stored into the "user_args" buffer and array variable "from" is going to be incremented again and point to the next character after the null terminator, which will cause it to be out of the argument's bounds and keeps doing it because it is inside a while loop. put it another way, since the size of the "user_args" buffer is calculated at the beginning of set_cmnd function and it has a limited size, but it keeps store out of bound characters into this buffer, it gives rise to a heap-based buffer overflow. 



<img src="/assets/images/202103/nbyBysGHcbo_jkep-kdgzxe-PhcnA3V8RL5aT-RGt-HvPkhMCK1pbA0ZnHlP72s18qTj13AxSC2PCYq1Guc-MK6eG1uRh5KMRMWyhDwN9sq2AMF8Se4VN5V4snuggYS2tDpFHCo7" alt="img" style="zoom:80%;" />

However, logically, if the Mode_shell or mode_login_shell is set, command line input cannot end with a backslash character because the set_cmnd function will determine whether model_shell, mode_edit, or mode_check is activated. If MODE_SHELL is activated, the parse-args function will then parse all command-line input, which will change single backslash to double backslash. Therefore, instead of sudo, attackers can execute sudo-edit. Sudo-edit will trigger sudo, but will not reset valid_flags and Mode_run. Therefore, attackers are able to skip the parsing part of the parse_args function, therefore, keep a single backslash, and finally trigger the heap_based overflow. 



# Online Video

We'll demo the CVE during the class, however, here's what we find great demo on the Internet. You could take it as a reference as ours.

**First one:**    
<iframe width="560" height="515" src="https://www.youtube.com/embed/hZg1OoyqXhs?start=38" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

**Second one:**       

<iframe width="560" height="515" src="https://www.youtube.com/embed/Cqom0wGyhGg" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



# Reference

1. https://www.youtube.com/watch?v=2_ZaNBl6qNo This is a very detailed explanation about the CVE, recommend to watch it if you want to learn more

2. https://github.com/blasty/CVE-2021-3156 The exploit we’ll use in this video.   

   Other stuff you might find useful

3. https://www.qualys.com/2021/01/26/cve-2021-3156/baron-samedit-heap-based-overflow-sudo.txt

4. https://datafarm-cybersecurity.medium.com/exploit-writeup-for-cve-2021-3156-sudo-baron-samedit-7a9a4282cb31