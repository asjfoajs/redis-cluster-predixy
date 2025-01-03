# 解决了什么？
我们在部署redis-cluster时往往需要部署proxy组件，但是大部分redis chart并不包proxy组件，所以我将用[predixy](https://github.com/joyieldInc/predixy)作为proxy来编写一个chart。

---

# 做了什么？
编写一个predixy的chart和[redis 20.6.1 · bitnami/bitnami](https://artifacthub.io/packages/helm/bitnami/redis)组合成一个chart。然后可以通过`helm install`部署predixy和redis-cluster。

# 怎么用？
## 1.单独部署predixy
1. 创建一个value.yaml文件

设置一下stroageclass和password，注意需要分别设置redis-cluster的password和predixy的password

```yaml
global:
  redis:
    password: "" #这个是redis-cluster的密码
  predixy:
    clusterServerPool:
      password: "" #这个是predixy的密码
  storageClass: ""
```

2. 安装chart

`helm -n redis-cluster install  predixy predixy -f value.yaml`

## 2.一起部署
1.直接修改value.yaml文件

注意这里面多了fullnameOverride的内容，因为两个子chart的模板都会用release-name去生成container-name等值，如果不覆盖,都会用启动时的ReleaseName导致命名混乱。

```yaml
global:
  redis:
    password: "" #这个是redis-cluster的密码
  predixy:
    clusterServerPool:
      password: "" #这个是predixy的密码
  storageClass: ""
#一起部署需要覆盖一下fullnameOverride，否则两个子chart的模板都会用release-name去生成container-name等
predixy:
  fullnameOverride: ""
redis-cluster:
  fullnameOverride: ""
```

2.安装chart

`helm -n redis-cluster install redis-cluster-predixy ./redis-cluster-predixy`

---

# 不足？
1. predixy和redis-cluster的chart都会用common的共享chart，但是目前时每个都引用了一份，应该把他抽出去，只使用一份。
2. 一起部署时，应该等redis-cluster启动完成后在启动predixy。
3. 很多文件如NOTES.txt、ingress、

