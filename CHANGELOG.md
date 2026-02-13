# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- EVENT_TTL_DAYS env var for automatic event expiration on startup (#16)

## [0.1.0] — 2026-02-09

### Added
- Initial release with Express + TypeScript server
- WebSocket real-time event streaming
- Multiple namespace support via NAMESPACES env var
- JSONL file persistence per namespace
- Single-page UI with event list and detail view
- Dark/light theme toggle
- Docker Compose with dev/prod profiles
- Caddy reverse proxy in dev profile

### Fixed
- XSS vulnerability via innerHTML interpolation (#3)
- Unbounded in-memory event growth (#6)

### Changed
- Simplified Docker Compose with profiles, removed docker-compose.dev.yml (#5)
- Expanded README documentation with usage examples (#1)