# Maze Writeup
by Azumi Yasukohchi


https://app.hackthebox.com/challenges/Maze 

## Overview



## Category

Reversing

## Approach


**1. Setting up the HackTheBox environment**

 I am going to use Kali Linux for this CTF

In the description it said that I do not need to have connection to HackTheBox this time. 




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


**2. Unzipping and looking through the maze.zip**


Unzipped:

<img width="554" height="248" alt="image" src="https://github.com/user-attachments/assets/456f7193-6bfc-43ed-ac92-68ce851a26dd" />

There is another zipped file inside it so i tried unzipping it with same password "hackthebox" but did not work so I assume that is part of the challenge for this CTF.

<img width="754" height="593" alt="image" src="https://github.com/user-attachments/assets/e9eb95c8-724b-4db0-adaa-172c2b8b2b87" />



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



**3. Tackling maze.exe first **

started with maze.exe because executables almost always contain the main logic and tell you how other files like the png/zip are used, while those are usually just data.


So let's Run "file" and "strings" on maze.exe to learn what kind of files there are.

<img width="710" height="84" alt="image" src="https://github.com/user-attachments/assets/3e4042d4-032a-4a3d-b143-7ce9432ce388" />


<img width="359" height="52" alt="image" src="https://github.com/user-attachments/assets/2b15439d-3469-4cef-b862-37619ebe6bd8" />


<img width="628" height="854" alt="image" src="https://github.com/user-attachments/assets/1b4ea9be-bae7-4d05-b1c2-189d0abe4b82" />

This made me think that the "maze.exe" is a python app in a single exe which leads me to an idea of .exe being related to a packer like Pyinstaller or py2exe.

So I used "grep" for pyinstaller fingerprints.

<img width="706" height="215" alt="image" src="https://github.com/user-attachments/assets/1016348c-b9ef-424e-8db1-f16d5ef1203f" />


and this confirmed that there is higher possibility of pyinstxtractor being useful to explore further on maze.exe





and our clue to use pyinstxtractor.py to unpack it. After unpacking, running file maze.pyc tells you it’s “python 3.8 byte-compiled,” so you switch to Python 3.8 and use uncompyle6 with that version to decompile and continue


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

