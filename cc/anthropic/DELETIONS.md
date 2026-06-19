# cc-anthropic.md deletions audit

Referenced by `cc-anthropic.md` header ("Deletions audit: fable-prompts/cc/anthropic/DELETIONS.md").
Records what was removed in slimming cc-anthropic.md from the verbatim CLAUDE-FABLE-5-CC
prompt down to a lean prompt for a PRIVATE CLI coding tool on Anthropic-native models.

Design principle: this file feeds Anthropic Claude models (opus/me/api) through Claude
Code. The model's TRAINING already embodies most product/compliance dispositions, and the
Claude Code harness injects the real tools at runtime. So we remove (a) web-UI-only
tools/features that do not exist on the CLI path, (b) Anthropic-internal reminders, and
(c) product/copyright/legal prose the trained model already embodies and a private coding
tool does not need. We KEEP behavioral-quality content (tone, evenhandedness, mistakes),
the safety floor (refusal_handling, harmful_content_safety, user_wellbeing), and the
Claude Code workflow sections.

Important (corrects a common assumption): on the API/CLI path there is NO separate
Anthropic "internal" system prompt layered in. cc-anthropic.md IS the system prompt
(plus the harness tool schemas + the user's CLAUDE.md). Copyright etc. are not redundant
with a hidden company prompt; they are redundant with the model's TRAINED disposition.

Tags:
- [TECHNICAL-FACT] feature is physically unavailable on the CLI path (web-UI rendering,
  claude.ai-hosted tool/widget).
- [RESEARCHED]     Anthropic-infrastructure-only (reminders/classifiers).
- [TESTED]         behaviorally verified not load-bearing (see evidence below).
- [TRAINED]        the model's training already embodies this; explicit prose is
  belt-and-suspenders with little value for a private coding tool.

## Removals (this slim, 2026-06-20)

| Removed                                  | Tag             | Why (short)                                                              |
|------------------------------------------|-----------------|--------------------------------------------------------------------------|
| Tool Definitions block (~535 lines)      | [TESTED]        | claude.ai web-UI tool schemas (container Bash/Write/Read; sports/image/map/weather widgets; OutputFilePath) + bracket-invoke format. The harness injects the real CLI tools at runtime. |
| using_WebSearch_tool (image search, 47L) | [TECHNICAL-FACT]| Inline image search renders in the claude.ai UI; not on the CLI path.    |
| artifact_usage_criteria (~45L)           | [TECHNICAL-FACT]| claude.ai artifact rendering (React/Tailwind, .jsx/.svg/.mermaid UI, {artifact} tags). Claude Code writes real files; file-vs-inline guidance retained in file_creation_advice. |
| CRITICAL_COPYRIGHT_COMPLIANCE (~43L)     | [TRAINED]       | 15-word / one-quote search-reproduction rules. Model is trained to avoid verbatim reproduction; belt-and-suspenders for a private coding tool. Restore if you do heavy web-research quoting. |
| critical_reminders: copyright bullet     | [TRAINED]       | The one HARD-LIMITS restatement bullet, removed for consistency. The search-discipline bullets in that section are KEPT. |
| product_information (~24L)               | [TRAINED]       | Anthropic product/billing/settings blurb for the claude.ai apps. The model can web-search docs.claude.com if asked. |
| anthropic_reminders (~8L)                | [RESEARCHED]    | Reminders/classifiers are delivered only by Anthropic infra to claude.ai sessions, not via a CLI system prompt. |
| legal_and_financial_advice (~4L)         | [TRAINED]       | The model already hedges legal/financial questions; a disclaimer line is not needed. |
| user_wellbeing (~36L)                    | [TRAINED]       | Mental-health / self-harm / disordered-eating / crisis-resource handling. The model retains these dispositions from training; near-zero trigger on a private single-user CLI coding tool. Removed on a later explicit request; refusal_handling + harmful_content_safety remain. |

Net: 1138 -> 416 lines, 94333 -> 40265 bytes (about 57% smaller).

## [TESTED] evidence -- tool block not load-bearing (dual path, 2026-06-20)

Claim: removing the prose tool block does not break tool-calling.

Method: built a cc-anthropic variant with only the tool block removed, ran a forced
file-create task under Claude Code print mode with `--output-format json`, on two backends.

Results:
- Anthropic path: serving model `claude-haiku-4-5-20251001`; file created with exact contents.
- CN path (cross-check): serving model `kimi-k2.7-code`; file created with exact contents;
  the harness init advertised the real Task/Bash/Edit/Write/Read/AskUserQuestion tools.

Conclusion: the structured tool schemas come from the harness at runtime, not from this
prose block, on both backends. Removal is safe; a stronger model needs the prose even less.

## NOT removed (deliberately kept)

- Safety floor: refusal_handling, harmful_content_safety. (user_wellbeing was cut on a
  later explicit request -- see the removals table; the model's trained wellbeing
  disposition remains, and refusal_handling + harmful_content_safety still cover the floor.)
- Behavioral quality: tone_and_formatting + lists_and_bullets, evenhandedness,
  responding_to_mistakes_and_criticism, knowledge_cutoff, core_search_behaviors,
  search_usage_guidelines, critical_reminders (minus the one copyright bullet).
- Claude Code workflow: memory_system, mcp_servers, computer_use (skills, file_creation,
  file_handling, producing_outputs, sharing_files, package_management), available_skills
  (the user's curated list), filesystem_configuration, anthropic_api_call_reference,
  citation_instructions, Identity Preamble, User Context.

## Pre-existing issues (not introduced here, not fixed here)

- Line 1 title and many kept sections still contain non-ASCII (em dashes, smart quotes)
  inherited from the source prompt. Out of scope for this slim (the rule is: do not add
  new non-ASCII; pre-existing in untouched text is not in scope).
- An orphaned fragment line ("hould never use [voice-note] blocks ...") sits just under the
  frontmatter; it predates this change.
