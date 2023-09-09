---
title: 10k GitHub Repositories Secret Scanning Code
date: 2022-03-10 12:00:00 +0500
categories: [Computers and Security]
tags: [github,programming,secret-scan,security]
---

The API used to search git repositories is &rarr; 

```
https://api.github.com/search/repositories
```

The code to pull 10000 repositories' URLs is as follows &rarr; 

```python
import time
import json
import requests

github_urls = []
stars = ['0..1', '2..3', '4..5', '6..8', '9..11', '12..15', '16..20', '21..25', '26..40', '41..60', '61..90', '91..120', '121..150']

for star_count in stars:
    for i in range(10):
        r = requests.get("https://api.github.com/search/repositories?q=stars:" + star_count + "&per_page=100&page=" + str(i), headers = {"Accept": "application/vnd.github.v3.text-match+json"})
        data = json.loads(r.text)
        print(star_count, i, data['total_count'])
        if not 'total_count' in data.keys():
            time.sleep(7)
            continue
        for j in data['items']:
            github_urls.append(j['html_url'])
        time.sleep(8)

file = open("github.urls", "w")
for i in github_urls:
    file.write(i)
    file.write("\n")
file.close()
```

To clone all repos, use the following snippet &rarr; 

```bash
for i in $(head -n 1000 ~/github.urls | sort -u); do git clone --depth=1 $i; sleep 5; done
```

Secrets can be found using the following bash script &rarr; 

```bash
echo "======AWS API Key"
grep -niroE "\b((A3T[A-Z0-9]|AKIA|AGPA|AIDA|AROA|AIPA|ANPA|ANVA|ASIA)[A-Z0-9]{16})\b" --exclude-dir ".git/"

echo "======AWS Secret Access Key"
grep -niroE "\baws_?(secret)?_?(access)?_?(key)?.?\s{0,30}.?\s{0,30}.?([a-z0-9/+=]{40})\b" --exclude-dir ".git/"

echo "======AWS Account ID"
grep -niroE "aws_?(account)_?(id)?.?\s{0,30}.?\s{0,30}.?([0-9]{4}-?[0-9]{4}-?[0-9]{4})" --exclude-dir ".git/"

echo "======AWS Session Token"
grep -niroE "(aws.?session|aws.?session.?token|aws.?token).?\s{0,30}.?\s{0,30}.?([a-z0-9/+=]{16,200})[^a-z0-9/+=]" --exclude-dir ".git/"

echo "======Slack Token"
grep -niroE "\b(xox[pboa]-[0-9]{12}-[0-9]{12}-[0-9]{12}-[a-z0-9]{32})\b" --exclude-dir ".git/"

echo "======Generic Secrets & Keys"
grep -niroE "(sshpass|password|pass|pw|secret|key|api|access).?(key)?.{0,20}\b([0-9a-z]{24,64})\b" --exclude-dir ".git/"

echo "======Google OAuth Client ID"
grep -niroE "([0-9]+-[a-z0-9_]{32})\.apps\.googleusercontent\.com" --exclude-dir ".git/"

echo "======Google OAuth Client Secret"
grep -niroE '.?client_secret.?\s{0,30}.?\s{0,30}"([a-zA-Z0-9_-]{24})"' --exclude-dir ".git/"

echo "======Google OAuth Access Token"
grep -niroE "\b(ya29\.[0-9A-Za-z_-]{20,64})\b" --exclude-dir ".git/"

echo "======Google API Keys"
grep -niroE "\b(AIza[0-9A-Za-z_-]{35})\b" --exclude-dir ".git/"

echo "======Stripe API Key"
grep -niroE "\b((sk|rk)_live_[a-z0-9]{24})\b" --exclude-dir ".git/" # live can be substituted for test to get test keys

echo "======GitHub App|Refresh|Personal|OAuth Access Token"
grep -niroE "\b((ghr|gho|ghp|ghu|ghs)_[a-zA-Z0-9]{36})\b" --exclude-dir ".git/"

echo "======Twilio API Key"
grep -niroE "twilio.{0,20}\b(sk[a-f0-9]{32})\b" --exclude-dir ".git/"

echo "======Facebook Access Token"
grep -niroE "\b(EAACEdEose0cBA[a-zA-Z0-9]+)\b" --exclude-dir ".git/"

echo "======Twitter Access Token"
grep -niroE "twitter[\s.]{1,4}[1-9][0-9]+-[0-9a-zA-Z]{40}" --exclude-dir ".git/"

echo "======Okta API Token"
grep -niroE "(okta|ssws).{0,20}\b(00[a-z0-9_-]{39})[a-z0-9_]\b" --exclude-dir ".git/"

echo "======bCrypt Hashes"
grep -niroE "(\$2[abxy]\$\d+\$[./A-Za-z0-9]{53})" --exclude-dir ".git/"
```
It can be invoked as follows &rarr; 

```bash
zsh secret_scan.sh | tee -a ~/secret_scan.log
```

To scan for multiple commits in a git repo, use the following &rarr; 

```bash
for i in $(git --no-pager log | grep "^commit" | cut -d ' ' -f2); do git checkout $i 1>/dev/null 2>/dev/null; zsh /persist/secret_scan.sh; done | tee -a secret_scan.log
```

The following script can convert the results to a JSON object and store in `secret_scan.json`. It can remove duplicate keys from the same file.

```python
import sys
import json

if len(sys.argv) > 1:
        filename = sys.argv[1]
else:
        filename = "secret_scan.log"
f = open(filename)
data = f.read().split("\n")[:-1]
f.close()

collection = {}
final = {}
index = ""

for i in data:
    if i[:6] == "======":
        index = i[6:]
        if not index in collection.keys():
            collection[index] = []
    else:
        collection[index].append(i)

for index in collection.keys():
    if len(collection[index]) == 0:
        continue
    else:
        x = sorted(collection[index])
        check = x[0]
        y = []
        y.append(check)
        for i in range(1, len(x)):
            splitone = check.split(':')
            splittwo = x[i].split(':')
            if splitone[0] == splittwo[0] and splitone[-1] == splittwo[-1]:
                continue
            else:
                check = x[i]
                y.append(check)
        final[index] = y

f= open("secret_scan.json", "w")
f.write(json.dumps(final))
f.close()
```
