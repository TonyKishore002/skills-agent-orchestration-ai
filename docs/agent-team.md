# Agent team

Use the following agent team to build Mona's Project Pulse dashboard.

- Orchestrator — model: Claude Opus 4.7 (copilot). Responsibility: coordinate Planner, Coder, and Designer; decompose work into phases, assign file scopes, run tasks in parallel or sequentially as needed, and verify integrated results. Definition: .github/agents/orchestrator.agent.md

- Planner — model: Claude Opus 4.7 (copilot). Responsibility: research the codebase, produce an ordered implementation plan with file assignments, dependencies, edge cases, and validation steps. Definition: .github/agents/planner.agent.md

- Coder — model: GPT-5.5 (copilot). Responsibility: implement code and runnable app support within assigned file scopes, create/testing support files when required (e.g., .vscode/launch.json), and validate changes. Definition: .github/agents/coder.agent.md

- Designer — model: Gemini 3.1 Pro (copilot). Responsibility: UI/UX, accessibility, information architecture, visual design, and Project Pulse styling (responsive layout, project cards, status badges). Definition: .github/agents/designer.agent.md

Note: Work will be orchestrated from this Codespace using the GitHub Copilot CLI. Agents follow the repository rules (they do not stage, commit, or push changes; the learner controls git operations).
