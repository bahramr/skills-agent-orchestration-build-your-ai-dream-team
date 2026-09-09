# Agent team

To build Mona's Project Pulse dashboard, I'm using GitHub Copilot CLI in a
Codespace to orchestrate a prebuilt team of four custom agents. Each agent is
defined in `.github/agents/` as a Markdown file with YAML frontmatter (name,
description, target model, allowed tools) followed by its system prompt.

## Orchestrator

- **Model:** Claude Opus 4.7 (copilot)
- **Definition:** `.github/agents/orchestrator.agent.md`
- **Responsibility:** Coordinates the Planner, Coder, and Designer. Breaks the
  request into phases from the Planner's plan, assigns each specialist an
  explicit file scope, decides what can run in parallel (non-overlapping file
  scopes, no data dependencies) versus sequentially, verifies the integrated
  result, and reports the outcome. Does not implement work itself.

## Planner

- **Model:** Claude Opus 4.7 (copilot)
- **Definition:** `.github/agents/planner.agent.md`
- **Responsibility:** Researches the repository and relevant documentation,
  then produces an implementation plan for the Orchestrator: ordered steps,
  file assignments, dependencies, parallelizable vs. sequential work, edge
  cases, and validation expectations. Does not write code.

## Designer

- **Model:** Gemini 3.1 Pro (copilot)
- **Definition:** `.github/agents/designer.agent.md`
- **Responsibility:** Guides the dashboard's UI/UX within its assigned file
  scope — usability, accessibility, information hierarchy, and visual
  design. For Project Pulse, ensures a polished (not bare) dashboard with
  project cards, status badges, and deterministic CSS hooks (`.dashboard`,
  `.project-card`).

## Coder

- **Model:** GPT-5.5 (copilot)
- **Definition:** `.github/agents/coder.agent.md`
- **Responsibility:** Implements the static dashboard files
  (`app/index.html`, `app/styles.css`, `app/project-data.json`) within its
  assigned file scope. May also create `.vscode/launch.json` when assigned,
  so the dashboard can be launched via the **Run Project Pulse Dashboard**
  configuration.

## How the team is used

All four agents are invoked from GitHub Copilot CLI running in a Codespace
(`copilot --allow-all --enable-all-github-mcp-tools`), where I select agents
with `/agent` and let the Orchestrator coordinate the Planner, Designer, and
Coder rather than doing all the work in a single undifferentiated prompt.
Every agent leaves git operations (staging, committing, pushing) to me.
