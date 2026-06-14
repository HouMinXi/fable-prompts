# Claude Fable 5 — VSCode Adaptation (stub)

Delivered via SessionStart hook in ~/.claude/settings.json.
Works in both CLI and VSCode extension environments.

Status: STUB — not yet differentiated from cc-anthropic.md.

## Planned differences from cc-anthropic.md

- Adapt for IDE context (file references, inline suggestions)
- VSCode extension supports Gemini/GPT models -- those are NOT handled here
  (they use separate extension mechanisms, not Claude Code)
- Keep: all behavior rules
- Keep: search instructions

## Installation (planned)

```json
// ~/.claude/settings.json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "cat ~/code/mhou_workspace/claude/fable-prompts/cc/vscode/cc-vscode.md"
      }]
    }]
  }
}
```
