---
title: Reddit Feeds via Discord WebHooks
date: 2022-02-18 12:00:00 -0000
categories: [Home Server]
tags: [home-lab,productivity,webhooks,discord,reddit]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

## News Feeds from Reddit

This code is to be run on a linux server. A subreddit page can be loaded without authentication and has a huge JSON object which contains information about the window that is loaded in the browser. This can be extracted and the corresponding titles of posts can be pulled from the first page.

By sorting the feed by Hot and adding a condition to allow posts with the upvote to downvote ratio of higher than 0.9, the best headlines can be collated and sent to the WebHook.

The following script uses regex to extract the JSON object and can get unique news headlines with their links across several sends. The uniqueness is guaranteed by keeping a log of the SHA256 hash of the headlines, which is cleared every month leaving only the last 6 entries. The following code is to be run as a cron job as listed in the first comment &rarr; 

```python
#!/usr/bin/python3
# cron job runs as follows - 0 20 * * *

import re
import json
import requests
import hashlib
import datetime

URL = "https://discord.com/api/webhooks/<><><>" # add you channel webhook
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


The script uses a file `reddit_technews_log` as a log to maintain SHA256 hashes of previously produced titles. A comparison to those hashes help keep the news headlines unique. Be sure to replace the WebHook URL with the appropriate URL as created within a Discord channel.

## Meme Feeds from Reddit

This is similar to the news feed. The difference is that embeds are being instead to post image type memes only. The code to do this is as follows and also needs to be deployed via cron jobs &rarr; 

```python
#!/usr/bin/python3
# cron runs as follows - 0 0 1,5,10,15,20,25 * *

import json
import requests
import re
import time
import hashlib

URL = "https://discord.com/api/webhooks/<><><>"

def discord_message(link):
    r = requests.post(URL, json = {"embeds": [{"image": {"url": link}}]})
    return

r = requests.get("https://www.reddit.com/r/memes/hot/", headers={"User-Agent": "Firefox"})
temp = re.search(r"<script id=\"data\">window\.___r = .*?;</script>", r.text).group(0)
data_1 = re.sub("<script id=\"data\">window\.___r = (.*?);</script>", r"\1", temp)
data_1 = json.loads(data_1)

posts_1 = data_1['posts']['models']

memes_collection = []

for i in posts_1.keys():
    if posts_1[i]['upvoteRatio'] > 0.9 and posts_1[i]['isSponsored'] == False:
        if 'content' in posts_1[i]['media'].keys() and posts_1[i]['media']['type'] == 'image':
            memes_collection.append(posts_1[i]['media']['content'])
    if len(memes_collection) > 2:
        break

for i in memes_collection:
    discord_message(i)
```
