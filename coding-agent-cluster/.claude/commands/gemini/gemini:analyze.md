# Codebase Analysis

## Gemini CLI

The Gemini CLI is designed for efficient codebase analysis, allowing you to query and analyze code files and directories directly from the command line. This guide provides examples and best practices for using the Gemini CLI effectively.

## Usage Guidelines

Use `gemini -p` for:

- Analyzing entire codebases or large directories
- Files totaling more than 100KB
- Project-wide pattern analysis
- Verifying feature implementations across codebase
- When current context window is insufficient

### File Inclusion Syntax

- Single file: `gemini -p "@src/main.py Explain this file"`
- Multiple files: `gemini -p "@package.json @src/index.js Analyze dependencies"`
- Entire directory: `gemini -p "@src/ Summarize architecture"`
- Current directory: `gemini -p "@./ Project overview"`
- All files flag: `gemini --all_files -p "Analyze project structure"`

### Implementation Verification Examples

- Feature check: `gemini -p "@src/ @lib/ Has dark mode been implemented?"`
- Auth verification: `gemini -p "@src/ @middleware/ Is JWT auth implemented?"`
- Pattern search: `gemini -p "@src/ Any React hooks for WebSocket connections?"`
- Security audit: `gemini -p "@src/ @api/ Are SQL injection protections implemented?"`

### Tips for Effective Gemini CLI Use

- Analyze code structure then chunk into smaller parts to use Gemini CLI effectively
