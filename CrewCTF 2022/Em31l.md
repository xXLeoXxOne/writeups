---
permalink: /CrewCTF%202022/Em31l/
---

# Em31l
## Pt. 1
We get an email file (.eml) and the challenge is to find out what was deleted from the file...

```
Received: from 10.197.33.204...
Return-Path: <crewctff@gmail.com>
X-Originating-Ip: [209.85.221.45]
Received-SPF: pass (domain of gmail.com designates 209.85.221.45 as permitted sender)
Authentication-Results: ...
X-Apparently-To: zheroboy2015@yahoo.com; Fri, 15 Apr 2022 06:03:02 +0000
X-YMailISG: ...
Received: from 209.85.221.45...
Received: by mail-wr1-f45.google.com with SMTP id u3so9536261wrg.3...
DKIM-Signature: ...
X-Google-DKIM-Signature: ...
X-Google-Smtp-Source: ABdhPJyaUwECUfiVnHLMvCVoOGSDlrFjphDMOXwSo8pSUztrUcs+gK7lHOKWwReyKsHGHeQG13Psbc5aQ2asjRuWTvE=
X-Received: by 2002:a05:6000:1564:b0:20a:7727:27b0 with ...
MIME-Version: 1.0
From: crew ctf <crewctff@gmail.com>
Date: Fri, 15 Apr 2022 08:02:49 +0200
Message-ID: <CAG+6dK2LBZJeWOCXgSp_JnxShdjjkuoXpcxp3ChSLQ6dfkUjZQ@mail.gmail.com>
Subject: Help me!
To: zheroboy2015@yahoo.com
Content-Type: multipart/alternative; boundary="000000000000c0332a05dcab29d2"
Content-Length: 677

--000000000000c0332a05dcab29d2
Content-Type: text/plain; charset="UTF-8"

Hey, crushed kiwi I hate this loop of college, and I need your help. Can
you meet me at lost immediately?

--000000000000c0332a05dcab29d2
Content-Type: text/html; charset="UTF-8"
Content-Transfer-Encoding: quoted-printable

...

--000000000000c0332a05dcab29d2--
```

I realized that it has to be from a google email so I sent myself an email and compared both. <br/>
The `X-Gm-Message-State:` Header is missing!

This was the solution for the first part of the challenge.

The second part was a little harder...

## Pt. 2

In this part, we still have the same email file from the first part but need to find out what the redacted word inside the email was. I informed myself and after 15 minutes of researching I found out that the `DKIM-Signature:` Header is important, specifically the body hash `bh=`. We need to correctly canonicalize and hash the body to be able to bruteforce the lost word. I sent myself another email and tried calculating it myself. It was hard to find out the correct way...

### Calculating the correct hash

```py
from Crypto.Hash import SHA256

data = open("data").read()
from base64 import b64encode

def hash_body(body: str) -> str:
    canonicalized_body = body.strip().encode().replace(b"\n",b"\r\n") + b"\r\n"
    bh = b64encode(SHA256.new(canonicalized_body).digest())
    return bh.decode()

print(hash_body(data))
```

With:
```
--0000000000002b27b205dcc6879c
Content-Type: text/plain; charset="UTF-8"

af

--0000000000002b27b205dcc6879c
Content-Type: text/html; charset="UTF-8"

<div dir="ltr">af</div>

--0000000000002b27b205dcc6879c--
```
Inside the data file.

### The solution

Now that we are able to correctly calculate the body hash, we can finally brute force the lost word!<br/>
I imported the body into my program and wrote a little script that could find the correct word from the original body hash and the rest of the email.<br/>
Lets just hope that the word is not too long and only contains letters...

```
--000000000000c0332a05dcab29d2
Content-Type: text/plain; charset="UTF-8"

Hey, crushed kiwi I hate this loop of college, and I need your help. Can
you meet me at lost immediately?

--000000000000c0332a05dcab29d2
Content-Type: text/html; charset="UTF-8"
Content-Transfer-Encoding: quoted-printable

<div dir=3D"ltr"><span style=3D"color:rgb(29,34,40);font-family:Helvetica,A=
rial,sans-serif;font-size:13px">Hey, crushed kiwi I hate this loop of colle=
ge, and I need your help. Can you meet me at=C2=A0</span>lost<span style=3D=
"color:rgb(29,34,40);font-family:Helvetica,Arial,sans-serif;font-size:13px"=
>=C2=A0immediately?</span><br></div>

--000000000000c0332a05dcab29d2--
```

```py
import sys

from Crypto.Hash import SHA256

target = "5AqaoLYxMopB/cECaLwYX3ZR0XSAPW38Fwpy5WHeO2M=" # Body hash from the eml file (bh=)
bod = open("data").read()
from base64 import b64encode

def hash_body(body: str) -> str:
    canonicalized_body = body.strip().encode().replace(b"\n",b"\r\n") + b"\r\n"
    bh = b64encode(SHA256.new(canonicalized_body).digest())
    return bh.decode()

print(hash_body(bod))

a = list("abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ")
for i in a:
    for ii in a:
        for iii in a:
            for iiii in a:
                e = i+ii+iii+iiii
                print(e)
                if hash_body(bod.replace("lost", e)) == target:
                    print("Heureka!")
                    sys.exit()
```

That worked!

The lost word was `abay`.

*Both challenges gave my team a 2nd blood in the ctf.*
