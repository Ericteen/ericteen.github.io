---
layout:     post
title:      "Github actions 基础"
subtitle:   "Github Actions Basics"
date:       2019-01-01 13:00:00
author:     "Eric"
header-img: "img/post-bg-01.jpg"
---

## 概念

github actions 的两个元素

- the **action** itself
- a **workflow** that uses actions

一个工作流可以包含多个动作。

**动作的类型**：

- 容器动作(container actions)
- JS 动作(JavaScript actions)

Docker 容器动作允许 github actions 和环境共同打包，并且只可以运行在 GitHub 持有的 Linex 环境中运行。

JS 动作可以将 Github actions 从环境中解耦，这样可以获得更快的执行速度但是也需要接收更多的依赖管理的责任。

### workflows

工作流是一个添加到你仓库的自动化流程。他可以由一个或多个工作组成，可以被定时或者通过事件触发。可以被用来在 Github 上创建，测试，打包，发布或者部署.

### Events

事件是指可以触发工作流的特定行为。

### Jobs

工作是由一系列步骤组成的在同一个执行器里面执行的东西。通常情况下，工作是并行执行的。

### Steps

step 是一个可以在工作中执行指令的单独任务。可以是 action 或者 shell 命令。

### Actions

actions 是由多个单独的命令来组成的 steps 来创建一个工作。Actions are the smallest portable building block of a workflow.

### runner

runner 是一个安装了 Github Actions runner 应用的服务器。

## 基础演示

.github/workflows/main.yml

```yml
name: A workflow for my Hello World file
on: push

jobs:
  build:
    name: Hello world action
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - uses: ./action-a
        with:
          MY_NAME: "Mona"
```

- `jobs:` is the base component of a workflow run
- `build:` is the identifier we are attaching to this job
- `name:` 工作的名称，工作流在工作的时候会在GitHub上显示
- `uses: actions/checkout@v1` 使用社区的叫做 `checkout` 动作来允许访问对应的仓库
- `with:` 用来设置输入值

