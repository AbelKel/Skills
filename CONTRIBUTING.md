# Contributing

Contributions should make a skill more useful, accurate, and easy to apply without expanding its scope unnecessarily.

## Adding or updating a skill

1. Keep each skill in a lowercase, hyphenated directory at the repository root.
2. Include a `SKILL.md` file with YAML frontmatter containing `name`, `description`, and `license`. The name should match the directory, and the description should explain when the skill is useful.
3. Keep supporting references and examples inside that directory. Use relative links that still work when the directory is installed by itself.
4. Include the applicable license notice inside a skill that is distributed independently. For this repository's original MIT-licensed material, keep that notice consistent with the root [LICENSE](LICENSE).
5. Update the [README skill catalog](README.md#available-skills) when adding a skill or changing its purpose.

Prefer focused instructions, concrete examples, and explicit boundaries. Respect the target repository's conventions and distinguish writing, review-only work, and behavior-preserving refactoring.

## Sources and attribution

Credit the authors of books, articles, and other material that informs a skill. Give the source title and a reliable link; include an edition or ISBN when it helps identify a book.

Write original instructions and examples. Do not contribute book PDFs, copied chapters or code listings, proprietary source code, credentials, personal configuration, or session artifacts. Having access to a source does not grant permission to redistribute it.

Clearly distinguish inspiration from authorship. Do not imply that a source author or publisher endorses a skill, and do not claim that this repository's license covers third-party source material.

## Before opening a pull request

- Check the frontmatter, skill name, examples, and all relative links.
- Confirm that the skill remains usable when copied without the rest of this repository.
- Explain the behavior or guidance changed, including any compatibility concerns.
- Preserve unrelated content and keep the diff focused.
- If adding executable scripts, include focused checks and document how to run them. The current Markdown-only repository has no automated build or test suite.

Use an issue to describe an unclear requirement or a reproducible problem. Pull requests should explain why a change is useful, not just what files changed.

## Contribution license

By submitting a contribution, you agree to make your original contribution available under this repository's [MIT License](LICENSE). Submit only material you have the right to contribute, and preserve applicable source credits and notices.
