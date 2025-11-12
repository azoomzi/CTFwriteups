# CyberPsychosis Writeup
by Azumi Yasukohchi



## Overview
https://app.hackthebox.com/challenges/Cyberpsychosis 

## Category
Reversing


## Approach

**1. Unzipping and looking through the CyberPsychosi.zip**
Since the file inside the zip file was .ko which is for kernel module, I am going to use Kali Linux for this CTF.(Turned out later that I specifically needed VM running 5.15 (like Ubuntu 22.04.5)

<img width="629" height="252" alt="image" src="https://github.com/user-attachments/assets/3eeb04a7-67c0-49b1-99da-513b02af2bbb" />


Unzipped:

<img width="548" height="513" alt="image" src="https://github.com/user-attachments/assets/048a1a7e-64f0-4acf-b33e-26c9159e4c8a" />




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



**2. Understanding general concept of "diamorphie.ko"**

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



**3. Looking through functions inside the "diamorphie.ko"**

In Step 2, I discovered that the file was a non-stripped ELF file. So I will list allthe functions inside.

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


**4. Triggering the privilege escalation**


Now that we know init_module() installs the hook, and hacked_kill() listens for signal 63 to trigger give_root(), we can exploit the rootkit manually to get root access. 


HOWEVER, problem occured.
<img width="681" height="95" alt="image" src="https://github.com/user-attachments/assets/22bca2bd-c284-46df-8718-458413d4e807" />
My kali vm im using does not support the kernel module diamorphine.ko...
So I downloaded Ubuntu 22.04.5 specifically.
Now I try again.

<img width="900" height="110" alt="image" src="https://github.com/user-attachments/assets/854f7fe7-7897-4f6a-bbb3-323a5c7cef96" />

Stlll was unable to initiate the module so had to install build essentials and headers.
Then cloned from github directory to my directory

<img width="741" height="213" alt="image" src="https://github.com/user-attachments/assets/6ada491a-8685-40ec-8474-38f9a4095330" />

<img width="902" height="313" alt="image" src="https://github.com/user-attachments/assets/58ee80b3-8edd-4d68-ac86-438e0104c791" />

Then I recompiled and created new diamorphine.ko that matched my kernel version.
I was finally able to initiate the module.

<img width="881" height="114" alt="image" src="https://github.com/user-attachments/assets/9255406e-25f6-4e61-bb01-07056216b6ed" />

Diamorphiine is stealthy so regular listing did not work. Had to do dmesg.

<img width="750" height="92" alt="image" src="https://github.com/user-attachments/assets/9e01de4f-ffea-4c33-8f30-29f0d1d765f5" />


I was finally able to see it being loaded


<img width="449" height="52" alt="image" src="https://github.com/user-attachments/assets/8cb2a1d2-fc7c-4717-8a53-c87674ec7a83" />
Now I did this to see the PID

<img width="537" height="64" alt="image" src="https://github.com/user-attachments/assets/c3caea22-3746-4d2a-aeb1-726576275258" />

Then I triggered the rootkit by kill -64 with the PID i found.
<img width="537" height="64" alt="image" src="https://github.com/user-attachments/assets/dec4dbf2-836f-431b-9d1a-f38398638ddd" />




