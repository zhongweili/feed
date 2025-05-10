[中文文档](./README.md) | [English Documentation](./README.en.md)

Forked from https://github.com/Seanium/FeedMe

# <p align="center">😋FeedMe</p>

<p align="center">
  <b>用 AI 重新定义你的 RSS 阅读体验，轻松部署到 GitHub Pages / Docker</b>
</p>

<p align="center">
  <a href="https://feedme.icu" target="_blank">🌐 在线演示</a> •
  <a href="#主要功能">✨ 功能</a> •
  <a href="#部署指南">🚀 部署</a> •
  <a href="#开发指南">💻 开发</a>
</p>

---

## 轻量、自由、为你定制

- 希望能够**一站式**了解各个信息源的新鲜事。又嫌市面上产品太"重"（**不想注册登录/下载 APP/启动桌面软件...**），一个静态页面正合适（响应式 / 亮暗主题，电脑手机都方便浏览）。
- 希望能够 **TLDR** 式地速览，于是做了 AI 摘要。
- **自由配置** RSS 源、AI 模型、更新频率。
- **开源、方便 Fork、零成本、轻松自部署**。

## 主要功能

- **多源 RSS 聚合**: 从多个信息源获取并整合 RSS 内容
- **AI 摘要生成**: 自动使用 LLM 为文章生成摘要
- **定时更新机制**: 通过 GitHub Actions 或 crond 定期自动更新内容
- **分类浏览**: 支持按分类查看不同信息源
- **主题切换**: 支持明暗主题切换
- **静态部署**: 可部署在 GitHub Pages 等静态托管服务上
- **Docker 部署**：轻松部署在本地服务器上

## 部署指南

### 方式一：GitHub Pages 部署

本项目使用 GitHub Actions 自动部署到 GitHub Pages，使用工作流处理数据更新和网站部署。

1. **Fork 或克隆仓库**到你的 GitHub 账号

2. **设置 GitHub Secrets**

   在项目顶端 Settings - 左侧 Secrets and variables -> Actions 中添加以下密钥（**Secrets**）：

   - `LLM_API_KEY`: 用于 AI 摘要生成的 API 密钥
   - `LLM_API_BASE`: LLM 服务的 API 基础 URL
   - `LLM_NAME`: 使用的模型名称

3. **启用 GitHub Pages**

   在仓库设置中，选择从 GitHub Actions 部署

4. **手动触发工作流**（可选）

   在 GitHub 仓库的 Actions 页面手动触发"更新数据并部署"工作流

#### 工作流说明

**更新数据并部署** (`update-deploy.yml`)：

- 触发条件：
  - 定时执行（每 3 小时一次）
  - 推送到 main 分支
  - 手动触发
- 执行内容：
  - 获取最新 RSS 内容并生成摘要
  - 构建静态网站
  - 根据仓库变量设置决定部署目标:
    - 总是部署到 GitHub Pages
    - 如果 `ENABLE_VERCEL_DEPLOYMENT` 为 `true` 则部署到 Vercel

#### 自定义部署配置

- **自定义 RSS 源**：
  编辑 `config/rss-config.js` 文件以修改或添加 RSS 源。每个源需要包含：

  - 名称
  - URL
  - 分类

- **修改更新频率**: 编辑 `.github/workflows/update-deploy.yml` 中的 cron 表达式

  ```yml
  # 例如，改为每天凌晨更新一次
  cron: "0 0 * * *"
  ```

- **调整保留条目数**: 修改 `config/rss-config.js` 中的 `maxItemsPerFeed` 值

- **自定义域名配置**:
  请按照以下内容设置，避免出现页面资源加载异常：

  - **不使用自定义域名**: 请删除目录下的 `CNAME` 文件
  - **使用自定义域名**: 在仓库设置的 GitHub Pages 部分添加自定义域名，并修改 CNAME 文件内容为自定义域名
  - **多平台部署**: 系统会自动处理不同平台（GitHub Pages/Vercel）的路径差异:
    - GitHub Pages: 自动使用 `/{仓库名}` 为 basePath
    - 自定义域名: 不会添加 basePath
    - Vercel: 不会添加 basePath

- **自定义摘要生成**：
  如果需要自定义摘要生成方法，比如遵循特定格式或切换摘要语言，请修改 `scripts\update-feeds.js` 中的 `prompt` 变量

### 方式二：Vercel 部署

将你的 GitHub 仓库导入到 Vercel：

1. 前往 [Vercel 导入页面](https://vercel.com/import/git)
2. 选择 "GitHub" 并授权访问
3. 搜索并选择你 fork 的 FeedMe 仓库
4. 保持默认设置，点击 "Deploy" 开始部署

**配置自动更新：**

1. 完成 Vercel 部署后，获取以下信息:
   - `VERCEL_TOKEN`: 从 [Vercel Tokens](https://vercel.com/account/tokens) 创建
   - `VERCEL_ORG_ID`: 从 [账户设置](https://vercel.com/account) > 常规 > 页面底部获取
   - `VERCEL_PROJECT_ID`: 从 [Vercel Dashboard](https://vercel.com/dashboard) > 你的项目 > 设置 > 常规 > 页面底部获取
2. 添加上述信息到仓库密钥（**Secrets**）（位置：Settings -> Secrets and variables -> Actions -> **Secrets**）
3. 添加仓库变量（**Variables**） `ENABLE_VERCEL_DEPLOYMENT` 并设为 `true`（位置：Settings -> Secrets and variables -> Actions -> **Variables**）

### 方式三：Docker 本地部署

此方式使用 Docker 在本地或服务器上运行 FeedMe，并通过容器内的 Cron 任务自动更新数据和重建，不依赖 GitHub Actions。

1.  **克隆仓库**

    ```bash
    git clone https://github.com/zhongweili/feed.git
    cd feedme
    ```

2.  **配置环境变量**
    复制 `.env.example` 文件为 `.env` 并填入必要的 API 密钥：

    ```bash
    cp .env.example .env
    ```

    编辑 `.env` 文件：

    ```dotenv
    LLM_API_KEY=你的_API_密钥
    LLM_API_BASE=LLM服务的API基础URL
    LLM_NAME=使用的模型名称
    ```

3.  **构建并启动 Docker 容器**

    ```bash
    docker-compose up --build
    ```

4.  **访问应用**
    应用将在 [http://localhost:3000](http://localhost:3000) 上可用。

5.  **自动更新**
    容器将根据 `config/crontab-docker` 中的配置（默认为每 3 小时）自动执行 `pnpm update-feeds` 和 `pnpm build`，并重新启动服务。
    如需修改更新频率，请编辑 `config/crontab-docker` 文件中的 cron 表达式（例如 `0 */6 * * *` 表示每 6 小时执行一次）。

## 开发指南

1. **克隆仓库**

   ```bash
   git clone https://github.com/zhongweili/feed.git
   cd feedme
   ```

2. **安装依赖**

   ```bash
   pnpm install
   ```

3. **配置环境变量**

   复制环境变量示例文件并编辑：

   ```bash
   cp .env.example .env
   ```

   填入以下内容：

   ```
   LLM_API_KEY=你的 API 密钥
   LLM_API_BASE=LLM服务的 API 基础 URL（例如：https://api.siliconflow.cn/v1）
   LLM_NAME=使用的模型名称（例如：THUDM/GLM-4-9B-0414）
   ```

   这些环境变量用于配置文章摘要生成功能，需要从 LLM 服务提供商获取

4. **更新 RSS 数据**

   ```bash
   pnpm update-feeds
   ```

   此命令会抓取 RSS 源并生成摘要，保存到 `data` 目录

5. **启动开发服务器**
   ```bash
   pnpm dev
   ```
   访问 [http://localhost:3000](http://localhost:3000) 查看应用

## 许可证

[MIT](LICENSE) © 2025
