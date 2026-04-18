# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] — 2026-04-18

### Added
- Initial release of prompt-forge as a Claude Skill (v0.1.0).
- Bilingual README.md (Spanish primary, English secondary).
- MIT LICENSE with copyright "Carlos San Martín González".
- .gitignore covering .DS_Store, __pycache__, *.pyc, node_modules, .env, IDE configs.
- SKILL.md placeholder (source of truth lives locally at ~/skills/prompt-forge/SKILL.md).
- references/ folder with decision-tree.md, patterns.md, templates.md placeholders.

### Genealogy

Evolved from **PromptCL v2.0** (custom GPT, Claude-only). Migrated to Claude Skill format for multi-tool support.

**Kept from PromptCL v2.0:**
- Specificity scoring.
- Criticality triage.
- Decision-tree technique selection (8-node binary tree).
- Pre-delivery self-check.
- Iteration loop with changelog.

**Discarded from PromptCL v2.0:**
- Meta-Prompting with simulated experts (fabricates in single-prompt).
- Prompt Chaining as embedded instruction (fabricates).
- Verbose pipeline output format.

**Added (new in prompt-forge):**
- Multi-tool routing for 30+ tools (Claude, GPT, o3/o4-mini, Gemini, Qwen, DeepSeek-R1, Ollama, Claude Code, Cursor, Midjourney, Stable Diffusion, ComfyUI, Sora, Runway, ElevenLabs, Zapier, and more).
- Reasoning-native model detection (no CoT on o3, o4-mini, DeepSeek-R1, Qwen3-thinking).
- Anti-fabrication hard rules enforced as a dedicated layer.

[0.1.0]: https://github.com/csanmarting/prompt-forge/releases/tag/v0.1.0
