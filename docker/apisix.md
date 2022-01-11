# APISIX

[官网](https://apisix.apache.org/zh/docs/helm-chart/apisix)

```bash
helm install apisix apisix/apisix  --set gateway.type=NodePort --set admin.allow.ipList="{0.0.0.0/0}"  -n apisix --create-namespace

```bash
