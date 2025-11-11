# CTF 1: SecretDoor Wrtieup
by Azumi Yasukohchi



## Overview


## Category



## Approach

**1. Unzipping and looking through the secretdoor.zip**
<img width="709" height="233" alt="image" src="https://github.com/user-attachments/assets/1a9f6948-b39a-4763-9f33-f17ea63f5075" />

I decided to look through what kind of code the python file contains

secretbox.py

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


