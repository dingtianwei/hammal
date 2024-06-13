# hammal

Hammal 是运行于 cloudflare workers 上的 Docker 镜像加速工具，用于解决获取 Docker 官方镜像速度缓慢以及完全无法获取 k8s.gcr.io 上镜像的问题。

### 食用方式

首先安装 wrangler 命令行工具 https://developers.cloudflare.com/workers/cli-wrangler/install-update

```
git clone https://github.com/tomwei7/hammal.git
cd hammal
mv wrangler.toml.sample wrangler.toml

# 获取 account_id id
wrangler whoami

# 创建 KV namespace
wrangler kv:namespace create hammal_cache

```

修改 wrangler.toml 文件填充 account_id 与 kv_namespaces 中的 id

```toml
name = "hammal"
type = "webpack"
account_id = "your account_id"
workers_dev = true
route = ""
zone_id = ""
webpack_config = "webpack.config.js"
kv_namespaces = [
	 { binding = "HAMMAL_CACHE", id = "you kv id" }
]
```

发布 workers

```
wrangler publish
```

发布 workers 可以获得类似 https://hammal.{your_name}.workers.dev  的地址，修改 registry-mirrors 地址为该地址即可

```
<<EOF sudo tee /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://hammal.{your_name}.workers.dev"
  ]
}
EOF
```

### 获取其他镜像源镜像

目前 hammal 支持获取下列站点的镜像
- `k8s.gcr.io`
- `gcr.io`
- `quay.io` 
- `ghcr.io`
- `registry.k8s.io`
- `docker.cloudsmith.io`

可以通过 `handler.ts` 中的 `ORG_NAME_BACKEND` 修改和删减；

```ts
const ORG_NAME_BACKEND:{ [key: string]: string; } = {
  "gcr": "https://gcr.io",
  "k8sgcr": "https://k8s.gcr.io",
  "quay": "https://quay.io",
  "ghcr": "https://ghcr.io",
  "k8s": "https://registry.k8s.io",
  "cloudsmith": "https://docker.cloudsmith.io"
}
```

#### 方式1 - 基于path的路由
```bash
# 拉取 k8s.gcr.io 镜像
docker pull hammal.{your_name}.workers.dev/k8sgcr/kubernetes-dashboard-amd64:v1.8.3

# 拉取 gcr.io 镜像
docker pull hammal.{your_name}.workers.dev/gcr/youlib/image:tags

# 拉取 quay.io 镜像
docker pull hammal.{your_name}.workers.dev/quay/coreos/flannel:v0.13.1-rc2
```

#### 方式2 - 基于域名的路由
为worker添加对应的自定义域名，如：`xxx.example.com`, `xxx` 与 `ORG_NAME_BACKEND`中的`key`值对应
```bash
# 拉取 k8s.gcr.io 镜像
docker pull k8sgcr.example.com/kubernetes-dashboard-amd64:v1.8.3

# 拉取 gcr.io 镜像
docker pull gcr.example.com/youlib/image:tags

# 拉取 quay.io 镜像
docker pull quay.example.com/coreos/flannel:v0.13.1-rc2

```


### TODO

- [ ] 私有仓库支持
- [ ] manifests/blob cache

