# Changelog

All notable changes to LinkedOut are documented here. The format follows [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/), and versions follow [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- _Add an entry here for every completed task in [TASKS.md](TASKS.md), referencing its T-ID._
- Frontend team plan in `docs/team/`: `TEAM_CONTRACT.md`, `README.md` and personal task files for Arko, Shoaib, Tuli and Abnt (frontend-only, mock-backed, fully parallel). Recorded as ASSUMPTIONS A-70 and A-71.

## [0.1.0] — 2026-11-27

### Added
- Initial documentation generated from master plan (`docs/LinkedOut_Detailed_Product_and_Engineering_Plan.md`):
  - Root `README.md` and `CLAUDE.md`.
  - `docs/PRD.md`, `docs/USER_STORIES.md`, `docs/FEATURES.md`, `docs/WORKFLOW.md`, `docs/ARCHITECTURE.md`, `docs/TECH_STACK.md`, `docs/DATABASE_SCHEMA.md`, `docs/API_SPEC.md`, `docs/UI_UX.md`, `docs/FOLDER_STRUCTURE.md`, `docs/SECURITY.md`, `docs/TESTING.md`, `docs/ENV_SETUP.md`, `docs/DEPLOYMENT.md`, `docs/ROADMAP.md`, `docs/TASKS.md`, `docs/CONTRIBUTING.md`, `docs/ASSUMPTIONS.md`, `docs/CHANGELOG.md`.
- Decisions recorded in `docs/ASSUMPTIONS.md`: NestJS backend, Redis-backed server sessions, per-confession anonymous labels stored in an isolated `vault` schema, six reactions (one per member per post), managed hosting (Vercel + Render + Cloudflare, EU region), GDPR privacy baseline, 2–3 person team with a 16-week MVP starting 30 November 2026.

[Unreleased]: https://github.com/linkedout-app/linkedout/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/linkedout-app/linkedout/releases/tag/v0.1.0
