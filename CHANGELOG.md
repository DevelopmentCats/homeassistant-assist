# Changelog

All notable changes to the Home Assistant Assist skill will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.1.0] - 2026-02-07

### Added
- Comprehensive README with badges, installation guide, and troubleshooting tips
- Tips for handling entity aliases, automation triggers, and common issues
- ClawHub installation badge and button

### Changed
- Restructured documentation for better readability
- Added comparison table showing token/API efficiency vs traditional approach

## [1.0.1] - 2026-02-07

### Changed
- Simplified skill philosophy: "fire and forget" — trust Assist to handle everything
- Removed over-engineered response parsing guidance
- Errors from Assist are now framed as HA config suggestions, not skill failures
- Streamlined documentation

## [1.0.0] - 2026-02-07

### Added
- Initial release of Home Assistant Assist skill
- Uses Home Assistant Conversation API (`/api/conversation/process`) for natural language control
- Supports any HA-integrated device (not just lights — anything HA exposes)
- Query support with smart response parsing
- Guidance on parsing `data.success[]` for robust answers when `speech` is ambiguous
- Area-aware commands: "turn off the bedroom"
- Token-efficient: passes natural language directly to HA instead of manual entity resolution

### Notes
- Use `/api/conversation/process`, NOT `/api/services/conversation/process` (service endpoint doesn't return full response)
- For queries, parse entity IDs to derive context when friendly names are duplicated
