---
title: H1B Lottery Simulator
date: 2022-04-02 12:00:00 +0500
categories: [Computers and Security]
tags: [programming,h1b]
---

The following code is a simulation for the H1B lottery selection process. This script takes into account all years of the lottery and gives a guess of whether you'll be selected in the 3 years or not. It can also take into account whether you're an advanced degree candidate or not. This simulation can be run in a for loop unattended with output being redirected to a file for calculations. Example &rarr; run the code 5000 times and `grep` the output for required items to calculate percentages such as likelihood of selected in the first year, or in the lotteries that can occur in the middle of the year.

```python
import sys
import random

# Assumes lottery is filed for a total of 3 times (typical)
def do_lottery(id):
    # Lottery for the Advanced Degree set
    for i in range(20000):
        temp = random.randint(1, 80000)
        if temp == id:
            print("Congratulations!")
            return 1
    # Lottery for the remaining set
    for i in range(65000):
        temp = random.randint(1, 280000)
        if temp == id:
            print("Congratulations!")
            return 1
    # Lottery that may happen middle of the year
    for i in range(10000):
        temp = random.randint(1, 210000)
        if temp == id:
            print("Congratulations - middle of the year!")
            return 1
    return 0

# Set the argument as "masters" if applicant has advanced degree
if len(sys.argv) > 1 and sys.argv[1] == "masters":
    your_id = random.randint(1, 20000)
else:
    your_id = random.randint(20001, 85000)

print("\n==============================")
print("Year 1 Lottery")
accepted = do_lottery(your_id)
if accepted:
    print("You got it in the first year!!\n==============================")
    exit()

print("Year 2 Lottery")
accepted = do_lottery(your_id)
if accepted:
    print("You got it in the second year!!\n==============================")
    exit()

print("Year 3 Lottery")
accepted = do_lottery(your_id)
if accepted:
    print("You got it in the third year!!\n==============================")
    exit()

print("Unlucky!!\n==============================")
```
