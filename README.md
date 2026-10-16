# Description
A multi-agent orchestrated development workflow that uses [Antigravity](https://antigravity.google/product/antigravity-cli/) as the source of truth for product and architecture requirements. Agents collaborate in a structured loop — from specification through implementation to review — keeping every change grounded in a written spec before the implementation starts.

## Agents

| Agent | Mode | Role |
|---|---|---|
| [@architect](<YOUR_SCHEDULER_PROJECT_WORKING_FOLDER>/scheduler/.agents/agents/architect.md) | primary | Drives the full delivery loop: clarifies requirements, creates OpenSpec specifications, delegates implementation, and evaluates review output. Never writes code itself. |
| [@developer-backend](<YOUR_SCHEDULER_PROJECT_WORKING_FOLDER>/scheduler/.agents/agents/developer-backend.md) | subagent | Implements the backend spec assigned by `@architect` using the `openspec-apply-change` skill. Runs all checks before reporting completion. |
| [@developer-frontend](<YOUR_SCHEDULER_PROJECT_WORKING_FOLDER>/scheduler/.agents/agents/developer-frontend.md) | subagent | Implements the frontend spec assigned by `@architect` using the `openspec-apply-change` skill. Runs all checks before reporting completion. |
| [@code-reviewer](<YOUR_SCHEDULER_PROJECT_WORKING_FOLDER>/scheduler//.agents/agents/code-reviewer.md) | subagent | Reviews every change against the OpenSpec change definition. Issues actionable change requests or approves; cannot modify code itself. |
| [@repo-scout](<YOUR_SCHEDULER_PROJECT_WORKING_FOLDER>/scheduler//.agents/agents/repo-scout.md) | subagent | Scans the repository and maintains `ARCHITECTURE.md` with detected stack, conventions, linting/test commands, and structural hotspots. |

## Workflow

```
User ──► @architect
             │
             ├─ calls @repo-scout (first run or on repo change)
             │
             ├─ guides user through opsx-explore / opsx-propose
             │   to build the OpenSpec specification
             │
             │  [on explicit user approval]
             │
             ├─► @developer  (opsx-apply)
             │       │
             │       └─► reports completion
             │
             ├─► @code-reviewer
             │       │
             │       ├─ change requests ──► @developer (loop)
             │       │
             │       └─ approved ──► @architect
             │
             └─► summary to user + opsx-archive suggestion
```

1. **Discovery** — `@architect` inspects the repo (via `@repo-scout` if needed) and collaborates with the user to define an OpenSpec specification using the `opsx-explore` and `opsx-propose` skills.
2. **Approval gate** — implementation only starts after the user explicitly approves.
3. **Implementation** — `@developer` applies the spec (`opsx-apply`), follows existing repo conventions, adds tests, and validates with linters/type-checkers before reporting back.
4. **Review** — `@code-reviewer` reads the OpenSpec change, runs `git diff`, checks correctness, security, and test coverage, and either requests changes or approves.
5. **Loop** — steps 3–4 repeat until `@code-reviewer` approves and `@architect` is satisfied.
6. **Archive** — `@architect` suggests archiving the change with `opsx-archive` and returns control to the user.

## How to Use

### Prerequisites

- [Antigravity](https://antigravity.google/product/antigravity-cli/) installed
- [OpenSpec](https://openspec.dev) installed

### Setup

Copy the agent definitions to the Project agents directory and restart Antigravity.

**macOS / Linux**
```bash
cp opencode/agents/* <YOUR_SCHEDULER_PROJECT_WORKING_FOLDER>/scheduler/.agents/agents/
```

**Windows**
```powershell
Copy-Item opencode\agents\* "<YOUR_SCHEDULER_PROJECT_WORKING_FOLDER>\scheduler\.agents\agents\"
```

### Start

If it is a new project, where you have not used openspec yet, run `openspec init` to set it up before launching opencode.

Open a new Antigravity session in your project directory and describe the change you want to make to `@architect`.

## Credits

This agent setup is based on the workflow described in [How I Write Software With LLMs](https://www.stavros.io/posts/how-i-write-software-with-llms/) by Stavros Korokithakis.
