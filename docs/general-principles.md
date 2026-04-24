# General Principles

Language-agnostic practices we apply across the whole codebase.

## 1. Code Readability

Write code for humans first, computers second. Use meaningful variable and function names that describe their purpose. Keep functions short and focused on a single task. Add comments only when the "why" isn't obvious from the code itself.

## 2. Consistency

Follow a consistent style throughout your codebase — naming conventions, indentation, bracket placement, and file organization. Adopt a style guide for your language (e.g., PEP 8 for Python, Airbnb for JavaScript) and use linters/formatters to enforce it automatically.

## 3. Version Control

Commit early and often with clear, descriptive commit messages. Use branches for features and fixes. Review code before merging to main branches.

See also: [Pull Requests](workflow/pull-requests.md).

## 4. Error Handling

Anticipate what can go wrong. Validate inputs, handle exceptions gracefully, and provide meaningful error messages. Fail fast and fail loudly during development; fail gracefully in production.

## 5. Testing

Write tests alongside your code. Cover the happy path, edge cases, and error conditions. Aim for tests that are fast, isolated, and repeatable. Treat test code with the same care as production code.

## 6. Documentation

Document the "what" and "why" at the project level (README, architecture decisions). Use docstrings for public APIs. Keep documentation close to the code and update it when the code changes.

## 7. Security

!!! danger "Never trust user input"
    Validate everything that crosses a trust boundary.

- Avoid hardcoding secrets — use environment variables or secret managers.
- Keep dependencies updated and audit them for vulnerabilities.

## 8. Performance

Measure before optimizing. Write clear code first, then profile to find actual bottlenecks. Optimize only what matters.

## 9. Simplicity

Prefer simple solutions over clever ones. Avoid premature abstraction. Delete dead code. The best code is often the code you didn't write.

## 10. Consistency over Cleverness

When adding new code to an existing area, **match what's already there**. If the rest of the file uses a particular pattern, style, or abstraction, use the same one. Your "better" way might genuinely be better, but it costs readers time to context-switch between styles. Propose a change across the whole codebase if you think something should be different — don't introduce a one-off exception.

This is especially true for UI work: always follow the styling of the most recently deployed feature. See [Systems Development → UI Prototyping](workflow/systems-development.md#2-ui-prototyping).

## 11. Clean as You Go

Leave the codebase a little better than you found it. If you touch a function to fix a bug and notice an obvious improvement nearby — a confusing name, a dead import, a comment that's now wrong — fix it in the same PR. Don't let small issues accumulate.

Counterpoint: don't turn a one-line bug fix into a 500-line refactor. Keep the cleanup proportional to the change.

## 12. Ask Before Building Big Things

For anything that takes more than a day or touches more than one part of the system, **talk it through with a teammate before you start**. A 15-minute conversation at the whiteboard stage saves hours of rework. The cost of talking is low; the cost of building the wrong thing is high.

See [Systems Development → Design and Planning](workflow/systems-development.md#1-design-and-planning).
