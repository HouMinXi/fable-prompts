# fable-prompts

Claude Fable 5 system prompt adaptations for different AI coding agent backends
and IDE integrations.

## Design Principle: Lossless Transformation

Every variant starts from the **full `cc-anthropic.md`** as the baseline.
Content is only removed when there is concrete evidence:

| Evidence type | Example |
|---------------|---------|
| `TECHNICAL-FACT` | OutputFilePath is a web-UI file rendering API, absent from all CLI environments |
| `TESTED` | AskUserQuestion (tappable buttons) confirmed broken on CN model APIs |
| `RESEARCHED` | anthropic_reminders: Anthropic classifiers are only relayed by Anthropic infrastructure |

Every removal is documented inline as `<!-- REMOVED [TYPE]: reason -->`.
Speculative removals ("this probably doesn't work") are not permitted.

## Structure

```
upstream/
  CLAUDE-FABLE-5.md        Original Anthropic system prompt (reference baseline)

cc/anthropic/
  cc-anthropic.md          Claude Code + Anthropic native models            (1138 lines)
                           (claude_me, claude_opus, claude_api/Vertex)
                           Injected via --system-prompt in .bashrc wrappers.

cc/cn/
  cc-cn.md                 Claude Code + CN model backends                  (763 lines)
                           (kimi, minimax, glm, mimo)
                           Removals vs anthropic: anthropic_reminders [RESEARCHED],
                           web-UI-only tools [TECHNICAL-FACT], AskUserQuestion
                           tappable variant [TESTED].

cc/copilot/
  cc-copilot.md            GitHub Copilot (VSCode Chat + copilot CLI)       (829 lines)
                           Injected via ~/.copilot/copilot-instructions.md
                           or VSCode codeGeneration.instructions setting.
                           Removals vs anthropic: anthropic_reminders [RESEARCHED],
                           web-UI-only tools [TECHNICAL-FACT].
                           Additions: copilot_ide_capabilities (getDiagnostics,
                           #codebase/#file/#selection context injection, plan_mode).
                           Keeps: memory_system, mcp_servers, skills, computer_use
                           (same agent SDK as Claude Code -- RESEARCHED).

cc/agy/
  cc-agy.md                agy / Gemini CLI (antigravity-cli)               (806 lines)
                           Injected via ~/GEMINI.md symlink.
                           Tool names adapted: Bash->run_shell_command,
                           Read->read_file, Write->write_file, Edit->replace,
                           WebSearch->google_web_search, WebFetch->web_fetch.
                           Removals: same web-UI tools + artifact_usage_criteria
                           + ListAvailableTools [TECHNICAL-FACT].
                           Adapted: memory_system (save_memory + GEMINI.md),
                           mcp_servers (path ~/.gemini/settings.json, @servername).
                           Additions: agy_capabilities section.

cc/cline/
  cline-rules.md           Cline Global Rules supplement                    (115 lines)
                           Injected via ~/Documents/Cline/Rules/fable5-rules.md.
                           ADDITIVE ONLY -- Cline already has its own full system
                           prompt (DefaultClaudeAgentPrompt) covering all tool defs.
                           Contains: tone, wellbeing, coding standards, security
                           checklist, code review classification, git workflow,
                           web search copyright rules.
                           Does NOT contain: tool definitions, identity, filesystem
                           config, memory/MCP sections.

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

## Injection methods by tool

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

**Updating (after cc-copilot.md changes):**

Both routes read the file at runtime -- there is no copy to refresh manually.
All you need is:

```bash
# On primary machine (fable-prompts is a submodule of mhou_workspace):
cd ~/code/mhou_workspace
git pull
git submodule update --init --recursive

# On macOS (standalone clone):
cd ~/code/fable-prompts
git pull
```

After pulling:
- **Route 1 (symlink):** effective immediately on next Copilot session -- no
  further action needed.
- **Route 2 (settings.json file path):** VSCode re-reads the file per session,
  but reload to be safe: `Cmd+Shift+P` -> `Developer: Reload Window`.

### agy / Gemini CLI (cc-agy.md)

Injected via `~/GEMINI.md`. The GEMINI.md is built from two sources:
1. code-review-graph MCP guidance (manually maintained)
2. cc-agy.md behavioral content (via `sed '1,/^---$/d'`)

The `~/GEMINI.md` is managed by mhou_workspace as `home/GEMINI.md` (symlinked).

**Setup:**
```bash
# ~/GEMINI.md is already managed by mhou_workspace symlink on the primary machine.
# On other machines:
git clone git@github.com:HouMinXi/fable-prompts.git ~/code/fable-prompts
mkdir -p ~/.gemini

# Build ~/GEMINI.md combining MCP guidance + cc-agy.md:
{
  echo "<!-- code-review-graph MCP tools -->"
  # ... add your MCP guidance header ...
  printf '\n---\n\n'
  sed '1,/^---$/d' ~/code/fable-prompts/cc/agy/cc-agy.md
} > ~/GEMINI.md
```

agy auto-loads `~/GEMINI.md` (global) and `GEMINI.md` / `.gemini/GEMINI.md`
(project-level) on every session.

### Cline -- Global Rules (cline-rules.md)

Cline auto-loads all `.md` files from `~/Documents/Cline/Rules/`. These rules
are **appended** to Cline's own system prompt (which already covers tool defs,
agent identity, and workspace info) inside a `USER'S CUSTOM INSTRUCTIONS` block.

```bash
mkdir -p ~/Documents/Cline/Rules
ln -s /path/to/fable-prompts/cc/cline/cline-rules.md \
      ~/Documents/Cline/Rules/fable5-rules.md
```

> **Note:** Do NOT inject cc-cn.md or cc-copilot.md as Cline Global Rules.
> Those are full system prompt replacements. Injecting a 700+ line system prompt
> as Global Rules duplicates Cline's own tool definitions and wastes tokens on
> every API call. Use cline-rules.md (115 lines, additive only) instead.

## Submodule of mhou_workspace

This repo lives at `claude/fable-prompts/` inside mhou_workspace as a git
submodule.

```bash
# After adding commits here, bump the pointer in mhou_workspace:
cd ~/code/mhou_workspace
git add claude/fable-prompts
git commit -m "chore/submodule: bump fable-prompts to <description>" # chore
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
