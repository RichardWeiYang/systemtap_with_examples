Below is the syntax of systemtap language.

```
probe PROBEPOINT [, PROBEPOINT] { [STMT ...] }
```

This is something like awk syntax, specify the PROBEPOINT to trigger the
action and specify what to do with STMT.

This would be to boring to just look at the definition. Take the HelloWorld as
an example.

```
probe begin
{ 
    printf("HelloWorld\n");
    exit(); 
}
```

* begin is a special probe point means at the beginning of the probe process
* printf()/exit() is what should do on **begin** event

With this script and write into file helloworld.stp, we can run it with

```
sudo stap -v helloworld.stp
```

The syntax seems simple, right?

