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

## Fable Prompt Characteristics

`cc-anthropic.md` (1138 lines) has a distinctive structure and rule style that
differs from typical system prompts. Understanding it helps when adapting to new
backends.

**Three-layer architecture**

| Layer | Content | Lines (approx) |
|-------|---------|----------------|
| Behavior rules | Identity, refusal, tone, wellbeing, search, copyright | ~400 |
| Tool definitions | Bash/Read/Write/Edit/WebSearch/WebFetch + full JSON schemas | ~500 |
| Capability extensions | Skills system, memory system, artifact criteria | ~240 |

**Key behavioral features**

- **Hard copyright limits** -- 15-word per-quote ceiling and one-quote-per-source
  max are marked `CRITICAL` / `NEVER`, not suggestions. The limits appear three
  times in the prompt (rule, self-check, consequence reminder) to resist drift.

- **User wellbeing guardrails** -- More detailed than most safety guidelines:
  no clinical diagnosis, no self-harm substitution techniques, no eating-disorder
  calorie guidance, no crisis-line confidentiality assurances. Each boundary has
  a rationale sentence.

- **Skills system** -- Any file creation or code task must be preceded by a
  `Read` of the relevant SKILL.md. Framed as a "required first step" to resist
  the model defaulting to training knowledge over environment-specific constraints.

- **Memory system** -- File-based cross-session persistence at
  `~/.claude/projects/<project>/memory/MEMORY.md`. Index always in context;
  individual memory files loaded on demand.

- **Search discipline** -- Explicit rules for when to search vs. not (binary
  event checks, current role holders, unrecognized entity rule: must search).
  Search result copyright limits match the inline copyright rules.

- **Anti-formatting bias** -- Explicitly prohibits over-use of bullets, bold,
  and headers. Bullets only when "(a) asked, or (b) content is multifaceted
  enough." Conversational answers stay prose.

- **Rule density** -- Behavior enforced with `NEVER` / `MUST` / `CRITICAL` /
  `STOP` markers throughout, not phrased as preferences. ~40+ hard-stop markers
  in the behavior section alone.

## CN Model API Compatibility

Research conducted June 2026 (Exa) on thinking/reasoning support across CN
model backends used in cc-cn.md variants.

| Model | API Format | Anthropic endpoint | thinking param | budget_tokens | reasoning output |
|-------|-----------|-------------------|----------------|---------------|-----------------|
| DeepSeek | OpenAI + Anthropic | `api.deepseek.com/anthropic` | `thinking.type` | **Ignored** | `reasoning_content` / `thinking` block |
| Kimi (Moonshot) | OpenAI only | none | `thinking.type` + `thinking.keep` | N/A | `reasoning_content` |
| GLM (Zhipu/Z.ai) | OpenAI + Anthropic | `api.z.ai/api/anthropic/v1` | `enable_thinking` | unknown | `reasoning_content` |
| MiMo (Xiaomi) | OpenAI only | none | `thinking.type` | N/A | `reasoning_content` |

**Key implications for proxies**

- **DeepSeek** (Anthropic endpoint): `thinking` supported end-to-end;
  `budget_tokens` silently ignored but model still reasons. `cache_control`
  ignored. CC sending `budget_tokens: 31999` causes no errors.

- **Kimi**: OpenAI-only. Proxy must translate CC's Anthropic `thinking` block
  to Kimi's `{"thinking": {"type": "enabled"}}`. K2.6+ requires
  `reasoning_content` preserved across turns for agent workflows -- dropping
  it causes context degradation (not an immediate error, but quality drops).

- **GLM**: Has Anthropic-compatible endpoint. Thinking mode uses a different
  parameter format (`enable_thinking` for local; API behavior not fully
  documented). Proxy should translate or strip `budget_tokens`.

- **MiMo**: OpenAI-only. **Critical for multi-turn agents**: in conversations
  containing tool calls, `reasoning_content` from the previous assistant message
  **must** be included in the next request or the API returns 400. The current
  cn-auth-proxy (pass-through only) does not handle this -- MiMo's dedicated
  proxy must preserve `reasoning_content` in conversation history.

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
