# cc-cn.md deletions audit

Referenced by `cc-cn.md` header ("Deletions audit: fable-prompts/cc/cn/DELETIONS.md").
This file records every section/tool removed in deriving `cc-cn.md` (CN model backends)
from `cc-anthropic.md` (verbatim CLAUDE-FABLE-5-CC).

Design principle (from the cc-cn.md header): cc-cn.md is LOSSLESS from cc-anthropic.md
except for content that is web-UI-only or Anthropic-infrastructure-only. Behavioral
guidance (refusal, tone, evenhandedness, mistakes, wellbeing, copyright, legal,
knowledge-cutoff) is RETAINED. Only the items below are removed.

Tags:
- [TECHNICAL-FACT] removed feature is physically unavailable on a CN/Claude Code path
  (web-UI rendering, claude.ai-hosted tool) -- a technical fact, not a judgment.
- [RESEARCHED]     removed after researching how the mechanism is delivered.
- [TESTED]         removed after a behavioral test confirmed it is not load-bearing.

Provenance note: this file was created 2026-06-20. Entries 1-7 are RECONSTRUCTED from
the inline `<!-- ... REMOVED [TAG] -->` breadcrumbs that were already in cc-cn.md plus a
header-set diff (cc-anthropic.md vs cc-cn.md). They are not an original change log.
Entries 8-9 are this session's removals, with the live evidence inline.

## Removals

| # | Removed                                  | Tag             | Why (short)                                                        | Inline breadcrumb |
|---|------------------------------------------|-----------------|--------------------------------------------------------------------|-------------------|
| 1 | anthropic_reminders (subsection)         | [RESEARCHED]    | Anthropic safety classifiers are relayed only by Anthropic infra; CN backends (Kimi/DeepSeek/GLM/MiMo) do not relay them. Behavioral safety stays via refusal_handling. | survives in cc-cn |
| 2 | using_WebSearch_tool / image search      | [TECHNICAL-FACT]| Inline image search requires the claude.ai backend; not available on the CN path. | survives in cc-cn |
| 3 | AskUserQuestion tappable-buttons variant | [TESTED]        | CN model APIs do not support the web-UI interactive-button tool. Confirmed via behavioral testing. | consolidated here (was inside tool block) |
| 4 | WebSearch sports (SportRadar)            | [TECHNICAL-FACT]| SportRadar scores API is web-UI only.                              | consolidated here |
| 5 | WebSearch image / display_map / places   | [TECHNICAL-FACT]| Image/map/places widgets are web-UI only.                          | consolidated here |
| 6 | OutputFilePath                           | [TECHNICAL-FACT]| Web-UI file-rendering tool; Claude Code shares paths as text.      | consolidated here |
| 7 | WebFetch weather                         | [TECHNICAL-FACT]| Weather-widget API is web-UI only.                                 | consolidated here |
| 8 | artifact_usage_criteria (~44 lines)      | [TECHNICAL-FACT]| claude.ai artifact system: renders .jsx/.html/.mermaid/.svg in the chat UI, React+Tailwind sandbox, localStorage-fails-in-Claude.ai, {artifact} tags. Claude Code has no artifact rendering layer; it writes real files. File-vs-inline guidance is retained in file_creation_advice. | survives in cc-cn |
| 9 | Tool Definitions block (~215 lines)      | [TESTED]        | ~215 lines of claude.ai web-UI tool schemas (BashInput/CreateFileInput/ViewInput container tools) plus the bracket-invoke call format. Redundant and conflicting: Claude Code injects the REAL CLI tool schemas at runtime. | survives in cc-cn |

## Entry 9 -- behavioral test evidence (2026-06-20)

Claim under test: removing the prose tool block does not break tool-calling on a CN model.

Method: built a cc-cn.md variant with the block absent and ran a forced file-create task
on kimi via Claude Code print mode (`claude_kimi -p ... --system-prompt <variant>
--permission-mode acceptEdits --output-format json`).

Result:
- Serving model (json modelUsage key): kimi-k2.7-code -- confirmed it was Kimi, not a fallback.
- Harness init advertised the REAL tools: Task, AskUserQuestion, Bash, Edit, Write, Read, ...
  i.e. the structured tool schemas come from the Claude Code harness at runtime, not from
  this prose block.
- Kimi created the target file with the exact requested contents -> tool call succeeded with
  the block ABSENT.

Conclusion: the block is not load-bearing for CN tool-calling. Removing it also makes the
cc-cn.md header claim ("only removes web-UI-only tools") true, which it was not while 215
lines of web-UI tool schemas remained.

### Supersede note (honest, non-silent)

Entry 3's original inline breadcrumb stated that the connector-options AskUserQuestion
(the ListAvailableTools companion) was KEPT while only the tappable variant was removed.
Removing the whole Tool Definitions block (entry 9) also removes that kept prose schema.
This is intentional and justified by the same test: AskUserQuestion appeared in the harness
init tools array, so the real AskUserQuestion + ListAvailableTools tools are injected at
runtime regardless of the prose. The prose schema was redundant. This supersedes the earlier
keep-decision rather than silently overriding it.

## Not removed (deliberately retained, despite external suggestions)

These are behavioral, not web-UI/infra, so they stay to preserve the lossless principle:
copyright (CRITICAL_COPYRIGHT_COMPLIANCE), legal_and_financial_advice, user_wellbeing,
product_information (already CN-customized). External analyses suggested cutting some of
these for "CN models do not enforce them"; that would change cc-cn.md from a lossless
behavioral mirror into an opinionated trim, which is out of scope for this file.
