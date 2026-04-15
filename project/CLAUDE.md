# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

<!-- TEMPLATE INSTRUCTIONS
Replace the placeholder sections below with your project's actual content.
The more specific and accurate this file is, the more effectively Claude can help you.
Delete these comment blocks when you're done filling things in.
-->

## Reference Files

<!-- List any additional context files you've placed in .claude/ alongside this one.
These are good for domain-specific knowledge too long to fit here.

Example:
| File | Contents |
|------|----------|
| [`.claude/preferences.md`](preferences.md) | **Read first.** Your workflow preferences and collaboration rules |
| [`.claude/domain_context.md`](domain_context.md) | Deep context on a specific subsystem |
-->

| File | Contents |
|------|----------|
| [`.claude/preferences.md`](preferences.md) | **Read first.** Workflow preferences and collaboration rules |

## Overview

<!-- Describe the project in 2-4 sentences: what it does, what tech stack it uses, and the main
entry points Claude should know about.

Example:
"This is a TypeScript monorepo for a robotics control web app. The frontend (packages/app/)
is React + Redux; the backend (packages/api/) is an Express server that talks to a ROS bridge.
Shared types live in packages/shared/. The main dev entry point is `npm run dev` from the root."
-->

## Key Commands

<!-- List the commands Claude will need most often — how to build, test, run, and format.
Be specific: include flags, environment requirements, and what the output looks like when it succeeds.

Example:
```bash
# Start development server
npm run dev

# Run all tests
npm test

# Lint and auto-fix
npm run lint -- --fix

# Build for production
npm run build
```
-->

## Repository Structure

<!-- Describe the directory layout. Focus on the parts Claude will touch.

Example:
```
src/
├── components/     # React UI components
├── api/            # Backend service clients (REST + WebSocket)
├── store/          # Redux slices and thunks
└── utils/          # Shared helpers (date formatting, math, etc.)
tests/
scripts/            # One-off build and migration scripts
```
-->

## Architecture

<!-- Explain the key design patterns and conventions used in this project.
What should Claude understand before modifying code?

Examples:
- "We use a manager-based pattern: all subsystems register with a central Manager class."
- "State is immutable; all updates go through Redux actions — never mutate directly."
- "All async operations use the Result<T, E> pattern, never throw exceptions."
-->

## Coding Standards

<!-- Describe the lint/format configuration and any style rules that aren't captured by tooling.

Example:
"We use Ruff for Python (120-char line length, configured in pyproject.toml). For TypeScript,
Prettier with 2-space indent and no semicolons. Run `npm run lint` before committing."
-->
