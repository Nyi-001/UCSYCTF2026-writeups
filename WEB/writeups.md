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

