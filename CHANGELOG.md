# Changelog

All notable changes to Gate are documented here.

This project follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.1] - 2026-04-30

### Added

- Boot heartbeat instrumentation. Gate now reports launch state to SolidDark so the activation funnel is observable. Anonymized aggregates only; no code or content data sent.

### Privacy

- See https://soliddark.net/tos for the full telemetry policy.

## [1.0.0] - 2026-04-27

### Added

- Initial public release of Gate, the agentic desktop OS for developers.
- Four-desk pipeline: Kitty intake, Strategist plan, Engineer build, Auditor verify.
- Up to eight robots per workspace with persistent skill databases.
- Robot classes: Natural, Librarian, Surgeon, Paranoid, Investigator, Contractor, Sprinter, Architect.
- Local LLM gateway (Rashomon) on port 14881 supporting Anthropic, OpenAI, Codex CLI, and local Ollama.
- BYO API key model.
- Three-day free trial with hardware-bound license.
- Linux x86_64 AppImage and .deb builds.
- SHA256 verification on all release assets.

### Security

- All telemetry payloads exclude code content. Only metadata (ticket cost, outcome, robot class) is collected.

[Unreleased]: https://github.com/AkrijSama/gate-public/compare/v1.0.1...HEAD
[1.0.1]: https://github.com/AkrijSama/gate-public/releases/tag/v1.0.1
[1.0.0]: https://github.com/AkrijSama/gate-public/releases/tag/v1.0.0
