systemtap support user write script in raw c language, which is called guru
mode.

When invoke a script in guru mode, **-g** option should be passed.

For more information, please refer to [RHEL Embedded C][1] and [Embedded C
exapmle][2]


# Task Info

```

%{
#include <linux/sched.h>
%}

function task_info:long (task:long) %{ /* pure */  
        struct task_struct *p = (struct task_struct *)((long)STAP_ARG_task);
	STAP_PRINTF("task pointer   : %p\n", STAP_ARG_task);
	STAP_RETVALUE = p->cpu;
%}

probe vfs.read
{
	if (execname() == "stapio")
		next;
	task = pid2task(pid());
	printf("task name      : %s\n", execname());
	printf("task running on: %d cpu\n", task_info(task));
	exit();
}
```

# Hijack kernel

Guru mode is so powerful that even you could modify the behavior of kernel on
the fly.

Here is an example to change 'm' to 'b' when you type on keyboard. Copied from
[How to Monkey-Patch the Linux Kernel][3] with little modification.


```
probe kernel.function("evdev_events") 
{
	for (i = 0; i < $count; i++) {
		# Changes 'm' to 'b'.
		if ($vals[i]->code == 50)
			$vals[i]->code = 48
	}
}
```

After doing so, you could never type 'm'.

[1]: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/systemtap_language_reference/ch03s06
[2]: https://zhengheng.me/2015/02/11/system-embedded-c/
[3]: https://blog.cloudflare.com/how-to-monkey-patch-the-linux-kernel/
