# Java Doctor

<p align="center">
  <a href="https://github.com/ajaywadhara/java-doctor/releases/latest">
    <img src="https://img.shields.io/github/v/release/ajaywadhara/java-doctor?include_prereleases&label=version" alt="Version">
  </a>
  <a href="https://opensource.org/licenses/MIT">
    <img src="https://img.shields.io/badge/license-MIT-green.svg" alt="License">
  </a>
  <a href="https://github.com/ajaywadhara/java-doctor/stargazers">
    <img src="https://img.shields.io/github/stars/ajaywadhara/java-doctor" alt="Stars">
  </a>
</p>

**AI-powered Java code health analyzer** that scans for security, performance, correctness, and architecture issues. Outputs a 0-100 score with actionable diagnostics and auto-detects 197 rules across 12+ categories.

## Overview

Java Doctor is an AI-powered code review skill that analyzes Java projects for issues. It automatically detects project technologies (Spring Boot, gRPC, JPA, Lombok) and loads relevant rules on-demand.

**197 rules** across 12 categories - from security vulnerabilities to concurrency safety to Spring best practices.

Works with popular AI coding agents:

| Agent | Support |
|-------|---------|
| Claude Code | Native (skills) |
| Cursor | Native (skills) |
| KiloCode | Native (skills) |
| GitHub Copilot | Via custom instructions |
| Windsurf | Native (skills) |

## Quick Start

```bash
# Install via skills CLI (recommended)
npx skills add ajaywadhara/java-doctor

# Or clone manually
git clone https://github.com/ajaywadhara/java-doctor.git

# Copy to your AI assistant's skills directory
# For Claude Code / Cursor / KiloCode:
cp -r java-doctor ~/.claude/skills/
# or
cp -r java-doctor ~/.cursor/skills/
# or
cp -r java-doctor ~/.kilocode/skills/
```

Then activate the skill and say: **"Run Java Doctor"**

## Features

### 197 Rules Across 12 Categories

| Technology | Detection | Rules |
|-----------|----------|-------|
| Core | Always | 108 rules (Security, Null Safety, Performance, Concurrency, etc.) |
| Spring Boot | `spring-boot-starter-parent` | +23 rules |
| gRPC | `grpc-java` | +26 rules |
| JPA/Hibernate | `hibernate`, `spring-data-jpa` | +15 rules |
| Lombok | `lombok` | +5 rules |
| Build Tools | Always | +20 rules |

### Version Detection
- **Java**: Auto-detects 8-25 from pom.xml/build.gradle
- **Spring Boot**: Auto-detects 3.x or 4.x

### Scoring
- **75-100**: Great - Production-ready
- **50-74**: Needs Work - Address warnings
- **0-49**: Critical - Blockers must be fixed

### Output Formats
- Markdown (default)
- JSON
- HTML
- SARIF (IDE integration)
- CSV

## Usage

```
"Run Java Doctor"
"Check my Java code"
"Find security issues in my code"
"Scan for performance problems"
"Analyze this Java project"
```

## Trigger Phrases

- `run java doctor`
- `scan my java code`
- `java code review`
- `find bugs in java`
- `check for security issues in java`
- `find performance problems`
- `analyze java code`

## Rule Categories (197 rules)

| Category | When Loaded | Description |
|----------|------------|-------------|
| Security (16) | Always | Hardcoded secrets, SQL injection, OWASP Top 10 |
| Null Safety (8) | Always | Optional.get(), null returns |
| Exception Handling (8) | Always | Swallowed exceptions |
| Performance (12) | Always | N+1 queries, String concatenation |
| Concurrency (12) | Always | Thread safety |
| Architecture (10) | Always | God classes, long methods |
| Logging (7) | Always | System.out, sensitive data |
| Checkstyle (35) | Always | Formatting, naming |
| Spring/Boot (23) | If detected | @Transactional, Boot 4.x |
| gRPC (26) | If detected | Channel reuse, TLS |
| JPA (15) | If detected | Hibernate issues |
| Build Tools (20) | Always | Dependencies, plugins |

## Project Structure

```
java-doctor/
├── SKILL.md              # Main skill definition
├── LICENSE               # MIT License
├── README.md            # This file
└── references/          # Detailed references
    ├── bug-patterns.md
    ├── security-checklist.md
    ├── performance-antipatterns.md
    ├── spring-best-practices.md
    ├── effective-java-mapping.md
    ├── version-specific-changes.md
    └── grpc-best-practices.md
```

## For AI Assistant Developers

### Tool Requirements
Adapt these tools to your assistant:

- **Bash**: Execute git commands, run analysis
- **Read**: Read source files
- **Write**: Generate reports
- **Glob**: Find Java files
- **Grep**: Search for patterns
- **question**: Ask user for clarification

### Token Usage
- **Startup**: ~100 tokens (metadata)
- **Activation**: ~4,000 tokens (core rules + detected tech)
- **References**: ~20,000 tokens (on-demand)

## Example Output

```markdown
# Java Doctor Report

## Summary
| Metric | Value |
|--------|-------|
| Score | 75/100 |
| Status | Needs Work |
| Java Version | 17 |
| Files Analyzed | 12 |
| Issues | CRITICAL: 2, ERROR: 3, WARNING: 8 |

## Issues
### Security (CRITICAL)
1. AuthService.java:42 - Hardcoded password
   Fix: Use @Value injection
```

## Contributing

1. Fork the repo
2. Create a feature branch
3. Submit a PR

## Skills Marketplace

### skills.sh

Install via CLI:
```bash
npx skills add ajaywadhara/java-doctor
```

## License

MIT License - See LICENSE file

## Author

Ajay Wadhara
