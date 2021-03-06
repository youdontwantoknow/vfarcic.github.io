## Hands-On Time

---

# Monitoring


## Cluster Setup
## (if not already running)

---

```bash
source cluster/kops

chmod +x kops/cluster-setup.sh

NODE_COUNT=3 NODE_SIZE=t2.medium \
    ./kops/cluster-setup.sh
```


## Prometheus Chart

---

```bash
export LB_ADDR=$(kubectl -n kube-ingress get svc ingress-nginx \
    -o jsonpath="{.status.loadBalancer.ingress[0].hostname}")

dig +short $LB_ADDR # Repeat until not empty.

LB_IP=$(dig +short $LB_ADDR | tail -n 1)

MON_ADDR="mon.$LB_IP.xip.io"

kubectl create ns mon

helm install stable/prometheus --name mon --namespace mon \
    --set server.ingress.enabled=true \
    --set server.ingress.hosts={"$MON_ADDR"}

kubectl -n mon rollout status deploy mon-prometheus-server
```


## Prometheus Chart

---

```bash
open "http://$MON_ADDR"

helm upgrade -i go-demo-3 ../go-demo-3/helm/go-demo-3 \
    --namespace go-demo-3 --set image.tag=1.0

kubectl -n go-demo-3 rollout status deployment go-demo-3

export LB_DNS=$(kubectl -n go-demo-3 get ing go-demo-3 \
    -o jsonpath="{.status.loadBalancer.ingress[0].hostname}")

curl http://$LB_DNS/demo/hello

open "http://$MON_ADDR"
```


## Prometheus Chart

---

* `container_memory_usage_bytes{namespace="go-demo-3"}`
* `sum(container_memory_usage_bytes{namespace="go-demo-3"})`
* `container_memory_usage_bytes{pod_name=~"go-demo-3-db.+"}`
* `container_memory_usage_bytes{pod_name=~"go-demo-3-db.+", container_name="db"}`
* `node_memory_MemTotal`
* `node_memory_MemFree`
* `node_memory_MemAvailable`


## Grafana Chart

---

```bash
GRAFANA_ADDR="grafana.$LB_IP.xip.io"

helm install stable/grafana --name grafana --namespace mon \
    --set ingress.enabled=true \
    --set ingress.hosts="{$GRAFANA_ADDR}" \
    --set persistence.enabled=true \
    --set persistence.accessModes="{ReadWriteOnce}" \
    --set persistence.size=1Gi

kubectl -n mon rollout status deploy grafana

open "http://$GRAFANA_ADDR"

kubectl get secret -n mon grafana \
    -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```


## Grafana Chart

---

```bash
echo "http://$MON_ADDR"
```

* Import dashboards *3131*, *1621*


## What Now?

---

```bash
kubectl delete ns mon
```
