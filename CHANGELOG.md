# Changelog

## 2026-06-15 — Initial

- `upstream/CLAUDE-FABLE-5.md`: Anthropic Fable 5 system prompt (reference)
- `cc/anthropic/cc-anthropic.md`: Claude Code adaptation for Anthropic native models
  - Tool names: bash_tool->Bash, create_file->Write, str_replace->Edit, view->Read
  - Paths: /mnt/skills/ -> ~/.claude/skills/, /home/claude -> /tmp
  - Sections replaced: memory_system (MEMORY.md), mcp_servers, available_skills (actual skills)
  - Sections removed: Artifacts storage, claude.ai MCP marketplace, image_search UI, citation tags
  - Environment: "web interface" -> Claude Code CLI
- `cc/cn/cc-cn.md`: stub for CN model adaptations (kimi/minimax/glm/mimo)
- `cc/vscode/cc-vscode.md`: stub for VSCode SessionStart hook
- `deepseek/NOTE.md`: documents why DeepSeek cannot receive system prompts
