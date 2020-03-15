See what happens if you omit the bash command for centos image,

```
$ docker run -it centos     
```

Thats because the centos image is configured to run bash as the default command anyway. You can doublechck that in the dockerfile using docker inspect:

```
docker inspect centos
```

Some images doesn't come with bash. For example you have to use sh for the busybox  image e.g.:

```
docker run -it busybox sh
```

