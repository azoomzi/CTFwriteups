# Maze Writeup
by Azumi Yasukohchi


https://app.hackthebox.com/challenges/Maze 

## Overview
This CTF challenge focused on unpacking and reversing a multi-stage PyInstaller maze instead of exploiting a memory bug. I unpacked maze.exe to recover its Python code, which revealed the ZIP password for enc_maze.zip and a partially broken decryption routine for a blob called maze. A hidden module (obf_path) provided a RNG seed and randint loop that let me rebuild the real XOR key, fully decrypt maze into an ELF (dec_maze), then use Ghidra to reverse the flag-checking logic and reconstruct the final flag from the stored integer array.


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



**3. Tackling maze.exe first**

started with maze.exe because executables almost always contain the main logic and tell you how other files like the png/zip are used, while those are usually just data.


So let's Run "file" and "strings" on maze.exe to learn what kind of files there are.


<img width="710" height="84" alt="image" src="https://github.com/user-attachments/assets/3e4042d4-032a-4a3d-b143-7ce9432ce388" />



.
<img width="359" height="52" alt="image" src="https://github.com/user-attachments/assets/2b15439d-3469-4cef-b862-37619ebe6bd8" />

.


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


But it gave me NO clue.

I thought that maybe going back to where it came from would help me understand what the maze file was for. the password "Y0u_Ar3_W4lkiNG_t0_Y0uR_D34TH" came from the file "maze.pyc"-->(decompiled)--> "maze_decompiled.py" so lets go back to the decompiled file and see if there are any other clues left inside the code.




<img width="919" height="706" alt="image" src="https://github.com/user-attachments/assets/13becb86-57ca-416c-bf0c-619c86afd6fa" />



Important things we learned here:
- ZIP password: Y0u_Ar3_W4lkiNG_t0_Y0uR_D34TH
- It decrypts a file called **"maze"** and writes **"dec_maze"**.

  
  <img width="966" height="113" alt="image" src="https://github.com/user-attachments/assets/cea0f779-83d3-4098-9e34-f8d1b7b6872c" />

- The second loop does NOTHING because KEY IS ALL ZERO (key = [0] * len(data)) 

Since x ^ 0 = x, the second step does nothing, which tells us the real key is missing and must come from somewhere else.

when i go back to the top, there is a suspcious module called obf_path (not a standard library) 

The code only calls obf_path.obfuscate_route() if i type the correct string Y0u_St1ll_1N_4_M4z3, so that function is on the “good path” and must be giving me something useful, not a dead end.



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


**5. Decompiling obf_path.pyc**


<img width="860" height="653" alt="image" src="https://github.com/user-attachments/assets/cd7a4db7-e3ee-4461-a8e9-1acc880aecfc" />


I decompiled it by using "uncompyle6". When I looked at the decompiled obf_path.pyc, it didn’t give me a nice readable function. So even after decompiling, the real logic is still hidden inside a big blob that gets marshal.loads() + exec()’d at runtime. That means I would have to reverse raw Python bytecode inside the blob, which is way more painful than I want for this CTF.








<img width="579" height="165" alt="image" src="https://github.com/user-attachments/assets/a19bc6c7-30a4-4b38-a6a3-ad927ff203e9" />



So the module inside the obf_path.pyc code ran, but instead of giving me the real hint it basically told me “you’re in the wrong place, go back to the previous path.” That made me think this module is meant to be executed from the original challenge directory, next to the other files (maze.exe, enc_maze.zip, maze.png), and that it probably uses one of those files as input and also since enc_maze.zip and maze were already explained in maze.pyc, the only unexplained file left was maze.png, so I suspected obf_path was using the PNG.




<img width="591" height="283" alt="image" src="https://github.com/user-attachments/assets/79e90bb1-f322-452a-a3e6-374620a526bb" />

Now I know the program wants me to use random.seed(493) and generate 300 values with randint(32,125) to build the missing key.








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


**6. Fixing the broken key logic in maze_decompiled.py**


Changes we need to make:

- Add import random
- Replace the key = [0] * len(data) block with RNG logic we the seed we got from obf_path.pyc.

```
# uncompyle6 version 3.9.3
# Python bytecode version base 3.8.0 (3413)
# Decompiled from: Python 3.8.18 (default, Nov 26 2025, 17:02:02) 
# [GCC 14.3.0]
# Embedded file name: maze.py
import sys #,obf_path
import random  # <--- added
ZIPFILE = "enc_maze.zip"
print("Look who comes to me :)")
print()
inp = input("Now There are two paths from here. Which path will u choose? => ")
if inp == "Y0u_St1ll_1N_4_M4z3":
    pass #obf_path.obfuscate_route()
else:
    print("Unfortunately, this path leads to a dead end.")
    sys.exit(0)
import pyzipper

def decrypt(file_path, word):
    with pyzipper.AESZipFile(file_path, "r", compression=(pyzipper.ZIP_LZMA), encryption=(pyzipper.WZ_AES)) as extracted_zip:
        try:
            extracted_zip.extractall(pwd=word)
        except RuntimeError as ex:
            try:
                try:
                    print(ex)
                finally:
                    ex = None
                    del ex

            finally:
                ex = None
                del ex


decrypt(ZIPFILE, "Y0u_Ar3_W4lkiNG_t0_Y0uR_D34TH".encode())
with open("maze", "rb") as file:
    content = file.read()
data = bytearray(content)
data = [x for x in data]


# FIXED: use seed(493) and 300 x randint(32,125) as the key
random.seed(493)
key = [random.randint(32, 125) for _ in range(300)]








for i in range(0, len(data), 10):
    data[i] = (data[i] + 80) % 256
else:
    for i in range(0, len(data), 10):
        data[i] = (data[i] ^ key[i % len(key)]) % 256
    else:
        with open("dec_maze", "wb") as f:
            for b in data:
                f.write(bytes([b]))

# okay decompiling maze.exe_extracted/maze.pyc





```

<img width="613" height="111" alt="image" src="https://github.com/user-attachments/assets/78b0148a-6ab8-422a-86ba-dc5c9c1d3f88" />


Now it should've given out the dec_maze file.

Let's list the directory for confirmation.

<img width="914" height="98" alt="image" src="https://github.com/user-attachments/assets/109da05d-df47-4134-91a4-9db9fca6316f" />

YAY

Now let's check the file type of this file.


<img width="944" height="84" alt="image" src="https://github.com/user-attachments/assets/4f578711-870e-4c88-bead-1930af8f78e0" />

Now that I know this file is an executable, I can open it in Ghidra to analyze what it actually does. My goal is to find the part of the code that reads my input, see how it checks that input against some internal values (like an encrypted array), and then use that logic to reconstruct the exact flag the program expects.





<img width="975" height="870" alt="image" src="https://github.com/user-attachments/assets/d253f307-25de-4ee4-85b6-a19e8f4d6682" />


I loaded dec_maze into Ghidra and then searched for the function fgets in the Symbol Tree on the left. Since fgets is commonly used to read user input, I right-clicked it and opened the Function Call Trees window (bottom-left). There was only one caller, FUN_00100169, so I double-clicked it to jump there. In the middle Listing and right Decompile panels you can see this function: it calls fgets(local_58, 0x40, stdin) to read my input, checks that the first three bytes are 'H', 'T', 'B', and then runs a loop that compares sums of three consecutive characters from my input against a hardcoded integer array in .rodata.





<img width="624" height="585" alt="image" src="https://github.com/user-attachments/assets/5bfe7ebe-c099-45cc-959a-93fda5947f70" />



After finding the flag-checking loop, I looked at the global array it was using for the comparisons. In the decompiler, this array was referenced as something like INT_ARRAY_00102060, so I double-clicked that symbol to jump to it in the Listing window (center). There, I right-clicked the address and chose Data → Define Array…, set the type to int and the length to 36, and Ghidra displayed the whole int[36] array you can see in the screenshot (values like 0DEh, 111h, 134h, etc). These 36 integers are the “encrypted sums” that the program compares against s[i-1] + s[i] + s[i+1].


<img width="1044" height="1026" alt="image" src="https://github.com/user-attachments/assets/abd972c5-7e80-4aba-93fe-ccd10693697c" />


I later copied them out and used them in a Python script to recover the original flag.




From the decompiled C code, I saw that the program first checked the prefix HTB, and then used the following rule in a loop:
encrypted[i] == input[i-1] + input[i] + input[i+1].
Since I already knew input[0] = 'H', input[1] = 'T', and input[2] = 'B', I could turn this around and solve for each next character:
input[i+1] = encrypted[i] - input[i-1] - input[i]. Using the 36 integers I copied from Ghidra, I wrote a small Python script that starts with [ 'H', 'T', 'B' ], applies this formula in a loop, and appends each newly recovered character to the flag string.                                                                                                                                                                                                                                               




<img width="554" height="252" alt="image" src="https://github.com/user-attachments/assets/0576865d-c782-45e4-a21d-3a91d1ba6175" />



now we got the flag, thanks for reading!!!


