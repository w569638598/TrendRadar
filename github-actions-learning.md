# GitHub Actions 运行机制学习文档

## 1. 概述

GitHub Actions 是 GitHub 提供的持续集成和持续部署(CI/CD)平台，允许您在 GitHub 仓库中自动化构建、测试和部署工作流程。本项目使用 GitHub Actions 实现热点资讯的定时抓取、处理和推送。

## 2. 工作流程文件结构

项目中的工作流程文件位于 [.github/workflows/trend-radar.yml](file:///d:/work/github/me/TrendRadar/.github/workflows/trend-radar.yml)。该文件定义了完整的自动化流程，包括触发条件、执行步骤和权限设置。

### 2.1 基本结构

```yaml
name: Trend Radar News Crawler and Pages Deployment

on:
  # 触发条件
  schedule:
    - cron: "*/30 * * * *" # 每30分钟运行一次
  workflow_dispatch: # 允许手动触发
  push:
    branches:
      - main
      - master

# 权限设置
permissions:
  contents: write
  pages: write
  id-token: write
```

### 2.2 触发条件说明

- `schedule`: 使用 cron 表达式设置定时任务，本项目设置为每30分钟运行一次
- `workflow_dispatch`: 允许通过 GitHub 网页界面手动触发工作流程
- `push`: 当向 main 或 master 分支推送代码时触发

### 2.3 权限设置

为确保工作流程可以正确执行，需要设置相应的权限：
- `contents: write`: 允许写入仓库内容
- `pages: write`: 允许部署 GitHub Pages
- `id-token: write`: 允许获取 ID 令牌

## 3. 工作流程任务

工作流程包含两个主要任务：`crawl-and-build` 和 `deploy-pages`。

### 3.1 crawl-and-build 任务

此任务负责抓取热点数据并构建内容：

```yaml
jobs:
  crawl-and-build:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.9"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          if [ -f requirements.txt ]; then pip install -r requirements.txt; else echo "requirements.txt not found"; fi

      - name: Verify required configuration files
        run: |
          # 检查并创建必要的配置文件

      - name: Run crawler
        env:
          FEISHU_WEBHOOK_URL: ${{ secrets.FEISHU_WEBHOOK_URL }}
          TELEGRAM_BOT_TOKEN: ${{ secrets.TELEGRAM_BOT_TOKEN }}
          # 其他环境变量
        run: |
          python main.py || echo "Crawler execution completed with status: $?"

      - name: Commit and push changes
        run: |
          git config --global user.name 'GitHub Actions'
          git config --global user.email 'actions@github.com'
          git add -A
          git diff --quiet && git diff --staged --quiet || (git commit -m "Auto update by GitHub Actions: $(date)" && git push) || echo "No changes to commit or push failed"
```

#### 步骤说明：

1. **Checkout repository**: 检出代码仓库
2. **Set up Python**: 设置 Python 环境，使用 3.9 版本
3. **Install dependencies**: 安装项目依赖
4. **Verify required configuration files**: 检查并创建必要的配置文件
5. **Run crawler**: 运行主程序抓取热点数据
6. **Commit and push changes**: 提交并推送生成的更改

### 3.2 deploy-pages 任务

此任务负责部署 GitHub Pages：

```yaml
  deploy-pages:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: crawl-and-build
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Pages
        uses: actions/configure-pages@v5

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '.'

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

#### 步骤说明：

1. **Checkout**: 检出代码仓库
2. **Setup Pages**: 配置 GitHub Pages
3. **Upload artifact**: 上传构建产物
4. **Deploy to GitHub Pages**: 部署到 GitHub Pages

## 4. 环境变量和 Secrets

项目使用 GitHub Secrets 来管理敏感信息：

```yaml
env:
  FEISHU_WEBHOOK_URL: ${{ secrets.FEISHU_WEBHOOK_URL }}
  TELEGRAM_BOT_TOKEN: ${{ secrets.TELEGRAM_BOT_TOKEN }}
  TELEGRAM_CHAT_ID: ${{ secrets.TELEGRAM_CHAT_ID }}
  DINGTALK_WEBHOOK_URL: ${{ secrets.DINGTALK_WEBHOOK_URL }}
  WEWORK_WEBHOOK_URL: ${{ secrets.WEWORK_WEBHOOK_URL }}
  GITHUB_ACTIONS: true
```

这些 Secrets 需要在 GitHub 仓库的 Settings > Secrets and variables > Actions 中配置。

## 5. 数据处理和存储

- 程序运行时会抓取多个平台的热点数据
- 数据按日期分类存储在 output 目录中
- 每天的数据存储在以日期命名的文件夹中
- 支持三种推送模式：daily（当日汇总）、current（当前榜单）、incremental（增量监控）

## 6. 配置文件管理

工作流程会检查必要的配置文件是否存在，如果不存在则创建默认配置：

- [config/config.yaml](file:///d:/work/github/me/TrendRadar/config/config.yaml): 主配置文件
- [config/frequency_words.txt](file:///d:/work/github/me/TrendRadar/config/frequency_words.txt): 关键词过滤配置文件

## 7. 最佳实践

### 7.1 Actions 版本管理

使用稳定版本的 GitHub Actions：
- `actions/checkout@v4`
- `actions/setup-python@v5`
- `actions/configure-pages@v5`
- `actions/deploy-pages@v4`

### 7.2 错误处理

在关键步骤中添加错误处理，避免单个步骤失败导致整个工作流崩溃：

```yaml
run: |
  python main.py || echo "Crawler execution completed with status: $?"
```

### 7.3 权限控制

只授予工作流程所需的最小权限，确保安全性。

## 8. 故障排除

### 8.1 工作流程未触发

1. 检查工作流文件是否在正确的目录中（[.github/workflows/](file:///d:/work/github/me/TrendRadar/.github/workflows/)）
2. 确认 YAML 语法是否正确
3. 手动触发工作流程测试

### 8.2 GitHub Pages 部署失败

1. 确保在仓库 Settings > Pages 中将 Source 设置为"GitHub Actions"
2. 检查工作流程权限设置
3. 验证工作流程文件中的部署步骤

### 8.3 通知功能不工作

1. 检查 GitHub Secrets 是否正确配置
2. 确认配置文件中的通知开关是否启用
3. 查看工作流程运行日志排查错误