# Unreal Insights 使用说明

这份文档是给正在和 AI 协作的人看的，不是给 Python 包维护者看的。

`unrealinsights/agent-harness` 提供的是一套 AI 可调用的 Unreal 性能采集与分析能力，让 Agent 能比较稳定地驱动 Unreal trace capture 和 Unreal Insights 导出流程。

## 这个能力能做什么

- 找到或构建与你当前 Unreal Engine 匹配的 `UnrealInsights.exe`
- 启动一次性 trace 采集
- 启动后台持续采集，并在你指定时机停止
- 查看当前后台采集状态
- 对当前活跃 trace 做 best-effort 快照
- 停止当前追踪会话
- 对已有 `.utrace` 导出：
  - `threads`
  - `timers`
  - `timing-events`
  - `timer-stats`
  - `timer-callees`
  - `counters`
  - `counter-values`
- 读取导出的 CSV，并总结热点、等待链和可能瓶颈

## 人类应该怎么调 AI

最有效的方式不是说“帮我分析性能”，而是补齐下面四类信息：

1. 引擎路径
2. 项目路径或目标可执行文件
3. 你关心的是“启动阶段”还是“运行时场景”
4. 你希望 AI 最终给你什么输出

## 推荐提法

### 只分析启动性能

```text
用 D:\code\D5\d5render-ue5_3 这套引擎，
分析 D:\code\D5\FusionEffectBuild 这个项目的启动性能。
先确保匹配版 UnrealInsights 可用，再抓一份启动 trace，
导出 timer-stats，并总结前 20 个热点。
```

### 持续采集，等我说停

```text
用 D:\code\D5\d5render-ue5_3 和 D:\code\D5\FusionEffectBuild
启动一份后台性能采集，不要阻塞，不要立刻退出项目。
先告诉我 trace 输出路径和当前状态。
等我说“停”时，再停止采集、做一次 snapshot，
导出 timer-stats 和 timing-events。
```

### 指定场景分析运行时性能

```text
先启动后台 trace，让我在项目里手动操作某个场景。
我说“现在停”之后，你帮我导出 timer-stats、timing-events，
重点看 GameThread、RenderThread 和任务系统等待时间。
```

### 自定义引擎没有 Insights 可执行文件

```text
这套引擎是源码版，
先帮我在 D:\code\D5\d5render-ue5_3 下面找或构建 UnrealInsights.exe，
然后再用它分析 trace。
```

## AI 最容易接住的关键词

下面这些关键词很适合直接写进需求：

- “确保匹配版 UnrealInsights”
- “后台持续采集”
- “等我说停”
- “做一次 snapshot”
- “导出 timer-stats”
- “导出 timing-events”
- “总结 top hotspots”
- “看 GameThread / RenderThread / WaitForTasks”

## 你可以期待 AI 自动做的事

如果描述足够完整，AI 通常会自动：

- 解析 `.uproject`
- 推导 `UnrealEditor.exe`
- 选择或构建匹配版 `UnrealInsights.exe`
- 生成 `.utrace`
- 导出 CSV
- 从 CSV 中提取热点
- 告诉你这份 trace 更像是：
  - 启动等待
  - 任务调度阻塞
  - GameThread 卡住
  - RenderThread / 渲染 pass 偏重
  - 音频或 streaming 相关问题

## 当前能力边界

当前版本已经支持：

- `capture run`
- `capture start`
- `capture status`
- `capture stop`
- `capture snapshot`
- `backend ensure-insights`

但它还不是完整的 Unreal GUI 自动化：

- 不直接驱动已经打开的 Unreal Insights 图形界面
- 不完整覆盖 live SessionFrontend / TraceStore 的 GUI 工作流
- `capture stop` 当前是对 harness 启动的进程树做 best-effort 停止，不是完整的 SessionServices 远程 `Trace.Stop`
- `capture snapshot` 当前是对活跃 trace 的 best-effort 文件快照，不是完整的 live trace-store 控制路径

## 一句话原则

把“路径 + 场景 + 时机 + 输出物”说清楚，AI 就最容易把这个能力真正调起来。
