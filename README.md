There is a kernel-headers Raspbian package now:
https://www.raspberrypi.org/documentation/linux/kernel/headers.md

rpi-source is helpful if you use rpi-update kernels or want to build an in-kernel module.

------

rpi-source installs the kernel source used to build rpi-update kernels and the kernel on the Raspian image.  
This makes it possible to build loadable kernel modules.  
It is not possible to build modules that depend on missing parts that need to be built into the kernel proper (bool in Kconfig).

The script uses sudo internally when self-updating and when making the links */lib/modules/$(uname -r)/{build,source}*

[Examples on how to build various modules](Examples-on-how-to-build-various-modules)

Dependencies
```text
sudo apt install git bc bison flex libssl-dev
```

Install
```text
sudo wget https://raw.githubusercontent.com/RPi-Distro/rpi-source/master/rpi-source -O /usr/local/bin/rpi-source && sudo chmod +x /usr/local/bin/rpi-source && /usr/local/bin/rpi-source -q --tag-update

```
Run
```text
rpi-source
```

### ncurses-devel

This library is needed when running 'make menuconfig'

```text
 *** ncurses-devel is NOT installed. Needed by 'make menuconfig'. On Raspberry Pi OS: sudo apt install ncurses-dev
```

Install prerequisites
```text
$ sudo apt install libncurses5-dev
```

## Misc

### make prepare
Always run 'make prepare' after changing the kernel config

Show kernel config diff between current and previous config
```text
$ scripts/diffconfig
 ENC28J60 n -> m
+ENC28J60_WRITEVERIFY n
```
Show config diff against the running kernel
```text
$ cd linux
$ zcat /proc/config.gz > .config.running
$ scripts/diffconfig .config.running .config
```

### ~/linux/Module.symvers

Module.symvers contains all the symbols the kernel and modules has exported, and was made during the kernel build process.
This file is used later to find needed symbols when building modules. If it was missing (or empty) during building of a module, you might get a not very helpful error message when loading the module. But dmesg will help:
```text
$ sudo modprobe elo
ERROR: could not insert 'elo': Exec format error
$ dmesg |tail -n 1
[19988.002342] elo: no symbol version for module_layout
```

But if we build a module that exports a symbol, and another module we build needs that symbol, we get warnings.  
The build system can't find that symbol in ~/linux/Module.symvers.  
We need to tell the build system where this symbol is. This can be done with KBUILD_EXTRA_SYMBOLS pointing to the needed Module.symvers in the build directory of the exporting module.  
See [mcp2515a](Examples-on-how-to-build-various-modules#mcp2515a) for an example.

* [6.3 Symbols From Another External Module](https://www.kernel.org/doc/Documentation/kbuild/modules.txt)

I had a situation after building several modules that loading the module failed with some format error. It turned out that ~/linux/Module.symvers had changed somehow.  
If that happens, there is a backup file Module.symvers.backup. The timestamps will differ if it has changed.


### Kernel taint
The kernel can be tainted in various ways. One of them is loading an out-of-tree module.
```text
$ cat /proc/sys/kernel/tainted
4096
```
From https://www.kernel.org/doc/Documentation/sysctl/kernel.txt
```text
4096 - An out-of-tree module has been loaded.
```
