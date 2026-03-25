# Agent Studio

A single-file AI-powered development dashboard that runs entirely in the browser. Paste `agents_dashboard.html` into any web server or open it locally — no build step, no dependencies, no backend required.

Seven specialised agents collaborate on your codebase, with full GitHub integration and an autonomous loop that can take an issue from requirements through to a merged PR with no human intervention.

---

## Agents

| Agent | Role |
|---|---|
| ⚡ **Coding** | Writes production-ready code, creates branches, commits files |
| ✦ **Design / UI** | Builds UI components, animations, and design systems |
| ◈ **Code Review** | Reviews PRs with a structured APPROVE / REQUEST_CHANGES / COMMENT verdict |
| ◉ **Full-Stack** | Architecture, backend APIs, databases, CI/CD configuration |
| ◎ **Requirements** | Breaks feature requests into structured JSON task breakdowns |
| ❖ **Lead Developer** | Decomposes tasks into atomic, file-level subtasks |
| ⟳ **Auto Loop** | Fully autonomous: issue → requirements → code → CI → review → merge |

---

## Stack Support

Switch between six technology stacks from the sidebar. Each stack has tailored system prompts, suggestion chips, agent tag subtitles, and a CI workflow — per agent.

| Stack | CI |
|---|---|
| 📱 Flutter / Dart | `flutter test` + `flutter build apk` |
| ⚛ React / TypeScript | `vitest` + `vite build` |
| ▲ Next.js / TypeScript | `vitest` + `next build` |
| 🟢 Node.js / Express | `jest` + `tsc` |
| 🐍 Python / FastAPI | `pytest` + `mypy` + `ruff` |
| 🔷 C# / .NET | `dotnet test` |

Stack choice is persisted to `localStorage`.

---

## GitHub Integration

Connect a GitHub Personal Access Token with `repo` and `project` scopes to unlock the full workflow:

- **Issues** — agents read and comment on issues; the Auto Loop creates task and subtask sub-issues automatically
- **Branches** — code is committed to feature branches; file SHAs are fetched from the target branch to prevent conflicts
- **Pull Requests** — PRs are opened automatically with `Closes #N` links
- **CI** — the loop polls GitHub Actions check-runs every 15s, auto-fixes failures, and re-runs CI
- **Reviews** — the Code Review agent posts a real GitHub PR review with a structured verdict
- **Merge** — approved PRs are squash-merged and source issues closed automatically
- **Polling** — PR notifications are polled every 60s and routed to the review agent automatically

Type `#123` in any chat to load a GitHub issue, or `PR#45` to send a pull request to the Code Review agent.

---

## Auto Loop

The ⟳ Auto Loop agent runs a fully autonomous pipeline. Enter a GitHub issue number (or a free-text prompt if GitHub is not configured) and the loop handles everything:

```
Issue / prompt
  → ◎ Requirements breakdown    (creates task issues on GitHub)
  → ❖ Lead Dev subtasks         (creates subtask sub-issues)
  → ⚡ Code all tasks            (commits to shared branch, closes issues)
  → Open PR
  → ◉ CI gate                   (polls check-runs, auto-fixes failures up to 3×)
  → ◈ Code review loop          (reviews, fixes, re-reviews up to 3×)
  → Human gate (optional)
  → Merge + close root issue
```

**Without GitHub:** the loop runs requirements → subtasks → code and produces a downloadable zip of the full directory structure.

**Controls:** the timeline shows every step with live status. Abort at any point. Resume after CI failures re-polls CI rather than skipping it. A "pause for approval before merge" checkbox is available before the final merge step.

---

## No-GitHub / Zip Export

All `file_changes` blocks generated during a session are accumulated in memory. When GitHub is not configured:

- Every agent response that produces files shows a **Download zip** card instead of a PR card
- The zip recreates the full directory structure with last-write-wins deduplication
- An `AGENT_STUDIO_README.md` is included listing every generated file and which agent wrote it

---

## Credential Security

API keys and GitHub tokens are stored in memory only by default — they are cleared on every page refresh.

To persist credentials between sessions, tick **"Save credentials encrypted between sessions"** in the connect modal and choose a passphrase. Credentials are encrypted with **AES-256-GCM** (key derived via PBKDF2, 250,000 iterations, random salt and IV per save) before being written to `localStorage`. The passphrase itself is never stored anywhere.

On reload, a small unlock modal prompts for the passphrase. Wrong passphrase shows an error without touching the stored blob. "Clear saved credentials" removes the encrypted blob entirely.

---

## Setup

1. Download `agents_dashboard.html`
2. Open it in a browser (file:// works, or serve from any static host)
3. Click **Connect Agent Studio** and enter:
   - **Anthropic API Key** — required (`sk-ant-...`)
   - **GitHub PAT** — optional, enables all GitHub features (`repo` + `project` scopes)
   - **Repository** — `owner/repo` format
4. Select a stack from the sidebar
5. Start chatting with any agent, or go straight to ⟳ Auto Loop

No npm install. No build. No server.

---

## Architecture

The entire application is a single `.html` file (~200KB). All agent logic, system prompts, GitHub integration, the orchestrator loop, and the UI are in one file for easy distribution and local use.

For hosting as a shared team tool, three changes are needed:

1. Replace `fetch('https://api.anthropic.com/v1/messages', ...)` with a call to your own proxy endpoint — this moves the API key server-side and removes the `anthropic-dangerous-direct-browser-access` header
2. Replace `ghRest` / `ghGraphQL` calls with a GitHub proxy route — the server looks up the user's token from their session
3. Replace `localStorage` conversation persistence with a `/api/conversations` endpoint backed by Postgres

The recommended hosting stack is a Node.js/Express or Hono server on [Render](https://render.com) (~$7–25/mo) with GitHub OAuth for authentication and Neon Postgres for per-user conversation history.

---

## Changelog

See the in-app changelog (click the version badge in the header) or the table below.

| Version | What changed |
|---|---|
| **v1.10.0** | Encrypted credential storage — AES-256-GCM, PBKDF2, unlock modal |
| **v1.9.0** | Version badge in header, clickable changelog modal |
| **v1.8.0** | `claudeFetch` retry wrapper — exponential backoff on 529/429 |
| **v1.7.0** | No-GitHub auto loop from free-text prompt |
| **v1.6.0** | Zip export — JSZip, session accumulation, full directory structure |
| **v1.5.0** | Six-stack selector — per-stack prompts, suggestions, CI YAML |
| **v1.4.0** | GitHub issue hierarchy — task and subtask sub-issues, auto-close on commit |
| **v1.3.0** | Branch SHA fix — fetchFileContent passes target branch ref |
| **v1.2.0** | CI/CD gate — starter workflow, check-run polling, auto-fix loop, resume re-checks |
| **v1.1.0** | Auto Loop orchestrator — full pipeline, review loop, human gate |
| **v1.0.0** | Initial release — 7 agents, GitHub integration, PR automation |

---

## Requirements

- A modern browser (Chrome, Firefox, Safari, Edge — Web Crypto API required for encrypted storage)
- An [Anthropic API key](https://console.anthropic.com/)
- Optionally: a GitHub Personal Access Token with `repo` and `project` scopes
