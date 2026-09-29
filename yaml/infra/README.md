# 從零開始建置 kind 環境

從零建一個 kind cluster，裝好 Cilium，再把 bobo 和 MySQL 部署上去。做完之後，從主機開 http://10.89.0.225:3000 就能用 bobo。從其他電腦要透過遠端桌面連進來，見第 9 步。

## 檔案

| 檔案 | 用途 |
| --- | --- |
| `kind.yaml` | 3 個節點，不裝預設 CNI，也不裝 kube-proxy |
| `cilium-values.yaml` | Cilium 的 Helm values |
| `cilium-lb-bobo.yaml` | bobo 專用的 LoadBalancer IP pool 和 L2 announcement 規則 |

bobo 和 MySQL 本身的 YAML 在上一層：`yaml/deploy.yaml`、`yaml/mysql.yaml`。

## 版本

| 元件 | 版本 |
| --- | --- |
| kind | v0.32.0，用 podman provider |
| kindest/node | v1.36.1 |
| Cilium | 1.20.1 |
| Gateway API CRD | v1.6.1 standard |

## 網段

| 用途 | CIDR |
| --- | --- |
| podman 網路 `kind`，也就是節點的網段 | 10.89.0.0/24 |
| Pod | 10.244.0.0/16 |
| Service | 10.96.0.0/16 |
| bobo 的 LoadBalancer IP | 10.89.0.224/28，bobo 固定用 10.89.0.225 |

LoadBalancer IP 一定要在節點的網段裡，主機才連得到。所以第 1 步要先把 podman 網路固定成 10.89.0.0/24。

## 事前準備

從一台剛裝好的 Ubuntu 26.04 server 開始，架構是 amd64，使用者要有 sudo 權限。下面的工具版本都跟目前在跑的環境一樣。

| 工具 | 版本 | 安裝方式 |
| --- | --- | --- |
| podman | 5.7.0 | Ubuntu 套件 |
| kind | v0.32.0 | 官方執行檔 |
| kubectl | v1.36.1 | 官方執行檔 |
| helm | v4.2.4 | 官方執行檔 |
| cilium CLI | v0.19.7 | GitHub release |

### 安裝系統套件

```sh
sudo apt-get update
sudo apt-get install -y podman git curl tar
podman --version
```

### 安裝 kind、kubectl、helm、cilium CLI

每個執行檔下載後都會先比對 sha256，比對失敗就不會安裝。

```sh
cd /tmp

# kind
curl -fLo kind https://kind.sigs.k8s.io/dl/v0.32.0/kind-linux-amd64
curl -fsL https://kind.sigs.k8s.io/dl/v0.32.0/kind-linux-amd64.sha256sum | awk '{print $1"  kind"}' | sha256sum -c
sudo install -m 0755 kind /usr/local/bin/kind

# kubectl
curl -fLO https://dl.k8s.io/release/v1.36.1/bin/linux/amd64/kubectl
echo "$(curl -fsL https://dl.k8s.io/release/v1.36.1/bin/linux/amd64/kubectl.sha256)  kubectl" | sha256sum -c
sudo install -m 0755 kubectl /usr/local/bin/kubectl

# helm
curl -fLO https://get.helm.sh/helm-v4.2.4-linux-amd64.tar.gz
curl -fsL https://get.helm.sh/helm-v4.2.4-linux-amd64.tar.gz.sha256sum | sha256sum -c
tar -xzf helm-v4.2.4-linux-amd64.tar.gz
sudo install -m 0755 linux-amd64/helm /usr/local/bin/helm

# cilium CLI
curl -fLO https://github.com/cilium/cilium-cli/releases/download/v0.19.7/cilium-linux-amd64.tar.gz
curl -fsL https://github.com/cilium/cilium-cli/releases/download/v0.19.7/cilium-linux-amd64.tar.gz.sha256sum | sha256sum -c
sudo tar -xzf cilium-linux-amd64.tar.gz -C /usr/local/bin

rm -rf kind kubectl helm-v4.2.4-linux-amd64.tar.gz linux-amd64 cilium-linux-amd64.tar.gz
cd ~
```

裝完確認一下版本：

```sh
kind version
kubectl version --client
helm version --short
cilium version --client
```

### 下載 repo

```sh
git clone https://github.com/braveantony/bobo-system.git ~/bobo-system
cd ~/bobo-system
K="kubectl --context kind-kind"
```

kind 節點跑在 root 的 podman 底下，所以 kind 和 podman 的指令都要加 sudo。後面的指令都在 repo 根目錄執行，而且會用到上面設的 `K` 變數。如果中途開了新的 terminal，要重新 `cd` 進 repo 並設定 `K`。

## 建置步驟

### 1. 建立 podman 網路

```sh
sudo podman network exists kind || sudo podman network create --driver bridge --subnet 10.89.0.0/24 --gateway 10.89.0.1 kind
sudo podman network inspect kind --format '{{range .Subnets}}{{.Subnet}} {{end}}'
```

第二行的輸出一定要有 10.89.0.0/24。如果 `kind` 網路已經存在但網段不同，要先 `sudo podman network rm kind` 再重建，不然後面的 LoadBalancer IP 會連不到。

### 2. 建立 cluster

```sh
sudo KIND_EXPERIMENTAL_PROVIDER=podman KIND_EXPERIMENTAL_PODMAN_NETWORK=kind \
  kind create cluster --config yaml/infra/kind.yaml
```

這時節點會是 NotReady，因為還沒裝 CNI，這是正常的。

### 3. 把 kubeconfig 合併到自己的帳號

cluster 是用 sudo 建的，kubeconfig 寫在 root 底下，要搬過來一般使用者才能用 kubectl。

```sh
mkdir -p ~/.kube
tmp=$(mktemp)
sudo KIND_EXPERIMENTAL_PROVIDER=podman kind get kubeconfig --name kind > "$tmp"
KUBECONFIG="$HOME/.kube/config:$tmp" kubectl config view --flatten > ~/.kube/config.new
mv ~/.kube/config.new ~/.kube/config && chmod 600 ~/.kube/config && rm "$tmp"
$K get nodes
```

### 4. 安裝 Gateway API CRD

`cilium-values.yaml` 有開 `gatewayAPI.enabled`，CRD 要在裝 Cilium 之前先裝好。

```sh
B=https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v1.6.1/config/crd/standard
for c in gatewayclasses gateways httproutes grpcroutes referencegrants backendtlspolicies tlsroutes; do
  $K apply --server-side -f $B/gateway.networking.k8s.io_${c}.yaml
done
```

### 5. 安裝 Cilium

```sh
helm repo add cilium https://helm.cilium.io/ && helm repo update
helm install cilium cilium/cilium --version 1.20.1 -n kube-system \
  --kube-context kind-kind -f yaml/infra/cilium-values.yaml
$K -n kube-system rollout status ds/cilium --timeout=15m
$K wait --for=condition=Ready node --all --timeout=5m
cilium --context kind-kind status --wait
```

三個節點都變成 Ready 才算完成。

### 6. 建立 bobo 的 IP pool 和 L2 規則

```sh
$K apply -f yaml/infra/cilium-lb-bobo.yaml
$K get ciliumloadbalancerippool bobo-pool
$K get ciliuml2announcementpolicy bobo-l2
```

這一步一定要在部署 bobo 之前做。順序反過來的話，bobo 的 service 會一直拿不到 IP。

### 7. 部署 MySQL 和 bobo

bobo 的 image 放在 GitHub Container Registry，`yaml/deploy.yaml` 已經指定好版本，直接套用就好。

```sh
$K create namespace bobo
$K apply -n bobo -f yaml/mysql.yaml
$K apply -n bobo -f yaml/deploy.yaml
$K -n bobo rollout status sts/mysql --timeout=5m
$K -n bobo rollout status deploy/bobo --timeout=2m
```

### 8. 驗證

```sh
$K -n bobo get svc bobo
curl -s -o /dev/null -w "%{http_code}\n" http://10.89.0.225:3000/
```

service 的 EXTERNAL-IP 要是 10.89.0.225，curl 要回 200。接著用瀏覽器開 http://10.89.0.225:3000，登入畫面這樣填：

| 欄位 | 值 |
| --- | --- |
| Host | mysql.bobo.svc.cluster.local |
| Port | 3306 |
| User | bigred |
| Password | bigred |

登入後應該會看到 `test` 資料庫。

### 9. 用遠端桌面連到 bobo

10.89.0.225 只有這台主機連得到，其他電腦看不到這個網段。要從自己的電腦用瀏覽器操作 bobo，可以在主機上跑一個有桌面的容器，再用遠端桌面連進去。

```sh
sudo podman run -itd -p 3389:3389 --name desktop --shm-size=2gb quay.io/flysangel/rdp:xfce4-v1.0.2
sudo podman exec desktop curl -s -o /dev/null -w "%{http_code}\n" http://10.89.0.225:3000/
```

第二行是從容器裡面測試能不能連到 bobo，要回 200。容器接在 podman 預設網路就連得到，不用另外接到 kind 網路。

接著在自己的電腦用遠端桌面連線，Windows 可以用內建的「遠端桌面連線」，macOS 可以用 Windows App。

| 欄位 | 值 |
| --- | --- |
| 電腦 | `<主機 IP>:3389` |
| 使用者名稱 | bigred |
| 密碼 | bigred |

登入後從左上角的應用程式選單打開 Firefox，網址輸入 http://10.89.0.225:3000，就會看到 bobo 的登入畫面。登入要填的值跟第 8 步一樣。

`--shm-size=2gb` 是給 Firefox 用的共享記憶體，太小的話分頁容易當掉。

## 日常操作

### 更新 Cilium 設定

改完 `cilium-values.yaml` 後執行：

```sh
helm upgrade cilium cilium/cilium --version 1.20.1 -n kube-system \
  --kube-context kind-kind -f yaml/infra/cilium-values.yaml
$K -n kube-system rollout status ds/cilium --timeout=10m
```

### 拆掉整個 cluster

```sh
sudo podman rm -f desktop
sudo KIND_EXPERIMENTAL_PROVIDER=podman kind delete cluster --name kind
kubectl config delete-context kind-kind
kubectl config delete-cluster kind-kind
kubectl config delete-user kind-kind
```

### 發佈新版的 bobo image

改了 bobo 的程式碼之後，要 build 新的 image 並推到 ghcr.io。

推 image 到 ghcr.io 要用 classic personal access token，而且要勾 `write:packages`。fine-grained token 不支援 Container Registry，推送時會被拒絕。

```sh
IMG=ghcr.io/braveantony/bobo:$(git rev-parse --short HEAD)
sudo podman build -t $IMG --label org.opencontainers.image.source=https://github.com/braveantony/bobo-system .
echo "<classic token>" | sudo podman login ghcr.io -u braveantony --password-stdin
sudo podman push $IMG
```

推完後把 `yaml/deploy.yaml` 的 image 改成新的 tag，commit 之後重新套用：

```sh
$K apply -n bobo -f yaml/deploy.yaml
$K -n bobo rollout status deploy/bobo --timeout=2m
```

## 注意事項

- MySQL 沒有掛 PVC，pod 重建資料就會不見。
- MySQL 用 hostNetwork，會直接佔用所在節點的 3306 port，所以只能跑一份。
- `yaml/mysql.yaml` 裡的密碼是明文，只適合測試環境。
- 多個 IP pool 的 CIDR 不能重疊，重疊的話 pool 會被標成 conflict。10.89.0.240/28 已經有其他服務在用，新增 pool 時要避開。
- 同時跑好幾個 kind cluster 時，可能要調高 inotify 上限：`sudo sysctl -w fs.inotify.max_user_instances=1024 fs.inotify.max_user_watches=1048576`
- cluster 改名的話，`cilium-values.yaml` 的 `k8sServiceHost` 要跟著改成 `<名稱>-control-plane`。
