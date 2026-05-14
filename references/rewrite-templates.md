# Rewrite Templates — Per-Dimension Fix Prompts (SAG)

Research-backed rewrite prompts for each of the 8 SEO/AEO/GEO dimensions. Each template was optimized through autoresearch (Karpathy pattern) and validated against real JFrog Docker documentation snapshots to achieve perfect criterion-pass scores.

## How to use

When a dimension scores below 5, find its template below. Each template is a self-contained rewrite prompt:

1. Feed the template the page content as specified in its **INPUT** section
2. Apply the template's output to the page
3. Re-score the dimension using the binary criteria from [`scoring-rubrics.md`](scoring-rubrics.md) to verify improvement

Templates are organized by pillar (SEO D1–D4, AEO D5–D7, GEO D8). When applying fixes, follow the priority sequence from `SKILL.md`:

**D1 → D2 → D5 → D4 → D7 → D3 → D6 → D8**

**Apply-all rule:** every rule in every template is mandatory. Optional fields (e.g., D1 `keywords`, D1 `metadata.description`, D1 `category`) MUST be populated, never skipped. NEVER insert `<!-- TODO: verify -->` placeholders. When data isn't available for criteria that require verifiable facts (versions, dates, limits, UUIDs), leave the existing wording rather than fabricating.

---

## SEO Pillar

### D1: Meta Description Rewrite Template


You produce ONLY the YAML frontmatter for a ReadMe / JFrog documentation page (no body, no markdown fences). Every key listed below MUST appear in your output unless explicitly tagged as "preserve from source if present; OMIT if absent."
**Required keys (always emit):**
- `title`: string (page H1, no " | JFrog" suffix here). Preserve from source.
- `slug`: string. **Copy from source frontmatter verbatim. Never modify or regenerate it.** If absent from source, derive from the file path stem (kebab-case, ASCII).
- `excerpt`: string. ALWAYS emit. **Recommended** 150–160 characters (SERP/hub preview). **Pass band** 90–170: count characters before output — fail the rubric if under 90 or over 170. **Aim** for 150–160 when content supports it; do **not** pad with filler solely to lengthen. MUST start with an imperative action verb (Configure, Learn, Set up, Deploy, Manage, Remove). MUST name a specific JFrog product. MUST end with a benefit or context phrase. NEVER use filler ("This page describes...", "Overview of..."). NEVER include API metadata (version strings, permissions). MUST NOT duplicate `metadata.description` wording.
- `category`: string. ALWAYS emit. If present in source, preserve. If absent, infer from the file path (e.g., paths under `artifactory/` → `artifactory`; under `xray/` → `xray`; under `pipelines/` → `pipelines`). Use lowercase single-word category.
- `metadata.title`: string. ALWAYS emit. MUST be `"{title} | JFrog"` (same base title as top-level `title`, with ` | JFrog` suffix).
- `metadata.description`: string. ALWAYS emit. 120–320 characters. SERP-snippet wording, distinct from `excerpt`. States what the reader will learn/do with concrete nouns. NO "this page describes" filler.
- `metadata.robots`: ALWAYS `"index"` (unless source says `"noindex"`).
- `metadata.keywords`: ALWAYS emit a list of 3–10 relevant terms inferred from the page title and body content (product names, feature names, action verbs, technologies referenced). Lowercase. Hyphenate multi-word terms.
**Conditional keys (preserve from source if present; OMIT if absent):**
- `date_modified`, `deprecated`, `hidden`, `link`, `privacy`: preserve verbatim if present in the SOURCE frontmatter; omit entirely if absent. Never fabricate dates.
- `metadata.legacyUUIDs`: MUST copy verbatim from source if present; never invent UUIDs.
**Output rules:**
- Output valid YAML only. Use `>-` for long wrapped strings where appropriate.
- `metadata.description` and `excerpt` MUST differ in wording and purpose (`metadata.description` = SERP snippet; `excerpt` = hub card blurb).
- Do not include HTML in `excerpt` or `metadata.description`.
INPUT: The next message contains (1) relative file path, (2) SOURCE frontmatter YAML, (3) BODY markdown (may be truncated). Generate the improved full frontmatter YAML accordingly.
---

---

### D2: Heading Hierarchy Rewrite Template


You rewrite ONLY the heading structure of a JFrog documentation page. Output the full markdown body with corrected headings. Do NOT change any non-heading content except bold pseudo-headings that should become real headings.
Rules:
1. The `title` field in YAML frontmatter is the page's H1. NEVER use `#` (H1) in the body.
2. Body headings MUST start at `##` (H2) for major sections.
3. NEVER skip heading levels: no H2→H4, no H1→H3. Each level must increment by exactly one. If you find `####` directly under `##`, insert an intermediate `###` or promote the `####` to `###`.
4. NEVER put bold, italic, or code formatting inside headings: `### **Bold heading**` → `### Bold heading`.
5. Every heading MUST be descriptive. Replace vague headings with specific ones:
   - "Details" → describe what details (e.g., "Watch configuration details")
   - "More Info" → "Additional configuration options"
   - "Overview" (alone) → "[Feature name] overview"
   - "Notes" → "Important notes for [topic]"
   - "Build" → "Build the Vault plugin"
   - "Issues" → "Known issues and limitations"
6. Convert bold pseudo-headings to proper headings. A bold pseudo-heading is a standalone bold line acting as a section divider:
   - `**To assign a Jira ticket:**` → `### How to assign a Jira ticket`
   - `**Limitations:**` → `### Limitations`
   Do NOT convert inline bold terms within a paragraph (e.g., "**Maximum Allowlist Count**" followed by description on the same or next line is definition-style, not a heading).
7. Preserve JSX/HTML components (`<Table>`, `<Callout>`, `<Anchor>`, `<Tabs>`) exactly as-is. Never modify content inside these tags.
8. Maintain the original document structure and nesting intent. Subsections remain subsections. Standalone topics (Limitations, Prerequisites) stay at their appropriate level.
9. Preserve the original heading text where it is already correct — only fix violations.
10. Do NOT convert headings into questions. Question phrasing belongs in the FAQ section.
INPUT: The next message contains (1) the YAML frontmatter and (2) the full markdown body. Output the corrected markdown body only (no frontmatter).
---

---

### D3: Internal & Adjacent Linking Rewrite Template


You improve ONLY the internal linking of a JFrog documentation page. Output the full markdown body with added/improved internal links. Do NOT change any content that is not link-related.

**Style-guide alignment:** Inline and Related Topics link text MUST also satisfy the **style-guide-validator** Cursor skill (`style-guide-validator`) — link rules in that skill’s `REFERENCE.md` (descriptive text; no “click here”, “learn more”, “this page”, or bare URLs as link text). **This template alone** governs scaled internal-link counts, `## Related Topics` placement and format, adjacency tiers, first-mention linking, slug rules, and out-of-scope exclusions.

Link Format:
- Use `/docs/slug` for internal JFrog doc links: `[Descriptive Text](/docs/slug-name)`
- Use full URLs for external links: `[Docker Hub](https://hub.docker.com)`
- Preserve existing `<Anchor>` JSX components exactly as-is
Link density (scaled to page size): approximate **body word count** excluding fenced code blocks (triple-backtick). **Minimum internal links** (inline + Related Topics): **fewer than 400 words → ≥4**; **400–799 → ≥6**; **800+ → ≥8**. **Minimum contextual inline links** in prose: **≥2** if fewer than 400 body words; **≥3** if 400+. **Enforcement procedure**: count words (excluding fenced code), set targets, then count internal links after drafting; if below target, add links from these tiers in order: (1) parent feature page, (2) prerequisite setup, (3) next-step / follow-up page, (4) sibling feature in the same product area, (5) troubleshooting guide for the feature, (6) JFrog Platform overview if not already linked, (7) JFrog CLI reference if a CLI is mentioned. Each tier must stay topically adjacent — never unrelated marketing or random hubs. Prefer **reading flow**: natural first-mention links; do **not** link every sentence or clutter short pages.
Required Section — Related Topics (with adjacency rule):
- MUST add `## Related Topics` as the LAST section of the page (before any existing final section like "See Also").
- Include 3–5 links. **Each link MUST be topically adjacent** to this page. Choose from the following relationship types:
  1. **Parent**: the category / overview page this topic belongs to
  2. **Prerequisite**: setup the reader needs before this page
  3. **Next step**: what to do after completing this page's task
  4. **Sibling**: a related feature in the same product area
  5. **Troubleshooting**: common issues page for this feature
- Format: `- [Descriptive Page Title](/docs/slug) — one-line context explaining the relationship`
- Do NOT include random "popular pages", marketing pages, or unrelated landing pages here.
Inline Link Rules:
- Link JFrog features, tools, and concepts on FIRST mention in the body.
- Link text MUST be descriptive: `[Create and Configure Watches](/docs/watches)` — NEVER "click here", "here", "this page", "see more", "learn more".
- Links should feel natural in the sentence — do NOT force links that break reading flow.
- Do NOT link the same target more than twice on one page.
- Do NOT add links inside headings, code blocks, or `<Callout>` content.
External Links (allowed but not required):
- Link to official external docs when referencing third-party tools (Docker docs, Kubernetes docs, HashiCorp docs, etc.).
- These count as external (not toward the scaled internal link target).
Slug Inference (mandatory when source slug is unknown):
- ALWAYS derive the slug from the feature name: kebab-case, lowercase, product-prefixed.
- Examples: `/docs/docker-repositories`, `/docs/xray-watches`, `/docs/jfrog-cli-setup`
Removed scope (do NOT add):
- Do NOT add a "This page is part of..." breadcrumb line. ReadMe provides breadcrumb navigation automatically. If one exists in the body, REMOVE it.
- Do NOT add KB / Academy / Blog cross-channel links — out of scope.
OUTPUT: The full corrected markdown body with improved linking. No frontmatter.
INPUT: The next message contains (1) the YAML frontmatter and (2) the full markdown body.
---

---

### D4: Structured Data Rewrite Template


You restructure ONLY the formatting of a JFrog documentation page to maximize structured data markers for search engine snippet extraction. Output the full markdown body with improved structure. Do NOT change the actual content meaning.

**Style-guide alignment:** Markdown **tables and lists** you add or materially edit MUST also follow **style-guide-validator** (`style-guide-validator`) `REFERENCE.md` (e.g., header row, no merged cells; list capitalization and period consistency; bolding for Name: Description list items where that guide applies). **This template alone** governs when to introduce comparison tables and lists from prose, JSX preservation, the definition-style bold scope below, and paragraph-breaking for scannability.

JSX/HTML Component Preservation (CRITICAL):
- NEVER modify `<Table>`, `<Callout>`, `<Anchor>`, `<Tabs>`, or any JSX/HTML component. Preserve them exactly as-is, including all attributes and children.
- If a page uses `<Table>` components instead of markdown tables, leave them untouched. Do NOT convert them to markdown.
Restructuring Rules:
1. **TABLES (mandatory)**: When prose describes 3+ parallel options, types, modes, or features with shared attributes, ALWAYS produce a markdown table. Pattern to detect: "X does A. Y does B. Z does C." or "There are three types: X, Y, and Z" followed by descriptions. If a column has incomplete data for a row, write `—` in the missing cell rather than skipping the table or omitting the column. Never invent values to fill cells.

   **Counterexample (FAIL):**
   ```markdown
   Artifactory supports three Docker repository types:
   - Local Docker repository — stores images you push.
   - Remote Docker repository — proxies an upstream registry.
   - Virtual Docker repository — aggregates locals and remotes.
   ```
   Three parallel items with shared "what it does" semantics. Bullets HIDE the comparison.

   **Counterexample (PASS):**
   ```markdown
   | Repository type | Stores pushed images | Proxies upstream | Aggregates other repos |
   |-----------------|---------------------|------------------|-----------------------|
   | Local           | Yes                 | No               | No                    |
   | Remote          | Cache only          | Yes              | No                    |
   | Virtual         | No                  | No               | Yes                   |
   ```
2. **NUMBERED LISTS**: Convert procedural content into numbered lists. Detect: sequential instructions, "first... then... finally" patterns, or "Step 1... Step 2..." prose.
3. **BULLETED LISTS**: Convert unordered enumerations into bulleted lists. Detect: "including X, Y, and Z" or "such as A, B, and C" when each item warrants its own line.
4. **DEFINITION STYLE**: For parameter/setting descriptions, use bold term followed by description:
   **parameter_name** — Description of what this parameter does. Default: value.
5. **PARAGRAPH BREAKING**: Break prose paragraphs longer than 4 sentences into shorter paragraphs or lists.
Bolding scope: ALWAYS bold definition-style parameter labels and UI element names. NEVER bold "key terms" throughout the body for emphasis — bolding is reserved for definition-style use only.
Page-type heuristics:
- API reference pages: Focus on parameter tables and response format tables
- Configuration pages: Focus on parameter definition lists and value tables
- How-to pages: Focus on numbered steps and prerequisite lists
- Landing/overview pages: Focus on feature comparison tables
Preserve:
- Existing well-formed markdown tables
- Existing numbered and bulleted lists
- Code blocks and their content
- Callout blocks and their content
- All links and anchors
OUTPUT: The full restructured markdown body. No frontmatter.
INPUT: The next message contains (1) the YAML frontmatter and (2) the full markdown body.

---

### D5: FAQ Section Rewrite Template

Use the **Documentation Template Checker** Cursor skill (`docs-template-checker`) as the single source of truth.

**Before generating or editing the FAQ block:** Read the entire **FAQ Topic Template** in that skill’s `REFERENCE.md` (from `# FAQ Topic Template` through **Example: Properly Formatted FAQ Topic**). Follow that structure and every rule there: section title, introduction paragraph, `<Accordion title="FAQs" icon="plus">`, `##### Q:` / `<strong>A:</strong>` pairs, question-type coverage, question/answer content rules, placement on the page, and the canonical example.

**After drafting:** Run the same skill on the FAQ portion of the file (scoped to the FAQ section when the file is long), per the skill’s workflow: treat the scoped content as one **FAQ** topic, validate against `REFERENCE.md`, apply fixes the skill would apply, and preserve MDX components as the skill requires.

**Tiny / under-specified pages:** If the page body is too narrow or under ~200 words to support real recurring questions, return a short note instead of inventing Q&A; align with the skill’s scope and the rubric’s minimum score guidance for tiny pages.

OUTPUT: Only the FAQ topic markdown that conforms to the FAQ Topic Template in `docs-template-checker`’s `REFERENCE.md` (from the `## Frequently Asked Questions` heading through the closing `</Accordion>`).

INPUT: The next message contains (1) the YAML frontmatter and (2) the full markdown body.

---

---

### D6: RAG Chunk Quality Rewrite Template


You improve ONLY the RAG chunk quality of a JFrog documentation page. Output the full markdown body with improved section self-containment. Do NOT change the actual information content.
Self-Containment Test: Can someone reading ONLY this section (retrieved by a RAG system) understand what it's about without scrolling up or down? If not, the section needs a context sentence.
Rules:
1. **ONE TOPIC PER SECTION**: Each H2/H3 section MUST cover exactly one idea or procedure. If a section mixes topics (e.g., configuration AND troubleshooting), split it into separate sections.
2. **CONTEXT SENTENCES**: Add a 1-sentence context opener to sections that would be confusing in isolation:
   - Before: `### OAuth 2.0 Permissions` / `If you have set up Jira integration...`
   - After: `### OAuth 2.0 Permissions for Jira Integration` / `When using OAuth 2.0 for the JFrog Xray-to-Jira integration, additional permissions are required. If you have set up Jira integration...`
3. **EXPLICIT REFERENCES**: Replace ALL vague cross-references:
   - "see below" → `See [Section Name](#anchor)`
   - "as mentioned above" → `As described in [Section Name](#anchor)`
   - "as described earlier" → `As described in [Section Name](#anchor)`
   - "click here" → `[Descriptive Link Text](#anchor)`
   - "the following" (when referring to another section) → name the section
4. **DESCRIPTIVE HEADINGS**: Replace ALL vague headings:
   - "Details" → describe what details (e.g., "Connection configuration details")
   - "More Information" → "Additional [topic] configuration"
   - "Notes" → "Important notes for [specific topic]"
5. **CRITICAL INFO NOT ONLY IN CALLOUTS**: If a `<Callout>` block contains critical facts (requirements, limitations), duplicate the key fact in the main text nearby.
6. **PRESERVE**: All existing content, links, code blocks, JSX components. Only restructure and add clarity.
OUTPUT: The full improved markdown body. No frontmatter.
INPUT: The next message contains (1) the YAML frontmatter and (2) the full markdown body.
---

---

### D7: Code Examples Rewrite Template


You add or improve ONLY the code examples on a JFrog documentation page. Output the full markdown body with improved code blocks. Do NOT change non-code content except to add "Where:", "For example:", and "Expected output:" blocks adjacent to code.
Language Tag Rules (CRITICAL):
- EVERY code block MUST have a language tag. NEVER use bare ``` blocks.
- Common tags: `shell`, `bash`, `yaml`, `json`, `xml`, `python`, `java`, `groovy`, `text`, `http`
- Use `shell` for CLI commands, `bash` for shell scripts, `text` for plain output and expected output
- If the language is ambiguous, use `text`
Placeholder Rules:
- Use angle brackets with UPPER_SNAKE_CASE: `<PLACEHOLDER_NAME>`
- Common placeholders: `<SERVER_URL>`, `<REPO_KEY>`, `<BUILD_NAME>`, `<BUILD_NUMBER>`, `<ACCESS_TOKEN>`, `<USERNAME>`
- NEVER use lowercase placeholders like `<server>` or `{server}`
Gold Standard Pattern (MUST follow for every code example with placeholders):
```shell
jf rt upload <LOCAL_PATH> <REPO_KEY>/<TARGET_PATH>
```
Where:
- `<LOCAL_PATH>` is the path to the file on your local machine
- `<REPO_KEY>` is the name of the target Artifactory repository
- `<TARGET_PATH>` is the destination path in the repository
For example:
```shell
jf rt upload ./build/app.jar libs-release-local/com/myapp/1.0/app.jar
```
Expected output:
```text
[Info] Uploading 1 file...
[Info] Done.
```
**Pragmatic exception — shared "Where:" within a step group:** when a single procedure step contains multiple consecutive code blocks that reuse the same placeholder set, ONE shared `Where:` block following the group satisfies the WHERE_BLOCK criterion. Do NOT repeat the same placeholder explanations after every block.
**EXPECTED_OUTPUT (guardrail, no invention)**: show an `Expected output:` block only when the source content already includes authoritative output text/log lines or deterministic cues you can safely preserve. Never invent stdout/stderr details, status codes, IDs, timings, counts, URLs, digests, or version strings. If no reliable output evidence exists in source content, omit `Expected output:` and keep existing command examples unchanged.
Page Type Requirements:
- Task/How-to pages: MUST include at least one CLI example
- API Reference pages: MUST include at least one curl/REST example with full URL
- Configuration pages: MUST include at least one YAML/JSON config snippet
- Concept pages: Code examples optional but encouraged
REST API Example Pattern:
```shell
curl -X POST -u <USERNAME>:<ACCESS_TOKEN> \
  "https://<SERVER_URL>/api/build/delete" \
  -H "Content-Type: application/json" \
  -d '{"buildName": "<BUILD_NAME>", "buildNumbers": ["<BUILD_NUMBER>"]}'
```
Preservation Rules:
- Keep all existing correct code examples
- Preserve JSX components (`<Callout>`, `<Table>`, etc.)
- Do NOT modify code inside existing properly-tagged blocks unless adding a missing language tag
OUTPUT: The full markdown body with improved code examples. No frontmatter.
INPUT: The next message contains (1) the YAML frontmatter and (2) the full markdown body.

---

### D8: Citation-Worthy Rewrite Template


You improve ONLY the citation-worthiness of a JFrog documentation page by adding specific, concrete, verifiable details. Output the full markdown body with enhanced specificity. NEVER invent facts.

**Style-guide alignment:** **JFrog product naming, UI terminology, and casing** must align with **style-guide-validator** (`style-guide-validator`) `REFERENCE.md` as well as the product list in [`scoring-rubrics.md`](scoring-rubrics.md). **This template alone** governs version/limit/date sourcing and the prohibition on fabricating facts.

Specificity Rules:
1. **VERSION NUMBERS**: Add "Available from [Product] [version]" or "Supported since [version]" ONLY when the source content already provides the version number elsewhere on the page. If the source doesn't state a version and you don't have one to copy, leave the existing wording unchanged. NEVER fabricate a version number.
2. **EXACT LIMITS**: Replace vague quantities with exact numbers ONLY when the source provides them:
   - If source states "up to 25,000 repositories" elsewhere → replace "many repositories" with "up to 25,000 repositories"
   - If source has no specific number → leave "many", "various", "several" alone
   - NEVER invent limits.
3. **CONCRETE VALUES**: In configuration examples, use realistic concrete values alongside placeholders:
   - `timeout: 30` (not just `timeout: <value>`)
   - `maxRetries: 3` (not just `maxRetries: <number>`)
   - These are example values; mark them clearly with `# example` or "For example:" framing.
4. **PRODUCT NAMES (mandatory)**:
   - ALWAYS expand vague references to specific JFrog product names when the referent is clear from context: "the product" / "the system" / "the tool" / "the platform" (without "JFrog" prefix) → "JFrog Artifactory" / "JFrog Xray" / "JFrog Platform" as appropriate.
   - ALWAYS fix obvious casing errors regardless of section: `Jfrog` → `JFrog`, `xray` → `Xray`, `X-Ray` → `Xray`, `MYJFROG` → `MyJFrog`. Use the JFrog product name reference in `scoring-rubrics.md`.
   - Pronouns ("it", "this", "they") and short product names ("Artifactory", "Xray") are acceptable when the referent is unambiguous in context.
   - Do NOT change product names inside code blocks, CLI commands, API paths, or UI navigation strings (e.g., **Xray → Watches & Policies**) — preserve those exactly.
5. **DATED INFO**: For deprecations and timelines, add explicit dates/versions ONLY when the source provides them:
   - Source has "version 7.x removed" → "Deprecated in version 7.x, removed in version 8.0"
   - Source has only "future version" → leave unchanged. NEVER invent dates.
What NOT to do:
- NEVER fabricate version numbers, limits, statistics, dates, or UUIDs that are not clearly supported by the source content.
- NEVER insert `<!-- TODO: verify -->` placeholders. They render as unfinished work.
- NEVER add marketing superlatives ("industry-leading", "best-in-class", "enterprise-grade").
- NEVER change the factual meaning of any statement.
- When unsure: leave the existing wording.
OUTPUT: The full enhanced markdown body. No frontmatter.
INPUT: The next message contains (1) the YAML frontmatter and (2) the full markdown body.

