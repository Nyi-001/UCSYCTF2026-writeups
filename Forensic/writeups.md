# Forensic Writeups



### Palindrome Bomb (100 points)

![](/images/Screenshot_2026-07-09_13-21-30.png)

when i extract the zip file i got  another zip file named **next.zip** and **layer_xxxx.txt** file. But the flag is fake flag

![](/images/Screenshot_2026-07-09_13-24-39.png)

So i realize that we have to extract the zip file till there is no another zip file to extract. So i ask my GPT :)

Here is my scirpt :

```python
#!/usr/bin/env python3
import zipfile
import re
import sys
from pathlib import Path

FLAG_RE = re.compile(rb"UCSYCTF\{[^}\r\n]{1,200}\}")

def safe_extract(zip_path: Path, out_dir: Path):
    with zipfile.ZipFile(zip_path, "r") as z:
        for member in z.infolist():
            target = (out_dir / member.filename).resolve()
            if not str(target).startswith(str(out_dir.resolve())):
                raise Exception(f"ZipSlip blocked: {member.filename}")
        z.extractall(out_dir)

def extract_chain(start_zip):
    start_zip = Path(start_zip).resolve()
    out_base = Path("extracted_layers")
    out_base.mkdir(exist_ok=True)

    current_zip = start_zip
    all_flags = []

    for depth in range(1, 5000):
        layer_dir = out_base / f"layer_{depth:04d}"
        layer_dir.mkdir(exist_ok=True)

        print(f"[+] Extracting depth {depth}: {current_zip}")
        safe_extract(current_zip, layer_dir)

        # collect flags from extracted files
        for f in layer_dir.rglob("*"):
            if f.is_file() and f.suffix.lower() != ".zip":
                data = f.read_bytes()
                for m in FLAG_RE.findall(data):
                    flag = m.decode(errors="ignore")
                    all_flags.append((depth, str(f), flag))
                    print(f"    [flag] {flag}  <-- {f}")

        # find next zip
        zips = list(layer_dir.rglob("*.zip"))
        if not zips:
            print("\n[+] No more zip files found. Finished.")
            break

        # prefer next.zip if it exists
        next_zips = [z for z in zips if z.name == "next.zip"]
        current_zip = next_zips[0] if next_zips else zips[0]

    print("\n========== SUMMARY ==========")
    print(f"Total flags found: {len(all_flags)}")

    unique_flags = []
    seen = set()
    for _, _, flag in all_flags:
        if flag not in seen:
            seen.add(flag)
            unique_flags.append(flag)

    print("\nUnique flags:")
    for flag in unique_flags:
        print(flag)

    print("\nPossible real candidates:")
    fake_words = ["wrong", "decoy", "fake", "path"]
    candidates = [
        flag for flag in unique_flags
        if not any(w in flag.lower() for w in fake_words)
    ]

    if candidates:
        for flag in candidates:
            print(flag)
    else:
        print("No obvious candidate found. Check the final extracted layer manually.")

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print(f"Usage: python3 {sys.argv[0]} evidence_bomb.zip")
        sys.exit(1)

    extract_chain(sys.argv[1])
```

```bash
python solve.py [evidance.zip]
```

and i got a lot of flags and the real flag is 

UCSYCTF{Again_and_Again}

---



### Ghost Protocol Logs  (50 points)

![](/images/Screenshot_2026-07-09_13-44-29.png)

I opened the log files with sublime text , and search the real attacker by filtering 

**Accepted publickey for**

![](/images/Screenshot_2026-07-09_13-47-08.png)

and i found the ssh key and the real attacker ip is 

```
203.0.113.66
```

and then i search the suspicious DNS domains by filtering with 

**cdn.cloudfront-akamai.net**

and i got hex chunks

![](/images/Screenshot_2026-07-09_13-49-56.png)

Decode it 

55435359 -> UCSY
4354467b -> CTF{
67683073 -> gh0s
745f0000 -> t_

so we got the first part of the flag **UCSYCTF{gh0st_**

So let's try other parts,

this time we know what the attacker ip address is 

So let's filer with it in other log files

when i filter at logs/web/access.log file 

![](/images/Screenshot_2026-07-09_13-54-05.png)

i got the last part of the flag

**pu4aa3y_rks1y}**

So let's find the middle part of the flag in logs/mail/mail.log 

![](/images/Screenshot_2026-07-09_13-56-04.png)

I got base64 encoded **X-Data-Chunk: cHIwdDBjMGxfbXVsdDFf**

![](/images/Screenshot_2026-07-09_13-57-10.png)

So the flag is **UCSYCTF{gh0st_pr0t0c0l_mult1_pu4aa3y_rks1y}**

---



### Git (100 points)

![](/images/Screenshot_2026-07-09_14-04-59.png)

unzip the zip file and in the .git, read the HEAD file , but nothing found and the flag.enc file is included fake flag. So i check all git history with 

```bash
git log --all --oneline --decorate
```



![](/images/Screenshot_2026-07-09_14-09-48.png)

BOOM!

---



### RAID 0 (200 points)

![](/images/Screenshot_2026-07-09_14-10-37.png)

There is two disk .raw file, one is FAT32 boot sector and other one is corrupt.

![](/images/Screenshot_2026-07-09_14-34-36.png)

after checking the strings of **the disk_chunk_B.raw**

![](/images/Screenshot_2026-07-09_14-46-15.png)

it contain PNG data

So i don't know what to do next, so let's search for what does RAID 0 mean and got that 

**RAID 0 (disk striping) splits data evenly across two or more drives without parity or redundancy. It provides maximum read/write speeds and combines total drive capacities, but offers zero fault tolerance: if one drive fails, all data in the array is lost**

as one drive fails in this challenge , we need to recover it 

So i ask chatgpt to give the script ,

```python
#!/usr/bin/env python3
from pathlib import Path

A = Path("disk_chunk_A.raw").read_bytes()
B = Path("disk_chunk_B.raw").read_bytes()

stripe = 65536
out = bytearray()

for i in range(0, max(len(A), len(B)), stripe):
    out += A[i:i+stripe]
    out += B[i:i+stripe]

# FAT32 boot sector says total sectors = 122624, sector size = 512
recovered = bytes(out[:122624 * 512])
Path("recovered.img").write_bytes(recovered)

# The recovered filesystem contains h.png.
# Its PNG magic appears at this offset in the rebuilt image.
png_start = recovered.find(b"\x89PNG\r\n\x1a\n")
iend = recovered.find(b"IEND", png_start)

png = recovered[png_start:iend + 8]
Path("h.png").write_bytes(png)

print("[+] rebuilt image: recovered.img")
print("[+] extracted PNG: h.png")
print("[+] open h.png to read the flag")
```

then got the png file 

![](/images/Screenshot_2026-07-09_14-49-04.png)

BOOM!

---



### Unknown (200 points)

![](/images/Screenshot_2026-07-09_14-58-55.png)

The challenge gives a unknown.xlsx file and i check with file cmd to know what the actual file type is 

![](/images/Screenshot_2026-07-09_15-08-38.png)

But actually the Miscrosoft Word 2007+ file is DOC/Word zip file structure.So unzipped it

![](/images/Screenshot_2026-07-09_15-10-05.png)

After checking many files, and i got the fake flag encoded from **word/document.xml** 

![](/images/Screenshot_2026-07-09_15-12-00.png)

But there is two interesting things

1. **word/media/image1.jpg**
2. **word/media/image2.jpg**

when i open it , nothing special , but let's analyze these images at https://aperisolve.com

and i found the flag from image1.jpg

![](/images/Screenshot_2026-07-09_15-14-11.png)

BOOM!

