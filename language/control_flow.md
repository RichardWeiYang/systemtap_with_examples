This part is really simple, if you have some basic knowledge about any
programming language.

I would like to give two examples here, and for more information you could
refer to [Ref 1][1].

# if condition

```
probe begin
{ 
	if (pid())
		printf("%d:%s\n", pid(), execname());
	else
		printf("invalid pid\n");
	exit();
}
```

# for loop

```
probe begin
{ 
	index = 0;
	for (; index < 10; index++)
		arr[index] = index;
	exit();
}
```

[1]: https://sourceware.org/systemtap/man/stap.1.html
