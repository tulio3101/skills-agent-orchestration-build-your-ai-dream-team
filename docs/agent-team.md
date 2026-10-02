# Agent team

I will use GitHub Copilot CLI in a Codespace to orchestrate a custom team for
building Mona's Project Pulse dashboard. The team is defined under
`.github/agents/`:

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| **Orchestrator** | Claude Opus 4.7 (copilot) | Breaks the dashboard work into phases, delegates tasks to the specialist agents, manages file ownership and dependencies, and verifies that the integrated result works together. | [`.github/agents/orchestrator.agent.md`](../.github/agents/orchestrator.agent.md) |
| **Planner** | Claude Opus 4.7 (copilot) | Researches the repository, documentation, dependencies, risks, edge cases, and validation needs, then produces an implementation plan without writing code. | [`.github/agents/planner.agent.md`](../.github/agents/planner.agent.md) |
| **Designer** | Gemini 3.1 Pro (copilot) | Defines and implements the dashboard's UI/UX within its assigned scope, focusing on usability, accessibility, information hierarchy, responsive behavior, visual clarity, project cards, status badges, priority treatment, and deterministic CSS hooks. | [`.github/agents/designer.agent.md`](../.github/agents/designer.agent.md) |
| **Coder** | GPT-5.5 (copilot) | Implements the assigned application code and runnable-app support, using clear, deterministic, testable behavior with explicit errors; for Project Pulse, this includes the assigned `.vscode/launch.json` configuration when needed. | [`.github/agents/coder.agent.md`](../.github/agents/coder.agent.md) |

The Orchestrator will use the Planner's research to define safe, non-overlapping
work packages, then coordinate the Designer and Coder before validating the
finished Project Pulse dashboard. All agents leave staging, committing, and
pushing changes to the learner.
