# Agent Studio

A single-file AI-powered development dashboard that runs entirely in the browser. Download `agents_dashboard.html`, open it locally or drop it on any static host — no build step, no dependencies, no backend required.

Eight specialised agents collaborate on your codebase across six technology stacks, with full GitHub integration and an autonomous loop that can take an issue from requirements through to a merged PR with no human intervention.

---

## Agents

| Agent | Icon | Role |
|---|---|---|
| **Coding** | ⚡ | Writes production-ready code, creates branches, commits files |
| **Design / UI** | ✦ | Builds UI components, animations, and design systems |
| **Code Review** | ◈ | Reviews PRs with a structured APPROVE / REQUEST_CHANGES / COMMENT verdict |
| **Full-Stack** | ◉ | Architecture, backend APIs, databases, CI/CD configuration |
| **Requirements** | ◎ | Breaks feature requests into structured JSON task breakdowns |
| **Architecture** | 🏛 | Produces a locked dependency and tooling spec all downstream agents must follow |
| **Lead Developer** | ❖ | Decomposes tasks into atomic, file-level subtasks |
| **Auto Loop** | ⟳ | Fully autonomous: issue → requirements → architecture → code → CI → review → merge |

---

## Stack Support

Switch between six technology stacks from the sidebar. Each stack has tailored system prompts, agent tag subtitles, suggestion chips, and a CI workflow — per agent.

| Stack | Language / Runtime | CI |
|---|---|---|
| 📱 Flutter / Dart | Dart 3 · Material 3 | `flutter test` + `flutter build apk` |
| ⚛ React / TypeScript | React 19 · Vite | `vitest` + `vite build` |
| ▲ Next.js / TypeScript | Next 15 · App Router | `vitest` + `next build` |
| 🟢 Node.js / Express | Node 20 · TypeScript | `jest` + `tsc` |
| 🐍 Python / FastAPI | Python 3.12 · async | `pytest` + `mypy` + `ruff` |
| 🔷 C# / .NET | .NET 9 · ASP.NET Core | `dotnet test` |

Stack choice is persisted to `localStorage` and drives the Architecture Agent's version-pinning focus.

---

## Auto Loop Pipeline

The ⟳ Auto Loop agent runs a fully autonomous pipeline. Enter a GitHub issue number (or a free-text prompt if GitHub is not configured) and the loop handles everything:

```
Issue / prompt
  → ◎ Requirements      (creates task issues on GitHub)
  → 🏛 Architecture     (locks exact versions and dependency constraints)
  → ❖ Lead Dev          (creates version-aware atomic subtasks + sub-issues)
  → ⚡ Code all tasks   (commits to shared branch, closes task/subtask issues)
  → Open PR
  → ◉ CI gate           (polls check-runs, auto-fixes failures up to 3×)
  → ◈ Code review loop  (reviews, fixes, re-reviews up to 3×)
  → Human gate (optional)
  → Merge + close root issue
```

**Architecture step:** The Architecture Agent runs after requirements and before Lead Dev. It receives the feature title, issue body, and requirements task summary, then produces a locked spec — exact pinned versions, known conflicts, patterns to follow. This spec is automatically injected into every downstream agent's system prompt for the rest of the run.

**Without GitHub:** the loop runs requirements → architecture → subtasks → code and produces a downloadable zip of the full directory structure.

**Controls:** the timeline shows every step with live status. Abort at any point. Resume after CI failures re-polls CI rather than skipping it. A "pause for approval before merge" checkbox is available before the final merge step.

---

## Architecture Agent

The Architecture Agent sits between Requirements and Lead Dev both in the sidebar and in the pipeline. Its output is a structured markdown spec document:

- **Stack** — framework, language, and version on one line
- **Locked dependencies** — exact pinned versions (no `^`, no `~`, no `>=`)
- **Tooling** — build tool, test runner, linter, CI runner
- **Constraints & known conflicts** — incompatibilities to avoid, breaking changes between versions
- **Integration points** — external services, SDK package names and versions
- **Patterns to use** — folder structure, naming conventions, error handling strategy

**In chat mode:** talking to the Architecture Agent directly captures its reply into a session-level `architectureSpec` variable. Every subsequent call to the Coding, Design, Full-Stack, and Lead Dev agents automatically includes the spec in their system prompt — so you can review and approve the spec before triggering the Auto Loop.

The spec is cleared when GitHub credentials change (new repo context).

---

## GitHub Integration

Connect a GitHub Personal Access Token with `repo` and `project` scopes to unlock the full workflow:

- **Issues** — agents read and comment on issues; the Auto Loop creates task and subtask sub-issues automatically
- **Branches** — code is committed to feature branches with correct SHA handling to prevent conflicts
- **Pull Requests** — PRs opened automatically with `Closes #N` links
- **CI** — polls GitHub Actions check-runs every 15s, auto-fixes failures, and re-runs CI
- **Reviews** — the Code Review agent posts a real GitHub PR review with a structured verdict
- **Merge** — approved PRs are squash-merged and source issues closed automatically
- **Polling** — PR notifications polled every 60s and auto-routed to the review agent

Type `#123` in any chat to load a GitHub issue, or `PR#45` to send a pull request to the Code Review agent.

---

## No-GitHub / Zip Export

All `file_changes` blocks generated during a session are accumulated in memory. When GitHub is not configured:

- Every agent response that produces files shows a **Download zip** card instead of a PR card
- The zip recreates the full directory structure with last-write-wins deduplication
- An `AGENT_STUDIO_README.md` is included listing every generated file and which agent wrote it
- The Auto Loop ends by triggering an automatic zip download instead of opening a PR

---

## Credential Security

API keys and GitHub tokens are stored in memory only by default — cleared on every page refresh.

To persist credentials between sessions, tick **"Save credentials encrypted between sessions"** in the connect modal and choose a passphrase. Credentials are encrypted with **AES-256-GCM** before being written to `localStorage`:

- Key derived via **PBKDF2** with 250,000 iterations
- Random **salt** (16 bytes) and **IV** (12 bytes) generated on every save
- Passphrase **never stored** anywhere — only you can decrypt

On reload, an unlock modal prompts for the passphrase. A wrong passphrase shows an error without touching the stored blob. "Clear saved credentials" removes the encrypted blob and returns to in-memory mode.

---

## Setup

1. Download `agents_dashboard.html`
2. Open it in a browser (`file://` works, or serve from any static host)
3. Click **Connect Agent Studio** and enter:
   - **Anthropic API Key** — required (`sk-ant-...`)
   - **GitHub PAT** — optional, enables all GitHub features (`repo` + `project` scopes)
   - **Repository** — `owner/repo` format
   - **Passphrase** — optional, encrypts credentials for persistence between sessions
4. Select a stack from the sidebar
5. Start chatting with any agent, or go straight to ⟳ Auto Loop

No npm install. No build. No server.

---

## Reliability

All Anthropic API calls go through a `claudeFetch` wrapper with automatic retry and exponential backoff:

- Retries on **529 Overloaded** and **429 Rate Limited** responses
- Backoff schedule: **5s → 10s → 20s → 40s** (up to 4 retries)
- Live retry status shown in the auto loop step timeline
- Hard failure after 4 attempts with a clear error message

---

## Technical Architecture

The entire application is a single `.html` file (~210KB). All agent logic, system prompts, GitHub integration, the orchestrator loop, crypto helpers, and the UI are in one file for easy distribution and local use.

For hosting as a shared team tool, three changes are needed to the JavaScript:

1. Replace `fetch('https://api.anthropic.com/v1/messages', ...)` with a call to your own proxy — moves the API key server-side and removes the `anthropic-dangerous-direct-browser-access` header
2. Replace `ghRest` / `ghGraphQL` with a GitHub proxy route — the server looks up each user's token from their session
3. Replace `localStorage` conversation persistence with a `/api/conversations` endpoint backed by Postgres

The recommended hosting stack is Node.js/Express or Hono on [Render](https://render.com) (~$7–25/mo) with GitHub OAuth for authentication and Neon Postgres for per-user conversation history.

---

## Changelog

Click the version badge (`v1.11.0`) in the header to open the in-app changelog, or see the summary below.

| Version | What changed |
|---|---|
| **v1.11.0** | Architecture Agent — locked dependency specs, injected into all downstream agents |
| **v1.10.0** | Encrypted credential storage — AES-256-GCM, PBKDF2, unlock modal |
| **v1.9.0** | Version badge in header, clickable changelog modal |
| **v1.8.0** | `claudeFetch` retry wrapper — exponential backoff on 529/429 |
| **v1.7.0** | No-GitHub Auto Loop from free-text prompt |
| **v1.6.0** | Zip export — JSZip, session accumulation, full directory structure |
| **v1.5.0** | Six-stack selector — per-stack prompts, suggestions, CI YAML |
| **v1.4.0** | GitHub issue hierarchy — task and subtask sub-issues, auto-close on commit |
| **v1.3.0** | Branch SHA fix — `fetchFileContent` passes target branch ref |
| **v1.2.0** | CI/CD gate — starter workflow, check-run polling, auto-fix loop, resume re-checks CI |
| **v1.1.0** | Auto Loop orchestrator — full pipeline, review loop, human gate |
| **v1.0.0** | Initial release — 7 agents, GitHub integration, PR automation |

---

## Requirements

- A modern browser with Web Crypto API support (Chrome, Firefox, Safari, Edge)
- An [Anthropic API key](https://console.anthropic.com/)
- Optionally: a GitHub Personal Access Token with `repo` and `project` scopes
