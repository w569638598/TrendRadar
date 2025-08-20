# GitHub Actions YAML 工作流文件官方 API 文档

## 官方文档链接

要获取最权威和详细的 GitHub Actions YAML 工作流文件语法信息，请参考以下官方文档：

1. [Workflow syntax for GitHub Actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions) - 这是 GitHub 官方提供的关于工作流语法的完整文档

2. [Events that trigger workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows) - 关于触发工作流的事件的详细说明

3. [GitHub Actions 入门指南](https://docs.github.com/en/actions/learn-github-actions)

4. [GitHub Actions 核心功能](https://docs.github.com/en/actions/learn-github-actions/essential-features-of-github-actions)

## 重要概念快速参考

虽然建议您查阅官方文档获取最详细的信息，但以下是一些关键概念的快速参考：

### jobs.<job_id>.runs-on

定义作业运行所需的运行器类型。常用值包括：

- `ubuntu-latest` - GitHub 提供的最新 Ubuntu Linux 运行器
- `windows-latest` - GitHub 提供的最新 Windows 运行器
- `macos-latest` - GitHub 提供的最新 macOS 运行器
- `self-hosted` - 自托管运行器

### jobs.<job_id>.steps

定义作业中要运行的步骤序列。每个步骤可以是：

1. 一个要运行的 shell 脚本
2. 一个要运行的 action（操作）

步骤按顺序执行，如果任何步骤失败，工作流将停止。

### jobs.<job_id>.steps[*].uses

选择要运行的 action。action 可以是：

- GitHub 提供的官方 action（如 `actions/checkout@v4`）
- 发布在 GitHub Marketplace 上的 action
- 您自己仓库中的 action（使用 `./` 路径引用）
- Docker Hub 上的 Docker 容器 action

### jobs.<job_id>.steps[*].with

为 action 指定输入参数。

## 更多信息

要了解所有可用的配置选项、语法细节和最佳实践，请访问 [GitHub Actions 官方文档](https://docs.github.com/en/actions).

## 1. 概述

GitHub Actions 是 GitHub 提供的持续集成和持续部署 (CI/CD) 平台，允许您在 GitHub 仓库中自动化构建、测试和部署工作流程。工作流通过存储在 `.github/workflows` 目录中的 YAML 文件进行配置。

## 2. 工作流基本结构

工作流文件必须使用 `.yml` 或 `.yaml` 扩展名，并放置在仓库的 `.github/workflows` 目录中。

一个基本的工作流文件结构如下：

```yaml
name: Workflow Name
on:
  # 触发条件
jobs:
  job1:
    # 作业配置
```

## 3. 核心语法元素

### 3.1 name

定义工作流的名称，GitHub 会在仓库的 "Actions" 标签页下显示工作流名称。

```yaml
name: CI/CD Pipeline
```

如果省略，GitHub 会显示工作流文件相对于仓库根目录的路径。

### 3.2 run-name

定义从工作流生成的工作流运行的名称。

```yaml
run-name: Deploy to ${{ inputs.deploy_target }} by @${{ github.actor }}
```

### 3.3 on

定义触发工作流运行的事件。可以定义单个或多个事件，也可以设置时间计划。

#### 3.3.1 单事件触发

```yaml
on: push
```

#### 3.3.2 多事件触发

```yaml
on: [push, fork]
```

#### 3.3.3 活动类型

某些事件具有活动类型，可以更精确地控制工作流的触发时机。

```yaml
on:
  label:
    types:
      - created
```

#### 3.3.4 过滤器

某些事件具有过滤器，可以进一步控制工作流的触发条件。

```yaml
on:
  push:
    branches:
      - main
      - 'releases/**'
```

### 3.4 jobs

工作流运行包括一项或多项作业 (jobs)，这些作业可以并行运行或按顺序运行。

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
```

## 4. 作业 (Jobs) 配置

### 4.1 runs-on

指定作业运行所需的运行器类型。

```yaml
runs-on: ubuntu-latest
```

常见的运行器类型：
- `ubuntu-latest`
- `windows-latest`
- `macos-latest`

### 4.2 steps

定义作业中的步骤序列。每个步骤可以是一个 shell 脚本或一个操作 (action)。

```yaml
steps:
  - name: Checkout repository
    uses: actions/checkout@v4
    
  - name: Set up Node.js
    uses: actions/setup-node@v4
    with:
      node-version: '18'
      
  - name: Run tests
    run: npm test
```

### 4.3 uses

指定要使用的操作 (action)。操作可以是：
- GitHub 提供的官方操作（如 `actions/checkout@v4`）
- 社区提供的操作
- 自定义操作

### 4.4 with

为操作指定输入参数。

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '18'
    registry-url: 'https://registry.npmjs.org'
```

### 4.5 needs

指定作业之间的依赖关系，控制作业的运行顺序。

```yaml
jobs:
  job1:
  job2:
    needs: job1
  job3:
    needs: [job1, job2]
```

### 4.6 permissions

定义授予 GITHUB_TOKEN 的权限。

```yaml
permissions:
  contents: read
  issues: write
```

### 4.7 strategy

定义作业的策略，例如矩阵策略用于在多个操作系统或版本上运行作业。

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node-version: [16, 18, 20]
```

## 5. 环境变量和上下文

### 5.1 环境变量

使用 `env` 定义环境变量：

```yaml
env:
  NODE_ENV: production
```

### 5.2 GitHub 上下文

使用 `${{ github }}` 访问 GitHub 提供的上下文信息：

```yaml
run-name: Deploy by @${{ github.actor }}
```

## 6. 表达式和条件

### 6.1 表达式

使用 `${{ }}` 语法编写表达式：

```yaml
${{ github.event_name == 'push' }}
```

### 6.2 条件

使用 `if` 关键字指定条件：

```yaml
- name: Upload coverage to Codecov
  if: ${{ github.event_name == 'push' }}
  run: bash <(curl -s https://codecov.io/bash)
```

## 7. 实际应用示例

以下是一个完整的 GitHub Actions 工作流示例：

```
name: CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4
      
    - name: Set up Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run tests
      run: npm test
      
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Deploy to production
      run: echo "Deploying to production"
```

## 8. 最佳实践

1. **使用稳定版本的操作**：指定操作的确切版本，如 `actions/checkout@v4`，而不是 `actions/checkout@main`

2. **合理设置权限**：只授予工作流所需的最小权限

3. **使用矩阵策略**：在多个环境上测试代码

4. **缓存依赖项**：使用缓存操作加速工作流运行

5. **错误处理**：添加适当的错误处理和恢复机制

## 9. 参考文档

- [GitHub Actions 官方文档](https://docs.github.com/en/actions)
- [Workflow syntax for GitHub Actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Events that trigger workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)