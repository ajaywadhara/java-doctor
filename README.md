# Java Doctor

<p align="center">
  <a href="https://github.com/ajaywadhara/java-doctor/releases/latest">
    <img src="https://img.shields.io/github/v/release/ajaywadhara/java-doctor?include_prereleases&label=version" alt="Version">
  </a>
  <a href="https://opensource.org/licenses/MIT">
    <img src="https://img.shields.io/badge/license-MIT-green.svg" alt="License">
  </a>
</p>

Comprehensive Java code health analyzer that scans for security, performance, correctness, and architecture issues. Outputs a 0-100 score with actionable diagnostics.

## Overview

Java Doctor is an AI-powered code review skill that analyzes Java projects for issues. It automatically detects project technologies and loads relevant rules.

Works with any AI assistant that supports custom skills or prompts (Claude Code, Cursor, GitHub Copilot, etc.).

## Quick Start

```bash
# Clone the repository
git clone https://github.com/ajaywadhara/java-doctor.git

# Copy to your AI assistant's skills directory
# For Claude Code:
cp -r java-doctor ~/.claude/skills/
```

Then activate the skill and say: **"Run Java Doctor"**

## Features

### Progressive Loading
Automatically detects project technologies and loads relevant rules:

| Technology | Detection | Rules |
|-----------|----------|-------|
| Core | Always | Security, Null Safety, Performance, Concurrency, etc. |
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
```

## Trigger Phrases

- `run java doctor`
- `scan my java code`
- `java code review`
- `find bugs in java`
- `check for security issues in java`
- `find performance problems`

## Rule Categories (~197 rules)

| Category | When Loaded | Description |
|----------|------------|-------------|
| Security | Always | Hardcoded secrets, SQL injection, OWASP Top 10 |
| Null Safety | Always | Optional.get(), null returns |
| Exception Handling | Always | Swallowed exceptions |
| Performance | Always | N+1 queries, String concatenation |
| Concurrency | Always | Thread safety |
| Architecture | Always | God classes, long methods |
| Logging | Always | System.out, sensitive data |
| Checkstyle | Always | Formatting, naming |
| Spring/Boot | If detected | @Transactional, Boot 4.x |
| gRPC | If detected | Channel reuse, TLS |
| JPA | If detected | Hibernate issues |
| Build Tools | Always | Dependencies, plugins |

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

## License

MIT License - See LICENSE file

## Author

Ajay Wadhara
