# k8s-redis
利用k8s部署redis服务

## 快速部署redis

```bash
# kubectl apply -f redis-ns.yaml 
namespace/ns-redis created

# kubectl create configmap example-redis-config --namespace=ns-redis --from-file=redis-config 
configmap/example-redis-config created

# kubectl apply -f redis-pod.yaml 
pod/redis created

# kubectl apply -f redis-svc.yaml

# kubectl -n ns-redis get configmap example-redis-config -oyaml
apiVersion: v1
data:
  redis-config: |
    maxmemory 2mb
    maxmemory-policy allkeys-lru
kind: ConfigMap
metadata:
  creationTimestamp: "2019-05-16T10:20:44Z"
  name: example-redis-config
  namespace: ns-redis
  resourceVersion: "4238219"
  selfLink: /api/v1/namespaces/ns-redis/configmaps/example-redis-config
  uid: 4397b749-77c4-11e9-989c-5255000115c0

# kubectl -n ns-redis get all -owide
NAME        READY   STATUS    RESTARTS   AGE   IP                NODE             NOMINATED NODE   READINESS GATES
pod/redis   1/1     Running   0          40s   192.152.203.142   192.168.35.151   <none>           <none>

NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE   SELECTOR
service/redis   NodePort   192.153.39.61   <none>        6379:26379/TCP   9s    app=redis
```

## 快速验证redis服务

1. 确认ConfigMap的配置已生效：

```bash
# kubectl -n ns-redis exec -it redis redis-cli
127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "2097152"
127.0.0.1:6379> CONFIG GET maxmemory-policy
1) "maxmemory-policy"
2) "allkeys-lru"
```

2. 确认redis服务可正常调用：

```bash
$ pip3 install redis
$ python
Python 3.6.3 (default, Oct  4 2017, 06:09:38) 
[GCC 4.2.1 Compatible Apple LLVM 9.0.0 (clang-900.0.37)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import redis
>>> r = redis.Redis(host='192.168.36.145', port=26379, db=0)
>>> r.set('foo', 'bar')
True
>>> r.get('foo')
b'bar'
```

3. 再次确认redis服务：

```bash
# kubectl -n ns-redis exec -it redis redis-cli
127.0.0.1:6379> get foo
"bar"
```

## 参考文档

- [Configuring Redis using a ConfigMap](https://kubernetes.io/docs/tutorials/configuration/configure-redis-using-configmap/)
- [DockerHub - Redis](https://hub.docker.com/_/redis?tab=description)
- [REDIS WITH PYTHON - 2018](https://www.bogotobogo.com/python/python_redis_with_python.php)
