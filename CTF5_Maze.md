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


and this confirmed that there is higher possibility of extracting pyinstaller being useful to explore further on maze.exe

I googled "pyinstaller extractor" and I found pyinstxtractor.py so I used it on maze.exe


<img width="703" height="268" alt="image" src="https://github.com/user-attachments/assets/0afd4d58-50a1-4c21-a6e0-37380b82866f" />

the extractor created maze.exe_extracted so I went in.


<img width="825" height="237" alt="image" src="https://github.com/user-attachments/assets/4b5ee220-8f9b-428c-aa25-aef35195f580" />


maze.pyc stands out the most compare to other .pyc so let's try deomplile this file.


<img width="766" height="79" alt="image" src="https://github.com/user-attachments/assets/f34deefc-f620-4eb9-8feb-e259e53b720e" />


Looking inside the decompiled file

<img width="874" height="723" alt="image" src="https://github.com/user-attachments/assets/bd680147-fbf9-41fe-aab7-c8a37af2c9ca" />

these two flag ish letters stands out to me.

Lets try using these as passwords to the locked zipped file.?

- Y0u_St1ll_1N_4_M4z3

<img width="727" height="600" alt="image" src="https://github.com/user-attachments/assets/1253ea71-838a-4647-8b07-8dc8f0681ae3" />

Wrong password

- Y0u_Ar3_W4lkiNG_t0_Y0uR_D34TH


<img width="949" height="304" alt="image" src="https://github.com/user-attachments/assets/ff165cc1-6086-4572-8806-57039b180407" />

CORRECT! It gave me a "maze" file with no extenstion on it.







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



**4. tackling "maze" plane file**


I tried to figure out what kind of file "maze" is.

<img width="351" height="65" alt="image" src="https://github.com/user-attachments/assets/19437cd1-700d-4a05-af46-fe8f2128a849" />


But it gave me no clue.




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

