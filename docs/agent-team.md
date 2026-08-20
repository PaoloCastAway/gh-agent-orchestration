# Agent team

To build Mona's Project Pulse dashboard, I am using a four-agent custom team defined under `.github/agents/`, orchestrated with GitHub Copilot CLI running in a Codespace.

## Orchestrator

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Coordinates the Planner, Coder, and Designer agents. Breaks the request into phased tasks, assigns explicit file scopes to avoid overlapping work, decides what can run in parallel vs. sequentially, and reports the integrated result. Does not implement anything itself.
- **Definition:** `.github/agents/orchestrator.agent.md`

## Planner

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Researches the repository, docs, dependencies, and edge cases, then produces an implementation plan with ordered steps, file assignments, dependencies, parallelizable work, edge cases, and validation expectations. Does not write code.
- **Definition:** `.github/agents/planner.agent.md`

## Coder

- **Model:** GPT-5.5 (copilot)
- **Responsibility:** Implements the Project Pulse application logic and structure within its assigned file scope, including support files like `.vscode/launch.json` (configured to run from the `app` folder and open `index.html`). Validates changes before reporting completion.
- **Definition:** `.github/agents/coder.agent.md`

## Designer

- **Model:** Gemini 3.1 Pro (copilot)
- **Responsibility:** Owns UI/UX, accessibility, information architecture, and visual design for the dashboard — project cards, status badges, priority treatment, spacing, and responsive layout with deterministic CSS hooks (`.dashboard`, `.project-card`).
- **Definition:** `.github/agents/designer.agent.md`

All four agents operate under a shared rule: none of them stage, commit, or push changes — git operations remain fully under the learner's control via Copilot CLI prompts.
