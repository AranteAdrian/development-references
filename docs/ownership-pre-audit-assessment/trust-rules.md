# 360° Repo Takeover — Trust Rules

These rules apply to all 360° Repo Takeover prompts. They must be followed before generating any output.

---

## Documentation Trust Rule

Existing documentation files (README, CONTRIBUTING, docs/, wiki, inline comments) may be outdated, incomplete, or contradictory to the actual codebase. Before referencing any claim from an existing document:

1. **Verify it against the code.** If a README says the app does X, confirm that X exists in the codebase.
2. **Flag contradictions explicitly.** If a document claims something the code contradicts, state: "The [file] claims [X], but the codebase shows [Y]."
3. **Prefer code over docs.** When there is a conflict, the codebase is the source of truth — not the documentation.
4. **Do not propagate stale information.** Never copy claims from existing docs into your audit without verification.
