# Heap0PicoCTF Writeup
by Azumi Yasukohchi


https://app.hackthebox.com/challenges/Cyberpsychosis 

## Overview


## Category
Reversing


## Approach

**1. Setting up the picoCTF environment**
Downloaded the files from the website and put it into my folders.

<img width="443" height="210" alt="image" src="https://github.com/user-attachments/assets/d03cf3a0-9f29-484f-873e-1408cd117e70" />


**2. Understanding general concept of "chall.c"**

<img width="417" height="192" alt="image" src="https://github.com/user-attachments/assets/fea90ab3-c350-4f44-a93b-f8a7e7c55095" />

From this image, I can see that as long as I change any letters on bico, it will be considered as unsafe and will print the flag.txt















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



**3. Understanding general concept of "diamorphie.ko"**

I wanted see what kind of file actually this is.
<img width="1010" height="115" alt="image" src="https://github.com/user-attachments/assets/33c0a7cf-c07a-46fb-899e-4cd6073dba4e" />

It showed that it is a ELF file with debug info + not stripped. This will allow me to explore the function names.

I also ran strings on the kernel module to see what human-readable data would be inside.
<img width="733" height="917" alt="image" src="https://github.com/user-attachments/assets/3dba38a4-c0cd-4b45-ab02-701da51b7b0b" />
<img width="232" height="30" alt="image" src="https://github.com/user-attachments/assets/22074b0f-ee04-4cc7-9c95-c77b070a3590" />
<img width="127" height="39" alt="image" src="https://github.com/user-attachments/assets/b2de26d3-8f2e-456e-abde-5f7bfbf7a478" />
<img width="170" height="36" alt="image" src="https://github.com/user-attachments/assets/d88d9252-f1e4-44f6-9f75-ed931de5c9bc" />


We can see that the .ko file is a rootkit that can escalate privileges from "commit_creds, prepare_creds, register_kprobe"

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



**4. Looking through functions inside the "diamorphie.ko"**

In Step 2, I discovered that the file was a non-stripped ELF file. So I will list all the functions inside.

<img width="718" height="275" alt="image" src="https://github.com/user-attachments/assets/d6765b40-0008-44b8-9707-e59e7cf230ec" />

Reasearching important functions
The function init_module() sets everything up when the module is loaded using insmod. It hooks into the system and installs the malicious logic by replacing normal syscalls with its own functions.
One of those is hacked_kill, a syscall hook that listens for special signals like kill -64. When the module receives signal 64, it does not actually kill the process. Instead, it calls give_root(), which escalates the current process to root privileges (UID 0).


References
https://linux.die.net/man/2/init_module
https://github.com/m0nad/Diamorphine#features
https://dirtycow.ninja/ 

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


**5. Triggering the privilege escalation**


Now that we know init_module() installs the hook, and hacked_kill() listens for signal 64 to trigger give_root(), we can exploit the rootkit manually to get root access. 

We will ncat to the HackTheBox IP address.

```
nc <target-ip> <port>
```

<img width="529" height="110" alt="image" src="https://github.com/user-attachments/assets/3aa2c1bb-2f02-4e30-b840-1e6539fb23c9" />

Now I am connected to the HackTheBox Target machine.

I tried to locate where the diamorphine.ko file is inside the target machine since we need to initate the module there.

```
find / -type f -name "diamorphine.ko" 2>/dev/null
```

<img width="470" height="76" alt="image" src="https://github.com/user-attachments/assets/67f8ac8e-f51b-4a49-bc3e-8475a5686933" />

Without root priviledge we can not use /dev/null.

We will escalte to root priviledge by using what I learned in step 3.

- "kill" the process and gain root priviledge

<img width="542" height="188" alt="image" src="https://github.com/user-attachments/assets/a217a36d-4983-44d2-912d-59de202b5953" />


<img width="541" height="144" alt="image" src="https://github.com/user-attachments/assets/4364ce67-fd5c-46ea-888d-447e641f638c" />

Seems that I will need to make the rootkit visible and remove the diamorphine in order for me to actually locate where the file exist.

<img width="599" height="323" alt="image" src="https://github.com/user-attachments/assets/ff67307b-60f7-41f5-97e2-c8f6f3e4427b" />


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


**6. Finding the flag**


Now we know that it is located in: /opt/psychosis/diamorphine.ko

We will go to that directory and see if there would be any hints or text file I can read.

<img width="602" height="192" alt="image" src="https://github.com/user-attachments/assets/370784de-805e-489d-94ea-dbcb00f57ce0" />

I found the flag! Thank you for reading!

