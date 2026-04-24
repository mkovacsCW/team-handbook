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
