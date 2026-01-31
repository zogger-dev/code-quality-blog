# Research Notes

Raw data and findings for potential use in blog posts.

---

## The "AI Code Quality Crisis" (GitClear, 2024-2025)

A massive study of 211 million lines of code found that while AI has increased productivity, it has caused a measurable decline in code quality:

- **Refactoring is dying**: The percentage of "moved" or "refactored" code dropped from 25% (2021) to under 10% (2025). Developers are just "adding" instead of "organizing."

- **The Duplication Explosion**: Commits containing copy-pasted (duplicated) code blocks increased by 800% in 2024.

- **The DORA Stability Link**: Google's 2025 DORA report found that for every 25% increase in AI adoption, there was a 7.2% decrease in delivery stability (increased bug rates).

---

## PyQu: Quantifying "Quality Gains" (2026)

Researchers introduced PyQu, a tool that uses machine learning to identify "quality-enhancing commits."

- **Methodology**: Analyzes 3.7 million commits to see which ones actually improve reliability and modularity vs. just adding features.

- **Significance**: Proves that we can now predict if a commit will improve long-term maintainability with ~85% accuracy based purely on low-level structural metrics.

---

## Already Cited in Mission Statement

- Minelli et al. (2015) - 70% reading, 15% writing code
- Daniotti et al. (2026) - AI adoption:- Cui et al. (2024) - Task completion speed with AI
- PyQu Research Team. (2026) - Structural metrics for quality

---

## Post Refinement Ideas & Backlog

### Optional is Not a Replacement for Null
- **Target Post**: `local-scope-efficiency.md`
- **Context**: Address the widespread pattern of wrapping local nulls in `Optional` just to achieve pseudo-null-safety.
- **Thesis**: `Optional` is a communication tool for return types, not a local variable container.

### The Nuance of Output Parameters
- **Target Post**: `avoiding-output-parameters.md`
- **Context**: While avoiding them is a foundational principle (especially for junior engineers), there are valid high-performance exceptions.
- **Key Addition**: Show examples of when you *should* use them (e.g., high-performance database code where a single buffer is allocated once and passed in for mutations/filling).
