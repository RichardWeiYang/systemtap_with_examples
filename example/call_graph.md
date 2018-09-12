systemtap provides some facilities to generate call graph,

* thread_indent(), pretty prefix
* probefunc(), function name being probed

Let's see an example on module e1000.

```
probe module("e1000").function("*").call
{
	printf("%s -> %s\n", thread_indent(1), probefunc());
}
probe module("e1000").function("*").return {
	printf("%s <- %s\n", thread_indent(-1), probefunc());
}

probe timer.s(3)
{
	exit();
}
```

The output may look like this:

```
     0 kworker/1:6(9096): -> e1000_watchdog
     7 kworker/1:6(9096):  -> e1000_has_link
    11 kworker/1:6(9096):  <- e1000_watchdog
    13 kworker/1:6(9096):  -> e1000_update_stats
   319 kworker/1:6(9096):   -> e1000_read_phy_reg
   386 kworker/1:6(9096):   <- e1000_update_stats
   388 kworker/1:6(9096):   -> e1000_read_phy_reg
   450 kworker/1:6(9096):   <- e1000_update_stats
   468 kworker/1:6(9096):  <- e1000_watchdog
   471 kworker/1:6(9096):  -> e1000_update_adaptive
   474 kworker/1:6(9096):  <- e1000_watchdog
   489 kworker/1:6(9096): <- 0xffffffffba4a770e
```

It contains

* number of microseconds since the last initial level of indentation
* process name and thread ID
* probe function name
