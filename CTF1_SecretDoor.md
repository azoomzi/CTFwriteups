# CTF 1: SecretDoor Wrtieup
by Azumi Yasukohchi



## Overview


## Category



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



##4. Decoding the hidden message from secret.png

After understanding how the message was hidden, the next step is to figure out how to extract it.

Since the original script stored the message inside the alpha channel (transparency) of the image, all we need to do is read those alpha values and reverse the encryption process.






