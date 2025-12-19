# Jenkins Pipeline Architecture / Jenkins Pipeline 架构

## English

### Does this project include the pipeline implementation?

**Short Answer:** No, the Jenkins core repository does not contain the pipeline implementation itself. Pipeline functionality is provided through separate plugins.

### Architecture Overview

This repository (`jenkinsci/jenkins`) contains the **Jenkins core** - the foundational server and plugin infrastructure. The core provides:

- Basic Jenkins functionality (job management, build execution, user management, etc.)
- Plugin extension points and APIs
- Infrastructure for plugins to integrate with Jenkins

### Pipeline Implementation Location

The Pipeline functionality is implemented as a **set of plugins**, not in the core repository. The main pipeline plugins include:

1. **workflow-aggregator** - The main pipeline plugin that aggregates all pipeline functionality
2. **workflow-job** - Defines the Pipeline job type
3. **workflow-cps** - Provides the Groovy DSL for Pipeline scripts
4. **workflow-basic-steps** - Basic Pipeline steps
5. **workflow-durable-task-step** - Durable task execution
6. **pipeline-model-definition** - Declarative Pipeline syntax
7. **pipeline-stage-step** - Stage step implementation
8. **And many more...**

### Pipeline Plugin Repositories

The Pipeline plugins are developed in separate GitHub repositories under the `jenkinsci` organization:

- [workflow-aggregator-plugin](https://github.com/jenkinsci/workflow-aggregator-plugin)
- [workflow-job-plugin](https://github.com/jenkinsci/workflow-job-plugin)
- [workflow-cps-plugin](https://github.com/jenkinsci/workflow-cps-plugin)
- [pipeline-model-definition-plugin](https://github.com/jenkinsci/pipeline-model-definition-plugin)
- [And many others](https://github.com/jenkinsci?q=workflow+OR+pipeline&type=repositories)

### How Core Supports Pipelines

While the core doesn't implement Pipeline, it provides the foundation:

1. **Extension Points** - APIs that Pipeline plugins use to integrate with Jenkins
2. **Job/Build Infrastructure** - Base classes like `Job`, `Run`, `Queue` that Pipeline extends
3. **Plugin System** - Mechanism for loading and managing Pipeline plugins
4. **API Compatibility** - Stable APIs that Pipeline plugins depend on

### References in Core

You can find references to Pipeline in the core repository:

- `core/src/main/resources/jenkins/install/platform-plugins.json` - Lists recommended pipeline plugins
- Various deprecation notices and compatibility notes mentioning Pipeline
- Comments in code discussing Pipeline-specific behavior

### For More Information

- **Pipeline Documentation**: https://www.jenkins.io/doc/book/pipeline/
- **Pipeline Plugins**: https://plugins.jenkins.io/ (search for "pipeline" or "workflow")
- **Plugin Development Guide**: https://www.jenkins.io/doc/developer/plugin-development/

---

## 中文 (Chinese)

### 该项目中包含 Pipeline 的实现部分吗？

**简短回答：** 不包含。Jenkins 核心仓库本身不包含 Pipeline 的实现。Pipeline 功能是通过独立的插件提供的。

### 架构概述

本仓库 (`jenkinsci/jenkins`) 包含 **Jenkins 核心** - 即基础服务器和插件基础设施。核心提供：

- Jenkins 基础功能（任务管理、构建执行、用户管理等）
- 插件扩展点和 API
- 插件与 Jenkins 集成的基础设施

### Pipeline 实现位置

Pipeline 功能是作为**一组插件**实现的，而不是在核心仓库中。主要的 Pipeline 插件包括：

1. **workflow-aggregator** - 聚合所有 Pipeline 功能的主要插件
2. **workflow-job** - 定义 Pipeline 任务类型
3. **workflow-cps** - 提供 Pipeline 脚本的 Groovy DSL
4. **workflow-basic-steps** - 基本的 Pipeline 步骤
5. **workflow-durable-task-step** - 持久化任务执行
6. **pipeline-model-definition** - 声明式 Pipeline 语法
7. **pipeline-stage-step** - Stage 步骤实现
8. **还有许多其他插件...**

### Pipeline 插件仓库

Pipeline 插件在 `jenkinsci` 组织下的独立 GitHub 仓库中开发：

- [workflow-aggregator-plugin](https://github.com/jenkinsci/workflow-aggregator-plugin)
- [workflow-job-plugin](https://github.com/jenkinsci/workflow-job-plugin)
- [workflow-cps-plugin](https://github.com/jenkinsci/workflow-cps-plugin)
- [pipeline-model-definition-plugin](https://github.com/jenkinsci/pipeline-model-definition-plugin)
- [更多插件](https://github.com/jenkinsci?q=workflow+OR+pipeline&type=repositories)

### 核心如何支持 Pipeline

虽然核心不实现 Pipeline，但它提供了基础：

1. **扩展点** - Pipeline 插件用于与 Jenkins 集成的 API
2. **任务/构建基础设施** - 基础类如 `Job`、`Run`、`Queue`，Pipeline 对其进行扩展
3. **插件系统** - 加载和管理 Pipeline 插件的机制
4. **API 兼容性** - Pipeline 插件依赖的稳定 API

### 核心中的 Pipeline 引用

您可以在核心仓库中找到对 Pipeline 的引用：

- `core/src/main/resources/jenkins/install/platform-plugins.json` - 列出推荐的 Pipeline 插件
- 各种提及 Pipeline 的弃用通知和兼容性说明
- 代码中讨论 Pipeline 特定行为的注释

### 更多信息

- **Pipeline 文档**: https://www.jenkins.io/doc/book/pipeline/
- **Pipeline 插件**: https://plugins.jenkins.io/ (搜索 "pipeline" 或 "workflow")
- **插件开发指南**: https://www.jenkins.io/doc/developer/plugin-development/
