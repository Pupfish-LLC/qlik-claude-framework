# Contributing to Qlik Development Framework

Thank you for your interest in the Qlik Development Framework for Claude Code.

## How This Repo Works

This repository is released as a **framework template**. It provides the architectural skeleton for a multi-agent Qlik development pipeline. You are free to fork it, expand upon it, and adapt it to your own needs under the MIT license.

We are not currently managing this repo through issues or pull requests. This is an intentional choice: the framework depends on tight internal consistency between agents, skills, and handoff contracts. A change to one component often has downstream effects across the entire pipeline. Maintaining that consistency requires deep familiarity with the full architecture.

## If You Want to Build On This

Fork the repo and make it your own. A few things worth knowing before you start:

- **Agent handoff contracts are coupled.** What one agent produces must match what the next agent expects as input. If you modify an agent's output format, check every downstream consumer.

- **Skills are loaded into specific agents.** The `skills` field in each agent's YAML frontmatter controls which domain knowledge that agent receives. If you change a skill, check which agents auto-load it (the architecture reference has the full mapping).

- **The orchestrator coordinates everything.** Agents cannot call other agents. All delegation flows through CLAUDE.md. If you add a new agent or change the pipeline sequence, the orchestrator logic needs to reflect that.

## Collaboration

We welcome conversations about the architecture, design decisions, and the future of agentic Qlik development. If you have ideas you'd like to discuss, reach out through [pupfish.io](https://pupfish.io).

## Professional Services

For assistance with implementation, customization, or building out the framework for your specific environment, visit [pupfish.io](https://pupfish.io).
