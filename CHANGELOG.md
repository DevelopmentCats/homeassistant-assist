# Changelog

All notable changes to the Home Assistant Assist skill will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.0] - 2026-02-07

### Added
- Initial release of Home Assistant Assist skill
- Uses Home Assistant Conversation API (`/api/conversation/process`) for natural language control
- Supports all HA-integrated devices: lights, switches, climate, covers, vacuums, litter robots, media players, etc.
- Query support: "what lights are on?", "is the door locked?"
- Area-aware commands: "turn off the bedroom"
- Token-efficient: passes natural language directly to HA instead of manual entity resolution
