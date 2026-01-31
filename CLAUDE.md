# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **Cognitive Load** - a Hugo-powered blog about code quality, readability, and software design. The site uses the PaperMod theme.

## Core Thesis

> **Optimize for engineering at the speed of thought.**

Cognitive load is the most important metric to optimize for in software development. What slows engineers down is the dissonance between what the developer _thinks_ and what the code _does_. When you can read and understand code as fast as you can think, you've achieved the ultimate readable codebase.

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
content/
├── _index.md          # Landing page content (unused with profileMode)
└── posts/             # All posts in flat structure (no subdirectories)
    ├── handle-the-positive-case.md
    ├── self-documenting-states.md
    └── ...
```

- `hugo.toml` - Hugo configuration (baseURL: cognitive-load.dev/)
- `themes/PaperMod/` - Theme (git submodule)

## URL Structure

- Landing page: `cognitive-load.dev/`
- Posts: `cognitive-load.dev/posts/{slug}/`

## Content Conventions

- Posts use Java for code examples
- Each post follows a pattern: anti-pattern example → best practice → detailed rationale
- Posts require YAML front matter with `title`, `date`, and `tags`
- Tags are lowercase, kebab-case: `readability`, `abstraction`, `optionals`, `testing`, `system-integrity`

## Writing Guidelines

- **Don't restate the blog thesis in every post.** Each post should have its own unique opening that communicates its central point quickly. The connection to cognitive load should be implicit, not explicit repetition.
- **Avoid clichéd use of the tagline.** Phrases like "read code at the speed of thought" should not appear in post content—reserve them for the blog's branding.

## Commit Message Format

Use structured prefixes:

- `feat(posts):` - New post content
- `docs:` - Documentation updates
- `chore:` - Maintenance tasks
