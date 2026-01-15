# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **Cognitive Load** - a Hugo-powered blog about code quality, readability, and software design. The site uses the PaperMod theme.

## Core Thesis

> **Optimize for engineering at the speed of thought.**

Cognitive load is the most important metric to optimize for in software development. What slows engineers down is the dissonance between what the developer *thinks* and what the code *does*. When you can read and understand code as fast as you can think, you've achieved the ultimate readable codebase.

Every post, principle, and pattern on this blog should anchor back to this thesis: **reduce cognitive load**. Whether discussing readability, abstraction, testing, or system design - the underlying goal is always minimizing the mental effort required to understand and work with code.

## Commands

```bash
# Start development server with drafts
hugo server -D

# Build static site (outputs to public/)
hugo
```

## Architecture

```
content/posts/
├── readability/       # Code flow and clarity patterns
├── abstraction/       # Design patterns and abstraction principles
├── optionals/         # Optional types and null handling strategies
├── testing/           # Testing strategies and anti-patterns
└── system-integrity/  # Concurrency and reliability patterns
```

- `hugo.toml` - Hugo configuration
- `themes/PaperMod/` - Theme (git submodule)

## Content Conventions

- Posts use Java for code examples
- Each post follows a pattern: anti-pattern example → best practice → detailed rationale
- Posts require YAML front matter with `title`, `date`, and `categories`

## Commit Message Format

Use structured prefixes:
- `feat(posts):` - New post content
- `docs:` - Documentation updates
- `chore:` - Maintenance tasks
