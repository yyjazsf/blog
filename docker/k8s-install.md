## k8s install

`createDate: 2022/01/12`

Optimize for China mainland users(默认使用国内镜像)

system: ubuntu 20

## 设置代理（建议）
```bash
apt install privoxy

nano ~/.zshrc
alias proxy="export HTTP_PROXY=http://127.0.0.1:8118;export HTTPS_PROXY=http://127.0.0.1:8118;export FTP_PROXY=http://127.0.0.1:8118;export NO_PROXY=localhost,127.0.0.0,127.0.1.1,127.0.1.1,10.0.0.0/8,192.168.0.0/16,mirrors.aliyun.com,registry.aliyuncs.com;"

```

`docker proxy（建议）`

- [docker pull proxy](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy)

## install docker

[Official Tutorials](https://docs.docker.com/engine/install/ubuntu/)

```bash
sudo usermod -aG docker $USER

sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "registry-mirrors": ["https://v24t206y.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## Disabling virtual memory 

```bash
# Temporarily Closed virtual memory 
sudo swapoff -a
nano /etc/fstab
# 注释 swap 那一行
```

## install k8s

```bash

# 添加并信任APT证书
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -

# 添加源地址
add-apt-repository "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main"

# 更新源并安装最新版 kubenetes
sudo apt update && apt install -y kubelet kubeadm kubectl
# 锁定版本 kubelet kubeadm kubectl 版本不匹配会导致异常错误
apt-mark hold kubelet kubeadm kubectl

nano ~/.zshrc
# 添加 completion，最好放入 .zshrc 中
source <(kubectl completion zsh)
source <(kubeadm completion zsh)
alias k=kubectl
compdef __start_kubectl k
```

## init master node

[Official Tutorials](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

--pod-network-cidr= 需要和你接下来要安装的 网络插件要相匹配

```bash

# use [calico](https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart)
kubeadm init --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers' --pod-network-cidr=192.168.0.0/16

# cd other sudo user's HOME folder which you want to use
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## set config 设置读取配置路径

```bash
nano ~/.zshrc
export KUBECONFIG=/etc/kubernetes/admin.conf
```

## install network plugin (use calico)

use [calico](https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart)

## add work node

[create-cluster-kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

if only have one machine， maybe you want to use master node to work
```bash
# 允许 master 节点调度 Pod
kubectl taint nodes --all node-role.kubernetes.io/master-
```

```bash
# 获取 work join 的 token
kubeadm token list
# token 过期后重新获取
kubeadm token create

# 获取 ca cert hash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | sha256sum | awk '{print $1}'

kubeadm join `replace this master IP`:6443 \
--token `replace this token` \
--discovery-token-ca-cert-hash \
`replace this ca cert hash`
```

## install ingress-nginx

[Official Tutorials](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start)
```bash
wget -c  https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.0/deploy/static/provider/cloud/deploy.yaml -O ingress-nginx.yaml
# update port 80 and 443 or use default random port
kubectl apply -f ingress-nginx.yaml
```

## Q/A
### helm install failed, how to fix it?
View logs by running commands `kubectl -n ingress-nginx get events --sort-by='{.lastTimestamp}'`

### 

