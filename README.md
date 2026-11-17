# openmaic-ci

OpenMAIC 镜像的云端构建仓库。**只放构建配方，不放源码**——源码在每次构建时从上游拉。

- 上游：<https://github.com/THU-MAIC/OpenMAIC>（MIT）
- 产物：`ghcr.io/elliot-13/openmaic:latest`（另有 `:构建序号`）
- 触发：手动 / 每周一 04:00 CST / 改 `Dockerfile` 或本工作流

## 为什么单独开一个仓库

家里 NAS 和小马都试过：

| 位置 | 问题 |
|---|---|
| NAS | Docker Hub 被掐（30–56 KB/s），`pnpm build` 把 swap 吃到 10G |
| 小马（Win10 + WSL2） | 能编译（拉镜像 8.9 秒、构建 6.4 秒），但 WSL 实例会卡死在 `LxssManager StopPending`，只有重启 Windows 能清 |

GitHub Actions 的 runner 有干净的 Linux + 直连 Docker Hub + 32G 内存，跑完即销毁，不存在残留状态。

## `Dockerfile` 相对上游改了什么

1. `corepack enable && corepack prepare pnpm@10.28.0` → `npm install -g pnpm@10.28.0`
   （Alpine 的 node 包已不带 corepack，上游这条会 exit 127）
2. `RUN pnpm build` → `NODE_OPTIONS="--max-old-space-size=2600" NEXT_TELEMETRY_DISABLED=1 pnpm build`
   （默认堆不够，会 `FATAL ERROR: Ineffective mark-compacts near heap limit`）
3. `NPM_REGISTRY` 默认值指向 `registry.npmjs.org`（runner 在国内之外的网络环境）

## 在 NAS 上更新镜像

```bash
docker pull ghcr.io/elliot-13/openmaic:latest
cd /vol1/1000/Docker/openmaic
# compose 里 image 指向 ghcr，或直接 tag 成原来的名字
/opt/data/scripts/dc -C /vol1/1000/Docker/openmaic up -d
```

## 注意

- 本仓库和镜像**不含任何密钥**：模型 key 在 NAS 的 `.env.local` 里，运行时才注入。
- 上游是 MIT，镜像里只有上游代码 + 上面两处构建修正。
