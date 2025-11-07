# Conventional Commit Message Guidelines

## Format

type(scope): description

## Recommended Length

- **Title (header)**: **≤ 72 characters**
- Keep the message short, concise and clear.

---

## Types

- **feat**: A new feature  
- **fix**: A bug fix  
- **docs**: Documentation only changes  
- **style**: Code style changes (formatting, no logic)  
- **refactor**: Code change that neither fixes a bug nor adds a feature  
- **perf**: Performance improvement  
- **test**: Adding or updating tests  
- **chore**: Maintenance tasks (build, tooling, dependencies)  

---

## Rules

1. Use **lowercase** for type and scope.
2. Keep the **description short and imperative**, no period at the end.
3. **Scope** is mandatory (e.g., `ui`, `api`).

---

## Examples

- feat(auth): add JWT authentication  
- fix(api): handle null values in response  
- docs(readme): update installation instructions  
- refactor(core): simplify data parsing logic  