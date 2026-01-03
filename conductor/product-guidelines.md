# Product Guidelines

## Voice & Tone
- **Pragmatic & Direct:** Address issues head-on with a focus on real-world engineering impact.
- **Educational & Encouraging:** The ultimate goal is growth. Explain *why* a practice is better in a way that empowers the reader rather than belittles them.
- **Authoritative yet Humble:** Take a strong stance on code quality standards based on experience, but explicitly recognize that multiple valid approaches often exist.
- **Style Note:** Actively work to eliminate any condescending undertones present in raw source material (e.g., PR comments).

## Content Principles
- **Readability First:** The primary goal of every snippet and explanation is clarity. If a stylistic choice makes the code harder to read, it should be revised.
- **Before vs After:** Use "Before" and "After" as the standard labels for comparing anti-patterns with their solutions.
- **The Power of Choice:**
    - When multiple valid solutions exist, provide a primary recommendation.
    - Present alternatives clearly with a balanced Pros/Cons analysis.
    - Explicitly state that when multiple "good" options are available, reviewers should defer to the author's preference.
- **Commentary in Code:**
    - Use inline comments sparingly. Code should be as self-documenting as possible.
    - For complex logic that requires detailed referencing, use numbered annotations (e.g., `// [1]`) linked to text explanations.

## Visual & Technical Style
- **Java Formatting:** Aim for Google Java Style as the baseline for all code snippets.
- **Flexibility:** Minor deviations from strict formatting are encouraged if they significantly improve readability for the blog format.
