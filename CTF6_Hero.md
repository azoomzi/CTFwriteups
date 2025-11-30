# Hero Writeup

by Azumi Yasukohchi

https://cybertalents.com/challenges/malware/hero 



## Overview



## Category

Malware Reverse Engineering 


## Approach

**1. Setting up the picoCTF environment**
Downloaded the files from the website and put it into my folders.

<img width="443" height="210" alt="image" src="https://github.com/user-attachments/assets/d03cf3a0-9f29-484f-873e-1408cd117e70" />






............................................................................................................................
.
.
............................................................................................................................
.
.
.............................................................................................................................
.
.
............................................................................................................................



**2. Understanding the Heap Layout of "chall.c"**

Before exploiting anything, I wanted to understand where these variables actually live in memory. The challenge says “heap0”, so it will be both variables (input_data and safe_var) living on the heap, not the stack. 

I looked throught the file chall.c and here are information that I thought it was helpful to win the flag.


#1. The check that decides if you win

<img width="367" height="58" alt="image" src="https://github.com/user-attachments/assets/481ec4b3-21d5-4a16-8e25-9bff7428b348" />


From this image, **i can see that if safe_var does not eaqual to "bico",** it will print out the flag.txt. So as long as I change any letters on bico, it should print the flag.txt


#2. Vulenrability of scaf(%s)

<img width="291" height="103" alt="image" src="https://github.com/user-attachments/assets/7cb091dc-5b1d-44ca-9ee2-3b204b0d0d4c" />


In here, I can see that they used %s (format specifier for input/output) which reads untill there is a white space and writes them into a buffer. Hacker probably used scanf so they could use %s which is dangerous because it will allow user to write as many information as they want. 

If we go back to how much letters can input_data accepts, it is only 5 bytes long.

<img width="407" height="162" alt="image" src="https://github.com/user-attachments/assets/39b77fc0-cffc-4ec4-820f-cb4ce7682894" />

So because %s can accept more than 5 bytes, it can easily be overflow the input_data.


#3. The heap allocations being next to each other

<img width="441" height="79" alt="image" src="https://github.com/user-attachments/assets/8a27cddc-8457-4ac6-af96-f964a3048cd2" />

Because their allocation is next to each other, there is a high possibility of malloc giving input_data and safe_var address being next to each other. This can cause text being spilled out of input_data container and into safe_var.
Therefore, if you overflow input_data, it will also overflow safe_var.




............................................................................................................................
.
.
............................................................................................................................
.
.
.............................................................................................................................
.
.
............................................................................................................................



**3. Getting the flag**

I ncat the picoCTF server and accessed to the target machine

<img width="740" height="280" alt="image" src="https://github.com/user-attachments/assets/33382eb7-0e24-4bb1-8e64-dd30a0ba47b8" />


I tried overwriting the safe_var by writing few As and printed the current state of heap. It seems that did not overwrite to safe_var yet. Looking at both addresses, subtracting them would give me how far their addresses are from each other


<img width="962" height="808" alt="image" src="https://github.com/user-attachments/assets/0bfc3aeb-3702-4dee-90f8-1eed69451e04" />


  0x5a448142b2d0
- 0x5a448142b2b0
= 0x20 = 32 bytes apart

Therefore,

1 byte = 1 A
32 bytes = 32 As

I will try to type 32 As more and that should give me the flag.

<img width="962" height="551" alt="image" src="https://github.com/user-attachments/assets/1255c232-4a44-4e78-9437-c641900fbf1c" />

input_data leaked next to safe_var!

Now i got the flag. Thanks for reading!










