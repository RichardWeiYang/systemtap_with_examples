Here is an example to measure duration of a function call.


```
global time
probe vfs.read
{
	time[tid()] = gettimeofday_ns();
}
probe vfs.read.return
{
	if(time[tid()]) {
		printf("vfs.read by %s took %d ns to execute\n",
			execname(), gettimeofday_ns() - time[tid()]);
		delete time[tid()];
		exit();
	}
}
```
