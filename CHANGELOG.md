# Changelog

All notable changes to this repository will be documented in this file.

---

## [Unreleased]

### Added

- Professional README landing-page structure.
- `docs/architecture.md` for system architecture and Mermaid/SVG workflow guidance.
- `docs/installation.md` for installation and verification steps.
- `docs/model-configuration.md` for provider/model/fallback/alias guidance with point-level source links.
- `docs/openai-gpt56-operating-standard.md` for July 2026 OpenClaw + OpenAI GPT-5.6 operating standards with inline source markers.
- `docs/cost-control.md` for model-tier, output-budget, tool-call, file-workflow, and cron cost controls with pricing/source markers.
- `docs/security.md` for token hygiene, log sanitization, and agent threat model controls with security-source markers.
- `docs/troubleshooting.md` for common errors and first checks.
- `docs/cron-automation.md` for cost-safe scheduled workflows with command/cost/security source markers.
- `docs/references.md` as the central source reference registry and citation-placement standard.
- `docs/external-research-july-2026.md` for external source notes, source-handling rules, and point-level citation decisions.
- `assets/openclaw-openai-gpt5x-workflow-dark.svg` for reliable dark workflow rendering on GitHub.
- `examples/config/.env.example` for safe environment variable templates.
- `examples/prompts/daily-brief.md` for a reusable daily brief prompt.
- `.github/PULL_REQUEST_TEMPLATE.md` for review quality control.
- `CONTRIBUTING.md` for documentation and branch standards.

### Changed

- Reorganized README from a long single-page guide into a standard repository landing page.
- Replaced hardcoded model examples with provider/model placeholders and verification-first guidance.
- Standardized code fences for GitHub syntax highlighting.
- Replaced README inline Mermaid architecture block with an SVG dark workflow for consistent rendering on GitHub main.
- Expanded model-configuration guidance for GPT-5.6 route verification, tier strategy, fallback rules, and aliases.
- Expanded cron guidance with model-tier selection, pricing/quota verification, stop conditions, and isolated-session control.
- Expanded security guidance with agent threat model, tool permission standards, and incident note template.
- Rewrote `openclaw_openai_gpt_5_x_lesson_th.md` to align with README, GPT-5.6 Sol/Terra/Luna strategy, verification-first workflow, cost controls, security controls, workshop checklists, and point-level source links.
- Added inline source markers such as `[OC-OPENAI]`, `[OC-MODELS]`, `[OA-GPT56]`, `[NEWS-REUTERS]`, and `[SEC-PRISM]` near supported claims, command patterns, pricing cautions, and security controls.
