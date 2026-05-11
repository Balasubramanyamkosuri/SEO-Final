# Stakeholder feedback — SAG rubric dimensions

Internal record of writer feedback on `seo-final` / SAG criteria and agreed rubric updates.

---

## 6. D1 excerpt length and D3 internal linking — Joanna (Slack sign-off)

**Context:** Concerns that (1) excerpts under the old 120-character floor failed LENGTH and encouraged padding to pass, and (2) a flat **8+** internal link requirement was unrealistic for short pages and hurt reading flow.

**Review request sent (summary):** Asked Joanna to sanity-check updated bands: D1 excerpt **recommended** 150–160 chars, **pass band** 90–170; D3 **LINK_COUNT_SCALED** (fewer than 400 body words → ≥4 internal; 400–799 → ≥6; 800+ → ≥8); **CONTEXTUAL_INLINE** (≥2 if fewer than 400 words, ≥3 if 400+); Related Topics **3–5** links unchanged.

**Joanna Krashniker (Slack, May 2026):** *"i assume 90 is a good no. We can test over time. I'll try tomorrow."*

**Status:** Tentative sign-off on **90** as the minimum excerpt length; link ladder **4 / 6 / 8** and inline **2 vs 3** to be validated with real pages over time.

**Source of truth in repo:** [`SKILL.md`](SKILL.md) (D1 LENGTH, D3 criteria), [`references/scoring-rubrics.md`](references/scoring-rubrics.md) (full criteria + JSON field names `link_count_scaled`, `approx_body_word_count`, `internal_link_target`).

---

## 7. Todo completion note

Track follow-ups from Joanna’s live passes on docs; fold any numeric tweaks into `SKILL.md` and `references/scoring-rubrics.md` in one coordinated update.
