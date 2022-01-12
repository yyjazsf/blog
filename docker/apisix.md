# APISIX

[官网](https://apisix.apache.org/zh/docs/helm-chart/apisix)

```bash
helm install apisix apisix/apisix  --set gateway.type=NodePort --set admin.allow.ipList="{0.0.0.0/0}"  -n apisix --create-namespace


# 
helm install apisix-ingress-controller apisix/apisix-ingress-controller   --set config.apisix.baseURL=http://apisix-admin:9180/apisix/admin  --set config.apisix.adminKey=edd1c9f034335f136f87ad84b625c8f1  -n apisix

```bash
