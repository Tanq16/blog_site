---
title: Expense Tracking in Home Lab
date: 2023-25-09 22:53:00 -0000
categories: [Home Server]
tags: [budgeter,expense-tracking,home-lab]
---

# WIP

make it easy to store expense data
do it without unnecessary details (unless needed)
    this is because generally the analysis contains of what we're spending on and when
    like spent 200$ last week in food, but total 300$ in the last month in food is more important than had cheesecake factory 3 times in the last week
    at least for me, because I would like to monitor trends rather than specifics
store monthy data in JSON
plottable in any sort of format using that data
use it in the local network with Siri Shortcuts
come out of the clutches of other apps that make it hard to export data after a year or 2, to lock you in

CLI deploy

portainer deploy

convert data export from other app into this format. I used it on Spendee with this code -

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

for any other platform, try to get csv export and modify the fields either in the CSV or in code to match each other and that will allow making JSON files

the other piece is backup, which I'm doing manually for my home lab server so this data is included, but you can easily set up rsync or similar from the server via cron job and make a tarball from the expense data and back that single file up.
