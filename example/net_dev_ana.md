Here is an example to evaluate the time elapsed on a net_dev handler and on
which cpu it runs.

```
global value
global netpstats
global cpustats
global init_time
probe begin
{
	init_time = gettimeofday_ms()
}
probe module("e1000").function("e1000_xmit_frame")
{
	value = gettimeofday_ns()
	cpustats <<< cpu()
}
probe module("e1000").function("e1000_xmit_frame").return
{
	diff = gettimeofday_ns() - value
	netpstats <<< diff
}
probe timer.s(5)
{
	exit();
}
probe end
{
	print(@hist_linear(netpstats,0, 15000, 500))
	print(@hist_log(cpustats))
	printf("Total time: %d miliseconds\n", gettimeofday_ms() - init_time)
}
```
