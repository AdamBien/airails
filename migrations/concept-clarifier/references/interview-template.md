# INTERVIEW.md Template

Template for the async questionnaire written to `migration/INTERVIEW.md`. Keep blocks self-contained — the expert reads this file without access to the codebase or the glossary.

```markdown
# Domain Interview: <system name>

Generated: <YYYY-MM-DD> — from open questions in migration/CONCEPTS.md

How to fill in: check one option per question or write your answer on the
free-text line. Skip questions you cannot answer. Add your name at the top
or per question — answers without a name cannot be recorded as confirmed.

Answered by: ______________________

---

## Q3 — Are "Account" and "Customer" the same concept?

**Context:** `T_ACCOUNT` (database) and the UI label "Customer" appear on the
same screens; the code uses both `AccountBean` and `CustomerRecord`.

- [ ] Same concept — one term should win
- [ ] Different concepts
- [ ] Depends (explain below)

**Answer / preferred term:** ______________________

---

## Q7 — What does the abbreviation `VTRG` stand for?

**Context:** table `T_VTRG`, columns `VTRG_NR`, `VTRG_BEGINN`; mapped by
`ContractDAO`.

**Answer:** ______________________
```

Rules:

- One `## <Q-id> — <question>` block per open question, in Q-id order.
- **Context** carries the evidence pointers from CONCEPTS.md, verbatim but readable — table names, labels, class names.
- Offer checkboxes only when the answer space is enumerable; always include a free-text line.
- Never pre-check an option or suggest a preferred answer — a nudged expert confirms the nudge, not the domain.
