# Rega's Town Writeup
by Azumi Yasukohchi

https://app.hackthebox.com/challenges/Rega's%2520Town 

## Overview


## Category


## Approach


**1. Setting up the HackTheBox environment**


I am going to use Kali Linux for this CTF. You do not need connection to HTB for this CTF so we are skipping the connect to openVPN part.

<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>



**2. Unzipping and looking through the CyberPsychosi.zip**

<img width="574" height="280" alt="image" src="https://github.com/user-attachments/assets/a3ac7198-89de-47c1-974a-f72337155950" />

<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>




Unzipped:

<img width="617" height="292" alt="image" src="https://github.com/user-attachments/assets/b5efb023-c9ea-44fc-8df3-9d3abdcedaa6" /> <img width="603" height="348" alt="image" src="https://github.com/user-attachments/assets/628f2071-02a9-461a-8eb7-7806dac000eb" />


<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>


It looks like the file does not have any extention.

<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>


**3. Understanding concept of "rega_town file"**

I wanted see what kind of file actually this is.

<img width="951" height="88" alt="image" src="https://github.com/user-attachments/assets/6ad9914b-6faa-4a18-8959-be52399cd29c" />

This file can be executable.

<p>&nbsp;</p>
<p>&nbsp;</p>

I decided to run it to see what the program does.
I needed to change the permission to be able to run it so
<img width="233" height="71" alt="image" src="https://github.com/user-attachments/assets/4d9ed2f3-0586-434d-9d04-56becc65a5f2" />

then:
<img width="301" height="118" alt="image" src="https://github.com/user-attachments/assets/88f9c25f-cb07-48f3-9344-d2f9fc12fd02" />

<p>&nbsp;</p>
<p>&nbsp;</p>



Since it was executable file, I decided to analyze it through Ghidra

<p>&nbsp;</p>
<p>&nbsp;</p>

I went to "Defined strings" and seached for "Welcome to" as that is where the code starts.

<img width="962" height="484" alt="image" src="https://github.com/user-attachments/assets/a953fee2-2e62-4017-ab63-28dfb82bb187" />
<p>&nbsp;</p>
<p>&nbsp;</p>


<img width="1412" height="1004" alt="image" src="https://github.com/user-attachments/assets/5779309c-2be0-4058-8eda-dffa850a8196" />
<p>&nbsp;</p>
<p>&nbsp;</p>

It lead me to core logic. Now we can find out how this binary decide whether my input is correct.
<p>&nbsp;</p>
<p>&nbsp;</p>

<img width="1412" height="1004" alt="image" src="https://github.com/user-attachments/assets/af7d4d82-c045-411a-9861-79039ae1bec1" />

Let's take a closer look at the decompile, specifically this part:
<p>&nbsp;</p>
<p>&nbsp;</p>


```
std::io::stdio::stdin();
std::io::stdio::Stdin::read_line();
...
local_168 = core::str::trim_end(&Var2);
local_154 = filter_input(local_168);
if (local_154 == 0) {
    print("Maybe next time :<");
}
else {
    ::alloc::string::to_string(&local_150,&local_200);
    uVar1 = check_input(&local_150);
    local_154 = uVar1 & local_154;
    if (local_154 != 0) {
        local_50 = ::alloc::vec::index<>(&local_180,5);
        ...
        if (*local_50 != L'0') panic;
        ...
        local_40 = ::alloc::vec::index<>(&local_180,9);
        ...
        if (*local_40 != L'r') panic;
        print("Correct one of us!!");
    }
}
```
<p>&nbsp;</p>
<p>&nbsp;</p>

Why this matters
From this we deduce:
1. The user input is stored in local_200 (a String).
2. It is trimmed --> local_168.
3. Validation has 2 phases:
4. filter_input(trimmed_input) --> returns 0 or 1
5. check_input(&local_150) --> returns 0 or 1
6. Then two extra checks:
   - chars[5] == '0'
   - chars[9] == 'r'
7. Only if all conditions are true → prints "Correct one of us!!".

<p>&nbsp;</p>
<p>&nbsp;</p>

So "main" is telling us:
There are exactly three layers of constraints:
1. filter_input (structural / regex)
2. check_input (content logic)
3. Two character checks at positions 5 and 9

<p>&nbsp;</p>
<p>&nbsp;</p>

Let's go to filter_input to see what the logic behind of the first step.
<p>&nbsp;</p>
<p>&nbsp;</p>
<img width="969" height="1035" alt="image" src="https://github.com/user-attachments/assets/05e5bb74-d943-4f6b-a3ed-5cd5af7e28bc" />
<p>&nbsp;</p>
<p>&nbsp;</p>
What I can logically conclude from this
<p>&nbsp;</p>
<p>&nbsp;</p>
memcpy(local_d8, &PTR_s_^.{33}$(?:... ), 0x90);
--> There is some static data, referenced by PTR_s_^.{33}…, copied into a local buffer.
<p>&nbsp;</p>
<p>&nbsp;</p>
core::array::iter::into_iter<&str,_9>(&local_268, (&str (*) [9])local_d8);
--> That static data is interpreted as an array of 9 &str values.

<p>&nbsp;</p>
<p>&nbsp;</p>
<img width="637" height="783" alt="image" src="https://github.com/user-attachments/assets/e758947f-b33d-4732-a3fc-d56c39e8688b" />

Now I know that filter_input enforces that the user input must match all 9 regex patterns stored in static memory near PTR_s_^.{33}...
<p>&nbsp;</p>
<p>&nbsp;</p>

Reasoning for where to look next:

Since now i know the patterns themselves decide what the flag looks like, I will inspect the data behind PTR_s_^.{33}...

I double clicked on the PTR_s_^.{33}... and it led me to the actual place where it stores the blob


<img width="1232" height="1042" alt="image" src="https://github.com/user-attachments/assets/e006b510-5036-40f1-b380-ab9acf00300f" />

I double clicked again to go to actual address which is 0042603b

<img width="1907" height="1024" alt="image" src="https://github.com/user-attachments/assets/54814936-b3d6-4c99-8c1a-c45106ccb6b6" />


I copied using "Copy Data" and it gave me the full blob:
<p>&nbsp;</p>
<p>&nbsp;</p>
```
^.{33}$(?:^[\x48][\x54][\x42]).*^.{3}(\x7b).*(\x7d)$^[[:upper:]]{3}.[[:upper:]].{3}[[:upper:]].{3}[[:upper:]].{3}[[:upper:]].{4}[[:upper:]].{2}[[:upper:]].{3}[[:upper:]].{4}$(?:.*\x5f.*)(?:.[^0-9]*\d.*){5}.{24}\x54.\x65.\x54.*^.{4}[X-Z]\d._[A]\D\d.................[[:upper:]][n-x]{2}[n|c].$.{11}_T[h|7]\d_[[:upper:]]\dn[a-h]_[O]\d_[[:alpha:]]{3}_.{5}
```
<p>&nbsp;</p>
<p>&nbsp;</p>
Since it is supposed to be 9 regex, we now need to separate them into 9 piecies.
<p>&nbsp;</p>
<p>&nbsp;</p>

The pattern is each one:
- Has balanced ( / ) / [] / {}.
- Uses anchors ^ and $ in a sane way.
- $ goes at the end of each
- ^ always goes in the front

so by following that pattern:

<p>&nbsp;</p>
<p>&nbsp;</p>

```
1.  ^.{33}$

2.  (?:^[\x48][\x54][\x42]).*

3.  ^.{3}(\x7b).*

4.  (\x7d)$

5.  ^[[:upper:]]{3}.[[:upper:]].{3}[[:upper:]].{3}[[:upper:]].{3}[[:upper:]].{4}[[:upper:]].{2}[[:upper:]].{3}[[:upper:]].{4}$

6.  (?:.*\x5f.*)

7.  (?:.[^0-9]*\d.*){5}.{24}\x54.\x65.\x54.*

8.  ^.{4}[X-Z]\d._[A]\D\d.................[[:upper:]][n-x]{2}[n|c].$

9.  .{11}_T[h|7]\d_[[:upper:]]\dn[a-h]_[O]\d_[[:alpha:]]{3}_.{5}

```
<p>&nbsp;</p>
<p>&nbsp;</p>

<p>&nbsp;</p>
<p>&nbsp;</p>




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

