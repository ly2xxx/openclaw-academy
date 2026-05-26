# OpenClaw Study Guide

A self-directed, file-by-file study plan for learning TypeScript/Node.js through
the OpenClaw codebase version "2026.5.21", structured for a long weekend with an interview goal in mind.

## Schedule

| Day | Topic | Goal |
|-----|-------|------|
| [Day 1](day1-typescript-foundations.md) | TypeScript & project setup | Read the build chain; understand types, modules, strict mode |
| [Day 2](day2-config-system.md) | Config & schema system | Understand Zod validation, config loading, JSON5, env substitution |
| [Day 3](day3-gateway-server.md) | Gateway server & WebSocket | HTTP server, WS protocol, RPC dispatch, auth |
| [Day 4](day4-agent-system.md) | Agent & LLM system | Conversation turns, model fallback, session persistence |
| [Day 5](day5-plugin-extension-system.md) | Plugin & extension architecture | Channel contract, plugin SDK, lifecycle hooks |
| [Day 6](day6-cron-and-ui.md) | Cron system & UI | Scheduled jobs, Lit web components, WS protocol |
| [Day 7](day7-interview-prep.md) | Review & interview prep | Architecture questions, patterns, talking points |

## How to use this guide

Each day file contains:
- **Concept brief** — the TypeScript/Node.js idea introduced that day
- **File sequence** — exact files to open, in order, with what to focus on
- **Key questions** — answer these in your own words to test understanding
- **Exercises** — small experiments you can do in the running code

## Prerequisites

You should be able to run:
```powershell
cd C:\code\openclaw
pnpm install          # install dependencies
pnpm run build        # compile TypeScript → dist/
```

You should be able to run this single test:
```powershell
pnpm test src/plugins/runtime/index.test.ts
node --import tsx --test src/index.test2.ts
```

**Markdown Preview Mermaid Support VS code extension is required for rendering mermaid diagrams in this guide.**

Keep VS Code open alongside — hover over any type to see its definition.
