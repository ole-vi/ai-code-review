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

---

## 🔧 Codex Setup (Automated)

If you’re running this inside **Codex**, use the included **`codex-startup.sh`** script in this repo (from the project root).

This script will:
1. Create a Python venv and install `llm` (plus Gemini/Claude plugins) and the `review` CLI.
2. Persist your API keys for both the `llm` tool and your shell (adds exports to `~/.bashrc`).
3. Generate helper scripts under `tools/`:
   - `tools/pre_pr_review.sh` – runs the multi-model review
   - `tools/auto_fix_from_review.sh` – asks an LLM for a minimal patch based on the review

### Required Codex Secrets (exact names)

Set these three secrets in Codex **exactly** as named:

| Secret Name         | Value (example)           | Used For                                   |
|---------------------|---------------------------|--------------------------------------------|
| `OPENAI_API_KEY`    | `sk-...`                  | OpenAI models (e.g., `gpt-4o`)             |
| `ANTHROPIC_API_KEY` | `sk-ant-...`              | Claude models (e.g., `claude-3-5-sonnet-*`)|
| `GOOGLE_API_KEY`    | `AIza...`                 | Gemini models (e.g., `gemini-2.0-flash`)   |

> The startup script will run `llm keys set` for each provider and export these in the shell (with fallbacks in `~/.bashrc`).

### Typical Codex Pre-PR Flow

```bash
# Run an initial review
bash tools/pre_pr_review.sh

# (Optional) Try an auto-fix pass based on .ai-review.md
bash tools/auto_fix_from_review.sh .ai-review.md || true

# Run a second review after fixes
bash tools/pre_pr_review.sh

# Open a PR (uses repo’s default branch as base)
gh pr create --fill \
  --body-file .ai-review.md \
  --title "chore: codex multi-model review" \
  --base "$(git remote show origin | sed -n '/HEAD branch/s/.*: //p')"
```

## 🗣 Using Codex with This Setup

With the Codex integration and `codex-startup.sh` in place, you can work entirely in **plain English**.

### Example
You could type to Codex:
> "Refactor `src/app/example.component.ts` to use async/await instead of promises and run a multi-model review before opening a PR."

### What Happens Behind the Scenes
When you make a request like this, Codex will:
1. **Interpret your instruction** and make the requested code change(s) in your repo.
2. **Run `tools/pre_pr_review.sh`** – this triggers the multi-model review (`review` CLI) with OpenAI, Gemini, and Claude using the API keys you set in Codex secrets.
3. **Save the AI review output** to `.ai-review.md`.
4. **Optionally run `tools/auto_fix_from_review.sh`** – this will feed `.ai-review.md` to an LLM to propose a minimal patch and apply it.
5. **Run a second review** so you can see if the fixes resolved the earlier feedback.
6. **Open a PR** using `gh pr create` with the `.ai-review.md` contents as the PR body.

Because `codex-startup.sh` persists your keys and sets everything up automatically, this works in every Codex session without re-configuring.

You can mix and match tasks — ask Codex to:
- Make a specific change and run the review
- Just run a review on your current branch
- Apply auto-fixes from the last review
- Open a PR after review passes

All using plain language instructions.
