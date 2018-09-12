As a kernel dev, I would build and install kernel from source. After doing so,
we need to reinstall linux-header and linux-image for systemtap to work with
the correct kernel version.

This is a kind of struggling to me again.

The step I found is a little tricky. Maybe there is a better way, while I just
managed to do this in the following steps. If you have a better way, I'd
appreciate it a lot.

The general idea is get from kernel make target **deb-pkg** and the document
in [Ubuntu wiki -- Systemtap][1].

I am curious about why the **deb-pkg** target doesn't work as it should. And
the first steps in [Ubuntu wiki -- Systemtap][1] doesn't function on my
machine. Maybe I missed some critical steps in both parts.

The good news is I managed to combine the first part in **deb-pkg** and the
last part in [Ubuntu wiki -- Systemtap][1], and finally get the package I
need.

# create rules

The first step is to run **make deb-pkg** in kernel source directory. I named
this "create rules", since this creates config files in debian directory.

Before running **make deb-pkg**, we need to make some modification as below.

```
diff --git a/scripts/package/Makefile b/scripts/package/Makefile
index 73503ebce632..a1117548120e 100644
--- a/scripts/package/Makefile
+++ b/scripts/package/Makefile
@@ -68,12 +68,12 @@ binrpm-pkg: FORCE
 clean-files += $(objtree)/*.spec
 
 deb-pkg: FORCE
-       $(MAKE) clean
+       #$(MAKE) clean
        $(CONFIG_SHELL) $(srctree)/scripts/package/mkdebian
        $(call cmd,src_tar,$(KDEB_SOURCENAME))
        origversion=$$(dpkg-parsechangelog -SVersion |sed 's/-[^-]*$$//');\
                mv $(KDEB_SOURCENAME).tar.gz ../$(KDEB_SOURCENAME)_$${origversion}.orig.tar.gz
-       +dpkg-buildpackage -r$(KBUILD_PKG_ROOTCMD) -a$$(cat debian/arch) -i.git -us -uc
+       #+dpkg-buildpackage -r$(KBUILD_PKG_ROOTCMD) -a$$(cat debian/arch) -i.git -us -uc
 
 bindeb-pkg: FORCE
        $(CONFIG_SHELL) $(srctree)/scripts/package/mkdebian

diff --git a/scripts/package/mkdebian b/scripts/package/mkdebian
index 663a7f343b42..40ec6caabc2c 100755
--- a/scripts/package/mkdebian
+++ b/scripts/package/mkdebian
@@ -212,7 +212,6 @@ binary-arch:
 
 clean:
        rm -rf debian/*tmp debian/files
-       \$(MAKE) clean
 
 binary: binary-arch
 EOF
```

* First we comment out "make clean" to forbid rebuild the kernel again. As you
know, each time to rebuild the whole kernel will take a huge amount of time.

* Second we comment out the "dpkg-buildpackage" command. I didn't manage to run
this command.

* Third we remote the "make clean" in debian/rules. Since in next step, the
command will first call the clean rule and then rebuild. If you don't want to
waste too much time, remove it.

# create package

After the previous step, we may get debian directory in kernel source code and
a tar file of the source tree.

Now it is time to run the following command to get the package we want.


```
AUTOBUILD=1 fakeroot debian/rules binary-arch skipdbg=false
```

Those packages are in parent directory.

Enjoy and have fun!


[1]: https://wiki.ubuntu.com/Kernel/Systemtap#Where_to_get_debug_symbols_for_kernel_X.3F
