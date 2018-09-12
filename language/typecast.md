A context variable may lost the type information since it is saved into a
integer variable. Then we need to use @cast to cast from an integer to a type.

See the [Manual Page][1] for more information.

**Tip:** systemtap use -> to dereference both pointer and struct.

```
@define file_struct(ptr) %(
    @cast(@ptr, "file", "kernel<linux/fs.h>")
%)

probe vfs.read
{
	printf("pos as param  : %lu\n", $file->f_pos);
	printf("struct pointer: %p\n", file);
	printf("f_version     : %lu\n", @file_struct(file)->f_version);
	printf("f_pos         : %lu\n", @file_struct(file)->f_pos);
	printf("raw deref     :\n%s\n", @file_struct(file)$);
	exit();
}
```

[1]: https://sourceware.org/systemtap/man/stap.1.html
