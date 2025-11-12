# CyberPsychosis Writeup
by Azumi Yasukohchi



## Overview
https://app.hackthebox.com/challenges/Cyberpsychosis 

## Category
Reversing


## Approach

**1. Unzipping and looking through the CyberPsychosi.zip**
Since the file inside the zip file was .ko which is for kernel module, I am going to use Kali Linux for this CTF.

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

Reasearching what these does:
|Function Name| What it does based on my research| 
|-------------|----------------------------------|
|hacked_getdents|It is used to hide files (When rootkits want to hide files or directories, they hook getdents functions and filter out entries.|

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




**4. RGBA mode**

Finally the important part, RGBA mode.
In the code, the image is converted to RGBA mode.

```
im = Image.open(s_img).convert("RGBA")
```

Based on my research (https://en.wikipedia.org/wiki/RGBA_color_model#:~:text=RGBA%20stands%20for%20red%20green,pixel%20is%20a%204D%20vector)

RGBA mode is a color model that expands on the traditional RGB (Red, Green, Blue) model by adding a fourth channel for 'A'= Alpha which controls the color's opacity or transparency.

In this script, the secret message is hidden inside the Alpha channel, the fourth value. Since Alpha affects transparency (and not color), changes to it are often invisible to the human eye unless you specifically look for them.
RGBA mode allows encrypted message to store by replacing the Alpha values of the first few pixels (along the top row). 




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





**5. Creating a decoder for secret.png**


I understood how the secret message was hidden, so now all I need to do is to figure how I'm going to reverse enginneer the logic and decrypt it.

Since the script encoded each character by:

1. Reversing the original message
2. XORing each character with the length of the output filename (len("secret.png") = 10)
3. And hiding the result inside the Alpha channel of each pixel

We will do the exact opposite of how it encrypted.
Which is:

(3.)  Figureing out the each pixel value of Alpha channel 

(2.)  XORing each Alpha value with 10 (the length of the output filename) to get back the original character

(1.)  Revesing the final result to get the original message.

I will use this logic to create a python code that will decrypt the message from secret.png

I will use original code, "secretdoor.py" as a base code

```
from PIL import Image

def prob(d_img, key_len=10, max_len=100):
    im = Image.open(d_img).convert("RGBA")
    p = im.load()
    c = 0
    msg = []

    for i in range(max_len):
        enc = p[c, 0][3]  # Read the alpha channel
        ch = chr(enc ^ key_len)  # Decrypt it using XOR
        if not ch.isprintable():
            break
        msg.append(ch)
        c += 1

    print("Hidden Message:", ''.join(msg[::-1]))  # Reverse back to original

prob("secret.png")

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





**6. Extracting the secret message**

Save above code as secret_decoder.py and save in the same folder as secret.png

<img width="636" height="223" alt="image" src="https://github.com/user-attachments/assets/f839edd7-79d2-4c55-b631-d29c09a87ca2" />


For me, I had to download PIL to run this code so might as well document it here.


``` cmd
pip install pillow
```

Then run he script inside the command prompt.

```
python secret_decoder.py
```

<img width="1068" height="186" alt="image" src="https://github.com/user-attachments/assets/e22f9357-375a-488d-b1fb-1f12fb599922" />

FINALLY
I decrypted the secret message:
1t_is_very_light_b0x

Thanks for reading!
