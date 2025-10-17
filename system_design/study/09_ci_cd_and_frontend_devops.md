# 09 — Front-End Deployment, CI/CD Pipelines & DevOps (Part 1 of 3)
*(Sections 1–2 | Extended Bilingual Concepts Version)*

---

## 1. Front-End Deployment Lifecycle

The front-end deployment process defines **how code transitions from local development to production safely and predictably.**

> 💡 **中文解释：** 前端部署生命周期描述代码从本地开发到生产环境的全过程，目标是安全、可预测、可回滚。

### 1.1 Typical Lifecycle Stages

```
Code → Build → Test → Package → Release → Deploy → Monitor
```

| Stage | Description |
|--------|--------------|
| **Build** | Transpile TS/JS, bundle with Webpack/Vite |
| **Test** | Unit & integration tests (Jest, Playwright) |
| **Package** | Compress, version, and create artifacts |
| **Release** | Tag and publish build (CDN/S3/registry) |
| **Deploy** | Push to production environment |
| **Monitor** | Track health, metrics, and errors |

> 💡 **中文解释：** 构建阶段负责编译与打包，发布阶段生成工件，部署阶段推送到环境，监控阶段跟踪系统健康。

### 1.2 Deployment Models

| Model | Description | Example |
|--------|--------------|----------|
| **Static Hosting** | Deploy static HTML/JS/CSS to CDN | Netlify, Vercel, AWS S3 + CloudFront |
| **SSR Hosting** | Deploy Node server that renders HTML | Next.js, Remix on AWS ECS/Lambda |
| **Hybrid Edge Deployment** | Edge SSR with cache revalidation | Vercel Edge, Cloudflare Workers |

> 💡 **中文解释：** 静态托管适合纯前端应用；SSR 部署需要 Node 环境；边缘部署结合 SSR 与 CDN 性能优势。

---

## 2. Environment Configuration & Secrets Management

### 2.1 Environment Isolation

Each environment (dev, staging, prod) must have **isolated configs and secrets**.

| Environment | Purpose | Example |
|--------------|----------|----------|
| **Development** | Local iteration | `localhost:3000` |
| **Staging** | Pre-prod testing | `staging.example.com` |
| **Production** | Public traffic | `www.example.com` |

**.env Example:**
```bash
NODE_ENV=production
API_URL=https://api.example.com
SENTRY_DSN=https://123@sentry.io/456
```

> 💡 **中文解释：** 通过 `.env` 文件管理不同环境的变量，避免硬编码到源码。

### 2.2 Secrets Management

Sensitive credentials should never be stored in git repositories.

**Best Practices:**
- Store secrets in vaults: AWS Secrets Manager, GCP Secret Manager, HashiCorp Vault.  
- Inject secrets at runtime or CI pipeline.  
- Use short-lived credentials with rotation.

**GitHub Actions Example:**
```yaml
env:
  NODE_ENV: production
  API_KEY: ${{ secrets.API_KEY }}
```

> 💡 **中文解释：** 机密信息应保存在安全的密钥库，并在运行时注入而非硬编码。

# 09 — Front-End Deployment, CI/CD Pipelines & DevOps (Part 2 of 3)
*(Sections 3–4 | Extended Bilingual Concepts Version)*

---

## 3. CI/CD Pipeline Design

Continuous Integration (CI) ensures code quality before merging; Continuous Deployment (CD) automates release and delivery.

> 💡 **中文解释：** CI（持续集成）用于合并前质量校验；CD（持续交付）自动化构建与部署。

### 3.1 Typical Pipeline Architecture

```
Developer → Pull Request → CI Runner → Build + Test → Artifact → CD → Production
```

### 3.2 Common Tools

| Category | Tools |
|-----------|--------|
| **CI** | GitHub Actions, Jenkins, GitLab CI, CircleCI |
| **Build** | Webpack, Vite, Rollup |
| **Test** | Jest, Vitest, Playwright, Cypress |
| **Deploy** | Netlify, Vercel, AWS Amplify, GitHub Pages |

---

### 3.3 Pipeline Example (GitHub Actions)

```yaml
name: Frontend CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-test-deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 22

      - name: Install dependencies
        run: yarn install --frozen-lockfile

      - name: Run tests
        run: yarn test --coverage

      - name: Build production bundle
        run: yarn build

      - name: Deploy to S3 + CloudFront
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          aws s3 sync ./dist s3://my-app-bucket --delete
          aws cloudfront create-invalidation --distribution-id E1234567 --paths "/*"
```

> 💡 **中文解释：** 以上 GitHub Actions 工作流包含构建、测试与部署阶段，可实现自动化端到端交付。

---

### 3.4 Pipeline Optimization

| Optimization | Technique | Tool |
|---------------|-----------|------|
| **Caching** | Reuse `node_modules` / build cache | GitHub Actions cache |
| **Parallel Jobs** | Split lint/test/build | matrix strategy |
| **Conditional Deploy** | Deploy only on main branch | `if: github.ref == 'refs/heads/main'` |
| **Preview Environments** | Per PR staging URL | Vercel / Netlify |

> 💡 **中文解释：** 缓存、并行与条件执行可显著减少流水线耗时；预览环境支持快速回归验证。

---

## 4. Versioning & Rollback Strategies

### 4.1 Versioning Models

| Strategy | Description |
|-----------|--------------|
| **Semantic Versioning** | Major.Minor.Patch pattern | `1.4.2` |
| **Commit-based** | Auto tag with CI commit SHA | `build-abc123` |
| **Date-based** | Daily build versions | `2025.10.15.01` |

> 💡 **中文解释：** 使用语义化版本或基于日期/提交的版本号便于追踪与回滚。

### 4.2 Rollback Mechanisms

**Approaches:**
1. Maintain N previous build artifacts.  
2. Rollback = redeploy previous version.  
3. Keep deployment immutable; never overwrite artifacts.  

```bash
aws s3 cp s3://my-app-bucket/releases/v1.2.3 ./ --recursive
```

> 💡 **中文解释：** 保留多个构建版本可实现快速回滚；工件应保持不可变性（immutable deployment）。

# 09 — Front-End Deployment, CI/CD Pipelines & DevOps (Part 3 of 3)
*(Sections 5–6 | Extended Bilingual Concepts Version)*

---

## 5. Deployment Strategies & DevOps Integration

### 5.1 Blue-Green Deployment

Run two identical environments — **Blue (current)** and **Green (new)**. Traffic switches only after Green passes validation.

```
User → Load Balancer → Blue (v1) / Green (v2)
```

> 💡 **中文解释：** 蓝绿部署通过双环境切换实现无缝升级，降低停机风险。

### 5.2 Canary Deployment

Gradually release to a small subset of users (1% → 10% → 100%). Rollback quickly if anomalies occur.

```yaml
# Example: Canary rollout in CI
- name: Deploy Canary (10%)
  run: deploy.sh --env=canary --percent=10
```

> 💡 **中文解释：** 金丝雀发布逐步放量，可在发现异常时即时回滚，常用于高风险更新。

### 5.3 Feature Flags

Enable or disable features dynamically without redeploying.

**Tools:** LaunchDarkly, Split.io, Unleash

```js
if (featureFlags.newCheckout) renderNewFlow();
```

> 💡 **中文解释：** 功能开关让部署与发布解耦，支持灰度测试与 A/B 实验。

---

### 5.4 DevOps for Front-End

| Category | Tool | Role |
|-----------|------|------|
| **Infrastructure** | Docker, Terraform | Containerized builds |
| **Monitoring** | Datadog, Prometheus | Pipeline metrics |
| **Logging** | Loki, CloudWatch | Build logs |
| **Alerting** | PagerDuty, Opsgenie | On-call notifications |

> 💡 **中文解释：** 前端 DevOps 集成包括容器化、日志、监控与告警系统。

**Dockerfile Example (Next.js):**
```Dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN yarn install --production
COPY . .
RUN yarn build
CMD ["yarn", "start"]
```

---

## 6. Interview-Oriented Summary

### 6.1 “Design a CI/CD pipeline for a React app”

**Answer Framework:**
1. CI: lint, test, build → generate artifact.  
2. CD: upload to S3 + CloudFront.  
3. Add caching + preview env + rollback.  
4. Monitor build status via Slack integration.

> 💡 **中文解释：** 面试回答应突出自动化与安全回滚机制。

### 6.2 “How do you ensure safe deployments?”

| Area | Practice |
|-------|-----------|
| **Pre-deploy** | Automated tests, code review, canary rollout |
| **Deploy** | Immutable artifact, blue-green release |
| **Post-deploy** | Monitor errors, roll back on threshold breach |

### 6.3 “How do you manage secrets in CI/CD?”

- Store secrets in CI vault (GitHub Secrets / HashiCorp Vault).  
- Access via environment injection.  
- Never log secret values.  

> 💡 **中文解释：** CI/CD 中机密管理是关键安全环节，需结合 Vault 与最小权限原则。

---

**End of Chapter 9 — CI/CD & Front-End DevOps**