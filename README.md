# Ananta — अनन्त

*Endless counsel.* A free chatbot that thinks alongside you with the combined wisdom of the
Bhagavad Gita, Krishna's counsel, the Mahabharata, the Upanishads, Homer's Iliad and Odyssey,
the Greek tragedians, Socrates, Plato, Aristotle, the Stoics — and the rest of the old voices.

**Live:** https://vaibhavkumar.is-a.dev/Ananta/

## How it works

```
index.html (GitHub Pages)  ──POST /chat──▶  ananta-brain (Cloudflare Worker)
  threads in localStorage                     rate limits in KV, cascades models:
                                              Groq 70b* → Gemini flash* → Workers AI 70b → Workers AI 8b
                                                                          (* only if key set — optional)
```

- **Zero-key by default** — the Workers AI steps use Cloudflare's free tier via the `[ai]`
  binding, so the service runs with no LLM account at all. Adding `GROQ_API_KEY` /
  `GEMINI_API_KEY` secrets upgrades quality; they're tried first when present.
- **Threads with context** — conversations live in the browser's localStorage; the last 20
  messages are sent as context with each question.
- **Free-tier limits** — 30 user messages per thread, 10 saved threads (client-side);
  60 requests/IP/day and 300 requests/day globally (worker-side, KV counters on salted IP
  hashes — no raw IPs stored, keys expire in 2 days).

## Files

- `index.html` — the whole front-end: single file, no build step, no CDN. Thread sidebar,
  minimal markdown rendering (bold/italic/blockquote), mobile drawer.
- `worker/worker.js` — the `ananta-brain` Cloudflare Worker (vaibhavpro9210 account):
  holds the system prompt, validates and truncates history, rate-limits, cascades models.
- `worker/wrangler.toml` — worker config; KV namespace `ANANTA_LIMITS`.

## Secrets (wrangler, never in repo)

- `IP_SALT` — required, random string for rate-limit hashes (set).
- `GROQ_API_KEY` — optional, console.groq.com.
- `GEMINI_API_KEY` — optional, aistudio.google.com.

Set with `cd worker && npx wrangler secret put GROQ_API_KEY`.

## Dev hooks

- `index.html#shot=welcome` — welcome screen (default when no threads).
- `index.html#shot=chat` — seeds an unsaved demo conversation for screenshots.
- `index.html#shot=card` — opens the quote-card preview modal on a demo quote.
- POST `{"history":[...], "debug":true}` to the worker to get the model-cascade errors back.

## Deploy

- Site: push `main` and `main:gh-pages` (Pages serves from `gh-pages`).
- Worker: `cd worker && npx wrangler deploy`.
