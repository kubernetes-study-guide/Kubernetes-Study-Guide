need to tag droplets for this to work. 


test your lb is up:


also setup port forwarding to 6443. 

```
$ telnet 188.166.136.150 6443
Trying 188.166.136.150...
Connected to 188.166.136.150.
Escape character is '^]'.
^CConnection closed by foreign host.
```

or do:

```
$ nc -v 188.166.136.150 6443
```


dns addresses can take a a few days to propragate through so here's a lb i created earlier. 


