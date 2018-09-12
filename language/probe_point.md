As we may see, the syntax of systemtap is 

```
probe PROBEPOINT [, PROBEPOINT] { [STMT ...] }
```

Probe point is the event to trigger the probe.

Here I will list some categories I have used. For more information, please
refer to [stapprobe][1] and [Built in probe point][2].

```
begin, end
timer.s(1)
vfs.read
kernel.function("alloc_pages*")
kernel.statement("*@mm/page_alloc.c:3663")
module("kvm").function("kvm_init")
process("/usr/bin/vi").begin
```

To get more information about a probe point, you could use -L option to query.

For example, you want to know all the probe point in vfs subsystem, you could
run

```
sudo stap -L 'vfs.*'
```

While I am wondering besides vfs, what are the other subsystems I could probe?

[1]: https://sourceware.org/systemtap/man/stapprobes.3stap.html
[2]: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/systemtap_language_reference/ch04s02
