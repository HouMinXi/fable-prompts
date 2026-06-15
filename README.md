# fable-prompts

Claude Fable 5 system prompt adaptations for different Claude Code backends
and IDE integrations.

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
                           Removes Anthropic product/classifier references,
                           web-UI tools, and hardcoded knowledge cutoff date.

cc/copilot/
  cc-copilot.md            GitHub Copilot (VSCode Chat + copilot CLI)
                           Injected via ~/.copilot/copilot-instructions.md
                           or VSCode codeGeneration.instructions setting.
                           Adds Copilot IDE capabilities (getDiagnostics,
                           #codebase context, browser automation).
                           Removes CC-specific tool schemas and skills system.

cc/vscode/
  cc-vscode.md             VSCode Claude Code extension via SessionStart hook.

deepseek/
  NOTE.md                  DeepSeek cannot receive system prompts --
                           the local filter proxy strips role:system.

tests/
  spec.md                  20 behavioral test cases for CN model adaptation
  run-auto.sh              Automated majority-vote tests (AUTO probes)
  run-judge.sh             Blind pairwise judge tests (JUDGE probes)
```

## Injection methods by backend

### Claude Code -- Anthropic native (cc-anthropic.md)

Injected via `--system-prompt` flag in `.bashrc` wrappers:

```bash
# In .bashrc (managed by mhou_workspace):
_CLAUDE_API_SYSTEM_PROMPT="$HOME/code/mhou_workspace/claude/fable-prompts/cc/anthropic/cc-anthropic.md"

# claude_me / claude_opus / claude_api wrappers pass:
#   --system-prompt "$_CLAUDE_SYSPROMPT_CONTENT"
```

### Claude Code -- CN models (cc-cn.md)

Injected via `_build_cn_sysprompt_args()` in `.bashrc`:

```bash
_CLAUDE_CN_API_SYSTEM_PROMPT="$HOME/code/mhou_workspace/claude/fable-prompts/cc/cn/cc-cn.md"

# kimi / minimax / glm / mimo wrappers call:
#   _build_cn_sysprompt_args "$@"
#   claude "${_SYSPROMPT_ARGS[@]}" "$@"
```

### GitHub Copilot -- CLI + VSCode Chat (cc-copilot.md)

Two injection routes; both point to the same file:

**Route 1 -- Global (CLI + VSCode Chat):**
```bash
mkdir -p ~/.copilot
ln -s /path/to/fable-prompts/cc/copilot/cc-copilot.md \
      ~/.copilot/copilot-instructions.md
```
Loaded automatically by `copilot` CLI and GitHub Copilot VSCode extension
on every session. No frontmatter required.

**Route 2 -- VSCode Chat only (via settings.json):**

| Platform | settings.json path |
|----------|--------------------|
| Linux Flatpak VSCode | `~/.var/app/com.visualstudio.code/config/Code/User/settings.json` |
| Linux RPM/deb VSCode | `~/.config/Code/User/settings.json` |
| macOS | `~/Library/Application Support/Code/User/settings.json` |

Add this key (use absolute path, not `~`):
```json
{
  "github.copilot.chat.codeGeneration.instructions": [
    {
      "file": "/absolute/path/to/fable-prompts/cc/copilot/cc-copilot.md"
    }
  ]
}
```

Then reload VSCode: `Cmd/Ctrl+Shift+P` -> `Developer: Reload Window`.

**macOS quick setup:**
```bash
git clone git@github.com:HouMinXi/fable-prompts.git ~/code/fable-prompts

# Route 1
mkdir -p ~/.copilot
ln -s ~/code/fable-prompts/cc/copilot/cc-copilot.md \
      ~/.copilot/copilot-instructions.md

# Route 2
python3 - << 'PYEOF'
import json, pathlib
p = pathlib.Path.home() / "Library/Application Support/Code/User/settings.json"
data = json.loads(p.read_text()) if p.exists() else {}
data["github.copilot.chat.codeGeneration.instructions"] = [
    {"file": str(pathlib.Path.home() / "code/fable-prompts/cc/copilot/cc-copilot.md")}
]
p.write_text(json.dumps(data, indent=4) + "\n")
print("Done:", p)
PYEOF
```

**Note:** Route 1 covers both CLI and VSCode; Route 2 is VSCode-only.
Use both together for maximum coverage (Copilot deduplicates identical content).

### Cline -- Global Rules (cc-cn.md)

Cline automatically loads all files from `~/Documents/Cline/Rules/`:

```bash
mkdir -p ~/Documents/Cline/Rules
ln -s /path/to/fable-prompts/cc/cn/cc-cn.md \
      ~/Documents/Cline/Rules/fable5-cn.md
```

The CN adaptation is used here because Cline is typically configured with
CN model backends (kimi-proxy, mimo-proxy, etc.) on this setup. When Cline
is backed by Copilot models via VS Code Language Model API, Cline Global
Rules inject at the Cline layer (not the Copilot layer), so cc-cn.md remains
appropriate.

## Submodule of mhou_workspace

This repo lives at `claude/fable-prompts/` inside mhou_workspace as a git
submodule.

```bash
# After adding commits here, bump the pointer in mhou_workspace:
cd ~/code/mhou_workspace
git add claude/fable-prompts
git commit -m "chore/submodule: bump fable-prompts to <description>"
```

## Updating on other machines

If cloned as part of mhou_workspace:
```bash
cd ~/code/mhou_workspace
git pull
git submodule update --init --recursive
```

If cloned standalone:
```bash
cd ~/code/fable-prompts
git pull
```
