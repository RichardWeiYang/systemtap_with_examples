In case the kernel and systemtap doesn't match, you need to download the
source code and build from source.

# Download Source

To build systemtap, we need to download systemtap and elfutils.

```
git clone git://sourceware.org/git/elfutils.git

git clone git://sourceware.org/git/systemtap.git
```

# Configure

```
$cd elfutils
$autoreconf -i


$cd ../systemtap
$./configure --with-elfutils=../elfutils
```

# Build & Install

```
$make
$sudo make install
```

If succeed, you would see stap under systemtap directory.
