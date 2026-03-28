---
title: Contribution Guidelines
description: How to contribute to the Kakitu ecosystem
---

# Contribution Guidelines

Kakitu is an open-source project. All repositories are open for contributions at [github.com/kakitucurrency](https://github.com/kakitucurrency).

---

## How to Contribute

### Reporting Bugs

1. Check [existing issues](https://github.com/kakitucurrency/kakitu-node/issues) to avoid duplicates
2. Open a new issue with:
   - Node version (`{"action":"version"}` output)
   - OS and Docker version
   - Steps to reproduce
   - Expected vs. actual behavior
   - Relevant log lines

### Suggesting Features

Open a GitHub Discussion or an issue labeled `enhancement`. Describe:
- The problem you're trying to solve
- Your proposed solution
- Alternatives you considered

### Submitting Pull Requests

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Write your changes with tests where applicable
4. Run the test suite: `./core_test`
5. Commit with a clear message: `feat: add kshs_ URI scheme support`
6. Open a pull request against `main`
7. Respond to reviewer feedback

### Documentation Improvements

The documentation source lives at [github.com/kakitucurrency/kakitu-docs](https://github.com/kakitucurrency/kakitu-docs). Fix typos, clarify explanations, add examples, or propose new pages.

---

## Code Standards

### C++ (kakitu-node)

- Follow the existing code style (clang-format)
- Write unit tests for new functionality in `core_test`
- RPC commands must be documented in `docs/commands/rpc-protocol.md`
- New config options must be added to `docs/running-a-node/configuration.md`

### Go (gokakitu)

- Follow `gofmt` formatting
- Write table-driven tests
- Export only what is necessary

### JavaScript/TypeScript (kakitucurrency-js)

- Use TypeScript with strict mode
- Follow ESLint rules
- All exported functions must have JSDoc comments

---

## Review Process

Pull requests are reviewed by maintainers. Expect feedback within a few business days for documentation changes, and within a week for code changes. Complex protocol changes may require community discussion before merging.

---

## Code of Conduct

All contributors are expected to be respectful and constructive. The Kakitu community values:
- Inclusive, welcoming communication
- Technical rigor and honesty
- Kenya-first thinking — solutions that work in low-connectivity, low-income contexts

Harassment or abusive behavior in any Kakitu space will result in removal.
