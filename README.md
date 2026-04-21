# google-python-style-skill

A NanoClaw skill that applies the [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html) during code reviews.

## What it does

When invoked, the agent checks Python code against the full Google style guide and reports violations using the standard tiered format (🔴 Must Fix / 🟡 Should Fix / 🟢 Nice to Have) with exact line citations and fixes.

## Covers

- Imports — ordering, grouping, one-per-line rules
- Naming — modules, classes, functions, constants, private identifiers
- Type annotations — `X | None` over `Optional`, built-in generics, public API requirements
- Docstrings — `Args:` / `Returns:` / `Raises:` sections, required contexts
- Formatting — line length, indentation, continuation, trailing commas
- Exceptions — no bare `except`, no `assert` for logic, `Error` suffix rule
- Strings & logging — no f-strings as logging args, no `+=` in loops
- True/False — implicit falsiness, explicit None checks
- Default arguments — no mutable defaults
- Comprehensions — no multi-clause nesting
- Function length — ~40 line guideline
- Main guard — `if __name__ == "__main__"`

## Installation

Copy `python-style-guide.md` into your NanoClaw skills directory:

```bash
cp python-style-guide.md /home/node/.claude/skills/python-style-guide/SKILL.md
```

Restart your NanoClaw container. The skill will appear as `/python-style-guide`.

## Source

Based on the official Google Python Style Guide:
https://google.github.io/styleguide/pyguide.html
