# LLM Code Review — Multi‑Model

Tiny CLI wrapper around `git diff` + the [`llm`](https://llm.datasette.io) tool to get fast, actionable PR reviews from multiple models at once.

---

## 🚀 One‑liner to run

```bash
review --models "gpt-4o,gemini-2.0-flash,anthropic/claude-3-7-sonnet-latest" origin/master...HEAD
```

> Swap `origin/master` for `origin/main` if your default branch is `main`.

---

## 1) Prereqs (once)

* **git** and **bash** (any Linux/macOS shell is fine)
* **llm** CLI (Python‑based)
* Optional: **bat** for nice Markdown output

**Quick install (recommended in a venv):**

```bash
python3 -m venv ~/.llm-env
source ~/.llm-env/bin/activate
pip install --upgrade pip llm
# (Optional) On Debian/Ubuntu for pretty output
sudo apt install -y bat && sudo ln -s /usr/bin/batcat /usr/local/bin/bat || true
```

Place the `review` script somewhere on your `PATH`, e.g. `~/bin/review`, and make it executable:

```bash
chmod +x ~/bin/review
```

---

## 2) Add API keys (once)

The script can call **OpenAI**, **Gemini**, and **Claude** via `llm`. Set keys like this:

```bash
llm keys set openai       # for OpenAI (e.g., gpt-4o)
llm keys set google       # for Gemini (e.g., gemini-2.0-flash)
llm keys set anthropic    # for Claude (e.g., anthropic/claude-3-7-sonnet-latest)
```

**Quick test:**

```bash
llm -m gpt-4o "hello"
llm -m gemini-2.0-flash "hello"
llm -m anthropic/claude-3-7-sonnet-latest "hello"
```

---

## 3) Usage

**Compare current branch to default branch:**

```bash
review --models "gpt-4o,gemini-2.0-flash,anthropic/claude-3-7-sonnet-latest" origin/main...HEAD
```

**Only staged changes:**

```bash
review --cached
```

**Add extra context (PR description, ticket, etc.):**

```bash
git log -1 --pretty=%B | review --context -
```

**Set models once per shell:**

```bash
export LLM_MODELS="gpt-4o,gemini-2.0-flash,anthropic/claude-3-7-sonnet-latest"
review origin/main...HEAD
```

---

## 4) What you get

* A short **readiness rating** (★☆☆ / ★★☆ / ★★★)
* **Blocking items** to fix before human review
* **Non‑blocking suggestions**
* **Test plan gaps**

---

## Troubleshooting

* **No output?** Probably no diff. Try `review --cached` or confirm your range (e.g., `origin/main...HEAD`).
* **Auth error?** Re‑run the key setup commands above.
* **Command not found?** Ensure `llm` and `review` are on your `PATH`, or activate your venv.
