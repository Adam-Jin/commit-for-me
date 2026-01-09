# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

cfme (Commit For Me) is a bash CLI tool that generates AI-powered commit messages using `aichat`. It uses the conventional-commits standard by default and supports customizable prompts and variables.

## Build & Development Commands

This project uses [Bashly](https://bashly.dev/) for CLI generation and [Shellspec](https://shellspec.info/) for testing.

```bash
# Generate the CLI script from src/
bashly generate

# Run all tests
shellspec --shell bash

# Run a specific test file
shellspec --shell bash spec/<test_file>_spec.sh

# Debug prompt parsing (prints substituted prompt without running AI)
./cfme --print-parsed-prompt
```

## Code Architecture

### Bashly Structure

- `src/bashly.yml` - CLI definition (flags, env vars, help text, version)
- `src/before.sh` - Runs before main command (setup directories, fetch defaults)
- `src/root_command.sh` - Main command logic (orchestrates the commit flow)
- `src/lib/*.sh` - Library functions sourced by the CLI

### Key Library Functions

| File | Purpose |
|------|---------|
| `render_prompt.sh` | Substitutes `<__TEMPLATE__>` strings in prompts |
| `load_vars_map.sh` | Parses YAML variables file into bash associative array |
| `fetch_and_wait_for_ai_response.sh` | Sends prompt to aichat and handles response |
| `extract_headers_from_response.sh` | Parses YAML response to extract commit headers |
| `pick_from_headers.sh` | Uses fzf for interactive selection |
| `build_commit_message.sh` | Assembles final commit message from selected entry |
| `edit_and_commit_or_abort.sh` | Opens editor for review, then commits |

### Prompt Template System

Prompts use `<__VARIABLE__>` syntax for substitution. Special variables:
- `<__GIT_DIFF__>` - Replaced with `git diff --cached` (required)
- `<__RESPONSE_FORMAT_REQUIREMENTS__>` - AI response format rules (required)
- `<__INSTRUCTIONS__>` - Runtime instructions via `-i` flag (optional)

Custom variables are defined in YAML files with either `value` (literal) or `value_from` (command output).

### Directory Structure

```
src/
├── bashly.yml          # CLI definition
├── before.sh           # Pre-command setup
├── root_command.sh     # Main logic
└── lib/                # Library functions
defaults/prompts/       # Default prompt templates
├── conventional-commits/
│   ├── default.md      # Prompt template
│   └── default-vars.yaml
spec/                   # Shellspec tests
release/cfme            # Generated release binary
```

## Testing

Tests are in `spec/*_spec.sh`. Each library function has a corresponding test file. Helper functions for tests are in `spec/helpers/`.

## Release Process

```bash
bashly generate
rm release/cfme
mv cfme release/cfme
chmod -x release/cfme
```
