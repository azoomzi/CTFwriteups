# CTF 1: SecretDoor Wrtieup
by Azumi Yasukohchi



## Overview
This CTF challenge involved a Python script called `secretbox.py` that hides a secret message inside an image using steganography. By analyzing the code, I discovered that the message was encrypted and stored in the image's alpha channel. I then wrote a custom decoder to extract the hidden message from the image file.


## Category
Image Steganography with XOR encryption


## Approach

**1. Unzipping and looking through the secretdoor.zip**
<img width="709" height="233" alt="image" src="https://github.com/user-attachments/assets/1a9f6948-b39a-4763-9f33-f17ea63f5075" />

I decided to look through what kind of code the python file contains




**2. Understanding general concept of "secretbox.py"**
```
import sys
from PIL import Image

def prob(s_img, msg, d_img):
	im = Image.open(s_img).convert("RGBA")
	p = im.load()
	c = 0
	msg = map(lambda x: ord(x) ^ len(d_img), msg[::-1])
	for i in range(0, len(msg)):
		enc = msg[i]
		p[c, 0] = (p[c, 0][0], p[c, 0][1], p[c, 0][2], enc)
		c += 1
	im.save(d_img)

if len(sys.argv) != 4:
	print "%s \"orignal.png\" \"secret message\" \"secret.png\"" % sys.argv[0]
	exit()

prob(sys.argv[1], sys.argv[2], sys.argv[3])
```

From what I see in this code, I thought that this code is what let the user create "sercret.png".

Why?

```
im = Image.open(s_img).convert("RGBA")
```

this part takes input image from the user. then saves it with this code

```
im.save(d_img)
```

Then, this part made me think that this code will take a "secret message" that the user typed in.

```
msg = map(lambda x: ord(x) ^ len(d_img), msg[::-1])
```


So therefore this part of code is the format of the command line,

```
if len(sys.argv) != 4:
	print "%s \"orignal.png\" \"secret message\" \"secret.png\"" % sys.argv[0]
	exit()
```

so if you type in the order of above "\"orignal.png\" \"secret message\" \"secret.png\"", for example,

```cmd
python secretbox.py original.png "random" secret.png
```

Then user will generate a "secret.png" with a sercret message embedded in it.






**3. Logic of hiding secret message inside the image**
Let's closely look at this part of the code again.

```
msg = map(lambda x: ord(x) ^ len(d_img), msg[::-1])
```

I have never seen lambda before so I asked AI to teach me what lambda is. It told me that lamba acts as function but smoother and quicker to type the whole logic. So basically:

```
def encrypt_char(x):
    return ord(x) ^ len(d_img)

msg = map(encrypt_char, msg[::-1])
```

So in here, "x" is the one character that we get from the user. For example, if we are using "random", the first letter that the code will take as "x" will be "r".

So below line tells python that we are making a function called encrypt_char and we will take one letter from the message as x

```
def encrypt_char(x):
```

```
    return ord(x) ^ len(d_img)
```

"ord" will take "x" and will get ASCII number of the character. For example, r = ord('r') = 114.
"len" will get (d_img) which is the LENGTH of the output image FILE NAME. For example, d_img = secret.png = len ("secret.png") = 10 characters long
"^" this is a bitwsie XOR operation which is a way to mix two numbers together.
so for example,

ord('r') = 114
len("secret.png") = 10
```
  	114 = 01110010
xor  10 = 00001010
------------------
      	= 01111000
	  	= 120
```	
114 ^ 10 = 120 (this number becomes the "encrypted" value

```
msg = map(encrypt_char, msg[::-1])
```

Then above line will reverse the whole message with msg[::-1].
So "random" becomes "modnar".
and map(encrypt_char,   ) applies the encryption to EACH character in that reversed message.








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
