# UCSYCTF2026 steganography writeups



### hide (20 points)

![](/images/Screenshot_2026-07-07_21-42-38.png)

The challenge gives a zip file . so i unzip it and got an image file

![](/images/Screenshot_2026-07-07_21-44-02.png)

when i open the image, 

![](/images/Screenshot_2026-07-07_21-44-31.png)

**111oneone**

So as a hint , the file name is steghide, we can hide the text and reveal it using a steganography tool called **steghide**

so let's extract it.

the password should be **111oneone**, got from the image.

![](/images/Screenshot_2026-07-07_21-48-42.png)

BOOM!

---



### Photoapp (30 points)

![](/images/Screenshot_2026-07-07_21-50-05.png)

The challenge give a **photoapp.apk** file , since apk files are just zip archives. I unzip it and got some directories and files

![](/images/Screenshot_2026-07-07_22-00-12.png)

then i found **res/drawable/app_logo.png** image file and opened it ,

![](/images/Screenshot_2026-07-07_22-01-15.png)

so when i checked with **exiftool** (a tool used to see metadata)

![](/images/Screenshot_2026-07-07_22-02-12.png)

there is two interesting things, 

**Description                     : App logo - check the pixels carefully
Comment                         : LSB holds the secret**

So i assume this may be a LSB challenge. and i ask **chagpt** to give me the script :)

```python
from PIL import Image

img = Image.open("res/drawable/app_logo.png")
pixels = list(img.getdata())

bits = []

# Extract LSB from red channel only
for r, g, b in pixels:
    bits.append(r & 1)

data = bytearray()

# Convert bits to bytes, MSB-first
for i in range(0, len(bits) - 7, 8):
    byte = 0
    for bit in bits[i:i+8]:
        byte = (byte << 1) | bit
    data.append(byte)

text = data.decode(errors="ignore")
print(text[:200])
```

![](/images/Screenshot_2026-07-07_22-05-56.png)

BOOM!

---



### J2k (50 points)

![](/images/Screenshot_2026-07-07_22-22-23.png)

The challenge gives j.zip file and unzip it and got j.jpg image file, 

when i open it , 

![](/images/Screenshot_2026-07-07_22-23-32.png)

nothing special, but this is a steganography challenge , there is a useful site colleting most stegano tools 

https://aperisolve.com/

we can use this site to reveal the flag, 

![](/images/Screenshot_2026-07-07_22-28-54.png)



we can also try this with jsteg tool in our machine 

![](/images/Screenshot_2026-07-07_22-29-39.png)

BOOM!

---



### Scramble (100 points)

 ![](/images/Screenshot_2026-07-07_22-36-20.png)

Got a png image file , and open it , but :(

![](/images/Screenshot_2026-07-07_22-37-54.png)

someting's wrong with the image byptes, so i check with **hexedit** tool

![](/images/Screenshot_2026-07-07_22-46-49.png)

and i found that a lot of png chunk's byte orders are incorrect. So i put the hex in a file with 

```bash
xxd scramble.png > hex.txt
```

and upload it to  chatgpt and ask for a script to fix the wrong byte orders :)

```python
import re

data = bytearray()

with open("hex.txt", "r", errors="ignore") as f:
    for line in f:
        m = re.match(r"^[0-9a-fA-F]+:\s*((?:[0-9a-fA-F]{4}\s*)+)", line)
        if m:
            data.extend(bytes.fromhex(m.group(1).replace(" ", "")))

fixed = bytearray()

for i in range(0, len(data), 2):
    fixed.extend(data[i:i+2][::-1])

with open("fixed.png", "wb") as f:
    f.write(fixed)
```

and i got a the fixed image and when i opened it, got a pastebin site link



![](/images/Screenshot_2026-07-07_22-54-54.png)

https://pastebin.com/raw/4PJGctXf

![](/images/Screenshot_2026-07-07_22-55-46.png)

BOOM!

---

