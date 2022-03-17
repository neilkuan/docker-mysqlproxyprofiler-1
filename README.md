# Docker image with mysql-proxy setup to profile your queries

## Motivation

Even if we have a lot of tools to pay attention at what we (or our teams) are doing in term of performances. It appears 
that sometimes people don't pay attention and specifically do not use indexes or simply don't think about
optimizing/checking their queries to databases. Most of the time because it works on development environments and 
performances are not really a concern at that time.

That sucks, but that happens, and with a database full of data it could literally kill your website on production.

With [eZ Platform](https://ezplatform.com), [Symfony](http://symfony.com) and recent PHP Framework you usually have a
"profiler" that allows you to see how many database queries the page is doing, how long they take, and if they are using
indexes, but most of the time you need to be in "development" mode to see that.

Agnostic to any framework or technology, this image provides a way to profile your queries during development but on 
production as well using MySQL Proxy and a LUA script that:

- logs queries
- indicates their execution time
- let you know if you are using indexes or not
- add SQL_NO_CACHE (in option) to be sure your are not using the cache when profiling

## Usage

You can use the image that way:

```bash
docker run -it --rm --name myproxy -e BACKEND=host:port plopix/docker-mysqlproxyprofiler
```

> by default the profiling adds `SQL_NO_CACHE` to all `SELECT` queries, you can turn it off adding `-e USE_NO_CACHE=0`

## Examples 

Let's say you have a container named `myprojectdatabase` that is your database.

```bash
docker run -it --rm --name myprojectdbproxy --link myprojectdatabase -e BACKEND=myprojectdatabase:3306 plopix/docker-mysqlproxyprofiler
```

You can now reconfigure your application configuration container to reach the proxy.

Using Symfony

```diff
parameters:
-    env(DATABASE_HOST): myprojectdatabase
+    env(DATABASE_HOST): myprojectdbproxy
```

> Assuming `myprojectdatabase` container internally listens on 3306.

If your are using docker only for your database and PHP on your host
```bash
docker run -it -p 3303:3306 --rm --link myprojectdatabase -e BACKEND=myprojectdatabase:3306 plopix/docker-mysqlproxyprofiler
```

```diff
parameters:
-    env(DATABASE_PORT): 3307
+    env(DATABASE_PORT): 3303
```

> Assuming `myprojectdatabase` internally container listens on 3306 AND that 3307 was the port mapped to container
`myprojectdatabase` (ran with -p '3307:3306')

**That is it! No more reason to forget indexes!**

Happy profiling!

## Go further 

In term of profiling and optimizations: have a look at [Blackfire.io](https://blackfire.io). That's awesome!


## Special thanks

- Patrick Allaert, for this `debug.lua` script: https://github.com/patrickallaert/MySQL-Proxy-scripts-for-devs



## Deploy to k8s cluster
see: [k8s-deployments.yaml](./k8s-deployments.yaml)
```diff
# edit line 31 to your db host and port
- value: "host:port"
+ value: "mysql.example.com:3306"
```

### Deploy to k8s
```bash
kubectl apply -f k8s-deployments.yaml
```

### Start the db proxy
```bash
# example
kubectl get pod -l 'app=proxysql'
NAME                        READY   STATUS    RESTARTS   AGE
proxysql-5d6d74cc44-hpjzm   1/1     Running   0          9m44s

kubectl port-forward pod/proxysql-5d6d74cc44-hpjzm 3306:3306

or

kubectl port-forward pod/$(kubectl get pod -l 'app=proxysql' -o=jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}') 3306:3306
```
