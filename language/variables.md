To all kinds of programming languages, type of variables means the data
language could manipulate.

For example, with the pointer type in C, it could be used in low level
programs.

systemtap provides almost the same type of variables as in C. This is easy to
understand, since the script will be translated to C and then to be made a
kernel module.

# Common variables

To define and use a variable is easy. Different from C, variables in systemtap
is type-less.

```
global times = 0

probe begin
{ 
	world = "World";
	lucky_number = 666;

	printf("Hello%s %d\n", world, lucky_number);
	times++;
	exit(); 
}

probe end
{
	times++;
	printf("%d times of probe event\n", times);
}
```

The example above defines variables with type

* string
* integer

Also it shows how to define a global variable **times**.

# Associate array

This looks like map to me.

```
global reads

probe vfs.read
{
	reads[execname()] ++
}

probe timer.s(3)
{
	printf("raw statistics\n")
	foreach (process in reads)
		printf("%016s : %d \n", process, reads[process])
	printf("---\n");
	printf("raw statistics in alphabetical order\n")
	foreach (process+ in reads)
		printf("%016s : %d \n", process, reads[process])
	printf("---\n");
	printf("top 3 reads\n")
	foreach (process in reads- limit 3)
		printf("%016s : %d \n", process, reads[process])
	printf("---\n");
	printf("bottom 3 reads\n")
	foreach (process in reads+ limit 3)
		printf("%016s : %d \n", process, reads[process])
	if(["stapio"] in reads) {
		exit()
	}
}
```

This example shows how to get the statistics of vfs.read in 3 seconds by
assign the times into corresponding reads[] slot.

One more interesting thing is we can order it by putting +/- at the related
field in foreach() clause. And get limit number of result by the **limit**
keyword.

Try this example by yourself to feel the behavior.

You could get more information in [Ref 1][1] and [Ref 2][2].

# Aggregate statistics

This type is more suitable for statistic analysis.

You could get more information in [Ref 3][3].

```
global reads1, reads2
global agg_reads
global agg_hist_linear

probe vfs.read
{
	reads1[execname()] <<< 1
	reads2[execname()] ++
	agg_reads[execname(), pid()] <<< 1
	agg_hist_linear <<< bytes_to_read
}

probe timer.s(3)
{
	printf("raw statistics in aggregate statistic\n");
	//  the following two outputs are identical
	foreach (process+ in reads1 limit 5)
		printf("%016s: %d\n", process, @count(reads1[process]));
	printf("raw statistics in normal way\n")
	foreach (process+ in reads2 limit 5)
		printf("%016s: %d\n", process, reads2[process]);

	printf("aggregate usage\n");
	foreach([process+,var2] in agg_reads limit 5) {
		printf("%016s (%05d)\n", process, var2);
		printf("\tcount: %d \n", @count(agg_reads[process,var2]));
		printf("\tsum  : %d \n", @sum(agg_reads[process,var2]));
		printf("\tmin  : %d \n", @min(agg_reads[process,var2]));
		printf("\tmax  : %d \n", @max(agg_reads[process,var2]));
	}

	printf("Analysis of the vfs read length\n");
	// read length 0-512, step is 32
	printf("Minimum read length: %d\n", @min(agg_hist_linear));
	printf("Maximum read length: %d\n", @max(agg_hist_linear));
	printf("Average read length: %d\n", @avg(agg_hist_linear));
	// the following two could use one at once
	print(@hist_linear(agg_hist_linear, 0, 512, 32));
	//print(@hist_log(agg_hist_linear));

	if(["stapio"] in reads1) {
		exit()
	}
}
```

You would have some interesting finding with this script.

BTW, @hist_linear() and @hist_log() could not use simultaneously?


[1]: https://nanxiao.me/systemtap-note-12-associate-array-and-foreach/
[2]: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/systemtap_beginners_guide/arrayoperators
[3]: https://sourceware.org/systemtap/langref/Statistics_aggregates.html
