---
title: Linux Device Drivers
date: 2019-09-12 12:00:00 +0500
categories: [Computers and Fun]
tags: [linux,drivers,kernel]
---

This is a practical study I made using some videos and written resources to understand device drivers to some extent.

## Building a module

Device drivers hide the details of how the device works. User activities are performed by standardized calls that are independent of the specific driver. Device drivers map these calls to device-specific operations that act in real hardware. Drivers can be integrated into the kernel as well as built separately and dynamically loaded at runtime when needed. These are called loadable modules. Linux looks at device drivers in three different fundamental types -

1. `Character devices`
2. `Block devices`
3. `Network devices`

Character device is one that can be accessed as a stream for bytes. Uses a char driver. Eg Keyboard, mouse, camera, etc. Most char devices are just data channels which can be accessed sequentially. Block devices are those which has a file system on it i.e., we can create files and directories and there is an option of moving back and forth. Reads from block devices are in chunks or blocks of data. Eg. pen drive, disks, etc.

Making of a simple dynamically loadable program driver which can be added and removed. `example.c` -

```cpp
#include<linux/init.h>
#include<linux/module.h>

int simplemod_init(void)
{
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
    return 0;
}
void simplemod_exit(void)
{
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
}
module_init(simplemod_init);
module_exit(simplemod_exit);
```

The init function gets called when the module is inserted/loaded in the kernel space. Module can be initialized here. The exit function is called when the module is removed/unloaded. Cleanups can be performed here. Last two lines are helper modules to identify the functions.

For a simple module to be compiled om a linux system, we need to have a build directory on which we will execute the module. Requirement is the kernel build directory for the version of the kernel running on the machine. To do that type on terminal -

```bash
uname -r
```

We need the directory which contains the binary for the returned version of linux. Therefore, to compile the module we make a makefile. `Makefile` -

```bash
obj-m:=example.o
```

This states a need to build a module (m for module). It then generates the specified `.o` file from the corresponding `.c` file and then generates the `.ko` file from the `.o` file. `:=` is used to initialize the obj-m list of files to be compiled to a string on the right hand side. `+=` is used to add to the obj-m list of files to be compiled. To build the module, type on terminal -

```bash
make -C /lib/modules/$(uname -r)/build M=$PWD modules
```

- `-C` is to change the directory to the `/lib/module/<running kernel>/build` directory. The make utility goes to the specified directory and pickup the makefile from there which would contain all info about processor, compilers to be used, optimization done to the kernel, etc.
- But we need to build the own module, not the kernel. So we specify `M=$PWD` which specifies to pickup the makefile from the present working directory. The `modules` in the end specifies what to do (build a module). We see the result as -

```
make: Entering directory '/usr/src/linux-headers-4.13.0-26-generic'
  CC [M]  /home/asus/TANISHQ/example.o
  Building modules, stage 2.
  MODPOST 1 modules
  CC      /home/asus/TANISHQ/example.mod.o
  LD [M]  /home/asus/TANISHQ/example.ko
make: Leaving directory '/usr/src/linux-headers-4.13.0-26-generic'
```

We would now have a file by the name `example.ko` which is a kernel object. This is the driver that will be inserted into kernel space. To insert this module we use a utility called `insmod`. It places the module from storage into the kernel space.

```bash
sudo insmod ./example.ko
```

sudo is used as we are changing something in the kernel space. The command will simply return. To check if the module is loaded, type command -

```bash
lsmod | grep example
```

This returns something like -

```
example             16384   0
```

To remove the module, we use `rmmod` as -

```
sudo rmmod example
```

The final prints will appear on log files.

```
sudo cat /var/log/syslog | grep inside
```

This gives result `Jan 24 19:59:57 asus kernel: [ 7803.715538] inside simple_module_init function Jan 24 20:00:13 asus kernel: [ 7819.304305] inside simple_module_exit function`

To see the effects of init and exit functions, create two files `exmod_init.c` and `exmod_exit.c`, each having only the init and the exit functions respectively.

```cpp
#include<linux/init.h>
#include<linux/module.h>

int simple_module_init(void)
{
        printk(KERN_ALERT "Inside %s function\\n",__FUNCTION__);
        return 0;
}

module_init(simple_module_init);
#include<linux/init.h>
#include<linux/module.h>

void simple_module_exit(void)
{
        printk(KERN_ALERT "Inside %s function\\n",__FUNCTION__);
}

module_exit(simple_module_exit);
```

Edit the Makefile to -

```
obj-m:=example.o
obj-m += exmod_init.o
obj-m += exmod_exit.o
```

Now after running the same make command, the result is -

```
make -C /lib/modules/$(uname -r)/build M=$PWD modules

make: Entering directory '/usr/src/linux-headers-4.13.0-26-generic'
  CC [M]  /home/asus/Tanishq/devdr2/example.o
  CC [M]  /home/asus/Tanishq/devdr2/exmod_init.o
  CC [M]  /home/asus/Tanishq/devdr2/exmod_exit.o
  Building modules, stage 2.
  MODPOST 3 modules
  CC      /home/asus/Tanishq/devdr2/example.mod.o
  LD [M]  /home/asus/Tanishq/devdr2/example.ko
  CC      /home/asus/Tanishq/devdr2/exmod_exit.mod.o
  LD [M]  /home/asus/Tanishq/devdr2/exmod_exit.ko
  CC      /home/asus/Tanishq/devdr2/exmod_init.mod.o
  LD [M]  /home/asus/Tanishq/devdr2/exmod_init.ko
make: Leaving directory '/usr/src/linux-headers-4.13.0-26-generic'
```

Now there are kernel modules for the two new files as well. On loading the `exmod_exit.ko` module, we do not get an alert after loading as there is no init function. On removing it, we get the required alert as there is an exit function.

```
Jan 25 01:24:16 asus kernel: [ 4316.859080] Inside simple_module_exit function
```

On loading the `exmod_init.ko` module, we get the desired alert as there is an init function.

```
Jan 25 01:25:21 asus kernel: [ 4381.815867] Inside simple_module_init function
```

But on using rmmod on the module we get an error as there is no exit function.

```
rmmod: ERROR: ../libkmod/libkmod-module.c:793 kmod_module_remove_module() could not remove 'exmod_init': Device or resource busy
rmmod: ERROR: could not remove module exmod_init: Device or resource busy
```

This module then continues to exist in the running modules list which can be checked using the `lsmod` command.

(This module is automatically unloaded after a reboot.)

---

## Building module using multiple files and module license and __init

For using multiple c files, edit the makefile to add a rule

```
obj-m := multimodule.o
multimodule.obj$ := exmod_init.o exmod_exit.o
```

The last line tells what files to use to build the `multifile` module. Also using `:=` again has reset the name from example.o to `multimodule.o`. Now to make the module, run -

```
!make
```

to run the previous run command for make. Now using `insmod` and `rmmod` gives the desired results.

Add a line to `example.c` -

```
MODULE_LICENSE("GPL");
```

after the header file inclusion lines. Without this line, the kernel assumes that we have a proprietary or private license for which we are not willing to share the source code. On running without this line the log consists of a line

```
Jan 29 01:48:05 asus-p8z77v kernel: [   95.022260] example: loading out-of-tree module taints kernel.
Jan 29 01:48:05 asus-p8z77v kernel: [   95.022263] example: module license 'unspecified' taints kernel.
Jan 29 01:48:05 asus-p8z77v kernel: [   95.022264] Disabling lock debugging due to kernel taint
Jan 29 01:48:05 asus-p8z77v kernel: [   95.022312] example: module verification failed: signature and/or required key missing - tainting kernel
```

The kernel thinks that the code is something that can potentially damage the kernel. Also some functionalities of the kernel will not be available without specifying the license (like lock debugging see above and USB support, etc.). Add `__init` before declaring the init function -

```
__init int simplemod_init(void)
```

The init function is only required to be called when the `insmod` function is called. After the `insmod` returns the init function is never going to be called again in the duration of the module. By specifying `__init`, we indicate that the function is only required when initialization is done. After that the module is not required to be present in the kernel space to save more space in RAM. So this releases some memory of the kernel RAM which can be used for some other purpose. So copy this to another file called `example2.c`. Add a loop to the `example.c` file -

```cpp
#include<linux/init.h>
#include<linux/module.h>

__initdata int count = 5;

__init int simplemod_init(void)
{
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
    int index;
    for(index = 0; index < count; index++)
    {
        printk(KERN_ALERT "Index = %d\\n", index);
    }
    return 0;
}
void simplemod_exit(void)
{
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
}
module_init(simplemod_init);
module_exit(simplemod_exit);
```

In the above code snippet, the value of the global variable count is used only by the init function, so to free up that space used by it after the execution of the `insmod` command, we use `__initdata` prefix.

### kernel difference (min size etc.)

The `__init` can be prefixed on some user defined functions as well but we need to make sure that the function is only invoked by the init function. If a function is declared `__init` and is also being called by the exit function, then on exiting when the call is made to the function which is not actually present in the RAM, the kernel experiences a page fault and therefore crashes. The output regarding size difference between modules written by same code except for the `__init` declared function.

```
example2                        12390   0
example                         12442   0
```

---

## Exporting symbols

Add a function called `mod_symbolex` to the example code.

```cpp
#include<linux/init.h>
#include<linux/module.h>

int mod_symbolex(void)
{
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
    return 0;
}
EXPORT_SYMBOL(mod_symbolex);
int simplemod_init(void)
{
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
    return 0;
}
void simplemod_exit(void)
{
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
}
module_init(simplemod_init);
module_exit(simplemod_exit);
```

The `EXPORT_SYMBOL` macro takes a name which can be a variable or a function which can be exported from this module to any other module in the kernel space. On a new `module.c` file, call the function and also declare a prototype.

```cpp
#include<linux/init.h>
#include<linux/module.h>

int mod_symbolex(void);
int simplemod_init(void)
{
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
    mod_symbolex();
    return 0;
}
void simplemod_exit(void)
{
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
}
module_init(simplemod_init);
module_exit(simplemod_exit);
```

On loading the module `use.ko` then `export.ko`, we get the error

```
insmod: ERROR: could not insert module ./use.ko: Unknown symbol in module
```

because the function used in that module isn't present in the kernel space. Therefore, we need to load the module `export.ko` first.

```
use                    16384  0
export                 16384  1 use
```

The log file also contains the respective function names.

---

## Module parameters

Module parameters are the arguments given to a module. To use this functionality we need to add a header file `linux/moduleparam.h`. Make a C source file -

```cpp
#include<linux/init.h>
#include<linux/module.h>
#include<linux/moduleparam.h>

int count=1;
module_param(count, int, 0);

int simplemod_init(void)
{
    int i=0;
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
    for(i = 0;i<count;i++)
        printk(KERN_ALERT "hi %d\\n", i);
    return 0;
}
void simplemod_exit(void)
{
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
}
module_init(simplemod_init);
module_exit(simplemod_exit);
```

`module_param` is a macro which takes three arguments - variable name, size and permission. If we load the module now, hi will be printed once. We can give the variable count as an argument in the following manner -

```
sudo insmod module.ko count=5
```

The name of the variable given as a argument should match the name specified in the module_param function. It is optional to pass a module parameter and so should have a default value. The permission's use can be demonstrated using code -

```cpp
#include<linux/init.h>
#include<linux/module.h>
#include<linux/moduleparam.h>

int count=1;
module_param(count, int, 0644);

int simplemod_init(void)
{
    int i=0;
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
    for(i = 0;i<count;i++)
        printk(KERN_ALERT "hi %d\\n", i);
    return 0;
}
void simplemod_exit(void)
{
    int i=0;
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
    for(i = 0;i<count;i++)
        printk(KERN_ALERT "bye %d\\n", i);
}
module_init(simplemod_init);
module_exit(simplemod_exit);
```

The permission format is the same as the unix permission format. The advantage is that now a variable store file is created in the location `/sys/module/<modulename>/parameters/count`. Since it has the permissions 644, we can edit it while the module is loaded. So if we edit this to 5, and then remove the module, then the message bye will be printed 5 times.

---

## Simple character driver module

`/dev` is an in-RAM file system that gets destroyed on reboot. The symbolic links to all devices on the system reside here. On listing the directory we get first character as c for character devices. The two numbers separated by a `,` are the major number and minor number. Major number is a way of identifying a driver which can be associated with a device. The minor number corresponds to instances of that device. Make a file `example.c` -

```cpp
#include<linux/init.h>
#include<linux/module.h>
#include<linux/fs.h>

ssize_t example_read(struct file *pfile, char __user *buffer, size_t length, loff_t *offset)
{
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
    return 0;
}
ssize_t example_write(struct file *pfile, const char __user *buffer, size_t length, loff_t *offset)
{
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
    return length;
}
int example_open(struct inode *pinode, struct file *pfile)
{
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
    return 0;
}
int example_close(struct inode *pinode, struct file *pfile)
{
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
    return 0;
}

struct file_operations example_file_operations={
    .owner=THIS_MODULE,
    .open=example_open,
    .read=example_read,
    .write=example_write,
    .release=example_close
};

int simplemod_init(void)
{
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
    register_chrdev(240, "simplechrdev", &example_file_operations);
    return 0;
}
void simplemod_exit(void)
{
    printk(KERN_ALERT "inside %s function\\n", __FUNCTION__);
    unregister_chrdev(240, "simplechrdev");
}
module_init(simplemod_init);
module_exit(simplemod_exit);
```

`fs.h` header file is used for support of character driver support. `register_chrdev()` is a function used to indicate to the kernel that this is a character device driver. It takes in arguments major number, name of the driver and the file operations. In the exit function we also need to deregister the driver. The struct is specified in the `fs.h` library. `example_operation` is the name given to the operation in that structure. The operations function prototypes are taken from the `/lib/modules/$(uname -r)/build/include/linux/fs.h` file. On the newer kernel the same file can be found in `/usr/src/linux-headers-4.13.0-32/include/linux/fs.h`. From these prototypes name the functions accordingly and specify names for the parameters. The open and close functions return 0 to indicate successful opening and closing of the files. The read returns 0 to indicate there is no data to return and read was successful. The write returns length to specify the length of data that has been written. There is also a command - `make -C /lib/modules/$(uname -r)/build M=$PWD clean` to remove all the clutter generated while building the module. Build the character driver module and insert. The device drivers loaded on a system are given in `/proc/devices`.

```
cat /proc/devices
```

This shows character device drivers first followed by the block device drivers. The first column is the major number and the second is the name for the driver. Output -

```
116 alsa
128 ptm
136 pts
180 usb
189 usb_device
204 ttyMAX
216 rfcomm
226 drm
240 simplechrdev
243 media
244 mei
245 kfd
```

Now we need to define a new device in `/dev`.

```bash
sudo mknod -m 666 /dev/mychar_device c 240 0
```

The last `0` is the minor number. The command `cat` basically opens a file and then reads the contents to display on STDOUT and then closes it. Since, linux does not differentiate between files and character devices, we can run cat command on the device and expect it to open the device, read it then close it. The output in `/var/log/syslog` shows this behavior. All the expected functions are called. The same can be done using echo command to write to the device (file, as linux treats it).

```bash
echo hello > /dev/mychar_device
```

Output -

```
Feb  2 22:08:43 Kubuntu-TR kernel: [42655.790679] inside simplemod_init function
Feb  2 22:10:39 Kubuntu-TR kernel: [42771.911791] inside example_open function
Feb  2 22:10:39 Kubuntu-TR kernel: [42771.911821] inside example_read function
Feb  2 22:10:39 Kubuntu-TR kernel: [42771.911842] inside example_close function
Feb  2 22:10:53 Kubuntu-TR kernel: [42785.883620] inside example_open function
Feb  2 22:10:53 Kubuntu-TR kernel: [42785.883651] inside example_write function
Feb  2 22:10:53 Kubuntu-TR kernel: [42785.883660] inside example_close function
Feb  2 22:11:10 Kubuntu-TR kernel: [42802.109737] inside simplemod_exit function
```

---
