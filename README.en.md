# OpenClaw Context Budget 🎛️

<div align="center">
  <a href="README.md">🇨🇳 中文</a> | <strong>🌐 English</strong>
</div>

<p align="center">
  <img src="./assets/readme/hero.svg" width="100%" alt="OpenClaw Context Budget — set each enabled model's context window to 60% of the vendor spec; flow: detect, decide, apply">
</p>

> **Context Check / Context Optimization** — the core goal is **context optimization**: turn "how large should this model's context window be?" into **one detection pass + one digit of confirmation**.
> It detects the models currently **enabled** in your installation (read dynamically at runtime — nothing hard-coded), fetches each model's latest published context length from its **vendor's official source**, proposes a window at **60%** of that value, and only writes the config after you confirm.

![license](https://img.shields.io/badge/license-MIT-green)
[![ClawHub](https://img.shields.io/badge/ClawHub-xiaoyaoclaw--context--budget-blue)](https://clawhub.ai/dtsola)

> 📌 Model names and numbers shown below are **examples**; everything is read dynamically at runtime.

---

## Why you need it

Context windows vary a lot between vendors (200K, 1M, …). When a model has no declared window, OpenClaw falls back to a **built-in default** (the source constant `DEFAULT_CONTEXT_TOKENS = 200000`). That causes three classes of problems:

- **Window underestimated** → context gets **compacted too early**, long tasks get cut off, compaction stalls
- **Window overestimated** → the model **loses focus** in overly long context, answer quality drops
- **No provenance** → the number in the config has no traceable source

This skill applies one fixed best practice: **effective window = vendor spec × 60%** (headroom to reduce attention dilution).

---

## Install

```bash
clawhub install xiaoyaoclaw-context-budget
```

(Or drop the skill folder into `<workspace>/skills/` or `~/.openclaw/skills/`.)

---

## Usage

| How | Examples |
|---|---|
| **Just say it (recommended)** | "context check", "context optimization", "optimize context", "check context window", "上下文检查", "上下文优化", "检查上下文" |
| Slash command | `/xiaoyaoclaw-context-budget` |
| When adding/switching a model | "I switched models, configure the window", "I added a new model, set its window" |
| Sub-commands | "undo last context adjustment" / "context check full" |

---

## Flow: Detect → Decide → Apply

**① Detect (automatic — you provide nothing)**
The skill enumerates the models in use, fetches each vendor's latest published window from official sources, compares against your config, and computes the suggested value at 60%.

- No differences → `✅ Check complete: all model windows look right. No changes needed.` **Done.**
- Differences → decision card ↓

**② Decide (you return a single digit)**

```
🎛️ 1 item can be adjusted
· <provider>/<model> (in use) ｜ vendor spec 1,000,000 ｜ current 200,000 → suggest 600,000
· <provider>/<model> (in use) ｜ vendor spec 1,000,000 ｜ current 600,000 ✅ no change
*suggested = vendor window × 60% (headroom against attention dilution)* ｜ source: <name> (fetched at)

Reply 1 to adopt ｜ 2 to keep current ｜ 3 for details
```

**③ Apply (automatic — 3-line receipt)**

```
✅ 1 item updated: <provider>/<model> 200,000 → 600,000
Verified (window is live)
To revert, reply "undo last context adjustment"
```

Snapshotting, writing, reload, and verification all happen behind the scenes.

---

## Boundaries

| Does | Does not |
|---|---|
| Set the context window of **enabled** models (at 60%) | `maxTokens` (output cap) or any other parameter |
| Read config, fetch official specs, write window values | Compaction thresholds (left at system default) |
| Keep a pre-change snapshot for rollback | Bulk-configure models that are not in use |
| Ask you when an official source can't be fetched | Write anything silently (always confirmed first) |
| — | **No change history, no audit trail** |

---

## Optional suggested settings (opt-in, never enforced)

### 1. Periodic self-check (weekly recommended)

Let the skill run "detect + compare" on a schedule. It stays **completely silent when nothing changed**, and only pings you when a vendor actually updates a window.

| Cadence | Best for |
|---|---|
| Mondays 09:00 (recommended) | Regular use |
| Every two weeks | Stable model set |
| Never | Manual "context check" only |

OpenClaw cron example:

```jsonc
{
  "schedule": { "kind": "cron", "expr": "0 9 * * 1", "tz": "Asia/Shanghai" },
  "payload": { "kind": "agentTurn", "message": "Run a context check; if nothing differs, reply NO_REPLY (stay silent)." },
  "sessionTarget": "isolated",
  "delivery": { "mode": "announce" }
}
```

### 2. Event trigger (optional enhancement)

When a new model appears in your config, the skill can proactively ask whether to configure its window. Requires extra change detection; the main flow works without it.

### 3. Adjustable ratio (60% by default)

Want a more aggressive or more conservative window? Just say so in chat:
> "context check, use 50%" / "set this model at 80%"

Per-model values are supported; the ratio is not hard-coded.

---

## Safety and rollback

- A **pre-change snapshot** is kept, so rollback is one command away (single overwrite file, no naming, no trail)
- Config is written only after your confirmation: **there is no silent-write path**
- Writes go through `config.patch` (**never `config.apply`**), touching only window fields

---

## FAQ

**How long does a check take?** About 30–60 seconds, mostly spent fetching official pages.

**What if the official source can't be fetched?** The skill asks you for a link, or lets you skip — it never silently keeps a stale value.

**Will it change my other settings?** No. Only model window fields.

**Why 60%?** Headroom for system prompts, tool output, and multi-turn context, to keep the model from losing focus in very long contexts. It is **attention protection**, not mechanical overflow headroom.

**Why not configure compaction thresholds too?** Because the threshold is a single **global** value that cannot be set per model — leaving it at the system default is the most robust choice.

---

## Ecosystem

One of the **xiaoyaoclaw** series (home / content / progress / knowledge / audit / input / check …).

- GitHub: <https://github.com/dtsola/xiaoyaoclaw-context-budget>
- ClawHub: <https://clawhub.ai/dtsola>
- 小遥Claw — "Put an AI assistant on your own computer": <https://www.yuque.com/dtsola/igp1aa/adcicbai2zlem0bz>

## License

MIT © 2026 dtsola
