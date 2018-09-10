The first step in the systemtap journey is to **INSTALL** it, otherwise how to play with it?

Generally, it is easy, while I faced some trouble at the first time, because there is some problems for systemtap on Ubuntu 16.04. So I met a lot of frustrated error messages at the very beginning. Finally, I managed to install and run it on Ubuntu 18.04. Hope this information would help you a little.

# What we need

Basically, to use systemtap we need to install not only the systemtap itself but also

* linux-headers and
* linux-image-debug

By knowing this, it would be more clear about what to do.

# Install systemtap

That's easy and obvious.

```
sudo apt-get install systemtap
sudo apt-get install gcc
```

Well, gcc is necessary to build systemtap script.

# Install linux-headers and linux-image-debug

Usually, the installation could be accomplished by apt.

Use "uname -a" and "aptitude search" to get the proper package name.

```
$aptitude search linux-image | grep dbg
i  linux-image-unsigned-4.15.0-33-generic-dbgsym - Linux kernel debug image for version 4.15.0 on 64 bit x86 SMP

$aptitude search linux-headers-4.15.0-33-generic
i A linux-headers-4.15.0-33-generic                                                                 - Linux kernel headers for version 4.15.0 on 64 bit x86 SMP                                                
p   linux-headers-4.15.0-33-generic:i386                                                            - Linux kernel headers for version 4.15.0 on 32 bit x86 SMP
```

Then install them.

In case you can't find the linux-image-debug package, refer to **Install debug file** part in [this post][1].

# Verify the installation

```
sudo stap -v -e 'probe begin { printf("Hello, World!\n"); exit() }'
```

```
sudo stap -v -e 'probe vfs.read {printf("read performed\n"); exit()}'
```

If both of them works fine, congratulations.

# HelloWorld

HelloWorld is the classical example in programming, so let's start with it.

```
$cat helloworld.stp
probe begin
{ 
	printf("HelloWorld\n");
	exit(); 
}
```

Then run this script

```
sudo stap -v helloworld.stp
```

If you see the "HelloWorld" is printed, you can start your journey now~


[Install SystemTap in Ubuntu 14.04][1]
[Ubuntu wiki -- Systemtap][2]

[1]: https://blog.jeffli.me/blog/2014/10/10/install-systemtap-in-ubuntu-14-dot-04/
[2]: https://wiki.ubuntu.com/Kernel/Systemtap#Where_to_get_debug_symbols_for_kernel_X.3F
