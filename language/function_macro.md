The concept is simple, what we need to know is the format in systemtap.


# Function and Macro

This is a similar concept as in c with a little different in keywords. And a
function could be defined with type declaration or not.

For more information, you could read [Ref 1][1]

```
@define add(a,b) %( ((@a)+(@b)) %)

function series(a)
{
	sum = 0;
	idx = 0;
	while (idx < a)
		sum += idx++;
	return sum;
}

// function with type declaration
function sum:long (a:long, b)
{
	return a + b;
}

probe begin
{
	// invoke a function
	printf("%d\n", series(5));
	// invoke a function with type declaration
	printf("%d\n", sum(1, 2));
	// invoke a macro
	printf("%d\n", @add(1, 2));
	exit();
}
```

[1]: https://sourceware.org/systemtap/man/stap.1.html#lbAJ
