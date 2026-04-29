# custom-agents

A repository for sharing custom GitHub Copilot agents, instructions, prompts, and skills so that teammates can clone and use them in their own VS Code workspaces.

## Overview

This repository is the central hub for Copilot customization files. Once cloned (or added as a workspace folder), VS Code with the GitHub Copilot Chat extension automatically picks up these files from the `.github/` directory — no extra configuration required.

## Structure

```
.
├── .github/
│   ├── agents/                        # Custom agent modes (*.agent.md)
│   │   └── aws-migration.agent.md
│   ├── instructions/                  # Always-on or file-scoped instructions (*.instructions.md)
│   ├── prompts/                       # Reusable prompt templates (*.prompt.md)
│   ├── skills/                        # Workflow skills with bundled assets (<name>/SKILL.md)
│   └── workflows/
│       └── copilot-setup-steps.yml    # Copilot cloud agent environment setup
└── README.md
```

| Folder | File type | VS Code chat usage |
|---|---|---|
| `.github/agents/` | `*.agent.md` | Select agent from the agent picker in chat |
| `.github/instructions/` | `*.instructions.md` | Auto-applied based on `applyTo` glob or `description` |
| `.github/prompts/` | `*.prompt.md` | Run via `/prompt-name` slash command in chat |
| `.github/skills/` | `<name>/SKILL.md` | Invoked automatically when description matches the task |

## Available Agents

### `aws-migration`

Migrates Java projects from AWS SDK v1 (`com.amazonaws`) to AWS SDK v2 (`software.amazon.awssdk`) while preserving compatibility with the `vtw-aws-common` helper jar.

**Covers:** `pom.xml` dependency cleanup, DynamoDB annotation migration, client builder patterns, Spring bean config updates, exception type mapping, and Maven build validation.

## How to Use

### Option A — Clone this repo as a standalone workspace folder

```bash
git clone <this-repo-url>
code custom-agents
```

Open the cloned folder in VS Code. All agents, instructions, prompts, and skills in `.github/` are immediately available.

### Option B — Add as a workspace folder alongside your project

In VS Code, use **File → Add Folder to Workspace…** and select this cloned repo. Both workspaces share Copilot customizations.

### Option C — Copy files directly into your project

Copy any file from `.github/agents/`, `.github/instructions/`, or `.github/prompts/` into the corresponding `.github/` subfolder of your own project repository.

## Contributing

1. Place new agents in `.github/agents/` as `<name>.agent.md`.
2. Place new instructions in `.github/instructions/` as `<name>.instructions.md`.
3. Place new prompts in `.github/prompts/` as `<name>.prompt.md`.
4. Place new skills in `.github/skills/<name>/SKILL.md`.
5. Update this README with a short description of the new customization.

## Copilot Setup

The [`copilot-setup-steps.yml`](.github/workflows/copilot-setup-steps.yml) workflow prepares the Copilot cloud agent environment when Copilot works directly on tasks in this repository.
