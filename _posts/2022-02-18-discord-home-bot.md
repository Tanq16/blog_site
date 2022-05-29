---
title: Home Bot via Discord
date: 2022-02-18 12:00:00 -0000
categories: [Home Server]
tags: [github,discord,webhook,server,home-lab,code,notifications]
---

# Battery Levels

The purpose of these notifications are to tell the owner about battery status of several devices in the home network whose charge is about to run out or might be on charge for a while. This is mainly effective with home servers for the scenario when battery needs to be preserved, such as plugging the server in when it is nearing around 20% charge capacity and unplugging it once it reaches 100% charge and stays connected.

Two devices configured for this are &rarr; Linux Home Server and spare iPhone 7.

### Linux Server

The following code reads the battery energy now as well as the full capacity. It also reads the status of the AC adapter and follows the cron job to tell the owner every 10 mins if the battery falls below 25%. It also tells the owner if the charger is connected with the battery at 100%, every 10 mins between 6 AM and 8 PM.

```python
#!/usr/bin/python3
# cron job needs to run as - 0,10,20,30,40,50 * * * *

import requests
import datetime

URL = "https://discord.com/api/webhooks/<><><>"

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

### iPhone

To do something similar as the Linux server, Siri Shortcuts can be used in iOS. The three cases this is deployed in are as follows &rarr; 

- Daily at 9:00 PM
- When battery falls below 30%
- When battery falls below 20%

The script setup for it is as follows &rarr; 

1. Get Battery Level
2. Add Battery Level to variable `batt` 
3. Get contents of URL (webhook)
	1. Set Method to POST
	2. Add header `Content-Type: application/json` 
	3. Add request body `{"content": "iPhone 7 Battery Report" + variable batt}` 

# News Feeds from Reddit

This is to be run on a linux server. A subreddit page can be loaded without authentication and has a huge JSON object which contains information about the window that is loaded in the browser. This can be extracted and the corresponding titles of posts can be pulled from the first page.

Sorting the feed by Hot and adding a condition to allow posts with the upvote to downvote ratio of higher than 0.9, the best headlines can be collated and sent to the WebHook.

The following script uses regex to extract the JSON object and can get unique news headlines with their links across several sends. The uniqueness is done by keeping a log of the SHA256 hash of the headlines, which is cleared every month leaving only the last 6 entries &rarr; 

```python
#!/usr/bin/python3
# cron job runs as follows - 0 20 * * *

import re
import json
import requests
import hashlib
import datetime

URL = "https://discord.com/api/webhooks/<><><>"
def discord_message(content = "test"):
    r = requests.post(URL, data = {"content": content})
    return

r = requests.get("https://www.reddit.com/r/technews/hot/", headers={"User-Agent": "Firefox"})
temp = re.search(r"<script id=\"data\">window\.___r = .*?;</script>", r.text).group(0)
data = re.sub("<script id=\"data\">window\.___r = (.*?);</script>", r"\1", temp)
data = json.loads(data)

with open("/home/tanq/installations/reddit_technews_log") as f:
    log_hashes = f.read().split("\n")

posts = data['posts']['models']
news_collection = []

for i in posts.keys():
    if posts[i]['upvoteRatio'] > 0.9 and posts[i]['isSponsored'] == False:
        if posts[i]['source']['url']:
            current_hash = hashlib.sha256(posts[i]['title'].encode()).hexdigest()
            if current_hash in log_hashes:
                pass
            else:
                with open("/home/tanq/installations/reddit_technews_log", "a") as f:
                    f.write(current_hash)
                    f.write("\n")
                news_collection.append((posts[i]['title'], posts[i]['source']['url']))
    if len(news_collection) > 6:
        break

content = "__*Here is today's news collection from* `r/technews` *subreddit:*__\n\n"
for i in news_collection:
    content = content + '[link](<' + i[1] + '>)' + ' - ' + i[0] + "\n"
content += "\n\n"

discord_message(content)

day = datetime.datetime.now().day
if day == 1:
	log_hashes = log_hashes[-14:]
	with open("/home/tanq/installations/reddit_technews_log", "w") as f:
		for i in log_hashes:
			f.write(i)
			f.write("\n")
```


The script also uses a file `reddit_technews_log` as a log to maintain SHA256 hashes of previously produced titles. Comparison to those hashes help keep the news headlines unique.

# Meme Feeds from Reddit

This is similar to the news feed. The difference is that embeds are being used for this to post image type memes only. The code to do this is as follows &rarr; 

```python
#!/usr/bin/python3
# cron runs as follows - 0 0 1,5,10,15,20,25 * *

import json
import requests
import re
import time
import hashlib

URL = "https://discord.com/api/webhooks/952312972691247105/m59i6ejPGWGpWNRUOSlJ1OPPXN_H5fPhxjDJXD3ddvU0moKQR2ddS6eW_HEx4qN1W-yS"

def discord_message(link):
    r = requests.post(URL, json = {"embeds": [{"image": {"url": link}}]})
    return

r = requests.get("https://www.reddit.com/r/memes/hot/", headers={"User-Agent": "Firefox"})
temp = re.search(r"<script id=\"data\">window\.___r = .*?;</script>", r.text).group(0)
data_1 = re.sub("<script id=\"data\">window\.___r = (.*?);</script>", r"\1", temp)
data_1 = json.loads(data_1)

r = requests.get("https://www.reddit.com/r/thensfwmemes/hot/", headers={"User-Agent": "Firefox"})
temp = re.search(r"<script id=\"data\">window\.___r = .*?;</script>", r.text).group(0)
data_2 = re.sub("<script id=\"data\">window\.___r = (.*?);</script>", r"\1", temp)
data_2 = json.loads(data_2)

posts_1 = data_1['posts']['models']
posts_2 = data_2['posts']['models']

memes_collection = []

for i in posts_1.keys():
    if posts_1[i]['upvoteRatio'] > 0.9 and posts_1[i]['isSponsored'] == False:
        if 'content' in posts_1[i]['media'].keys() and posts_1[i]['media']['type'] == 'image':
            memes_collection.append(posts_1[i]['media']['content'])
    if len(memes_collection) > 2:
        break

for i in posts_2.keys():
    if posts_2[i]['upvoteRatio'] > 0.9 and posts_2[i]['isSponsored'] == False:
        if 'content' in posts_2[i]['media'].keys() and posts_2[i]['media']['type'] == 'image':
            memes_collection.append(posts_2[i]['media']['content'])
    if len(memes_collection) > 5:
        break

for i in memes_collection:
    discord_message(i)
```

# GitHub Notifications

WebHooks can be added to GitHub repositories to send all details to discord channels. The WebHook URL should be appended with a `/github` before adding it to GitHub. Also, the content type should be changed to `application/json`.
