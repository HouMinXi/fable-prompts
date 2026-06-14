# fable-prompts

Claude Fable 5 system prompt adaptations for different Claude Code backends.

## Structure

```
upstream/
  CLAUDE-FABLE-5.md        Original Anthropic system prompt (reference baseline)

cc/anthropic/
  cc-anthropic.md          Claude Code + Anthropic native models
                           (claude_me, claude_opus, claude_api/Vertex)
                           Injected via --system-prompt in .bashrc wrappers.

cc/cn/
  cc-cn.md                 Claude Code + CN model backends
                           (kimi, minimax, glm, mimo)
                           Shorter version; removes Anthropic product references.

cc/vscode/
  cc-vscode.md             VSCode extension via SessionStart hook.

deepseek/
  NOTE.md                  DeepSeek cannot receive system prompts --
                           the local filter proxy strips role:system.
```

## Submodule of mhou_workspace

This repo lives at `claude/fable-prompts/` inside mhou_workspace.

### .bashrc wiring

```bash
# Anthropic native (claude_me, claude_opus, claude_api):
_CLAUDE_API_SYSTEM_PROMPT="$HOME/code/mhou_workspace/claude/fable-prompts/cc/anthropic/cc-anthropic.md"

# CN models (kimi, minimax, glm, mimo) -- separate variable (TODO):
# _CLAUDE_CN_SYSTEM_PROMPT=".../cc/cn/cc-cn.md"
```

## Submodule update

```bash
cd ~/code/mhou_workspace
git submodule update --remote claude/fable-prompts
```
