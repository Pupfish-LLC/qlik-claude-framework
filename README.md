# Qlik Development Framework for Claude Code

A multi-agent development framework that accelerates end-to-end Qlik Sense application development using [Claude Code](https://claude.ai/claude-code), Anthropic's agentic coding tool.

## What This Is

This framework provides a complete `.claude/` directory structure that turns Claude Code into a coordinated Qlik development pipeline. Seven specialized AI agents handle distinct phases of development, from requirements gathering through documentation, orchestrated by a central project manager that enforces quality gates and manages dependencies.

The framework produces comprehensive, domain-aware artifacts: specifications, data models, ETL scripts, expressions, visualization designs, and documentation. A Qlik developer provides architectural guidance and execution feedback at key checkpoints. The framework iterates to production-ready quality.

## Architecture Overview

See [CLAUDE.md](CLAUDE.md) for the full orchestration logic, pipeline gates, and context-passing strategy.

### Pipeline Phases

| Phase | Agent | Output |
|-------|-------|--------|
| 0 | Requirements Analyst | Platform Context Document (brownfield ingestion) |
| 1 | Requirements Analyst | Project Specification Document |
| 2 | Source Profiler (skill) | Source Profile Report with architecture classification |
| 3 | Data Architect | Data Model Specification (app architecture, star schema, ETL design, cross-layer field mapping) |
| 4 | Script Developer | Complete .qvs script files with diagnostics |
| 5 | Expression Developer | Expression Catalog + runnable expression-variables.qvs |
| 6 | Viz Architect | Sheet specifications, master item definitions, manual build checklist |
| 7 | QA Reviewer | QA Review Report + data quality validation |
| 8 | Doc Writer | Full documentation set |

### Key Design Decisions

**Brownfield-first.** Most real projects inherit existing artifacts. The pipeline begins with context ingestion (existing apps, shared subroutine libraries, platform conventions, upstream data architecture) before requirements gathering.

**Source architecture-aware.** Different upstream architectures (Data Vault 2.0, star schema, normalized OLTP, flat files) require fundamentally different consumption patterns. The framework classifies source architecture and designs accordingly.

**Human-in-loop execution validation.** Script correctness, synthetic key detection, data quality discovery, and expression evaluation all require loading data in the Qlik engine. The pipeline includes explicit execution validation gates where a developer loads artifacts, reports results, and the framework iterates.

**MCP-optional.** The framework works without database connectivity. When MCP connections are available, source profiling and data quality validation are automated. Without them, the framework produces structured templates for manual completion.

### Skills Library

13 domain knowledge modules loaded on-demand by agents:

**Core:** qlik-naming-conventions, qlik-data-modeling, qlik-load-script, qlik-expressions, qlik-security, qlik-performance, qlik-visualization

**Workflow:** source-profiler, platform-conventions, data-quality-validator, qlik-project-scaffold, qlik-review-checklist, qlik-deploy

Skills encode Qlik-specific patterns, anti-patterns, and best practices: cross-layer field naming strategies, dual-timestamp incremental loads for SCD2 satellites, null handling patterns, diagnostic query templates, and more.

## What's Included

The architectural skeleton is fully open: detailed orchestration rules, directory structure, agent definitions (with frontmatter specifying roles, tool access, and skill dependencies), skill definitions (with frontmatter and descriptions), hook configurations, orchestrator logic, and pipeline state schema. This gives you the complete blueprint for the multi-agent pipeline.

Agent system prompts and skill domain knowledge content have been developed separately and are not included in this release. Each file contains structural placeholders showing what belongs there. The framework is designed so that practitioners can author their own content using the architecture as a guide, or contact [Pupfish](https://pupfish.io) to collaborate on complete implementation.

## Directory Structure

```
qlik-claude-framework/
├── CLAUDE.md                          # Orchestrator (pipeline management)
├── .claude/
│   ├── settings.json                  # Hook configurations
│   ├── agents/                        # 7 specialized subagents
│   ├── skills/                        # 13 domain knowledge modules
│   └── hooks/                         # Validation scripts
├── inputs/                            # User-provided source materials
│   ├── source-documentation/
│   ├── existing-apps/
│   ├── platform-libraries/
│   └── upstream-architecture/
└── artifacts/                         # Framework-generated output
    ├── 04-scripts/
    ├── 07-qa-reports/
    └── 08-documentation/
```

## Requirements

- [Claude Code](https://claude.ai/claude-code) (CLI tool from Anthropic)
- Familiarity with Claude Code's [subagent architecture](https://code.claude.com/docs/en/sub-agents), [skills](https://code.claude.com/docs/en/skills), and [CLAUDE.md configuration](https://code.claude.com/docs/en/memory)
- A Qlik Sense environment (Cloud or client-managed) for execution validation
- Optionally: MCP database connections for automated source profiling

## How It Works

With agent system prompts and skill content in place, the workflow is:

1. Clone this repo into your Qlik project directory
2. Place source materials in `inputs/` (existing apps, schema docs, upstream architecture docs)
3. Run Claude Code. The orchestrator reads `CLAUDE.md` and begins the pipeline
4. Provide business requirements conversationally during Phase 1
5. Review and approve the Project Specification
6. The framework produces data model, scripts, expressions, and viz specs
7. Load artifacts in Qlik Sense at execution validation gates and report results
8. The framework iterates until validation passes
9. Complete documentation is generated

## The Shift

This framework changes what it means to do Qlik development. The developer's role shifts from writing load scripts and expressions to providing clear architectural specifications and validating execution results. That requires genuine expertise in data architecture, business requirements, and the Qlik associative engine. The mechanical work of translating architecture into code is what the agents handle.

## Contributing

This repo is released as a framework template. See [CONTRIBUTING.md](CONTRIBUTING.md) for details on forking, extending, and collaboration.

## License

MIT License. Copyright (c) 2025 Pupfish, LLC. See [LICENSE](LICENSE) and [NOTICE](NOTICE) for details.

## Author

Created by [Pupfish, LLC](https://pupfish.io).

For assistance with implementation, customization, or enhancement, visit [pupfish.io](https://pupfish.io).
