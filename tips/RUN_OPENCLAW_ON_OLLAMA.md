

**Title:** MiniMax-M2: The Unsung Hero Keeping My OpenClaw Alive After Anthropic Cut Support

**Body:**

If you run OpenClaw (or any third-party AI agent), today's news might have hit you hard: **Anthropic just cut off third-party support.** No more API access for many self-hosted automation tools. Just like that.

So now what?

For me, the answer was already sitting on my machine: **MiniMax-M2:cloud** from Ollama. And honestly? I'm impressed.

## The Backstory

I woke up this morning to find my OpenClaw instance had stopped working. Anthropic had pulled the plug on third-party support. My daily flight checks, WhatsApp automations, kanban health checks — all dead in the water.

But here's the thing: I'd already been running MiniMax-M2 as my primary model for a while. Why? Because it's:
- **Reliable** (Ollama's cloud infrastructure — always available)
- **Capable** (beats Claude Sonnet 4 on several agentic benchmarks)
- **Predictable pricing** (Ollama Pro/Max subscription, not pay-per-token)

Today proved why that matters.

## What is MiniMax-M2?

It's a 32B parameter model from MiniMax, optimized for coding and agentic workflows. Here's what makes it special:

**🏆 Benchmarks that speak for themselves:**

| Benchmark | MiniMax-M2 | Claude Sonnet 4 | Claude Sonnet 4.5 |
|-----------|------------|----------------|-------------------|
| SWE-bench Verified | 69.4 | 72.7 | 77.2 |
| Terminal-Bench | **46.3** | 36.4 | 50 |
| BrowseComp | **44** | 12.2 | 19.6 |
| GAIA (text only) | **75.7** | 68.3 | 71.2 |
| AA Intelligence | **61** | 57 | 63 |

It actually **beats** Claude Sonnet 4 on several agentic benchmarks, including Terminal-Bench and BrowseComp. That's the kind of model you want running your automations.

## How to Set Up Ollama

**1. Install Ollama:**
```bash
# Windows (winget)
winget install Ollama.Ollama

# Or download from https://ollama.com
```

**2. Sign up for Ollama Pro** (~$20/month) or Max ($100/month)

**3. Pull the model:**
```bash
ollama pull minimax-m2:cloud
```

**4. Verify it's running:**
```bash
ollama list
```

Ollama runs on `http://127.0.0.1:11434` by default — perfect for OpenClaw integration.

## OpenClaw Config

Add this to your `openclaw.json` (edit the paths to match your setup):

```json5
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://127.0.0.1:11434/v1",
        apiKey: "ollama-local",
        models: [
          {
            id: "minimax-m2:cloud",
            name: "minimax-m2:cloud",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 }
          }
        ]
      }
    }
  },
  agents: {
    defaults: {
      model: {
        primary: "ollama/minimax-m2:cloud",
        fallbacks: [
          "ollama/qwen3-coder:480b-cloud",
          "ollama/gpt-oss:120b-cloud",
          "anthropic/claude-sonnet-4-6",
          "ollama/glm-4.6:cloud"
        ]
      },
      models: {
        "ollama/minimax-m2:cloud": { alias: "MiniMax-M2" }
      }
    }
  }
}
```

Key steps:
1. Add `minimax-m2:cloud` to your `models.providers.ollama.models` array
2. Set it as your **primary** model
3. Keep fallbacks for redundancy
4. Restart OpenClaw gateway

## My Real-World Test Results

I've been running MiniMax-M2 for a while now. Here's what it's handled:

✅ **Daily cron jobs** — Flight price monitoring, WhatsApp messages, kanban health checks — all running reliably

✅ **Codebase analysis** — Used Repomix to analyze new GitHub repos, understand architecture, summarize code

✅ **Complex automation** — Built multi-step browser automation scripts (Playwright + LLM)

**Verdict:** It's not quite Claude-level for deep reasoning, but for 90% of automation tasks? It's indistinguishable. And more importantly — **it's always available.**

## The Bigger Picture

This is what self-hosted AI should be about: **owning your infrastructure.** When you rely on a single API provider, you're one policy change away from disaster. MiniMax-M2 via Ollama gives you a reliable, always-available backbone that doesn't have unpredictable rate limits and won't suddenly cut you off overnight.

The future of AI automation isn't about chasing the latest GPT — it's about sustainable, independent backbones. MiniMax-M2 might just be yours.

---

**Links:**
- MiniMax-M2: https://ollama.com/library/minimax-m2
- Ollama: https://ollama.com
- Ollama Pricing: https://ollama.com/pricing

---