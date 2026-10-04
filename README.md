# CI Templates - 共享部署模板

GitHub Actions 可复用工作流模板，用于将项目部署到腾讯云服务器 (62.234.100.4)。

## 模板列表

| 模板 | 适用场景 | 说明 |
|------|----------|------|
| `deploy-pm2.yml` | Next.js / Node.js 项目 | PM2 管理进程，支持 Prisma |
| `deploy-docker.yml` | Docker 项目 | docker-compose 部署 |

## 快速使用

### 1. PM2 项目 (Next.js)

在项目的 `.github/workflows/deploy.yml` 中：

```yaml
name: Deploy to Tencent Cloud

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    uses: amapig/ci-templates/.github/workflows/deploy-pm2.yml@main
    with:
      project_name: 你的项目名
      repo_name: 你的仓库名
      deploy_path: /root/你的项目路径
      pm2_process: PM2进程名
      branch: ${{ github.ref_name }}
      env_vars: |
        DATABASE_URL=${{ secrets.DATABASE_URL }}
        NEXTAUTH_SECRET=${{ secrets.NEXTAUTH_SECRET }}
        NEXTAUTH_URL=${{ secrets.NEXTAUTH_URL }}
        AUTH_SECRET=${{ secrets.AUTH_SECRET }}
    secrets:
      SERVER_SSH_KEY: ${{ secrets.SERVER_SSH_KEY }}
      GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 2. Docker 项目

```yaml
name: Deploy to Tencent Cloud

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    uses: amapig/ci-templates/.github/workflows/deploy-docker.yml@main
    with:
      project_name: 你的项目名
      repo_name: 你的仓库名
      deploy_path: /root/你的项目路径
      branch: ${{ github.ref_name }}
      env_vars: |
        DATABASE_URL=${{ secrets.DATABASE_URL }}
        # 其他环境变量...
    secrets:
      SERVER_SSH_KEY: ${{ secrets.SERVER_SSH_KEY }}
      GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## 配置参数

### deploy-pm2.yml 输入参数

| 参数 | 必填 | 默认值 | 说明 |
|------|------|--------|------|
| `project_name` | ✅ | - | 项目名称 (用于脚本命名) |
| `repo_name` | ✅ | - | GitHub 仓库名 |
| `deploy_path` | ✅ | - | 服务器部署目录 |
| `pm2_process` | ✅ | - | PM2 进程名 |
| `env_vars` | ✅ | - | 环境变量 (每行 KEY=VALUE) |
| `branch` | ❌ | `main` | 部署分支 |
| `pm2_mode` | ❌ | `restart` | `restart`=快速重启 / `delete-recreate`=重建进程 |
| `build_cmd` | ❌ | `npm run build` | 构建命令 |
| `prisma_generate` | ❌ | `true` | 是否运行 `prisma generate` |
| `prisma_migrate` | ❌ | `true` | 是否运行 `prisma db push` |
| `server_host` | ❌ | `62.234.100.4` | 服务器 IP |
| `server_user` | ❌ | `root` | 服务器用户名 |

### deploy-docker.yml 输入参数

| 参数 | 必填 | 默认值 | 说明 |
|------|------|--------|------|
| `project_name` | ✅ | - | 项目名称 |
| `repo_name` | ✅ | - | GitHub 仓库名 |
| `deploy_path` | ✅ | - | 服务器部署目录 |
| `env_vars` | ❌ | `''` | 环境变量 (每行 KEY=VALUE) |
| `branch` | ❌ | `main` | 部署分支 |
| `server_host` | ❌ | `62.234.100.4` | 服务器 IP |
| `server_user` | ❌ | `root` | 服务器用户名 |

## GitHub Secrets 配置

每个项目仓库需要配置以下 Secrets：

| Secret | 说明 | 获取方式 |
|--------|------|----------|
| `SERVER_SSH_KEY` | 服务器 SSH 私钥 | 从 `~/.ssh/tencent_cloud` 获取内容 |
| `GITHUB_TOKEN` | GitHub 自动提供 | 无需配置，GitHub Actions 自动生成 |
| 项目特有 Secrets | 如 `DATABASE_URL`, `NEXTAUTH_SECRET` 等 | 在 `env_vars` 中通过 `${{ secrets.XXX }}` 引用 |

## 项目配置速查

| 项目 | 类型 | deploy_path | pm2_process | 端口 |
|------|------|-------------|-------------|------|
| jiankaopaike | PM2 | `/root/jiankao/jiankaopaike` | `jiankaopaike` | 3001 |
| after-school-management | PM2 | `/root/after-school-management` | `after-school` | 3333 |
| school-lunch-waste-monitor | PM2 | `/root/school-lunch-waste-monitor` | `lunch-monitor` | 1111 |
| runwise-platform | PM2 | `/root/runwise-platform` | `runwise` | 8080 |
| shengxue-ai-planner | PM2 | `/root/shengxue-ai-planner` | `shengxue-ai` | 3000 |
| quantitative | Docker | `/root/quantitative` | - | 3001/8000 |

## PM2 重启模式说明

- **`restart`** (默认): 使用 `pm2 restart`，进程环境变量不变，速度快
- **`delete-recreate`**: 先 `pm2 delete` 再 `pm2 start`，会重新加载 `.env` 文件中的环境变量

当修改了 `.env` 中的环境变量时，必须使用 `delete-recreate` 模式才能生效。

## 更新模板

修改本仓库的模板后，所有引用它的项目会自动使用最新版本（如果引用 `@main` 分支）。

如需锁定版本，可以引用特定 commit：
```yaml
uses: amapig/ci-templates/.github/workflows/deploy-pm2.yml@<commit-sha>
```
