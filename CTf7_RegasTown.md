# Rega's Town Writeup
by Azumi Yasukohchi

https://app.hackthebox.com/challenges/Rega's%2520Town 

## Overview


## Category


## Approach


**1. Setting up the HackTheBox environment**


I am going to use Kali Linux for this CTF. You do not need connection to HTB for this CTF so we are skipping the connect to openVPN part.

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

<img width="574" height="280" alt="image" src="https://github.com/user-attachments/assets/a3ac7198-89de-47c1-974a-f72337155950" />





Unzipped:

<img width="617" height="292" alt="image" src="https://github.com/user-attachments/assets/b5efb023-c9ea-44fc-8df3-9d3abdcedaa6" /> <img width="603" height="348" alt="image" src="https://github.com/user-attachments/assets/628f2071-02a9-461a-8eb7-7806dac000eb" />




It looks like the file does not have any extention.

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



**3. Understanding general concept of "rega_town file"**

I wanted see what kind of file actually this is.

<img width="951" height="88" alt="image" src="https://github.com/user-attachments/assets/6ad9914b-6faa-4a18-8959-be52399cd29c" />

This file can be executable.


I decided to run it to see what the program does.
I needed to change the permission to be able to run it so
<img width="233" height="71" alt="image" src="https://github.com/user-attachments/assets/4d9ed2f3-0586-434d-9d04-56becc65a5f2" />

then:
<img width="301" height="118" alt="image" src="https://github.com/user-attachments/assets/88f9c25f-cb07-48f3-9344-d2f9fc12fd02" />




Since it was executable file, I decided to analyze it through Ghidra


I went to "Defined strings" and seached for "Welcome to" as that is where the code starts.

<img width="962" height="484" alt="image" src="https://github.com/user-attachments/assets/a953fee2-2e62-4017-ab63-28dfb82bb187" />



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

