# Skills

Reusable agent skills for writing, reviewing, and refactoring code with GitHub Copilot. Each skill packages task-specific instructions and supporting examples in a self-contained directory.

This repository contains Markdown-based guidance, not an application. There are no packages to install or build steps to run.

## Available skills

| Skill | Purpose | Supporting material |
| --- | --- | --- |
| [clean-code](clean-code/SKILL.md) | Improve readability, cohesion, explicit behavior, and maintainability while respecting repository conventions and preserving intended behavior. | [Field guide](clean-code/REFERENCE.md), [examples](clean-code/EXAMPLES.md) |

## Installation

### With GitHub CLI

With GitHub CLI **2.90.0 or later**, inspect the skill before installing it:

```powershell
gh skill preview AbelKel/Skills clean-code/SKILL.md
gh skill install AbelKel/Skills clean-code/SKILL.md --agent github-copilot --scope user
```

The user scope installs a personal skill for Copilot across local projects. To install it only in a particular repository, run the following from that repository:

```powershell
gh skill install AbelKel/Skills clean-code/SKILL.md --agent github-copilot --scope project
```

The explicit `SKILL.md` path is intentional: this collection keeps skill directories at the repository root, outside the CLI's standard `skills` directory convention.

See GitHub's [skill installation documentation](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills#managing-skills-with-github-cli) for supported clients and current options.

### Manual installation

Clone or download this repository:

```powershell
git clone https://github.com/AbelKel/Skills.git
```

Review the skill's instructions, then copy the entire `clean-code` directory, including its supporting files and `LICENSE`, to one of these locations:

| Scope | Destination on Windows |
| --- | --- |
| Personal skill for supported local Copilot clients | `%USERPROFILE%\.copilot\skills\clean-code` |
| Project skill | `<your-repository>\.github\skills\clean-code` |

If a destination already exists, review your local changes before replacing it. Simply cloning this repository does not register its root-level skill directories with Copilot.

In **Copilot CLI**, start a new session or run `/skills reload`, then use `/skills info clean-code` to confirm discovery. Other clients have their own reload and discovery behavior. Personal skills are not automatically available in cloud environments; see GitHub's [agent skills documentation](https://docs.github.com/en/copilot/concepts/customizations/agent-skills).

## Using clean-code

Ask for the skill explicitly when you want it applied:

```text
Use clean-code to implement this feature using the repository's existing patterns.
Use clean-code to review this diff. Report findings; do not edit files.
Use clean-code to refactor this module without changing its public behavior.
```

The skill requests default use for coding tasks, but the client decides when to load it. Installing it alone is not a guarantee of always-on activation.

Its guidance favors practical judgment over arbitrary size limits or speculative abstractions. Review-only requests remain read-only, and refactoring should preserve behavior rather than quietly introduce new policies.

## Credit and source material

The `clean-code` skill was inspired by **Robert C. Martin's _Clean Code: A Handbook of Agile Software Craftsmanship_**, with credit to the book's contributing authors.

**Source:** Robert C. Martin and contributors, *Clean Code: A Handbook of Agile Software Craftsmanship*, first edition, ISBN **978-0-13-235088-4**. See the [publisher's book page](https://www.informit.com/store/clean-code-a-handbook-of-agile-software-craftsmanship-9780132350884).

Credit for the source book and its ideas belongs to its authors. The skill's instructions and examples are original adaptations prepared by AbelKel, informed by that reading; they are not copied book text, code listings, or a replacement for the book.

The book PDF is not included or needed to use the skill. This is an independent project, not an official companion or an endorsement by the book's authors or publisher. The book remains subject to its own copyright and licensing terms.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for skill structure, review guidance, and source-attribution requirements.

## License

The repository's original instructions, examples, and documentation are available under the [MIT License](LICENSE).

That license does **not** apply to the source book or other third-party material. Keep source credits and applicable notices when sharing a skill. A copy of the license is included in `clean-code` so it travels with a standalone installation.
