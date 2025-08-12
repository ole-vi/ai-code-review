# review — LLM-powered code review from your terminal

A tiny CLI that pipes `git diff` into your preferred LLM and returns a concise, actionable review. It supports one or multiple models (to compare outputs side-by-side), adds a simple 3-star **readiness** rating, and optionally pretty-prints with `bat`.

> **What it does:**
> Runs `git diff …` → feeds the diff + a focused prompt into `llm` → prints a structured review with:
>
> * **Readiness:** ★☆☆ / ★★☆ / ★★★ with a one-line reason
> * **Blocking items** (must fix before human review)
> * **Non-blocking suggestions**
> * **Test-plan gaps**

---

## Features

* Works with any **`git diff` syntax** (unstaged, staged, branch vs branch, three-dot, file filters, etc.)
* **Multi-model comparisons** via `--models "gpt-4o,gemini-2.0-flash"`
* **Readable output** using `bat` if available
* **Automatic diff context scaling** if the diff looks too big for model context
* **Extra context** via `--context "…"`, `--context -` (stdin), or repeat the flag

---

## Requirements

* **git**
* **llm** (Simon Willison’s CLI): [https://llm.datasette.io/](https://llm.datasette.io/)
* **bat** *(optional but nice)*

### Install `llm` (recommended: virtualenv)

```bash
python3 -m venv ~/.llm-env
source ~/.llm-env/bin/activate
pip install --upgrade pip llm
```

List available models:

```bash
llm models
```

### Configure providers

**OpenAI** (built-in):

```bash
llm keys set openai
# Paste your OpenAI API key when prompted
# Or: export OPENAI_API_KEY=sk-...
```

**Gemini** (via plugin):

```bash
llm install llm-gemini
llm keys set google
# Paste your Google API key (or export GOOGLE_API_KEY=...)
```

> If `pip` is “externally managed” on your distro, use a venv (above) or `pipx install llm`.

---

## Install the script

1. Save the script as `~/bin/review`:

```bash
mkdir -p ~/bin
# copy the script content into ~/bin/review
chmod +x ~/bin/review
```

2. Ensure `~/bin` is on your PATH (add to `~/.bashrc` or `~/.zshrc`):

```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

3. Make sure `llm` is on your PATH (activate your venv if needed):

```bash
source ~/.llm-env/bin/activate
which llm
```

4. (Optional) Install `bat`:

* Debian/Ubuntu: `sudo apt install bat`
  (Binary may be `batcat`; symlink for convenience:)

  ```bash
  sudo ln -s /usr/bin/batcat /usr/local/bin/bat
  ```
* macOS: `brew install bat`

---

## Usage

```
review [--verbose] [--models LIST] [--context TEXT|-] [git-diff-arguments...]
```

### Common flows

Review **unstaged** changes:

```bash
review
```

Review **staged** changes:

```bash
review --cached
```

Review current branch vs default branch:

```bash
review origin/master...HEAD
# or for repos using main:
review origin/main...HEAD
```

*Compare two models side-by-side (this is a good go-to review command)*:

```bash
review --models "gpt-4o,gemini-2.0-flash" origin/master...HEAD
```

Add extra context:

```bash
review --context "Focus on auth and RBAC changes" origin/master...HEAD

# From a file or command:
cat PR_DESCRIPTION.md | review --context - origin/master...HEAD
git log -1 --pretty=%B | review --context - --cached
```

Limit to specific files:

```bash
review origin/master...HEAD -- src/app/ src/lib/
```

Adjust diff context lines (default is `-U10`):

```bash
review -U5 origin/master...HEAD
```

### Environment variables

* `LLM_MODEL` — default model when `--models` isn’t provided.
  Example:

  ```bash
  export LLM_MODEL=gpt-4o
  ```
* `LLM_MODELS` — default list for multi-model runs.
  Example:

  ```bash
  export LLM_MODELS="gpt-4o,gemini-2.0-flash"
  ```

---

## What the prompt asks the model to do

* Assign a **3-star readiness**:

  * **★★★** Ready for human review (coherent, low risk, tests/docs covered)
  * **★★☆** Needs minor fixes first (quick wins before wasting reviewer time)
  * **★☆☆** Not ready (significant risks/gaps)

* Focus on: architecture, correctness/edge-cases, performance, security, maintainability, and **test coverage**.

* Output a tight, structured review with **blocking**, **non-blocking**, and **test-plan gaps**.

---

## Troubleshooting

* **“❌ Missing required command llm”**
  Activate your venv or install `llm`:

  ```bash
  source ~/.llm-env/bin/activate
  pip install --upgrade pip llm
  ```

* **“No changes found to review.”**
  Your diff is empty. Try `review --cached`, or ensure your branch actually differs:

  ```bash
  git fetch origin
  git diff origin/master...HEAD | head
  ```

* **“Git diff command failed. Check your arguments.”**
  Verify the branch or remote exists:

  ```bash
  git fetch origin
  git branch -r | grep origin
  ```

* **Large diffs**
  The script auto-reduces `-U` to fit within a token budget. If still too large, review a subset of files or break up the PR.

* **`bat` not found**
  It’s optional. Install it or skip pretty printing.

---

## Privacy

This sends your diff content to the selected model provider(s). Don’t use it on sensitive code unless you’re comfortable with that provider’s data policies and configured settings.

---

## License

**Unlicense** — public domain. Do whatever you want.
