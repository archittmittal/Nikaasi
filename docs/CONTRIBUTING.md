# Contributing to Nikaasi

## Overview

Thank you for your interest in contributing to Nikaasi. This document outlines the engineering standards, contribution guidelines, and quality gates required for all contributions to this repository.

---

## 1. Code of Conduct and Principles

- Maintain a respectful, inclusive, and professional environment.
- Prioritize citizen clarity, accessibility, and algorithmic fairness in every technical decision.
- Adhere strictly to the zero live PII policy and synthetic sandbox boundary requirements.

---

## 2. Git Branching and Workflow

We follow a structured branch naming convention:

| Branch Prefix | Usage | Example |
| :--- | :--- | :--- |
| `feat/` | New functional capability or user journey | `feat/exit-date-attestation` |
| `fix/` | Bug fix or validation rule correction | `fix/jaro-winkler-threshold` |
| `docs/` | Architectural or technical documentation update | `docs/production-documentation-suite` |
| `refactor/` | Code restructuring without behavioral change | `refactor/state-machine-types` |
| `test/` | Adding or updating unit/integration tests | `test/preflight-cross-validation` |

---

## 3. Commit Message Conventions

All commit messages must strictly follow the **Conventional Commits** specification:

```
<type>(<scope>): <subject>

[optional body]

[optional footer(s)]
```

### Allowed Types

- `feat`: A new feature or capability.
- `fix`: A bug fix.
- `docs`: Documentation changes only.
- `style`: Changes that do not affect the meaning of the code (formatting, white-space, etc.).
- `refactor`: A code change that neither fixes a bug nor adds a feature.
- `perf`: A code change that improves performance.
- `test`: Adding missing tests or correcting existing tests.
- `chore`: Changes to the build process or auxiliary tools.

### Guidelines
- Do NOT use emojis in commit messages, branch names, or pull request descriptions.
- Use the imperative mood in the subject line (e.g., `feat: implement SLA state machine` instead of `feat: implemented SLA state machine`).
- Keep the first line under 72 characters.

---

## 4. Coding Standards and Formatting

- **Language:** TypeScript (Strict mode enabled).
- **Styling:** Tailwind CSS utility classes following mobile-first ordering.
- **Formatting:** Prettier with 2-space indentation, single quotes, and trailing commas where valid.
- **Linting:** ESLint with recommended React and Next.js rule sets. Zero warnings permitted in CI quality gates.

---

## 5. Pull Request Submission Checklist

Before submitting a Pull Request, ensure that:

1. [ ] The branch is branched from the latest `main` branch.
2. [ ] All TypeScript types compile without errors (`npm run typecheck` or `tsc --noEmit`).
3. [ ] Code follows formatting and linting rules (`npm run lint`).
4. [ ] Unit and heuristic tests pass successfully.
5. [ ] No real citizen PII, production tokens, or external API dependencies have been introduced.
6. [ ] The PR description clearly details the problem being addressed and the technical approach taken.
7. [ ] Documentation has been updated to reflect any architectural or schema changes.
