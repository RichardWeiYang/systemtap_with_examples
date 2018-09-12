One example on printing stack trace:

```
probe kernel.function("path_get").return 
{
	printf("in process [%s]\n", execname());
	print_regs();
	print_backtrace();
	print("-----------------------------------------\n");
	exit();
}
```

The output would looks like this:

```
In process [gnome-shell]
RIP: ffffffffba6724f5
RSP: ffffb6494439bc40  EFLAGS: 00000246
RAX: 0000000100000000 RBX: ffff929fbd0ce820 RCX: 0000000200000000
RDX: 0000000100000000 RSI: 0000000200000000 RDI: ffff929f5fa8c4d8
RBP: ffffb6494439bc78 R08: 0000000000027060 R09: ffff929f5fa8c480
R10: 2d09220074617473 R11: 0000000000000004 R12: ffffb6494439bd90
R13: ffff929f028b7810 R14: ffff929f31906240 R15: ffff929f028b7800
FS:  00007f65c89f8ac0(0000) GS:ffff92a01fd80000(0000) knlGS:0000000000000000
CS:  0010 DS: 0000 ES: 0000 CR0: 0000000080050033
CR2: 000055a8d8369d80 CR3: 00000000315a4003 CR4: 00000000000606e0
Returning from:  0xffffffffba6810c0 : path_get+0x0/0x30 [kernel]
Returning to  :  0xffffffffba6724f5 : do_dentry_open+0x45/0x310 [kernel]
 0xffffffffba673b4f : vfs_open+0x4f/0x80 [kernel]
 0xffffffffba68679e : path_openat+0x66e/0x1770 [kernel]
 0xffffffffba6888ab : do_filp_open+0x9b/0x110 [kernel]
 0xffffffffba67403b : do_sys_open+0x1bb/0x2c0 [kernel]
 0xffffffffba674174 : SyS_openat+0x14/0x20 [kernel]
 0xffffffffba403ae3 : do_syscall_64+0x73/0x130 [kernel]
 0xffffffffbae00081 : entry_SYSCALL_64_after_hwframe+0x3d/0xa2 [kernel]
 0x0
-----------------------------------------
```
