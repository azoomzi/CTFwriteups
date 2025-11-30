# Hero Writeup
by Azumi Yasukohchi

https://cybertalents.com/challenges/malware/hero

## Overview


## Category
Reversing Malware


## Approach


**1. Setting up the HackTheBox environment**

 I am going to use Kali Linux for this CTF



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



Unzipped:


<img width="536" height="384" alt="image" src="https://github.com/user-attachments/assets/c6356006-453c-4acb-be66-bdc1def5c210" />


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



**3. Understanding general concept of "Hero.unknown"**

Seeing contents inside the file(I added comments so it's easier to understand):

```
# def gen(i):
#     return i ^ 11

  1           0 LOAD_CONST               0 (<code object gen at 0x7f1e8f527710, file "<dis>", line 1>)
              2 LOAD_CONST               1 ('gen')
              4 MAKE_FUNCTION            0
              6 STORE_NAME               0 (gen)


# def gen2(i):
#     return 14 ** i

  4           8 LOAD_CONST               2 (<code object gen2 at 0x7f1e8f5277c0, file "<dis>", line 4>)
             10 LOAD_CONST               3 ('gen2')
             12 MAKE_FUNCTION            0
             14 STORE_NAME               1 (gen2)


# f = open("flag.txt", "r")

  7          16 LOAD_NAME                2 (open)
             18 LOAD_CONST               4 ('flag.txt')
             20 LOAD_CONST               5 ('r')
             22 CALL_FUNCTION            2
             24 STORE_NAME               3 (f)


# o = []

  8          26 BUILD_LIST               0
             28 STORE_NAME               4 (o)


# r = f.readlines()[0]

  9          30 LOAD_NAME                3 (f)
             32 LOAD_METHOD              5 (readlines)
             34 CALL_METHOD              0
             36 LOAD_CONST               6 (0)
             38 BINARY_SUBSCR
             40 STORE_NAME               6 (r)


# for i in range(len(r)):
#     o.append(ord(r[i]))

 10          42 LOAD_NAME                7 (range)
             44 LOAD_NAME                8 (len)
             46 LOAD_NAME                6 (r)
             48 CALL_FUNCTION            1
             50 CALL_FUNCTION            1
             52 GET_ITER
        >>   54 FOR_ITER                22 (to 78)
             56 STORE_NAME               9 (i)

 12          58 LOAD_NAME                4 (o)
             60 LOAD_METHOD             10 (append)
             62 LOAD_NAME               11 (ord)
             64 LOAD_NAME                6 (r)
             66 LOAD_NAME                9 (i)
             68 BINARY_SUBSCR
             70 CALL_FUNCTION            1
             72 CALL_METHOD              1
             74 POP_TOP
             76 JUMP_ABSOLUTE           54


# s = []

 14     >>   78 BUILD_LIST               0
             80 STORE_NAME              12 (s)


# for i in range(len(o)):
#     t = gen(i)
#     f = gen2(t)
#     s.append(~(f * o[i]))

 15          82 LOAD_NAME                7 (range)
             84 LOAD_NAME                8 (len)
             86 LOAD_NAME                4 (o)
             88 CALL_FUNCTION            1
             90 CALL_FUNCTION            1
             92 GET_ITER
        >>   94 FOR_ITER                40 (to 136)
             96 STORE_NAME               9 (i)

 16          98 LOAD_NAME                0 (gen)
            100 LOAD_NAME                9 (i)
            102 CALL_FUNCTION            1
            104 STORE_NAME              13 (t)

 17         106 LOAD_NAME                1 (gen2)
            108 LOAD_NAME               13 (t)
            110 CALL_FUNCTION            1
            112 STORE_NAME               3 (f)

 18         114 LOAD_NAME               12 (s)
            116 LOAD_METHOD             10 (append)
            118 LOAD_NAME                3 (f)
            120 LOAD_NAME                4 (o)
            122 LOAD_NAME                9 (i)
            124 BINARY_SUBSCR
            126 BINARY_MULTIPLY
            128 UNARY_INVERT
            130 CALL_METHOD              1
            132 POP_TOP
            134 JUMP_ABSOLUTE           94


# print(s)

 20     >>  136 LOAD_NAME               14 (print)
            138 LOAD_NAME               12 (s)
            140 CALL_FUNCTION            1
            142 POP_TOP


# print(len(s))

 21         144 LOAD_NAME               14 (print)
            146 LOAD_NAME                8 (len)
            148 LOAD_NAME               12 (s)
            150 CALL_FUNCTION            1
            152 CALL_FUNCTION            1
            154 POP_TOP
            156 LOAD_CONST               7 (None)
            158 RETURN_VALUE


# def gen(i):
#     return i ^ 11

Disassembly of <code object gen at 0x7f1e8f527710, file "<dis>", line 1>:
  2           0 LOAD_FAST                0 (i)
              2 LOAD_CONST               1 (11)
              4 BINARY_XOR
              6 RETURN_VALUE


# def gen2(i):
#     return 14 ** i

Disassembly of <code object gen2 at 0x7f1e8f5277c0, file "<dis>", line 4>:
  5           0 LOAD_CONST               1 (14)
              2 LOAD_FAST                0 (i)
              4 BINARY_POWER
              6 RETURN_VALUE


```










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

