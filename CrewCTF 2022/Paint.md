---
permalink: /Paint/
---

# Paint

We are given a .pcap file and the hint that the file was captured while somebody was drawing something on a ps4. Inside many HID packets.<br/>
Lets open it with wireshark! <br/>
At first I didn't see the HID Data Tab inside Wireshark but after trying other things I finally saw it:

<img src="./Paint/hiddata.png" width="500" height="350" />

I exported all the relevant packets with Wireshark and got a 50mb json file...

## Parsing the data

At first sight, everything seems normal but after trying to read all the data, I realized that there duplicate keys in the json file. Each button has the same key in the file so I had to first rename all the keys so that I could proceed.

### Fixing the mistake
```py
t = open("data.json").read().split("\n")
counter = 0
out = ""

for e in t:
    o = e
    if "usbhid.data.button" in e:
        counter+=1
        e.replace("ton","ton"+str(counter))
    if "usbhid.data.padding" in e:
        counter = 0
    out = out+e+"\n"

open("data_fixed.json","w").write(out)
```

The json file has objects that look like this now:

```json
"usbhid.data_tree": {
    "usbhid.data.report_id": "0x01",
    "usbhid.data.axis.x": "123",
    "usbhid.data.axis.y": "124",
    "usbhid.data.axis.z": "123",
    "usbhid.data.axis.rz": "124",
    "usbhid.item.local.usage": "0x39",
    "usbhid.data.button1": "0",
    "usbhid.data.button2": "0",
    "usbhid.data.button3": "0",
...
```
Important for us are just the x & y axis as well as the 2nd button which indicates if the user is drawing.
### Compressing the data
A 50mb file takes some time to process so I converted it to a smaller filesize by just keeping everything relevant:
```py
import json

data = json.load(open("data_fixed.json"))

new = list()

for e in data:
    if "usbhid.data_tree" in e["_source"]["layers"]:
        tree = e["_source"]["layers"]["usbhid.data_tree"]
        new.append(tree)

json.dump(new, open("small.json","w"))
```

Still 14mb but thats enough.

## Visualizing the movements

I wanted to visualize the movements of the cursor/controller so I choosed the turtle package in python which also animates it nicely.

Using the coordinates I got, I let the turtle draw an image. Whenever the 2nd Button gets pressed, I switched the turtle to drawing mode.

<img src="./Paint/turtle.png" width="200" height="200" />

It looks like movements... but not like a real drawing... <br/> From the shape of it, I would guess that it represents the stick itself!

Now we need to convert that to a proper image, where the movements of the stick correspond to the acceleration of the cursor.

```py
import json
import turtle
s = turtle.getscreen()
s.screensize(100000,10000)

t = turtle.Turtle()
t.pensize(2)
t.penup()
t.speed(10)

f = json.load(open("small.json"))

a = input()

for e in f:
    x = t.xcor()
    y = t.ycor()
    lxacc = int(e["usbhid.data.axis.x"])-123
    lyacc = int(e["usbhid.data.axis.y"])-123
    t.setx(x + int(lxacc/31))
    t.sety(y + (int(lyacc/31))*-1)

    if e["usbhid.data.button2"] != "0":
        t.pendown()
    else:
        t.penup()

p = input()
```
I needed to choose the canvas size so high as I did not know how to properly downscale everything. <br/>
The final result looks like this:

<img src="./Paint/flag.jpg" width="1000" height="100" />

Done!
