Here is some more interesting script.

# Process CPU distribution

This one shows how one process run on different cpus.

```
global one, two
global time
function div:string (a:long, b:long)
{
	total = a + b;
	mod = (100*a)%total;
	return sprintf("%d.%d%%", (100*a)/total, mod*100/total);
}
probe timer.profile
{
	if (cpu() == 0)
		one[execname(), tid()]++
	if (cpu() == 1)
		two[execname(), tid()]++
}
probe timer.s(5)
{
	exit();
}
probe begin
{
	time = gettimeofday_s()
}
probe end
{
	printf("%20s %5s %19s %19s\n", "process", "TID", "CPU0", "CPU1");
	foreach([p, t] in two){
		printf("%20s %5d %19s %19s\n", p, t,
			div(one[p, t], two[p, t]), div(two[p, t], one[p,t]));
		delete one[p,t];
	}
	foreach([p, t] in one){
		printf("%20s %5d %18d%% %18d%%\n", p, t, 100, 0);
	}
	time = gettimeofday_s() - time;
	printf("\nTotal............................: %5d secs.\n", time);
}
```


Next one would be more suitable for more cpus, while a little better for less
process.

```
#Argument on line
# ($1) is the number of cpus - 1 in the system
# ($2) is the process name regex
global sample
global time
probe timer.profile
{
	process = execname();
	if ($# == 2 && process =~ @2)
		sample[process, tid()] <<< cpu();
}
probe timer.s(5)
{
	exit();
}
probe begin
{
	time = gettimeofday_s()
}
probe end
{
	foreach([p, t] in sample){
		printf("process: %-20s TID: %5d\ncpu\tsamles\n", p, t)
		print (@hist_linear(sample[p,t], 0, $1, 1))
	}
	time = gettimeofday_s() - time
	printf("\nTotal............................: %5d secs.\n", time);
}
```
