# MarkdownLint One-Document Cleanup Operating Contract

## Purpose

This document defines a reusable operating contract for a local coding agent that cleans Markdown formatting issues with MarkdownLint in VS Code or an equivalent local lint environment.

The system has one goal: align exactly one Markdown document with the active MarkdownLint rules while preserving the document’s existing content.

## Scope

### In scope

The agent must:

- process exactly one Markdown document per run;
- use the active MarkdownLint rules for the target document;
- apply formatting-only fixes that preserve content;
- report diagnostics that cannot be safely fixed;
- recheck target-scoped diagnostics after editing;
- produce the required cleanup report.

### Out of scope

The agent must not:

- clean more than one Markdown document in a run;
- perform broad repository-wide cleanup;
- rewrite prose for clarity, tone, style, concision, or grammar;
- add, remove, summarize, or reorganize substantive content;
- alter examples, code, commands, configuration snippets, or inline code content;
- change heading names, table cell values, link text, or link targets unless this contract explicitly permits the exact formatting-only change;
- insert MarkdownLint disable comments unless the user explicitly authorizes that mitigation;
- change MarkdownLint configuration unless the user explicitly asks for configuration changes.

## Operating Principles

### Content authority

The target document is authoritative. The agent may correct formatting that MarkdownLint flags. The agent must not improve content.

### One-document boundary

The target document is the only editable document. The agent may read only:

| Context source | Permitted use |
| --- | --- |
| Target Markdown document | Required edit target and content-preservation source. |
| MarkdownLint diagnostics for the target | Required source for rule violations. |
| Active MarkdownLint configuration for the target | Required source for rule settings and enabled rules. |
| Companion reference document | Required operating contract. |
| Local script or package metadata | Permitted only when needed to run MarkdownLint for the target and only when it does not expand cleanup scope. |

The agent must not read unrelated Markdown files to infer style, normalize the repository, or improve consistency outside the target document.

### Smallest compliant change

For each diagnostic, the agent must choose the smallest change that satisfies the active rule and preserves content.

Tie-break order:

1. Whitespace-only change outside code, links, and table cell content.
2. Markdown delimiter or marker change that preserves rendered text and structure.
3. Local structural formatting change, such as adding a blank line around a block.
4. Report unresolved instead of changing content.

## Inputs

| Input | Required | Default | Validation |
| --- | ---: | --- | --- |
| `target_markdown_file` | Yes | None | Must identify exactly one existing Markdown file. |
| `companion_reference_file` | Yes | None | Must be readable before edits begin. |
| `markdownlint_configuration_source` | No | Active target-resolved configuration when available | Must be the configuration actually used for target diagnostics. |
| `diagnostics_source` | No | VS Code target diagnostics, then local target-only CLI diagnostics | Must be scoped to the target document only. |
| `user_authorized_content_changes` | No | Empty set | Must be explicit, rule-specific, and target-specific. |

If the input identifies zero Markdown files, more than one Markdown file, or a directory, the agent must stop with `TARGET_NOT_EXACTLY_ONE_DOCUMENT`.

## Outputs

| Output | Required | Description |
| --- | ---: | --- |
| Modified target document | Conditional | Written only when safe or permitted conditional formatting fixes exist. |
| Cleanup report | Yes | Markdown report using the required format. |
| Unresolved diagnostic list | Yes | Empty only when no unresolved diagnostics remain. |
| Verification result | Yes | States whether target-only MarkdownLint recheck completed. |

The agent must not emit a success claim unless target-only verification completed or the report clearly marks verification as unavailable.

## Active MarkdownLint Configuration

## Configuration discovery

The agent must determine the active MarkdownLint rules for the target document in this order:

1. Use target-scoped diagnostics already produced by VS Code MarkdownLint when available.
2. Use a local MarkdownLint command or workspace lint task only when it can be run for the target file alone.
3. Inspect target-applicable MarkdownLint configuration only when needed to understand rule settings or diagnostics.

The agent must record the configuration source as one of these values:

| Value | Meaning |
| --- | --- |
| `workspace_config_resolved` | A workspace MarkdownLint configuration was resolved for the target. |
| `workspace_settings_resolved` | VS Code workspace settings were resolved for the target. |
| `user_settings_unseen` | Diagnostics exist, but user-level settings are not directly visible to the agent. |
| `tool_default` | No explicit configuration was found, and the lint tool supplied default rules. |
| `configuration_unresolved` | The agent could not determine which rules are active. |

## Configuration failure behavior

| Condition | Required behavior |
| --- | --- |
| No MarkdownLint configuration is found, but target diagnostics are available | Proceed using the diagnostics and record `tool_default` or `user_settings_unseen`, as applicable. |
| No configuration and no diagnostics are available | Stop with `LINT_CONTEXT_UNAVAILABLE`. Do not edit. |
| Multiple configurations are present | Use only the configuration that the linter resolves for the target. If the active source cannot be determined, stop with `CONFIGURATION_AMBIGUOUS`. |
| Configuration parse error | Stop with `CONFIGURATION_INVALID`. Do not edit. |
| Configuration requires repository-wide linting | Do not run repository-wide linting. Run target-only linting or stop with `TARGET_SCOPED_LINT_UNAVAILABLE`. |

## Diagnostic Classification Model

Every diagnostic must be classified before editing.

| Classification | Definition | Edit permission |
| --- | --- | --- |
| `safe_formatting` | A change that modifies only Markdown formatting and cannot change wording, meaning, examples, links, code, commands, table cell content, or section intent. | Apply without additional user authorization. |
| `conditional_formatting` | A change that appears formatting-only but could affect meaning if context is misread. | Apply only when the formatting-only path is clear from local context. Otherwise report unresolved. |
| `content_sensitive` | A change that adds, removes, rewrites, renames, restructures, or reinterprets substantive content. | Do not apply unless the user explicitly authorized that exact content-affecting change. |

### Classification algorithm

For each diagnostic, in ascending order by line, column, rule ID, and message:

1. Identify the affected region: prose, heading, list marker, list body, blockquote, table syntax, table cell, link syntax, link text, link destination, code fence marker, code block body, inline code, image syntax, HTML, front matter, or reference definition.
1. Identify the minimal candidate fix.
1. Classify the candidate fix by observable effect.
1. Apply the rule matrix override in this document.
1. If two classifications apply, choose the more conservative classification using this precedence:

```text
content_sensitive > conditional_formatting > safe_formatting
```

1. If the fix depends on guessing intent, report unresolved.

## Rule Handling Matrix

This table gives default classifications for common MarkdownLint rules. The active configuration still controls whether a rule is enabled and what parameters apply.

| Rule | Default classification | Permitted formatting-only path | Must report unresolved when |
| --- | --- | --- | --- |
| `MD001` heading increment | `content_sensitive` | None by default. | Fixing requires changing heading levels or inserting headings. |
| `MD003` heading style | `safe_formatting` | Convert heading marker style while preserving heading level and exact heading text. | The conversion would change heading level, title text, or section structure. |
| `MD004` unordered list style | `safe_formatting` | Change only bullet marker characters according to active style. | Marker change would alter nesting or list grouping. |
| `MD005` list indentation consistency | `conditional_formatting` | Adjust indentation when list membership is unambiguous and item text is unchanged. | Indentation determines whether text is a code block, nested list, or continuation paragraph. |
| `MD007` unordered list indentation | `conditional_formatting` | Adjust unordered-list indentation when hierarchy is unambiguous. | Nesting intent is unclear or code-block membership may change. |
| `MD009` trailing spaces | `safe_formatting` outside code | Remove flagged trailing spaces outside code and outside intentional hard breaks. | The spaces are inside code, command output, diagrams, or intentional hard-break formatting. |
| `MD010` hard tabs | `conditional_formatting` | Replace tabs outside code or in non-semantic indentation when configured. | Tabs are inside code, Makefiles, command examples, alignment-sensitive text, or unknown language blocks. |
| `MD011` reversed links | `conditional_formatting` | Swap delimiters only when exact text and URL are preserved and intent is obvious. | The text may be intentionally literal or the URL/text boundary is unclear. |
| `MD012` multiple blank lines | `safe_formatting` | Reduce consecutive blank lines outside code to the configured maximum. | Blank lines occur inside code or preformatted content. |
| `MD013` line length | `conditional_formatting` | Wrap prose without changing words, inline code, links, table cells, headings, or examples. | The long line is code, command text, a URL, a table row, a heading, or semantically structured text. |
| `MD014` command prompt markers | `content_sensitive` | None by default. | Removing `$` would change command examples or presentation semantics. |
| `MD018` missing heading space | `safe_formatting` | Insert required space after ATX heading marker. | The line may be code or literal text rather than a heading. |
| `MD019` multiple heading spaces | `safe_formatting` | Reduce marker-to-text spacing to configured style. | The line may be code or literal text rather than a heading. |
| `MD020` closed heading missing spaces | `safe_formatting` | Insert required internal spaces around closed ATX markers. | The line may be code or literal text rather than a heading. |
| `MD021` closed heading multiple spaces | `safe_formatting` | Reduce internal closed-heading spacing to configured style. | The line may be code or literal text rather than a heading. |
| `MD022` headings surrounded by blank lines | `safe_formatting` | Add or remove surrounding blank lines according to active rule settings. | The heading appears inside a context where adding a blank line changes list, quote, or code structure. |
| `MD023` heading starts at beginning | `conditional_formatting` | Remove unintended leading spaces only when the line is clearly a heading. | Indentation may intentionally place the line in code, quote, or list context. |
| `MD024` duplicate headings | `conditional_formatting` | Append a short, relevant slug only to duplicate heading instances after the first occurrence (example: `### Trust and provenance (slsa)`), keeping heading text and section intent intact. | Deduplicating would change section identity, ambiguity remains about which context to preserve, or wording changes are required beyond a short suffix. |
| `MD025` multiple top-level headings | `content_sensitive` | None by default. | Fixing requires changing heading levels or document title structure. |
| `MD026` heading punctuation | `content_sensitive` | None by default. | Fixing requires changing heading text. |
| `MD027` blockquote marker spacing | `safe_formatting` | Normalize spaces after `>` while preserving quoted text. | Spacing is inside code or preformatted quote content. |
| `MD028` blank line in blockquote | `conditional_formatting` | Add blockquote markers to blank quote lines only when quote scope is clear. | It is unclear whether the blank line intentionally ends the quote. |
| `MD029` ordered list prefix | `conditional_formatting` | Renumber only when numbers are Markdown presentation and item text is unchanged. | Numbers encode priorities, versions, procedures, references, or values. |
| `MD030` list marker spacing | `safe_formatting` | Adjust spaces after list markers according to active settings. | Spacing is alignment-sensitive or changes continuation text. |
| `MD031` fenced code block blank lines | `safe_formatting` | Add blank lines around fenced code blocks. | The fence is inside a tight list or quote where spacing changes rendering semantics. |
| `MD032` lists surrounded by blank lines | `conditional_formatting` | Add blank lines around lists when list grouping remains unchanged. | The change would convert a tight list to a loose list or alter paragraph grouping. |
| `MD033` inline HTML | `content_sensitive` | None by default. | Fixing requires replacing, deleting, or rewriting HTML. |
| `MD034` bare URL | `conditional_formatting` | Wrap the exact URL in angle brackets or preserve exact target in Markdown link syntax. | The URL boundary includes punctuation, line wrapping, display text, or intentional bare formatting. |
| `MD035` horizontal rule style | `safe_formatting` | Change only the horizontal-rule marker to the configured style. | The line may be meaningful text rather than a horizontal rule. |
| `MD036` emphasis used as heading | `content_sensitive` | None by default. | Fixing requires converting prose emphasis into a heading or changing document structure. |
| `MD037` spaces inside emphasis markers | `conditional_formatting` | Remove marker-adjacent spaces when emphasized text remains exact. | Spaces may be intentional text content. |
| `MD038` spaces inside code spans | `content_sensitive` | None by default. | Fixing changes inline code content or command syntax. |
| `MD039` spaces inside link text | `conditional_formatting` | Trim marker-adjacent spaces when link text remains the same words. | Spaces may be intentional or part of visible link text. |
| `MD040` fenced code language | `conditional_formatting` | Add an existing or unambiguous language label; use `text` only for plain text blocks. | The language is uncertain or a wrong label could misrepresent code. |
| `MD041` first line top-level heading | `content_sensitive` | None by default. | Fixing requires adding, moving, or changing a document title. |
| `MD042` empty links | `content_sensitive` | None by default. | Fixing requires adding or removing link destinations. |
| `MD043` required heading structure | `content_sensitive` | None by default. | Fixing requires adding, removing, renaming, reordering, or releveling headings. |
| `MD044` proper names capitalization | `content_sensitive` | None by default. | Fixing changes wording or capitalization. |
| `MD045` image alt text | `content_sensitive` | None by default. | Fixing requires writing new alt text or changing image semantics. |
| `MD046` code block style | `conditional_formatting` | Convert indented versus fenced code only when code bytes and code meaning are preserved. | Indentation, fence choice, or language context affects code interpretation. |
| `MD047` final newline | `safe_formatting` | Ensure exactly one final newline if required. | Never, unless file format is not plain text. |
| `MD048` code fence style | `conditional_formatting` | Change fence marker style while preserving code block body and nesting. | Existing content contains the target fence delimiter or nesting is ambiguous. |
| `MD049` emphasis style | `safe_formatting` | Change emphasis delimiters while preserving emphasized text. | Delimiter change would alter parsing or visible text. |
| `MD050` strong style | `safe_formatting` | Change strong delimiters while preserving strong text. | Delimiter change would alter parsing or visible text. |
| `MD051` link fragments | `conditional_formatting` | Correct fragment syntax only when exact target heading is unambiguous. | Target intent is unclear or fixing requires changing heading text. |
| `MD052` undefined references | `content_sensitive` | None by default. | Fixing requires adding references, changing labels, or changing links. |
| `MD053` unused references | `content_sensitive` | None by default. | Fixing requires deleting reference definitions that may be intentional content. |
| `MD054` link and image style | `conditional_formatting` | Convert style while preserving exact text, URL, title, alt text, and reference meaning. | Conversion would remove reuse, change link definitions, or alter displayed text. |
| `MD055` table pipe style | `safe_formatting` | Add, remove, or normalize leading/trailing pipes without changing cells. | A row’s cell boundaries are ambiguous. |
| `MD056` table column count | `content_sensitive` | None by default. | Fixing requires adding, deleting, splitting, or merging table cells. |
| `MD058` table surrounding blank lines | `safe_formatting` | Add blank lines before or after tables when block boundaries are clear. | Adjacent text is intentionally part of the table or boundary is ambiguous. |
| `MD059` descriptive link text | `content_sensitive` | None by default. | Fixing requires writing or changing link text. |
| `MD060` table column style | `safe_formatting` | Normalize table pipe spacing or delimiter spacing while preserving cells and alignment markers. | Pipe positions or escaped pipes make cell boundaries ambiguous. |
| Unknown enabled rule | `conditional_formatting` by default | Apply only if the candidate fix satisfies the classification algorithm. | The rule requires content judgment or the active rule behavior is unclear. |

## Content Preservation Contract

The agent must preserve:

- wording;
- meaning;
- factual content;
- section intent;
- examples;
- code block body content;
- command text and command output;
- configuration snippets;
- inline code content;
- link text and link targets;
- image targets and alt text;
- table cell values;
- document ordering;
- heading text;
- terminology.

The agent may change:

| Change type | Allowed when |
| --- | --- |
| Whitespace outside content-bearing regions | Required by an active lint rule and not semantically meaningful. |
| Blank lines around headings, lists, code fences, tables, and blocks | Required by an active lint rule and does not alter grouping semantics. |
| Markdown delimiter style | Rendered meaning and visible text remain unchanged. |
| List marker style | List hierarchy and item text remain unchanged. |
| Table pipe spacing | Cell count, cell order, cell text, and alignment markers remain unchanged. |
| Final newline | Required by active lint rule. |

The agent must not silently apply a change when a reasonable reader could interpret the change as content-affecting.

## Deterministic Cleanup Workflow

### Step 1. Confirm target

The agent must confirm that the run targets exactly one Markdown file.

Failure behavior:

| Condition | Result |
| --- | --- |
| No target file | Stop with `TARGET_NOT_EXACTLY_ONE_DOCUMENT`. |
| More than one target file | Stop with `TARGET_NOT_EXACTLY_ONE_DOCUMENT`. |
| Target is not Markdown | Stop with `TARGET_NOT_MARKDOWN`. |
| Target is missing or unreadable | Stop with `TARGET_UNREADABLE`. |

### Step 2. Identify active lint rules

The agent must identify target-scoped MarkdownLint diagnostics and the active configuration source.

The agent must not run a repository-wide lint command unless the user explicitly authorized repository-wide diagnostics. Even with authorization, edits must remain limited to the target file.

### Step 3. Read target document

The agent must read enough of the target document to preserve content for every changed region.

If the whole document fits available context, read the whole document. If the document is too large for a single context window, use local file operations to inspect the full file and changed regions. If the agent cannot inspect changed regions and enough surrounding context to preserve content, stop with `CONTEXT_LIMIT_UNSAFE`.

### Step 4. Collect diagnostics

The agent must collect the pre-cleanup diagnostics for the target document.

Diagnostic ordering must be stable:

```text
1. line number ascending
2. column number ascending
3. rule ID ascending lexical order
4. diagnostic message ascending lexical order
```

### Step 5. Classify diagnostics

The agent must classify every diagnostic using the classification model and rule matrix.

The agent must not rely on tool auto-fix status alone. A fixable rule can still be content-sensitive under this contract.

### Step 6. Apply safe fixes

The agent must apply all non-conflicting `safe_formatting` fixes first.

If two safe fixes conflict, the agent must apply the one that fixes the earlier diagnostic in stable order, re-run diagnostics, and reclassify remaining diagnostics.

### Step 7. Apply conditional fixes

The agent may apply `conditional_formatting` fixes only when:

- the affected content is locally clear;
- the change preserves all content listed in the content-preservation contract;
- the change is the smallest lint-compliant change;
- no rule-specific unresolved condition applies.

If any condition fails, the agent must report the diagnostic unresolved.

### Step 8. Block content-sensitive fixes

The agent must not apply `content_sensitive` fixes unless the user explicitly authorized that exact rule and exact kind of content change for the target file.

MarkdownLint disable comments are content-sensitive. The agent must report them as possible mitigations only when useful; it must not insert them by default.

### Step 9. Recheck diagnostics

The agent must re-run target-scoped MarkdownLint diagnostics after edits.

Iteration rule:

```text
The agent may perform up to 5 cleanup cycles.
A cycle is classify -> edit -> target-only lint recheck.
Stop earlier when no safe or permitted conditional fixes remain.
```

After 5 cycles, remaining diagnostics must be reported unresolved with reason `MAX_CLEANUP_CYCLES_REACHED` unless another more specific reason applies.

### Step 10. Produce report

The report must include:

- target file path;
- configuration source;
- diagnostics source;
- result status;
- files edited;
- applied changes table;
- unresolved diagnostics table;
- verification summary.

## Failure Handling

| Failure state | Required behavior | Report code |
| --- | --- | --- |
| No MarkdownLint configuration found | Use target diagnostics if available; otherwise stop. | `LINT_CONTEXT_UNAVAILABLE` when no diagnostics exist. |
| Multiple configurations present | Use linter-resolved target config; otherwise stop. | `CONFIGURATION_AMBIGUOUS` |
| Diagnostics unavailable | Do not edit unless the user explicitly requests manual formatting without lint verification. | `DIAGNOSTICS_UNAVAILABLE` |
| Local lint command unavailable | Use VS Code target diagnostics if available; otherwise stop. | `TARGET_SCOPED_LINT_UNAVAILABLE` |
| Rule conflicts with content preservation | Leave unresolved and explain the content affected. | `CONTENT_PRESERVATION_CONFLICT` |
| Rule requires changing meaning | Leave unresolved. | `CONTENT_SENSITIVE_DIAGNOSTIC` |
| Table cannot be safely reformatted | Leave unresolved; preserve table content. | `TABLE_FORMAT_UNSAFE` |
| Code block cannot be safely reformatted | Leave unresolved; preserve code. | `CODE_FORMAT_UNSAFE` |
| Link cannot be safely reformatted | Leave unresolved; preserve link text and target. | `LINK_FORMAT_UNSAFE` |
| Intentional formatting conflicts with lint rule | Leave unresolved; identify possible explicit waiver or config change. | `INTENTIONAL_FORMAT_CONFLICT` |
| Target too large for safe inspection | Stop or edit only diagnostics whose changed regions can be fully inspected; report limit. | `CONTEXT_LIMIT_UNSAFE` |
| Post-cleanup lint recheck unavailable | Report verification incomplete; do not claim clean. | `POSTCHECK_UNAVAILABLE` |

## Review Process

Before finalizing, the agent must review the diff and verify:

| Check | Pass condition |
| --- | --- |
| Edited file set | Only the target document was edited. |
| Wording preservation | No prose wording changed except explicitly authorized content-sensitive changes. |
| Code preservation | Code block bodies, commands, command output, inline code, and config snippets are unchanged unless explicitly authorized. |
| Link preservation | Link text and targets are unchanged unless explicitly authorized or conditionally safe under the rule matrix. |
| Table preservation | Table cell text, cell order, and row order are unchanged unless explicitly authorized. |
| Heading preservation | Heading text and document order are unchanged unless explicitly authorized. |
| Diagnostic handling | Every pre-cleanup diagnostic is fixed, transformed into a new diagnostic after recheck, or listed unresolved. |
| Verification | Target-only lint recheck completed or is explicitly marked unavailable. |

## Required Cleanup Report

The agent must output this exact report shape:

```markdown
## MarkdownLint Cleanup Report

- Target document: `<path>`
- Configuration source: `<workspace_config_resolved | workspace_settings_resolved | user_settings_unseen | tool_default | configuration_unresolved>`
- Diagnostics source: `<VS Code Problems | CLI | unavailable>`
- Result: `<clean | clean_with_unresolved | no_safe_changes | stopped>`
- Files edited: `<target only | none>`

### Changes Applied

| Rule | Location | Classification | Change |
| --- | --- | --- | --- |
| `<rule>` | `<line[:column]>` | `<safe_formatting | conditional_formatting>` | `<concise description>` |

### Unresolved Diagnostics

| Rule | Location | Classification | Reason left unresolved | Required user decision |
| --- | --- | --- | --- | --- |
| `<rule>` | `<line[:column]>` | `<classification>` | `<reason>` | `<specific authorization or config decision>` |

### Verification

- Pre-cleanup diagnostic count: `<n>`
- Post-cleanup diagnostic count: `<n or unavailable>`
- Target-only lint recheck completed: `<yes | no>`
- Content-preservation review completed: `<yes | no>`
```

Result meanings:

| Result | Meaning |
| --- | --- |
| `clean` | Target-only lint recheck completed and no diagnostics remain. |
| `clean_with_unresolved` | Safe and permitted conditional fixes were applied, and remaining diagnostics are listed. |
| `no_safe_changes` | Diagnostics exist, but none can be fixed under this contract without user authorization. |
| `stopped` | A required precondition failed before safe editing could proceed. |

## Acceptance Criteria

A cleanup run is complete only when all applicable criteria pass:

| ID | Criterion | Pass condition |
| --- | --- | --- |
| AC-001 | One-document scope | Exactly one Markdown document was targeted. |
| AC-002 | No unrelated edits | No file other than the target document was edited. |
| AC-003 | Context boundary | The agent used only permitted context sources. |
| AC-004 | Active rule source | The active MarkdownLint diagnostics or unresolved diagnostic source is reported. |
| AC-005 | Classification | Every diagnostic considered for editing was classified before change. |
| AC-006 | Safe-first workflow | Safe fixes were applied before conditional fixes. |
| AC-007 | Content-sensitive protection | No content-sensitive fix was applied without explicit authorization. |
| AC-008 | Content preservation | Wording, meaning, code, commands, links, table cells, heading text, examples, and order were preserved. |
| AC-009 | Smallest change | Each applied change was the smallest lint-compliant change available under this contract. |
| AC-010 | Target-only recheck | MarkdownLint was re-run for the target document only, or verification was reported unavailable. |
| AC-011 | Unresolved reporting | Every remaining diagnostic is listed with a reason and required user decision. |
| AC-012 | Report format | The final report follows the required report shape. |
| AC-013 | Two-implementer consistency | Two competent agents using the same prompt, companion document, target file, and active MarkdownLint configuration would produce materially equivalent formatting changes and materially equivalent unresolved-diagnostic reports. |

## Two-Independent-Implementers Test

This system passes the two-independent-implementers test when two local agents, given the same target document, active MarkdownLint configuration, diagnostics, cleanup prompt, and companion reference document, satisfy all of these conditions:

| Dimension | Required equivalence |
| --- | --- |
| Scope | Both edit the same single target file and no other file. |
| Safe fixes | Both apply the same safe formatting fixes, except for whitespace-equivalent changes with identical Markdown rendering. |
| Conditional fixes | Both apply a conditional fix only when the same local evidence makes the formatting-only path clear. |
| Content-sensitive diagnostics | Both leave the same content-sensitive diagnostics unresolved unless the same explicit authorization exists. |
| Reporting | Both report the same unresolved rules, affected locations, and required user decisions. |
| Verification | Both report whether target-only postcheck passed, failed, or was unavailable. |

## Default Conservative Rule

When this contract, the active MarkdownLint diagnostic, or the target document leaves doubt, the agent must preserve content and report the diagnostic unresolved.
