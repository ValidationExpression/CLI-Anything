# Unreal Insights Human Guide

This document is for people collaborating with AI, not for Python package maintainers.

`unrealinsights/agent-harness` gives AI agents a reliable way to drive Unreal
trace capture and Unreal Insights analysis workflows from the terminal.

## What This Capability Can Do

- find or build an `UnrealInsights.exe` matching your current Unreal Engine
- start a one-shot trace capture
- start a background capture and let it run until you decide to stop
- inspect the current background capture status
- create a best-effort snapshot copy of the active trace
- stop the tracked capture session
- export an existing `.utrace` into:
  - `threads`
  - `timers`
  - `timing-events`
  - `timer-stats`
  - `timer-callees`
  - `counters`
  - `counter-values`
- read exported CSV files and summarize hotspots, waits, and likely bottlenecks

## How Humans Should Ask AI To Use It

The most effective prompt is not “analyze performance,” but a prompt that gives:

1. engine path
2. project path or target executable
3. whether you care about startup or runtime behavior
4. what output you want back

## Good Prompt Patterns

### Startup performance

```text
Use D:\code\D5\d5render-ue5_3 to analyze startup performance for
D:\code\D5\FusionEffectBuild.
First ensure a matching UnrealInsights.exe exists, then capture a startup trace,
export timer-stats, and summarize the top 20 hotspots.
```

### Long-running capture until I say stop

```text
Use D:\code\D5\d5render-ue5_3 and D:\code\D5\FusionEffectBuild
to start a background performance capture.
Do not block and do not exit the project immediately.
Tell me the trace path and current status first.
When I say "stop", stop the capture, make a snapshot, export timer-stats and
timing-events, then summarize the results.
```

### Runtime scene analysis

```text
Start a background trace for this project, let me interact with the scene
manually, and wait.
When I say "stop now", export timer-stats and timing-events and focus on
GameThread, RenderThread, and task-system waits.
```

### Source-built engine without UnrealInsights.exe

```text
This is a source-built engine.
First find or build UnrealInsights.exe under D:\code\D5\d5render-ue5_3,
then use it to analyze the trace.
```

## Keywords AI Handles Well

These phrases are especially useful:

- “ensure matching UnrealInsights”
- “background continuous capture”
- “wait until I say stop”
- “make a snapshot”
- “export timer-stats”
- “export timing-events”
- “summarize top hotspots”
- “look at GameThread / RenderThread / WaitForTasks”

## What AI Can Usually Do Automatically

If you give enough context, AI can usually:

- resolve the `.uproject`
- infer `UnrealEditor.exe`
- choose or build a matching `UnrealInsights.exe`
- generate `.utrace`
- export CSV reports
- extract hotspots from CSV
- tell you whether the trace looks more like:
  - startup waiting
  - task-scheduler blocking
  - GameThread stalls
  - RenderThread / render-pass pressure
  - audio or streaming side effects

## Current Scope

The current version supports:

- `capture run`
- `capture start`
- `capture status`
- `capture stop`
- `capture snapshot`
- `backend ensure-insights`

It is not full Unreal GUI automation:

- it does not drive an already-open Unreal Insights GUI panel directly
- it does not fully manage live SessionFrontend / TraceStore UI workflows
- `capture stop` is a best-effort stop of the harness-launched process tree, not
  full SessionServices remote `Trace.Stop`
- `capture snapshot` is a best-effort filesystem snapshot of the active trace,
  not a full live trace-store control path

## One-Sentence Rule

Give AI the path, the scene, the timing, and the desired output, and it will be
able to use this capability much more effectively.
