# SEO + AEO + GEO Optimizer (SAG) — Scoring Reference

Research-backed binary eval criteria, JSON output schemas, fix templates, JFrog naming rules, and the golden page template. The parent skill **`seo-final`** embeds a scoring cheat sheet in `SKILL.md`; open this file for full criteria details, borderline judgment guidance, and structured output schemas.

All criteria below were optimized through autoresearch (Karpathy pattern) and validated against real JFrog Docker documentation pages.

---

## Page Type Classification

Before scoring, classify the page. Page type affects which dimensions are weighted more heavily.

| Type | Characteristics | Typical Headings |
|------|----------------|-----------------|
| **Task** | Step-by-step procedure, "how to" content | Prerequisites, Steps, Configuration |
| **Concept** | Explanatory, architecture, background | Overview, Architecture, How It Works |
| **Reference** | API docs, CLI reference, parameter tables | Syntax, Parameters, Options, Returns |
| **Landing** | Hub/overview page linking to child pages | Getting Started, Products, Resources |

### Dimension Weight Adjustments by Page Type

| Dimension | Pillar | Task | Concept | Reference | Landing |
|-----------|--------|------|---------|-----------|---------|
| D1 Meta description | SEO | Full | Full | Full | Full |
| D2 Heading hierarchy | SEO | Full | Full | Full | Full |
| D3 Internal & adjacent linking | SEO | Full | Full | Half | Full |
| D4 Structured data | SEO | Full | Full | Full | Half |
| D5 FAQ section | AEO | Full | Full | Skip | Half |
| D6 RAG chunk quality | AEO | Full | Full | Full | Half |
| D7 Code examples | AEO | Full | Half | Full | Skip |
| D8 Citation-worthy | GEO | Full | Full | Full | Half |

- **Full** = scored 1–5, weight 1.0
- **Half** = scored 1–5, weight 0.5
- **Skip** = excluded from composite for that page type

---

## Scoring Rubrics — Binary Eval Criteria

### SEO Pillar

#### D1: Meta Description Quality

Evaluates the `excerpt` field in YAML frontmatter. Do NOT treat `metadata.description` as the excerpt. If no `excerpt` field exists, all criteria fail — score 1.

**Criteria (6 binary checks):**

1. **PRESENCE** — `excerpt` field exists in frontmatter
2. **LENGTH** — 150–160 characters (120–170 acceptable range; outside = fail)
3. **ACTION_VERB** — starts with an imperative action verb (Configure, Learn, Set up, Deploy, Manage, Remove, etc.); third-person forms like "Configures" or "Removes" do not count
4. **PRODUCT_NAME** — names a specific JFrog product (Artifactory, Xray, Distribution, Curation, Pipelines, JFrog CLI, JFrog Platform)
5. **USER_BENEFIT** — states what the user will learn or do, ending with a benefit or context phrase
6. **NO_FILLER** — free of generic filler ("This page describes...", "Overview of...", single-word excerpts) AND free of embedded API metadata (version numbers like "Since: X.Y.Z", permission/security requirements, changelog entries)

**Score mapping:** 6/6=5, 5/6=4, 3–4/6=3, 2/6=2, 0–1/6=1

**JSON output schema:**
```json
{
  "score": "<1-5>",
  "criteria_results": {
    "presence": "<true|false>",
    "length": "<true|false>",
    "action_verb": "<true|false>",
    "product_name": "<true|false>",
    "user_benefit": "<true|false>",
    "no_filler": "<true|false>"
  },
  "excerpt_text": "<the actual excerpt found, or null>",
  "char_count": "<number or null>",
  "violations": ["<specific violation referencing actual page content>"],
  "fix_suggestion": "<a complete, ready-to-use improved excerpt of 150-160 chars>"
}
```

**Violation rules:** Every violation must reference specific text or facts from the page (the actual excerpt text, character count, page title). Never use generic statements.

**Fix template:**

Before:
```yaml
excerpt: "Subscription Information"
```

After:
```yaml
excerpt: "Configure SAML SSO authentication in JFrog Platform with Azure AD, Google, Okta, or Keycloak. Step-by-step setup for enterprise identity providers."
```

---

#### D2: Heading Hierarchy

**Criteria (6 binary checks):**

1. **NO_H1_IN_BODY** — body contains zero `# ` (H1) headings; H1 comes only from frontmatter title
2. **STARTS_AT_H2** — first heading in the body is `##` (H2), not H3 or deeper; if body has NO headings, this passes
3. **NO_SKIPPED_LEVELS** — no heading level is skipped (H2→H4 without H3, H3→H5 without H4); check every pair of consecutive headings
4. **NO_FORMATTING_IN_HEADINGS** — no bold (`**`), italic (`*` or `_`), or inline code (backticks) inside any heading line
5. **DESCRIPTIVE_HEADINGS** — every heading describes its section content; fail if ANY heading is just "Details", "More Info", "Notes", "Overview" (without qualifier), "Additional", "Other"
6. **NO_BOLD_PSEUDO_HEADINGS** — no standalone bold lines acting as section dividers (e.g., `**To assign a ticket:**` on its own line); definition-style bold (bold term + description on same/next line) is acceptable

**Scanning rules:**
- Extract all lines matching `^#{1,6}\s` as headings
- Scan for `^\*\*[^*]+\*\*:?\s*$` as potential bold pseudo-headings
- Count heading levels numerically: # = 1, ## = 2, ### = 3, etc.
- For skip detection: compare each heading level to the previous; if current > previous + 1, it's a skip

**Score mapping:** 6/6=5, 5/6=4, 3–4/6=3, 2/6=2, 0–1/6=1

**JSON output schema:**
```json
{
  "score": "<1-5>",
  "criteria_results": {
    "no_h1_in_body": "<true|false>",
    "starts_at_h2": "<true|false>",
    "no_skipped_levels": "<true|false>",
    "no_formatting_in_headings": "<true|false>",
    "descriptive_headings": "<true|false>",
    "no_bold_pseudo_headings": "<true|false>"
  },
  "heading_list": ["## Heading 1", "### Heading 2"],
  "bold_pseudo_headings": ["**text**"],
  "skipped_level_pairs": ["## H2 → #### H4"],
  "violations": ["<specific violation with line content>"],
  "fix_suggestions": ["<before → after for each violation>"]
}
```

**Fix template:**

Before:
```markdown
### **What Are Watches in Xray?**
#### Details
###### More Info
```

After:
```markdown
## What are watches in Xray?
### Watch components and configuration
#### Advanced watch settings
```

---

#### D3: Internal & Adjacent Linking

Internal-doc linking dimension. Cross-channel external links (KB / Academy / Blog) are out of scope.

Count both markdown links `[text](url)` AND `<Anchor>` JSX components. Categorize as internal (`/docs/` or relative) or external (full URLs).

**Criteria (6 binary checks):**

1. **LINK_COUNT_8_PLUS** — 8+ internal links total (inline + Related Topics); external links do not count
2. **RELATED_TOPICS** — `## Related Topics` section (or `## See Also`) at or near bottom with 3–5 descriptive links
3. **CONTEXTUAL_INLINE** — 3+ contextual inline links connecting JFrog concepts naturally in sentences
4. **DESCRIPTIVE_TEXT** — ALL link text is descriptive; fail if ANY link uses "click here", "here", "this page", "see more", "learn more", or bare URLs
5. **FIRST_MENTION_LINKED** — JFrog features/products/tools referenced in body are linked on first mention
6. **ADJACENT_LINKS** — Related-Topics targets are topically adjacent to this page: parent category, prerequisite, next-step / follow-up, sibling feature, troubleshooting guide. Random or weakly-related links fail this criterion.

**Score mapping:** 6/6=5, 5/6=4, 3–4/6=3, 2/6=2, 0–1/6=1

**JSON output schema:**
```json
{
  "score": "<1-5>",
  "criteria_results": {
    "link_count_8_plus": "<true|false>",
    "related_topics_section": "<true|false>",
    "contextual_inline": "<true|false>",
    "descriptive_text": "<true|false>",
    "first_mention_linked": "<true|false>",
    "adjacent_links": "<true|false>"
  },
  "internal_link_count": "<number>",
  "external_link_count": "<number>",
  "non_descriptive_links": ["<link text that is not descriptive>"],
  "unlinked_features": ["<JFrog features mentioned but not linked>"],
  "non_adjacent_related_targets": ["<Related-Topics targets that are not topically adjacent>"],
  "violations": ["<specific violation descriptions>"],
  "fix_suggestions": ["<specific links to add with suggested text and target>"]
}
```

**Fix template — Related Topics with adjacency:**

```markdown
## Related Topics
- [Create and Configure Watches](/docs/watches) — parent feature this page configures
- [Set Up Security Policies](/docs/policies) — prerequisite policy setup
- [View Violation Reports](/docs/reports) — next step after a watch fires
- [Troubleshoot Xray Watches](/docs/xray-watch-troubleshooting) — common issues
```

Each link target should be one of: parent / prerequisite / next step / sibling feature / troubleshooting. Avoid putting unrelated marketing or top-level landing pages here.

---

#### D4: Structured Data Markers

**Criteria (5 binary checks):**

1. **HAS_TABLES** — comparison/feature tables present where content warrants them (both markdown `| col |` and JSX `<Table>` count); fail only if prose describes 3+ parallel items that should be a table
2. **NUMBERED_STEPS** — procedural content uses numbered lists; fail if step-by-step instructions are prose
3. **BULLETED_LISTS** — unordered items use bulleted lists; fail if enumerations of 3+ items are buried in prose
4. **DEFINITION_STYLE** — parameter labels and UI element names use bold formatting with clear definitions; bolding is reserved for definition-style use, not general emphasis
5. **CONTENT_SCANNABLE** — content organized for snippet extraction; fail if any section has 5+ consecutive prose sentences without structural breaks

**Score mapping:** 5/5=5, 4/5=4, 3/5=3, 2/5=2, 0–1/5=1

**JSON output schema:**
```json
{
  "score": "<1-5>",
  "criteria_results": {
    "has_tables": "<true|false>",
    "numbered_steps": "<true|false>",
    "bulleted_lists": "<true|false>",
    "definition_style": "<true|false>",
    "content_scannable": "<true|false>"
  },
  "table_count": "<number>",
  "list_count": "<number>",
  "prose_wall_sections": ["<headings of sections with 5+ unbroken prose sentences>"],
  "violations": ["<specific violation with section name and content description>"],
  "fix_suggestions": ["<specific restructuring: what to convert and how>"]
}
```

**Fix template — comparison table:**

Before:
```markdown
Artifactory supports local repositories that store your artifacts,
remote repositories that proxy external sources, and virtual
repositories that aggregate multiple repositories.
```

After:
```markdown
| Repository Type | Stores Artifacts | Proxies External | Aggregates Repos |
|-----------------|-----------------|-----------------|-----------------|
| Local           | Yes             | No              | No              |
| Remote          | No (cache only) | Yes             | No              |
| Virtual         | No              | No              | Yes             |
```

---

### AEO Pillar

#### D5: FAQ Section

Pages with <200 words of body content may not warrant a FAQ — score 3 minimum.

**Criteria (6 binary checks):**

1. **FAQ_HEADING** — `## Frequently Asked Questions` section exists (exact heading text, H2 level)
2. **QUESTION_COUNT** — contains 3–8 Q&A pairs formatted as H3 headings followed by answer text
3. **NATURAL_LANGUAGE** — questions phrased as users would naturally ask (starting with How/What/Why/When/Which/Can/Does/Is); fail if jargon-heavy or declarative
4. **SELF_CONTAINED** — each answer understandable without reading the rest of the page; fail if any answer says "as described above" or assumes prior reading
5. **CONCISE_ANSWERS** — each answer is 1–3 sentences; fail if any exceeds 4 sentences
6. **ANCHOR_LINKS** — at least one answer links to a detailed section using `[text](#anchor)` format

**Score mapping:** 6/6=5, 5/6=4, 3–4/6=3, 2/6=2, 0–1/6=1

**JSON output schema:**
```json
{
  "score": "<1-5>",
  "criteria_results": {
    "faq_heading": "<true|false>",
    "question_count_3_plus": "<true|false>",
    "natural_language": "<true|false>",
    "self_contained": "<true|false>",
    "concise_answers": "<true|false>",
    "anchor_links": "<true|false>"
  },
  "question_count": "<number>",
  "questions_found": ["<list of question headings>"],
  "violations": ["<specific violation descriptions>"],
  "fix_suggestions": ["<specific improvements or sample FAQ entries>"]
}
```

**FAQ generation guidance (5 question types):**
1. PRIMARY HOW-TO: "How do I [primary action described on this page]?"
2. COMMON MISTAKE: "What happens if I [common misconfiguration or error]?"
3. KEY DIFFERENCE: "What is the difference between [option A] and [option B]?"
4. PREREQUISITE: "What do I need before [performing this action]?"
5. RELATED FEATURE: "Can I also [related capability]?"

**Fix template:**

```markdown
## Frequently Asked Questions

### How do I push a Docker image to Artifactory?
Tag your image with the Artifactory registry prefix, then run `docker push`.
See [Push a Docker Image](#push-a-docker-image) for detailed steps.

### What's the difference between local and remote Docker repositories?
Local repositories store images you build internally. Remote repositories
proxy and cache images from external registries like Docker Hub.

### Can I use Docker repositories with Podman?
Yes. Artifactory supports OCI-compliant container images. Configure Podman
to use your Artifactory registry URL.
```

---

#### D6: RAG Chunk Quality

Measures how well each section performs when retrieved independently by a RAG/AI chatbot.

**Criteria (5 binary checks):**

1. **ONE_TOPIC_PER_SECTION** — each H2/H3 section covers exactly one idea or procedure; fail if any section mixes unrelated topics (e.g., setup + troubleshooting in one section)
2. **SELF_CONTAINED** — each section understandable without context from other sections; fail if any starts with "As mentioned above" logic or assumes prior reading
3. **NO_VAGUE_REFERENCES** — no "see below", "as mentioned above", "as described earlier", "click here", "the following section"; only explicit `[Section Name](#anchor)` references
4. **DESCRIPTIVE_HEADINGS** — no vague headings: "Details", "More Info", "More Information", "Notes", "Additional", "Other" (without qualifier)
5. **CLEAN_BOUNDARIES** — section content matches its heading; no content drift; fail if critical information appears only inside `<Callout>` blocks with no duplication in main text

**Score mapping:** 5/5=5, 4/5=4, 3/5=3, 2/5=2, 0–1/5=1

**JSON output schema:**
```json
{
  "score": "<1-5>",
  "criteria_results": {
    "one_topic_per_section": "<true|false>",
    "self_contained": "<true|false>",
    "no_vague_references": "<true|false>",
    "descriptive_headings": "<true|false>",
    "clean_boundaries": "<true|false>"
  },
  "section_count": "<number>",
  "vague_references_found": ["<exact vague reference text and surrounding context>"],
  "vague_headings_found": ["<heading text>"],
  "mixed_topic_sections": ["<heading of section that mixes topics>"],
  "violations": ["<specific violation descriptions>"],
  "fix_suggestions": ["<specific improvements>"]
}
```

---

#### D7: Code Examples

**Page type exemptions:** Concept-only pages without procedures are exempt — score 5 with a note.

**Criteria (6 binary checks):**

1. **CODE_PRESENT** — at least one code example exists on procedural, reference, or config pages
2. **LANGUAGE_TAGS** — ALL code blocks have language tags (`shell`, `yaml`, `json`, etc.); even one bare ``` block = fail (forward-only enforcement: legacy bare blocks are tracked as remediation backlog, not failed audits, when explicitly noted in the page header)
3. **PLACEHOLDER_FORMAT** — placeholders use `<UPPER_SNAKE_CASE>` in angle brackets; fail if any uses `{curly_braces}`, `$variable`, or `<lowercase>`
4. **WHERE_BLOCK** — each code example with placeholders has a "Where:" explanation listing each placeholder. Pragmatic exception: when consecutive code blocks within a single procedure step group reuse the same placeholders, one shared "Where:" block following the group satisfies this criterion.
5. **CONCRETE_EXAMPLE** — a "For example:" block with concrete, realistic values follows at least one generic example
6. **EXPECTED_OUTPUT** — expected output shown for at least one block when the command produces visible output (CLI command, curl, build, scan)

**Score mapping:** 6/6=5, 5/6=4, 3–4/6=3, 2/6=2, 0–1/6=1

**JSON output schema:**
```json
{
  "score": "<1-5>",
  "page_type": "<Task | Reference | Config | Concept | Landing>",
  "exempt_page": "<true|false>",
  "criteria_results": {
    "code_present": "<true|false>",
    "language_tags": "<true|false>",
    "placeholder_format": "<true|false>",
    "where_block": "<true|false>",
    "concrete_example": "<true|false>",
    "expected_output": "<true|false>"
  },
  "code_block_count": "<number>",
  "untagged_blocks": "<number>",
  "placeholders_found": ["<list of placeholders>"],
  "unexplained_placeholders": ["<placeholders without Where: explanation>"],
  "violations": ["<specific violation descriptions>"],
  "fix_suggestions": ["<specific code improvements>"]
}
```

**Gold standard code example pattern:**

```shell
docker login <ARTIFACTORY_REGISTRY>
```

Where:
- `<ARTIFACTORY_REGISTRY>` is your Artifactory server URL (e.g., `mycompany.jfrog.io`)

For example:
```shell
docker login mycompany.jfrog.io
```

---

### GEO Pillar

#### D8: Citation-Worthy Content

**Criteria (5 binary checks):**

1. **VERSION_NUMBERS** — specific version numbers present where relevant ("Available from", "Since version", "Supported in X.Y.Z"); concept pages not tied to specific versions are exempt
2. **EXACT_LIMITS** — exact limits/thresholds/quantities stated; fail if "many", "various", "several", "multiple" used where a specific number exists
3. **CONCRETE_VALUES** — config examples include concrete realistic values, not just abstract placeholders everywhere; at least one example with a real value
4. **PRODUCT_NAMES** — specific JFrog product names used consistently and with correct casing — not "the product", "the system", "the tool", "the platform" (without "JFrog" prefix); incorrect casings ("Jfrog", "xray", "X-Ray") fail this criterion
5. **DATED_INFO** — deprecations, timelines, and feature introductions include explicit dates or version numbers; fail if "will be removed in a future version" without specifying when

**Score mapping:** 5/5=5, 4/5=4, 3/5=3, 2/5=2, 0–1/5=1

**JSON output schema:**
```json
{
  "score": "<1-5>",
  "criteria_results": {
    "version_numbers": "<true|false>",
    "exact_limits": "<true|false>",
    "concrete_values": "<true|false>",
    "product_names": "<true|false>",
    "dated_info": "<true|false>"
  },
  "specifics_found": ["<list of specific facts: version numbers, limits, exact values>"],
  "vague_phrases_found": ["<list of vague phrases that should be specific>"],
  "casing_errors": ["<incorrect product-name casings found>"],
  "violations": ["<specific violation descriptions>"],
  "fix_suggestions": ["<specific additions: 'Replace X with Y'>"]
}
```

---

## JFrog product name reference

Used by D8 PRODUCT_NAMES. Use these canonical forms; the bracketed strings are common error patterns to detect:

- **JFrog Artifactory** (NOT: Jfrog, jfrog, JFROG, Artifactory Server, JFrog repository manager)
- **JFrog Xray** (NOT: Jfrog Xray, xray, X-Ray, Xray scanner)
- **JFrog Distribution** (NOT: distribution service)
- **JFrog Curation** (NOT: curation service)
- **JFrog Pipelines** (NOT: CI/CD pipelines without JFrog prefix)
- **JFrog CLI** (NOT: jfrog command line, jf tool — except as the literal CLI command `jf`)
- **JFrog Platform** (when referring to the whole suite)
- **MyJFrog** (NOT: My JFrog, myJFrog, MYJFROG, the MyJFrog Platform)
- **JFrog Access** (NOT: the access service, the auth service)
- **JFrog Connect** (NOT: the connect service)
- **JFrog Workers** (NOT: the workers service)
- **JFrog Mission Control** (NOT: mission control)

Pronouns and short product names are acceptable when the referent is unambiguous in context. D8 PRODUCT_NAMES enforces canonical casing only.

---

## Grading Scale

| Composite Score | Grade | Meaning |
|----------------|-------|---------|
| 4.5 - 5.0 | **A** | Citation-ready. Top-notch SEO/AEO/GEO. Minimal or no changes needed. |
| 3.5 - 4.4 | **B** | Strong. Minor gaps that are quick to fix. |
| 2.5 - 3.4 | **C** | Needs work. Several dimensions require attention. |
| 1.5 - 2.4 | **D** | Significant gaps. Major restructuring recommended. |
| 1.0 - 1.4 | **F** | Critical gaps across most dimensions. Full rewrite needed. |

### Composite Score Calculation

1. Score each applicable dimension 1–5 using binary criteria (Step 2 of the SKILL workflow).
2. Apply weight adjustments per page type (Full = 1.0, Half = 0.5, Skip = excluded).
3. Composite = weighted sum / weighted count.
4. Also report pillar sub-scores: SEO (D1–D4 mean), AEO (D5–D7 mean), GEO (D8 score).
5. Step 3 of the SKILL workflow auto-applies fixes for every dimension below 5/5; Step 4 reports the post-fix composite alongside the pre-fix composite.

---

## The Golden Page Template

A perfectly scored (5.0) page follows this structure.

````markdown
---
title: "[Descriptive, Keyword-Rich Title]"
excerpt: "[Imperative verb] [what user learns/does] [with JFrog product]. [Benefit or key context]. 150-160 chars."
category: "[Product Category]"
metadata:
  title: "[Same title] | JFrog"
  description: "[120-320 chars; SERP snippet wording, distinct from excerpt]"
  robots: index
  keywords:
    - "[keyword 1]"
    - "[keyword 2]"
    - "[keyword 3]"
---

## [Primary section heading — descriptive, not vague]

[First sentence directly answers the section topic.]

1. Step one with **bold UI elements**
2. Step two
3. Step three

```shell
example-command --flag <PLACEHOLDER>
```

Where:
- `<PLACEHOLDER>` is [explanation]

For example:
```shell
example-command --flag my-value
```

Expected output:
```text
Operation completed successfully.
```

## [Comparison or differentiation section]

[Direct answer in first sentence.]

| Feature | Option A | Option B |
|---------|----------|----------|
| Key trait 1 | Value | Value |
| Key trait 2 | Value | Value |

## [Secondary action section]

[Steps or explanation.]

## Frequently Asked Questions

### [Question 1 — most common user question]?
[1-3 sentence answer. Link to detailed section.]

### [Question 2 — common mistake or confusion]?
[1-3 sentence answer.]

### [Question 3 — related feature question]?
[1-3 sentence answer.]

## Related Topics
- [Parent feature page](/docs/parent-slug) — overview of this feature area
- [Prerequisite setup](/docs/prereq-slug) — what to configure first
- [Next step / follow-up](/docs/next-slug) — what to do after this task
- [Sibling feature](/docs/sibling-slug) — related capability in the same product area
````
