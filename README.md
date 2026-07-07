# Angular Code Review Agent

A reusable code-review agent for **Claude Code** and **GitHub Copilot Chat** that performs a full project-wide review of a modern Angular 15+ application and writes a structured report to `ANGULAR_CODE_REVIEW.md`.

## What it does

Discovers the whole codebase itself and evaluates it against 10 focus areas:

1. Architecture & Module Structure
2. Components
3. State & Data Flow
4. RxJS Usage & Memory Leaks
5. Performance
6. HTTP & Error Handling
7. Forms
8. Accessibility
9. Security
10. Code Quality

The report contains findings by severity (Critical / High / Medium / Low) with `file:line` locations and concrete recommendations, a 1–5 scorecard, and the top 3 fixes to tackle first.

**Read-only** — never edits source. The only file written is `ANGULAR_CODE_REVIEW.md`.

## Repo layout

```
.claude/agents/angular-code-reviewer.md            ← Claude Code format
.github/prompts/angular-code-reviewer.prompt.md    ← GitHub Copilot Chat format
```

Same prompt content in both. Install whichever format matches the tool you use.

## Prerequisites

- **Claude Code:** [Claude Code](https://claude.com/claude-code) installed, with access to the Opus model (`claude-opus-4-7`).
- **GitHub Copilot:** VS Code 1.90+ with GitHub Copilot Chat extension, `chat.promptFiles` enabled, and **Agent mode** selected in the chat panel.

## Installation — Claude Code (Windows / PowerShell)

### Step 1 — Clone this repo

```powershell
git clone https://github.com/Rudra-Infinnium/claude-copilot-angular-review-agent.git
```

### Step 2 — Pick an install scope

**Option A — Personal install** (available in all your projects)

```powershell
New-Item -ItemType Directory -Force "$HOME\.claude\agents" | Out-Null
Copy-Item "claude-copilot-angular-review-agent\.claude\agents\angular-code-reviewer.md" "$HOME\.claude\agents\"
```

**Option B — Per-project install** (ships with the code, teammates get it on `git pull`)

From inside the target Angular project:

```powershell
New-Item -ItemType Directory -Force ".claude\agents" | Out-Null
Copy-Item "..\claude-copilot-angular-review-agent\.claude\agents\angular-code-reviewer.md" ".claude\agents\"
git add .claude\agents\angular-code-reviewer.md
git commit -m "Add angular-code-reviewer agent"
git push
```

### Usage

In Claude Code, open an Angular project and ask:

> Use the `angular-code-reviewer` agent to review this project

Report is written to `ANGULAR_CODE_REVIEW.md` at the project root. Run `/agents` in Claude Code to verify the agent is loaded.

## Installation — GitHub Copilot Chat (VS Code)

Copilot prompt files live under `.github/prompts/` **inside the target project** (they are per-workspace, not global).

### Step 1 — Clone this repo

```powershell
git clone https://github.com/Rudra-Infinnium/claude-copilot-angular-review-agent.git
```

### Step 2 — Copy the prompt into your target project

From inside the target Angular project:

```powershell
New-Item -ItemType Directory -Force ".github\prompts" | Out-Null
Copy-Item "..\claude-copilot-angular-review-agent\.github\prompts\angular-code-reviewer.prompt.md" ".github\prompts\"
git add .github\prompts\angular-code-reviewer.prompt.md
git commit -m "Add angular-code-reviewer prompt for Copilot"
git push
```

### Step 3 — One-time VS Code setup

- Enable `"chat.promptFiles": true` in VS Code settings.
- In the Copilot Chat panel, open the mode picker and select **Agent**. (Without Agent mode, Copilot cannot write the report file automatically.)

### Usage

In Copilot Chat, type:

```
/angular-code-reviewer
```

Copilot runs the review and writes `ANGULAR_CODE_REVIEW.md` at the workspace root.

## macOS / Linux install

<details>
<summary>Click to expand</summary>

```bash
git clone https://github.com/Rudra-Infinnium/claude-copilot-angular-review-agent.git

# Claude Code — personal install
mkdir -p ~/.claude/agents
cp claude-copilot-angular-review-agent/.claude/agents/angular-code-reviewer.md ~/.claude/agents/

# Claude Code — per-project (from inside the target project)
mkdir -p .claude/agents
cp ../claude-copilot-angular-review-agent/.claude/agents/angular-code-reviewer.md .claude/agents/

# Copilot — per-project (from inside the target project)
mkdir -p .github/prompts
cp ../claude-copilot-angular-review-agent/.github/prompts/angular-code-reviewer.prompt.md .github/prompts/
```

</details>

## Customization

Both files are Markdown with frontmatter + a system prompt. Edit freely:

**Claude Code (`.claude/agents/angular-code-reviewer.md`)**
| Field | What it does |
|---|---|
| `name` | Identifier used to invoke the agent |
| `description` | How Claude decides when to auto-invoke |
| `tools` | Tools the agent may use |
| `model` | `opus`, `sonnet`, `haiku`, or a pinned ID |

**Copilot (`.github/prompts/angular-code-reviewer.prompt.md`)**
| Field | What it does |
|---|---|
| `mode` | `agent`, `ask`, or `edit` |
| `description` | Shown in the Copilot mode picker |

The body below the frontmatter is the prompt — adjust focus areas, output format, and rules to match your team's standards.

## Related repos

- .NET code-review agent: [claude-copilot-dotnet-review-agent](https://github.com/Rudra-Infinnium/claude-copilot-dotnet-review-agent)
- Python code-review agent: [claude-code-review-agent](https://github.com/Rudra-Infinnium/claude-code-review-agent)

## License

MIT — see [LICENSE](LICENSE).
