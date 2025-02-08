# What Problem Does This Solve?
When deploying a Redis cluster, we often need to include a proxy component. However, most Redis charts do not come with a proxy component. To address this, I created a Helm chart using [Predixy](https://github.com/joyieldInc/predixy) as the proxy.

---

# What Has Been Done?
1. Developed a **Predixy** Helm chart.
2. Combined the Predixy chart with the [redis 20.6.1 · bitnami/bitnami](https://artifacthub.io/packages/helm/bitnami/redis) into a unified chart.
3. This combination allows deploying both **Predixy** and **Redis Cluster** via a single `helm install` command. After deployment, the architecture looks as follows:

---

# How to Use It?
## **1. Deploy Predixy Separately**
1. **Create a **`**value.yaml**`** file**

Configure the `storageClass` and passwords for both the Redis cluster and Predixy. Ensure they are set independently:

```yaml
global:
  redis:
    password: "" 
  predixy:
    clusterServerPool:
      password: "" 
  storageClass: ""
```

2. **Install the Chart**

Use the following command to install Predixy:

`helm -n redis-cluster install  predixy predixy -f value.yaml`

---

## **2. Deploy Predixy and Redis Cluster Together**
1. **Modify the **`**value.yaml**`** file**

When deploying both components together, include the `fullnameOverride` field to prevent naming conflicts. Both subcharts use the release name to generate container names and other values. Without overriding this, deployment may lead to naming issues.

```yaml
global:
  redis:
    password: ""
  predixy:
    clusterServerPool:
      password: ""
  storageClass: ""

redis-cluster:
  fullnameOverride: ""
predixy:
  fullnameOverride: ""
  clusterFullNameOverride: ""#Consistent with redis-cluster.fullnameOverride

```

2. **Install the Chart**

Deploy both Predixy and Redis Cluster together using:

`helm -n redis-cluster install redis-cluster-predixy ./redis-cluster-predixy`

---

# Shortcomings
1. **Duplicate Dependencies**:  
Both the Predixy and Redis Cluster charts use a shared `common` chart. Currently, each includes its own copy. This should be refactored to share a single instance of the `common` chart.
2. **Startup Sequencing**:  
When deploying together, Predixy should wait for the Redis Cluster to be fully initialized before starting.
3. **Incomplete Files**:  
Some files, such as `NOTES.txt` and configurations for ingress, are missing and need to be completed.
4. **Prometheus Integration**:  
The Redis Cluster chart supports Prometheus integration, but this has not yet been added to Predixy.

Subsequent supplements