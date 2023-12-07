# Docker Commands List

## **Show Docker system information**
This command will look up how much space is used (-v is used for verbose)
You'll get information about:

* Images space usage,
* Containers space usage,
* Local Volumes space usage, and
* Build cache usage.

``` bash
docker system df -v
```

## **Clean up unused and dangling images**

``` bash
docker image prune
```

## **Clean up dangling images only**

``` bash
docker image prune -a
```

## **Clean up stopped containers**

``` bash
docker container prune
```

## **Clean up unused volumes**

``` bash
docker volume prune
```

## **Clean up used volumes**

``` bash
docker volume prune -f
```

