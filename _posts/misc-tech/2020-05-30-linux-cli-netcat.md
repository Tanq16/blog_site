---
title: Basics of Linux Commands and NetCat
date: 2020-05-30 12:00:00 -0000
categories: [Computers and Security]
tags: [linux,tool,netcat]
---

A shell is an interface to the operating system's services. All process are created by `fork()` and given a new direction by the `exec()` process. The trace of forks and calls can be seen from the strace command.

## Using shell script

This is more like using a shell to create more shell commands. A shell script is an executable which is executed by the terminal or the shell interpreter. Linux's default shell is usually the bourne again shell or `bash`. The default commands available for the shell can be written as a script to perform more complex functions and can be run as an executable. Aliases can also be used.

## Using programming languages like c and c++

The compiled code produces and executable which can be treated as a command. Example `C++` snippet &rarr;

```cpp
#include<iostream>

using namespace std;

int main()
{
    cout<<"Enter name - ";
    string s;
    cin>>s;
    cout<<endl<<"Welcome "<<s<<". System ready to rumble!!"<<endl;
    return 0;
}
```

## Common ground for the above two methods

In both cases, we need to run the commands using `./command` in the specific directory. This is not like a true command. All standard commands are stored as executables or as links to the executables in specific directories like `/bin/` or `/usr/bin/` etc. To actually call the command we do not specify the directory, we just specify the name without `./`. There is a `PATH` variable which stores the locations of all the mentioned folders, which are searched when a command is called. To emulate the working of standard commands, we need to add the directory to the `PATH` using a command as follows &rarr;

```bash
export PATH = "$PATH:<directory to be added>"
```

This will be valid for the particular session on the bash terminal. Therefore, to make the command always available, we need to add this command to the `.bashrc` or `.profile` file in the home directory.

## Complexity increased - Use python

Many packages are available for python which can be used to create commands for us which might actually help us in day to day work instead of using the commands as simple hello world programs. There are two types to classify this use as &rarr;

1. Standard for different packages.
2. Using [optparse](https://www.ibm.com/developerworks/aix/library/au-pythocli/) Refer to python code snippet for example on nmap package for python.

```python
#! /usr/bin/env python3

import nmap
import sys

nm=nmap.PortScanner()
i = 0
for ip in sys.argv:
    i = i+1
    if i==1:
        continue
    else:
        nm.scan(ip, '21-443')
        for host in nm.all_hosts():
            print("{} ({})".format(host, nm[host].hostname()))
            for proto in nm[host].all_protocols():
                for kk in nm[host][proto].keys():
                    if (kk==80) or (kk==443):
                        print("{} : {}".format(proto,kk))
```

> `#!` is the shebang directive. Using optparse, the commands we make will be very similar to the existing command structure. We can even have the help section and error messages inbuilt which make the commands more interactive.
{: .prompt-info }

## Argparse

The package `optparse` is now deprecated and replaced by `argparse` based on `optparse`. An example for a simple command to compute squares using `argparse` is as follows &rarr;

```python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument("square", type=int,
                    help="display a square of a given number")
parser.add_argument("-v", "--verbosity", type=int, choices=[0, 1, 2],
                    help="increase output verbosity")
args = parser.parse_args()
answer = args.square**2
if args.verbosity == 2:
    print("the square of {} equals {}".format(args.square, answer))
elif args.verbosity == 1:
    print("{}^2 == {}".format(args.square, answer))
else:
    print(answer)
```

The output is as follows &rarr;

```
$ python x.py
usage: x.py [-h] [-v {0,1,2}] square
x.py: error: too few arguments

$ python x.py 4
16

$ python x.py -v 0 4
16

$ python x.py -v 1 4
4^2 == 16

$ python -v 2 4
the square of 4 equals 16
```

---

## NetCat 

### Use netcat to chat

Use listen command on the server side. `nc -l -p <port>` Connect to the server from client on the same port.`nc <server> <port>` Can be done on the same machine also.

### Use netcat to send a file

Use server machine to give command`nc -v -w 30 -p <port> -l < file.txt` On the client machine give command `nc -v -w 2 <server ip> <port> > receivedfile.txt` -w is for wait and the -v is for verbose output. The file.txt will be received as receivedfile.txt on the client machine. Works on local machine as well. Test.

### Netcat for banner grabbing

It can be done with telnet as well but this does not alter the stream of data unlike telnet. Just nc to the ip using a specific port to get the info regarding the server running.

### Port scanning

Use -z for zero input output. `nc -v <ip> -z <port-start>-<port-end>` -n option can also be used. This does not do a DNS scan on the given ip address which does save time. Even -w can be used to wait specified number of seconds.

### Execute remote shell on windows

To execute a remote shell on windows, get an nc command executed as follows `nc -lp <port> -vv -e cmd.exe` Then execute nc on the attacking machine as follows `nc <ip> <port>` It is always unencrypted. Since this is always unsafe, there is a version of netcat called cryptcat which has two fish encryption. If the listener uses the -e option it is called a direct shell. If the connecting machine uses the -e option it is called a reverse shell. This can help in situations where not both the machines on the network can port forward.

### Netcat to transfer files between two systems on a network

To transfer a folder having the files to be sent, on the sender use command `tar -cf - <foldername> | nc -l -p 1337` On the receiving machine type command `nc <sender ip> 1337 | tar -xf -`

### Ncat

ncat is a more modern version of netcat which is implemented by nmap libraries. It has support for multiple protocols and transmission over ssl. On the listener type `ncat [options] [arguments] --allow <allowed ip> -vln <port> --ssl` Only connections from allowed ip will be allowed will have an encrypted channel. The connection will be allowed but not established form the allowed ip if the command from that ip does not contain the ssl option. On allowed ip type `ncat -vn <listener> <l port> --ssl`

## Resources

[Argparse](https://docs.python.org/3/library/argparse.html#module-argparse)
