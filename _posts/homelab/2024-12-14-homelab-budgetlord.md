---
title: Expense Tracking in Home Lab
date: 2024-12-14 10:10:00 -0400
categories: [Home Server]
tags: [budgetlord,expense-tracking,home-lab]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

## Whats and Whys

I wrote a tool called [BudgetLord](https://github.com/Tanq16/budgetlord) for one straightforward reason - to track expenses. There are `so` `So` `sO` `SO` many tools out there that do the same. And they do it so well, with so many features. What I wrote is very simple, so why not just use one of the existing tools? I was using one too! I preferred Spendee for a while but migrated from it. I also looked at Cashew, but still migrated away from it. I wanted something for my home server and something that does exactly what I want.

BudgetLord is primarily a simple API-based service to which I can add data using HTTP requests. I wanted data addition to be very quick, so HTTP requests allow me to add data through a Siri Shortcut on my iPhone, which also doubles as a quick and friendly UI to add data. I can also create a custom pie chart the way I want to. Lastly, I wanted to do it without adding unnecessary details (unless needed). This is because, generally, the analysis that matters to me needs information on what I'm spending on and when. For example, "*I spent 200$ last week on food, but a total of 300$ in the last month on food*" is more important than "*I had cheesecake factory 3 times in the last week*". This means skipping the notes when needed, wallet type, budget book, etc., and sticking to 2 main things - `category` and `amount`. I also don't need to maintain my entire net worth on an app by tracking income because income trickles in regularly, and I can monitor that through my banking apps and emails. Monitoring trends rather than specifics is essential, so I decided to store data in JSON to make it easily readable on a monthly basis.

Another reason for getting out of third-party apps is to reduce tracking and service vendor lock-in. I appreciate the hard work of app developers and hope they get funded for their work, but at the same time, if I can write code in some capacity, why commit to another app unnecessarily when it's more interesting this way? Also, The vendor locking thing shows up in random places for different apps; for example, I couldn't export `my` expense data on a third-party app that was older than a year without a premium subscription..... 🤔 ???? it's my data......!!! 🙄 You can gatekeep features but not my own data!! Alas, here's my app.

## Deployment

Make a persistence directory on your homelab machine &rarr;

```bash
mkdir -p $HOME/budgetlord_data
```

Then run the application through this Docker command &rarr;

```bash
docker run -d \
  --name budgetlord \
  -p 8080:8080 \
  -v $HOME/budgetlord_data:/app/data \
  tanq16/budgetlord:main
# use tag :main_arm for ARM64 image
```

After this, the container will be deployed at `localhost:8080`. A better way to deploy it is using docker-compose either directly or through Portainer/Dockge/etc. &rarr;

```yaml
services:
  BudgetLord:
    image: tanq16/BudgetLord:main
    container_name: BudgetLord
    volumes:
      - /home/username/budgetlord_data:/expense-data # replace with correct path
    ports:
      - 5002:8080
```

Lastly, you could also download the releases from the [project releases](https://github.com/Tanq16/BudgetLord/releases) to run the app as a binary (available for Linux, MacOS, and Windows for both ARM64 and x86_64).

## Backing Up

Backups are another piece of the puzzle. I'm doing manual backups for my home lab server, so this data is included automatically. Still, you can easily set up `rsync` for backing up to local NAS or other more sophisticated backup utilities triggered via cron jobs. In the spirit of automating everything, you could also use Dropbox API and Python in a script triggered by cron jobs to make a tarball from the expense data and back that single file to the cloud (I have yet to attempt this one).

## Migration

I migrated from Spendee after exporting a CSV from the service. Custom scripts are required for different types of exports. For Spendee, I used this code to convert my expense data to the format I needed &rarr;

```python
import csv
import json
from datetime import datetime

# Define a dictionary to map category names to actual values
category_mapping = {
    "Credit Card Offer": "ignore", "Food & Drink": "food", "Miscellaneous ": "personal",
    "Car/Cab": "travel", "Family": "family", "Groceries": "groceries", "Healthcare": "personal",
    "Home": "home", "Investments": "investing", "Miscellaneous": "personal", "Personal": "personal",
    "Refunds": "ignore", "Salary": "ignore", "Shopping": "shopping", "Subscriptions": "subscriptions",
    "Transport": "travel", "Travel": "travel"
}

def preprocess_data(row):
    amount = -float(row["Amount"])
    category = category_mapping.get(row["Category name"], row["Category name"].lower())
    reduced_date = datetime.strptime(row["Date"], "%Y-%m-%dT%H:%M:%S+00:00").strftime("%d-%m-%Y")
    return {
        "date": reduced_date,
        "category": category,
        "amount": amount,
        "note": row["Note"]
    }

final = {}
with open('transactions.csv', 'r') as csvfile:
    reader = csv.DictReader(csvfile, delimiter=',')
    for row in reader:
        processed_data = preprocess_data(row)
        month_year = processed_data["date"][3:10]
        if processed_data["category"] == "ignore":
            continue
        if processed_data["amount"] < 0:
            continue
        if not month_year in final:
            final[month_year] = []
        final[month_year].append(processed_data)

for i in final:
    with open(i + "-expenses.json", 'w') as jsonfile:
        json.dump(final[i], jsonfile, indent=4)
```

For any other platform, try to get CSV export ready and modify the fields directly in the CSV to match those of the above script or modify the script to match the export data.

## Analysis Views

The app presents two interfaces that automatically adapt to dark mode preferences &rarr;

Dashboard View &rarr;

  - Interactive pie chart showing expense distribution
  - Custom legend with percentage breakdowns
  - Real-time category totals
  - Seamless month-to-month navigation

Table View &rarr;

  - Chronological expense listing
  - Quick category reference
  - Formatted currency display
  - Responsive mobile design
  - Expense deletion with confirmation

Singular expenses can be deleted from the Table view for convenience.

## Adding Data

To add expense data to the application, use a cURL request like so &rarr;

```bash
curl -X PUT http://localhost:8080/expense \
-H "Content-Type: application/json" \
-d '{
    "name": "Groceries",
    "category": "Food",
    "amount": 75.50,
    "date": "2024-03-15T14:30:00Z"
}'
```

The `budgetlord` can also be used as a client to add data by pointing it to the server like so &rarr;

```bash
budgetlord -addr "192.168.1.13:8080"
```

This will automatically set the time for you and ask the other details before making the HTTP request (abstracted).

## Tips

In addition to using the binary or cURL to add expenses, iPhones allow using Siri Shortcuts to make HTTP requests. A shortcut can be defined to add expenses easily, which is what I do too.

Lastly, the application can also be installed as a PWA (progressive web app), which also helps add an icon to smartphone home screens. This opens up the app easily to allow analysis as needed.
