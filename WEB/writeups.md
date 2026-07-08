# UCSYCTF 2026 WEB writeup



### space X (100 points)

![](/images/Screenshot_2026-07-07_19-54-58.png)

When i check the challenge , it says 404 not found.

![](/images/Screenshot_2026-07-07_20-02-33.png)

 So as an hunter ,when we don't have any information about the target. Just do information gathering, when we collect more information about the target , we will find more attack surfaces.So let's start with directory fuzzing.....

After fuzzing a little while , i got a directory named **view**

![](/images/Screenshot_2026-07-07_20-18-58.png)

when i check that directory, 

![](/images/Screenshot_2026-07-07_20-15-03.png)

At this step, i think we have to try just one thing called playing with **Request Method** 

So i changed the method from **GET** to **POST**

![](/images/Screenshot_2026-07-07_20-19-47.png)

then in the response, **Submit an XML document for preview. Shared starter fragment: docs/welcome.xml**

it means we can try **XXE** attack , (the vulnerability caused by xml file parsers)

So i prepare my exploit: 

1. first we have to change the content type to (**Content-Type: application/xml**)

2. ```xml
   <?xml version="1.0"?>
   <root xmlns:xi="http://www.w3.org/2001/XInclude">
     <xi:include href="file:////flag.txt" parse="text"/>
   </root>
   ```

![](/images/Screenshot_2026-07-07_20-28-04.png)

BOOM!

if you are not familiar with xxe attack , i would like to recommend studying from https://portswigger.net/web-security/xxe

---



### Api (200 points)

![](/images/Screenshot_2026-07-07_20-30-41.png)



When i enter the challenge site, 

![](/images/Screenshot_2026-07-07_20-31-53.png)

There is nothing special, so i viewed the page source and got javascript file named **main.491f8fcf.js**

and after reading a little while , i got the admin credential 

![](/images/Screenshot_2026-07-07_20-37-12.png)

But i don't know where i have to use it, so just do directory fuzzing :)

![](/images/Screenshot_2026-07-07_20-58-05.png)

After fuzzing one time, i got nothing, so i think another way that nowadays, most modern are using api call. So we should try api endpoint fuzzing.....

![](/images/Screenshot_2026-07-07_21-00-05.png)

now i got one api endpoint called **admin**

![](/images/Screenshot_2026-07-07_21-01-14.png)

403 forbidden!

So we have to use admin credentials got from javascript file.i don't know where to use these credentials.

So i try **/api/login** 

![](/images/Screenshot_2026-07-07_21-04-08.png)

nothing special, but every login page is for updating the database , it should be **POST** method

![](/images/Screenshot_2026-07-07_21-06-04.png)

so it's clear that we have to put admin credential in this with json format , because the server response is json format.

but before using the credential, i think we need to crack the password.

![](/images/Screenshot_2026-07-07_21-10-29.png)

can't crackable :( 

But when check the javascript file again, i notice a note
**notes: "TODO remove migration credential before production"**

i think that means During migration, the application accepted the value of `migrationHash` as a valid login secret for the administrator account.

so i try login with password hash

![](/images/Screenshot_2026-07-07_21-24-40.png)

heeheehee we got the token. So let's enter the admin dashboard with that token

since most REST api use Bearer token : 

![](/images/Screenshot_2026-07-07_21-28-12.png)

BOOM!

---
### Secure Plugin Store (300 points)

![](/images/Screenshot_2026-07-08_19-51-45.png)

This is a **malicious plugin upload attack** that leads to **server-side file read / data exfiltration**.

The main vulnerablility is untrusted plugin with weak sandboxing.

So let's do exploitation

![](/images/Screenshot_2026-07-08_20-04-05.png)

There is a plugin upload function. So try to upload a simple zip file.

![](/images/Screenshot_2026-07-08_20-14-06.png)

response: **No manifest file found (requires manifest.json)**

So we need manifest.json file in the plugin. I search what is manifest.json and how it structures

and i found that 

**The `manifest.json` file is a core configuration document used across various software platforms. It acts as an instruction manual, providing the system or browser with essential metadata and setup rules on how to launch, display, and run an application**

So here is my simple manifest :

```json
{
  "manifest_version": 3,
  "name": "pwn",
  "author": "test",
  "version": "1.0.0",
  "description": "simple test plugin",
  "background": {
    "service_worker": "payload.py"
  }
}
```

zipped it and upload it , 

![](/images/Screenshot_2026-07-08_20-25-20.png)

So the plugin upload is successful now, then we don't know where to call that manifest.json file. So same as always  , let's do directory fuzzing.....

![](/images/Screenshot_2026-07-08_20-34-41.png)

Got one dir, try further fuzzing that api....

![](/images/Screenshot_2026-07-08_20-35-53.png)

now we got three dir, and we can call manifest.json file at

```
 http://187.127.100.53:9000/api/plugin/a6a062f9-e036-49ef-98c7-9bb47152ce45/manifest.json
```

So what we gonna do now. The challenge's hint says that the flag file is at /opt/app/instance/flag.txt.

we have to put our exploit payload file with manifest.json together in the plugin. But putting the payload file normally doesn't work. So we need to hide it in a hidden directory.

```
upload.zip
├── manifest.json
└── .hidden/
    └── payload.py
```

and the last hint says after reading the flag, we have to output the flag content in the /xxx/xxx/xxx-xxx-xxx/xxx.xx

that means the output file should be at

```
 /api/plugin/a6a062f9-e036-49ef-98c7-9bb47152ce45/outputfile.json
```

So the python exploit code is 

```python
import base64
from pathlib import Path

p = Path(__file__).resolve().parent.parent
flag = Path("/opt/app/instance/flag.txt").read_text().strip()

(p / "result.json").write_text(
    "{\"flag\":\"" +
    base64.b64encode(flag.encode()).decode() +
    "\"}"
)
```

Upload it and read the flag, 

![](/images/Screenshot_2026-07-08_20-49-56.png)

![](/images/Screenshot_2026-07-08_20-50-41.png)

BOOM!


