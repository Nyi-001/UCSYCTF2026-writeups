# MISC writeups



### multi-layered (20 points)



![](/images/Screenshot_2026-07-07_23-37-52.png)

There is a zip file given . So we need to unzip it, but need password

so i crack it with john 

```
zip2john layers.zip > hash
```

```
john hash --wordlist=/usr/share/wordlists/rockyou.txt
```

the password is layers and unzip it , need password, crack it again , got the password and same as that way again and again and then  i notice that the password of the zip file is it's filename

![](/images/Screenshot_2026-07-07_23-41-17.png)

so i ask chatgpt for script 
import zipfile
from pathlib import Path
import shutil
import re

start = Path("layers.zip")
workdir = Path("extracted_layers")
shutil.rmtree(workdir, ignore_errors=True)
workdir.mkdir()

current_zip = start
password = current_zip.stem

for depth in range(100):
    print(f"[+] Layer {depth}: {current_zip.name}, password = {password}")

```python
with zipfile.ZipFile(current_zip) as z:
    files = [x for x in z.infolist() if not x.is_dir()]
    if not files:
        print("[-] Empty archive")
        break

    target = files[0]

    try:
        data = z.read(target, pwd=password.encode())
    except RuntimeError as e:
        print(f"[-] Failed password: {password}")
        raise e

outpath = workdir / target.filename
outpath.write_bytes(data)

match = re.search(rb"UCSYCTF\{[^}]+\}", data)
if match:
    print("[+] FLAG:", match.group().decode())
    break

if zipfile.is_zipfile(outpath):
    current_zip = outpath
    password = current_zip.stem
else:
    print("[+] Final file:", outpath)
    print(data.decode(errors="ignore"))
    break
```

![](/images/Screenshot_2026-07-08_00-38-21.png)

BOOM!

---



### Needle in the Hash (30 points)

![](/images/Screenshot_2026-07-08_00-42-45.png)

file10000.zip 

password => file10000

so unzip it and got thousands of .txt files.

![](/images/Screenshot_2026-07-08_00-44-06.png)

But the challenge give us a hint 

**Thousands of files. Thousands of flags. Only one is genuine. Can you separate the real "b1fc80" from the fake?**

so i filter the file name with 

![](/images/Screenshot_2026-07-08_00-47-28.png)

BOOM!

---



### Look!! (50 points)

![](/images/Screenshot_2026-07-08_01-10-00.png)

The challenge gives ezgif-8cd3e7e8aec577d5.gif file , i opened it, 



![](/home/zane001/Desktop/ucsyctf/misc/look/ezgif-8cd3e7e8aec577d5.gif)

in the image , we can see the text, so how to cat that text 

when i check the medata , 

![](/images/Screenshot_2026-07-08_01-14-41.png)

i got the hint (**https://ezgif.com/video-to-gif**) 

so i go to the site and catch the text with 

1. Go to **Split** / **GIF splitter**.
2. Upload the GIF.
3. Click **Split to frames**.
4. Check the extracted frames one by one.
5. Around frame **42**, you will see this hidden text at the top-left:

```
VUNTWUNURnsqKioqKioqKn0=
```

decode it and got flag

![](/images/Screenshot_2026-07-08_01-17-21.png)

BOOM!

---

