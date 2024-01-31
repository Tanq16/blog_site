---
title: Battery Monitor via Discord WebHooks and Siri Shortcuts
date: 2022-02-18 12:00:00 -0000
categories: [Home Server]
tags: [home-lab,webhooks,discord,siri-shortcuts,battery-monitor]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

## The What and Why

This post details how Siri shortcuts and discord webhooks can be used in tandom with a smart plug to turn a server charger on or off based on battery levels. This is generally only helpful for servers that are actually laptops ith decent battery that needs to be preserved rather than keeping it plugged in for long. Another more centralized way to do that would be to use open source Home Assistant docker container with supported smart plugs to do everything directly from the server. This post deals with doing it via Siri shortcuts instead, since that was what I had available for the HomeKit supported devices I own.

The other purpose of these notifications are to just tell the owner about battery status of several server devices in the home network whose charge is about to run out or might be on charge for a while, such as plugging the server in when it is nearing around 20% charge capacity and unplugging it once it reaches 100% charge and stays connected. The notifications can help in manually turning the plug on or off.

This post will go over 2 possible methods, the first one to just inform about battery levels, the second to automatically perform an action via Siri shortcuts. Obviously, Siri shortcuts can be replaced with anything else that fulfils the purpose, but I only had that to experiment with at the time.

## Notifications via Discord WebHooks

The following code reads the battery energy level now as well as the full capacity. It also reads the status of the AC adapter and follows the cron job to tell the owner every 10 mins if the battery falls below 25%. It also tells the owner if the charger is connected with the battery at 100%, every 10 mins between 6 AM and 8 PM.

```python
#!/usr/bin/python3
# cron job needs to run as - 0,10,20,30,40,50 * * * *

import requests
import datetime

URL = "https://discord.com/api/webhooks/<><><>" # replace with the url from the channel

def discord_message(percentage = "", content = "Cosmos13 Battery Level = "):
    r = requests.post(URL, data = {"content": content + str(percentage)})
    return

with open("/sys/class/power_supply/BAT0/energy_now") as f:
    now = f.read()
with open("/sys/class/power_supply/BAT0/energy_full") as f:
    full = f.read()
with open("/sys/class/power_supply/ADP0/online") as f:
    ac_connected = int(f.read().strip("\n"))

percentage = (int(now.strip("\n"))*100)//int(full.strip("\n"))
timenow = datetime.datetime.now()

if percentage < 29 and ac_connected == 0:
    discord_message(percentage)
elif ac_connected == 1 and percentage > 95 and timenow.hour > 7 and timenow.hour < 21:
    discord_message(content = "Cosmos13 AC Still Connected")
```

## Automation via Siri Shortcuts and Flask

The following method is another way to do the turn off and turn on automatically. The requirements are as follows &rarr;

1. Have a smart plug connected via Google Home or Home Kit (in this case it is via Google Home)
2. A flask server running on the server machine
3. Consistent automation for Siri Shortcuts

The web server must deploy the following code &rarr;

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from flask import Flask
import datetime

app = Flask(__name__)
@app.route('/')
def index():
    with open("/sys/class/power_supply/BAT0/energy_now") as f:
        now = f.read()
    with open("/sys/class/power_supply/BAT0/energy_full") as f:
        full = f.read()
    with open("/sys/class/power_supply/ADP0/online") as f:
        ac_connected = int(f.read().strip("\n"))
    percentage = (int(now.strip("\n"))*100)//int(full.strip("\n"))
    timenow = datetime.datetime.now()
    if percentage < 19 and ac_connected == 0:
        return "PlugIn"
    elif ac_connected == 1 and percentage > 95:
        return "PlugOut"
    else:
        return "NoStatus"
if __name__ == '__main__':
    app.run(debug=True, port=9090, host='0.0.0.0')
```

Running this using `python3 server.py &` serves a web server that informs the status of the battery with `PlugIn`, `PlugOut` and `NoStatus` at a web request. Next, program a shortcut on Siri Shortcuts using the following &rarr;

1. Get Content from URL `http://server:9090`
2. Get Text from Input
3. If text is equal to `PlugIn`, run the Google Assistant action in the shortcut to turn smart plug on
4. If text is equal to `PlugOut`, run the Google Assistant action in the shortcut to turn smart plug off
5. If text is equal to `NoStatus`, do nothing
6. (BONUS) &rarr; A test can be added in a wrapping `if` statement to check if the variable has a value in these 3 items list; if not, then play a notification to inform that something might be wrong with the web server

Next is the hard part &rarr; Make automations on the iPhone device to run this shortcut at every required interval. The values can be modified according to the interval required.

As best practice, also make an automation to check the battery level of iPhone device twice each day.

To monitor the battery levels for the iPhone using Siri shortcuts, three cases can be considered &rarr; 

- Daily at 9:00 PM
- When battery falls below 30%
- When battery falls below 20%

The script setup for it is as follows &rarr; 

1. Get Battery Level
2. Add Battery Level to variable `batt` 
3. Get contents of URL (discord webhook)
	1. Set Method to POST
	2. Add header `Content-Type: application/json` 
	3. Add request body `{"content": "iPhone 7 Battery Report" + variable batt}`
