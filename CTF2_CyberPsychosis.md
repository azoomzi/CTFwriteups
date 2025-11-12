# CyberPsychosis Writeup
by Azumi Yasukohchi



## Overview
https://app.hackthebox.com/challenges/Cyberpsychosis 

## Category
Reversing


## Approach


** 1. Setting up the HackTheBox environment **

Since the file inside the zip file was .ko which is for kernel module, I am going to use Kali Linux for this CTF

Inside that VM, go to https://app.hackthebox.com/challenges/Cyberpsychosis then START the instance. Then click on "Connect to HTB".
After that, I clicked on "starting point", then selected "openvpn".


I downloaded the openvpn file. I moved it to same directory as my CTF 2 folder.
Then I started the openvpn

<img width="888" height="245" alt="image" src="https://github.com/user-attachments/assets/5e255d4d-972d-4252-817a-abb21b6d5638" />


I made sure that my Kali VM is connected to HackTheBox by pinging the ip address that was provided.

<img width="219" height="88" alt="image" src="https://github.com/user-attachments/assets/3729d7d1-f789-4599-af4e-f2cfa143e5c8" />


<img width="822" height="151" alt="image" src="https://github.com/user-attachments/assets/775e90a6-532c-4a96-995b-63fae04dee6b" />



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


**2. Unzipping and looking through the CyberPsychosi.zip**

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


**4. Triggering the privilege escalation**


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

<img width="243" height="98" alt="image" src="https://github.com/user-attachments/assets/3e951f49-7d69-4dc3-8ef1-7b782d5f5e5e" />

<img width="541" height="144" alt="image" src="https://github.com/user-attachments/assets/4364ce67-fd5c-46ea-888d-447e641f638c" />

Seems that I will need to make the rootkit visible and remove the diamorphine in order for me to actually locate where the file exist.

<img width="599" height="323" alt="image" src="https://github.com/user-attachments/assets/ff67307b-60f7-41f5-97e2-c8f6f3e4427b" />


**5. Finding the flag**


Now we know that it is located in: /opt/psychosis/diamorphine.ko

We will go to that directory and see if there would be any hints or text file I can read.

<img width="602" height="192" alt="image" src="https://github.com/user-attachments/assets/370784de-805e-489d-94ea-dbcb00f57ce0" />

I found the flag! Thank you for reading!

