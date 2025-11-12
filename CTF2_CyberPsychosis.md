# CyberPsychosis Writeup
by Azumi Yasukohchi



## Overview
https://app.hackthebox.com/challenges/Cyberpsychosis 

## Category
Reversing


## Approach


**1. Setting up the HackTheBox environment **

Since the file inside the zip file was .ko which is for kernel module, I am going to use Kali Linux for this CTF.(Turned out later that I specifically needed VM running 5.15 (like Ubuntu 22.04.5). So some of the screenshot will be shown from kali vm.

Inside that VM, go to https://app.hackthebox.com/challenges/Cyberpsychosis then START the instance. Then click on "Connect to HTB".
After that, I clicked on "starting point", then selected "openvpn".


I downloaded the openvpn file. I moved it to same directory as my CTF 2 folder.
Then I started the openvpn

<img width="888" height="245" alt="image" src="https://github.com/user-attachments/assets/5e255d4d-972d-4252-817a-abb21b6d5638" />


I made sure that my Ubuntu VM is connected to HackTheBox by pinging the ip address that was provided.

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

<img width="911" height="128" alt="image" src="https://github.com/user-attachments/assets/bc727667-86da-4fee-b8fe-302c7dbe2cea" />

Now I am connected to the HackTheBox Target machine.

We will escalte to root priviledge by using what I learned in step 3.

- "kill" the process
  
<img width="243" height="98" alt="image" src="https://github.com/user-attachments/assets/3e951f49-7d69-4dc3-8ef1-7b782d5f5e5e" />

- 



I tried to locate where the diamorphine.ko file is inside the target machine since we need to initate the module there.

```
find / -type f -name "diamorphine.ko" 2>/dev/null
```

<img width="482" height="88" alt="image" src="https://github.com/user-attachments/assets/5ccee706-6048-4d1d-a3dd-f4de31df3cad" />

Now we know that it is located in: /opt/psychosis/diamorphine.ko

We will go to that directory and see if there would be any hints or text file I can read.

<img width="728" height="302" alt="image" src="https://github.com/user-attachments/assets/6fa7fb09-48e4-46ec-90b9-cc656c37e071" />

<img width="690" height="86" alt="image" src="https://github.com/user-attachments/assets/29d4b12b-059a-465e-aa81-183ad80a43c4" />

Thanks for reading!

============================================================================================================================

HOWEVER, problem occured. SKIP THIS PART IF YOU PROPERLY DOWNLOADED UBUNTU.

<img width="681" height="95" alt="image" src="https://github.com/user-attachments/assets/22bca2bd-c284-46df-8718-458413d4e807" />
My kali vm im using does not support the kernel module diamorphine.ko...
So I downloaded Ubuntu 22.04.5 specifically.
Now I try again.

<img width="900" height="110" alt="image" src="https://github.com/user-attachments/assets/854f7fe7-7897-4f6a-bbb3-323a5c7cef96" />


I had to change to 5.15.0-82 generic inside my ubuntu to actually run it.

<img width="902" height="189" alt="image" src="https://github.com/user-attachments/assets/56dab739-9056-4d42-b191-0bdf5e8a1645" />

this has to match

```
sudo apt install linux-image-5.15.0-82-generic linux-headers-5.15.0-82-generic
```

Then reboot and go to GNU GRUB screen then select Advanced option for ubuntu 
<img width="663" height="622" alt="image" src="https://github.com/user-attachments/assets/97a93678-9913-4492-aeef-36853ac8325b" />
Then select Select Ubuntu, with Linux 5.15.0-82-generic, hit enter.


======================================================================================================================

I ran it again and this time there was no error

<img width="844" height="48" alt="image" src="https://github.com/user-attachments/assets/5f37d179-dab8-4f3d-b0ca-74537423f88b" />

This confirms that I was able to load the module.

I was finally able to initiate the module.

<img width="881" height="114" alt="image" src="https://github.com/user-attachments/assets/9255406e-25f6-4e61-bb01-07056216b6ed" />

Diamorphiine is stealthy so regular listing did not work. Had to do dmesg.

<img width="899" height="121" alt="image" src="https://github.com/user-attachments/assets/db3ba20d-8b73-42bc-8e9e-97d758b6efd1" />

I was finally able to see it being loaded


<img width="449" height="52" alt="image" src="https://github.com/user-attachments/assets/8cb2a1d2-fc7c-4717-8a53-c87674ec7a83" />

Now I did this to see the PID

<img width="537" height="64" alt="image" src="https://github.com/user-attachments/assets/c3caea22-3746-4d2a-aeb1-726576275258" />

Then I triggered the rootkit by kill -64 with the PID i found.

<img width="756" height="64" alt="image" src="https://github.com/user-attachments/assets/f54307d7-4530-4247-866f-ed7e3ad1b462" />

Ran this to reveal the module (they modified it from kill -63 to kill -46). (all of these numbers were based on my research from the creator's feature page: https://github.com/m0nad/Diamorphine#features)

<img width="757" height="86" alt="image" src="https://github.com/user-attachments/assets/837df789-9e60-45af-a961-d3c79f3eff79" />

Then confirmed that the rootkit is gone

**5. Finding the flag**




