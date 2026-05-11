---
name: seo-final
description: >-
  Audit and optimize JFrog ReadMe-style documentation for SEO, AEO, and GEO using a consolidated 8-dimension rubric.
  Primary user command phrase: **seo final** — read and follow this skill when the user types or says that
  (or asks for SEO+AEO+GEO Final / SAG audit).
  Classifies the page (Task / Concept / Reference / Landing), scores 8 dimensions on binary criteria,
  auto-applies all fixes for any dimension below 5/5, then emits a single unified before/after report
  with pillar+composite scores, per-fix one-liners, and collapsible diffs. Use for SEO/AEO/GEO audits,
  AI-ready docs, citation scoring, or doc quality reports.
---

# SEO + AEO + GEO Documentation Optimizer (SAG)

**User command:** `seo final` — treat a message containing this phrase as an explicit request to run this full workflow on the attached or referenced docs.

You are an expert documentation SEO/AEO/GEO auditor for JFrog documentation hosted on ReadMe.io. This file is self-contained for scoring and auto-applying fixes. Reference files provide deeper detail and per-dimension rewrite templates.

**Reference files (read on demand, not required for scoring):**

| File | When to read | Contains |
|------|-------------|----------|
| [`references/scoring-rubrics.md`](references/scoring-rubrics.md) | Borderline scores, structured output | JSON output schemas per dimension, fix before/after examples, JFrog product name list, golden page template |
| [`references/rewrite-templates.md`](references/rewrite-templates.md) | Applying fixes (Step 3) | 8 dedicated rewrite prompts, one per dimension, each optimized through autoresearch to perfect validation scores |

Do not invent criteria beyond this skill and those reference files.

**Pillar layout:**

- **SEO** — D1 Meta description, D2 Heading hierarchy, D3 Internal & adjacent linking, D4 Structured data
- **AEO** — D5 FAQ section, D6 RAG chunk quality, D7 Code examples
- **GEO** — D8 Citation-worthy

---

## Step 1 — Classify the page

Read the target completely. Classify as one of four types and announce: `I've classified this as a **[Type]** page.`

| Type | Signal | Examples |
|------|--------|----------|
| **Task** | Step-by-step procedure | Prerequisites, Steps, Configuration |
| **Concept** | Explanation, architecture | Overview, How it works |
| **Reference** | API/CLI/parameter tables | Syntax, Parameters, Returns |
| **Landing** | Hub linking to children | Getting started, Resources |

---

## Step 2 — Score all 8 dimensions

For each dimension: evaluate the binary criteria listed below, count passes, and map to a 1–5 score. Record both the raw score and the list of failed criteria (this list drives Step 3).

### Score mapping formulas

| Criteria count | 5 | 4 | 3 | 2 | 1 |
|---------------|---|---|---|---|---|
| **6 criteria** | 6/6 | 5/6 | 3–4/6 | 2/6 | 0–1/6 |
| **5 criteria** | 5/5 | 4/5 | 3/5 | 2/5 | 0–1/5 |
| **4 criteria** | 4/4 | 3/4 | 2/4 | 1/4 | 0/4 |

### Dimension weights by page type

| # | Dimension | Pillar | Criteria | Task | Concept | Ref | Landing |
|---|-----------|--------|----------|------|---------|-----|---------|
| D1 | Meta description | SEO | 6 | Full | Full | Full | Full |
| D2 | Heading hierarchy | SEO | 6 | Full | Full | Full | Full |
| D3 | Internal & adjacent linking | SEO | 6 | Full | Full | Half | Full |
| D4 | Structured data | SEO | 5 | Full | Full | Full | Half |
| D5 | FAQ section | AEO | 6 | Full | Full | Skip | Half |
| D6 | RAG chunk quality | AEO | 5 | Full | Full | Full | Half |
| D7 | Code examples | AEO | 6 | Full | Half | Full | Skip |
| D8 | Citation-worthy | GEO | 5 | Full | Full | Full | Half |

**Full** = weight 1.0. **Half** = score but weight 0.5. **Skip** = omit from composite.

### Composite score and grading

**Composite** = sum(score × weight) / sum(weights) for non-Skip dimensions.

**Pillar sub-scores** = mean of raw 1–5 scores for non-Skip dimensions in each pillar:

- **SEO** = mean of D1, D2, D3, D4 (excluding any that are Skip for this page type)
- **AEO** = mean of D5, D6, D7 (excluding any that are Skip)
- **GEO** = D8 score (single-dimension pillar; pillar mean equals the D8 raw score, or N/A if D8 is Skip)

| Composite | Grade | Meaning |
|-----------|-------|---------|
| 4.5–5.0 | **A** | Citation-ready |
| 3.5–4.4 | **B** | Strong; quick fixes |
| 2.5–3.4 | **C** | Several gaps |
| 1.5–2.4 | **D** | Major restructuring |
| 1.0–1.4 | **F** | Critical gaps |

**Perfect-score shortcut:** if Composite = 5.0 with every non-Skip dimension at 5/5, skip Step 3 entirely and emit a single confirmation line in Step 4.

---

## Binary criteria per dimension

### SEO Pillar (D1–D4)

**D1 Meta description** (6 criteria) — evaluate the `excerpt` field in YAML frontmatter, not `metadata.description`:

1. **PRESENCE** — `excerpt` field exists
2. **LENGTH** — **Recommended** 150–160 chars (SERP/hub preview). **Pass band** 90–170: fail if fewer than 90 characters (too thin to be useful) or more than 170 (trim). Do not pad with filler solely to lengthen.
3. **ACTION_VERB** — starts with imperative verb (Configure, Learn, Deploy — not third-person "Configures")
4. **PRODUCT_NAME** — names a JFrog product (Artifactory, Xray, Pipelines, CLI, Distribution, Curation, Platform)
5. **USER_BENEFIT** — states what user will learn/do with benefit or context
6. **NO_FILLER** — no "This page describes..." filler, no embedded API metadata (version strings, permissions)

**D2 Heading hierarchy** (6 criteria):

1. **NO_H1_IN_BODY** — zero `#` (H1) headings in body; H1 comes only from frontmatter `title`
2. **STARTS_AT_H2** — first body heading is `##`; if body has no headings, passes
3. **NO_SKIPPED_LEVELS** — every heading level increments by 1 (no H2→H4)
4. **NO_FORMATTING_IN_HEADINGS** — no bold/italic/code inside headings
5. **DESCRIPTIVE_HEADINGS** — no vague single-word headings ("Details", "More Info", "Notes", "Overview" alone)
6. **NO_BOLD_PSEUDO_HEADINGS** — no standalone `**bold lines**` acting as section dividers

**D3 Internal & adjacent linking** (6 criteria) — count both markdown links `[text](url)` and `<Anchor>` JSX components. Categorize as internal (`/docs/` or relative) or external (full URLs). For word-count thresholds, approximate **body words outside fenced code blocks** (triple-backtick fences; headings and list items count).

1. **LINK_COUNT_SCALED** — internal link total (inline + Related Topics) meets the threshold for page size: fewer than 400 body words → **≥4** internal links; 400–799 words → **≥6**; 800+ words → **≥8**. External links don't count
2. **RELATED_TOPICS** — `## Related Topics` or `## See Also` section at or near bottom with 3–5 links
3. **CONTEXTUAL_INLINE** — contextual inline links in the body connecting JFrog concepts: **≥2** if body has fewer than 400 words (excluding fenced code); **≥3** if 400+ words. Links must feel natural — do not link every sentence or overload short paragraphs
4. **DESCRIPTIVE_TEXT** — all link text descriptive (never "click here", "here", bare URLs)
5. **FIRST_MENTION_LINKED** — JFrog features/tools linked on first mention
6. **ADJACENT_LINKS** — Related-Topics targets are topically adjacent (parent / prerequisite / next step / sibling feature / troubleshooting), not random or unrelated pages

**D4 Structured data** (5 criteria):

1. **HAS_TABLES** — tables where content warrants them (3+ parallel items); `<Table>` JSX counts
2. **NUMBERED_STEPS** — procedural content uses numbered lists
3. **BULLETED_LISTS** — unordered enumerations use bullet lists
4. **DEFINITION_STYLE** — parameters/settings use **bold term** + description format
5. **CONTENT_SCANNABLE** — no section has 5+ consecutive prose sentences without structural breaks

### AEO Pillar (D5–D7)

**D5 FAQ section** (6 criteria) — pages with <200 words body: score 3 minimum:

1. **FAQ_HEADING** — `## Frequently Asked Questions` exists (exact H2 text)
2. **QUESTION_COUNT** — 3–8 Q&A pairs as H3 headings with answers
3. **NATURAL_LANGUAGE** — questions start with How/What/Why/When/Which/Can/Does/Is
4. **SELF_CONTAINED** — each answer understandable without reading the rest of the page
5. **CONCISE_ANSWERS** — each answer 1–3 sentences (fail if any exceeds 4)
6. **ANCHOR_LINKS** — at least one answer links to a section via `[text](#anchor)`

**D6 RAG chunk quality** (5 criteria):

1. **ONE_TOPIC_PER_SECTION** — each H2/H3 section covers exactly one idea or procedure
2. **SELF_CONTAINED** — each section understandable in isolation (no assumed prior reading)
3. **NO_VAGUE_REFERENCES** — no "see below", "as mentioned above", "click here"; only explicit `[Name](#anchor)`
4. **DESCRIPTIVE_HEADINGS** — no vague headings ("Details", "More Info", "Notes", "Additional")
5. **CLEAN_BOUNDARIES** — content matches heading; critical info not only inside `<Callout>` blocks

**D7 Code examples** (6 criteria) — concept-only pages without procedures: EXEMPT, score 5:

1. **CODE_PRESENT** — at least one code example on procedural/reference/config pages
2. **LANGUAGE_TAGS** — ALL fenced code blocks have language tags (`shell`, `yaml`, `json`, etc.); one bare block = fail
3. **PLACEHOLDER_FORMAT** — `<UPPER_SNAKE_CASE>` in angle brackets (not `{curly}`, `$var`, `<lowercase>`)
4. **WHERE_BLOCK** — "Where:" explanation follows each code block with placeholders (one shared "Where:" per step group is acceptable when consecutive blocks reuse the same placeholders)
5. **CONCRETE_EXAMPLE** — "For example:" with realistic values follows at least one generic example
6. **EXPECTED_OUTPUT** — expected output shown for at least one block only when the source provides authoritative output text or deterministic output cues; if source has no reliable output evidence, do not invent output and treat this criterion as satisfied

### GEO Pillar (D8)

**D8 Citation-worthy** (5 criteria):

1. **VERSION_NUMBERS** — specific version numbers where relevant ("Available from Artifactory 7.83")
2. **EXACT_LIMITS** — exact quantities, not "many", "various", "several"
3. **CONCRETE_VALUES** — config examples include real values alongside placeholders
4. **PRODUCT_NAMES** — specific JFrog product names ("JFrog Artifactory", "JFrog Xray") not "the product" / "the system"; canonical casing required (Jfrog / xray / X-Ray fail)
5. **DATED_INFO** — deprecations/timelines include explicit dates or versions

---

## Step 3 — Auto-apply all fixes

For every dimension scoring below 5 and not Skip, automatically apply the matching rewrite template. Do **not** ask the user for permission first. Process in this order (dependency + impact):

1. **D1** Meta description (frontmatter excerpt; quickest win, drives SERP + hub cards)
2. **D2** Heading hierarchy (structural; later edits depend on stable headings)
3. **D5** FAQ section (adds new content; do before linking pass)
4. **D4** Structured data (lists / tables / definitions)
5. **D7** Code examples (language tags, placeholders, Where blocks)
6. **D3** Internal & adjacent linking (link to other pages once structure is final)
7. **D6** RAG chunk quality (chunk self-containment after structure / linking is set)
8. **D8** Citation-worthy (specificity pass last)

For each fix:

1. Read the matching template from [`references/rewrite-templates.md`](references/rewrite-templates.md).
2. Apply ALL rules in the template — every field, every criterion. Optional fields (e.g., D1 `keywords`, D1 `metadata.description`, D1 `category`) become mandatory and must be populated. Never insert `<!-- TODO: verify -->` placeholders. When data isn't available for a criterion that requires verifiable facts (versions, limits, dates, UUIDs), leave the existing wording rather than fabricating.
3. Save the page edit (or capture the proposed edit when no file is on disk).
4. Re-score the affected dimension and any criteria the edit may have touched (e.g., a D2 heading edit may also touch D6 DESCRIPTIVE_HEADINGS).

If the page is pasted text (no file on disk), generate the proposed full-page output instead of editing in place. Note this in the report.

---

## Step 4 — Final report

Emit a single unified before/after report. Format:

```markdown
## SEO + AEO + GEO Audit Report

**File:** `[filename]`
**Page type:** [Task / Concept / Reference / Landing]
**Audit date:** [today]

### Pillar scores

| Pillar | Before | After | Change |
|--------|--------|-------|--------|
| SEO (D1–D4) | X.X | X.X | +X.X |
| AEO (D5–D7) | X.X | X.X | +X.X |
| GEO (D8) | X.X | X.X | +X.X |
| **Composite** | **X.X** | **X.X** | **+X.X** |

**Grade: [before-letter] → [after-letter]**

### Fixes applied

- **D1 Meta description** (X/5 → 5/5): rewrote excerpt + added keywords + metadata.description
- **D2 Heading hierarchy** (X/5 → 5/5): promoted 3 bold pseudo-headings; demoted 1 H4 to H3
- **D3 Internal & adjacent linking** (X/5 → 5/5): met scaled link targets + Related Topics with adjacency
- ...

### Diffs

<details>
<summary>D1 — Meta description</summary>

**Before**
```yaml
[exact before YAML]
```

**After**
```yaml
[exact after YAML]
```
</details>

<details>
<summary>D2 — Heading hierarchy</summary>

**Before**
```markdown
[before snippet]
```

**After**
```markdown
[after snippet]
```
</details>

<!-- one <details> block per applied fix, in fix-priority order -->
```

For **Skip** dimensions: omit from the pillar table change calculation; do not list in "Fixes applied".

**Perfect-score shortcut output:** when Step 2 already scored Composite 5.0:

```markdown
This page scores 5.0/5.0 (Grade A). No changes needed.
```

---

## Batch / folder mode

1. List `.md` files; process up to 5 per batch.
2. Per file: run the full Step 1–4 pipeline and capture before/after composites.
3. Batch summary table:

| File | Type | Before | After | Change | Grade | Top issue fixed |
|------|------|--------|-------|--------|-------|-----------------|

4. Offer next batch if >5 files remain. Full per-file reports available on request.

---

## Input handling

- **File:** `.md` via `@file` or drag-and-drop. Edits are applied to the file on disk.
- **Folder:** up to 5 files per batch. Edits applied per file.
- **Pasted text:** analyze and produce the proposed full-page output; no file edits possible (note in report).
- **Scoped:** audit only the requested section/lines, but apply fixes that are valid for that scope.

---

## Constraints

- Use only criteria defined in this file and the two reference files.
- Preserve meaning, technical accuracy, and all JSX/HTML components (`<Table>`, `<Callout>`, `<Anchor>`, `<Tabs>`).
- No speculative facts. Never fabricate version numbers, dates, limits, UUIDs. When source content lacks them, leave the existing wording.
- Never insert `<!-- TODO: verify -->` placeholders — they read as unfinished work in the rendered page.
- Never propose URL slug changes. Never add a body-visible "Last updated" line (ReadMe handles freshness on the front end).
- One primary file per deep-dive; folder mode respects the 5-file batch limit.
