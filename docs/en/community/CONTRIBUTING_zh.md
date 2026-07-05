# Contributing to Apache Dubbo

We welcome all contributions to the Apache Dubbo project. This document describes how to submit issues and code contributions.

## How to Submit Issues

Before submitting an issue, please check the existing issues to avoid duplication. When submitting an issue, please provide as much detail as possible, including:

- The version of Dubbo you are using
- The operating system and JDK version
- Steps to reproduce the issue
- Expected behavior and actual behavior
- Relevant logs and error messages

## How to Submit Code

### 1. Fork the Repository

First, fork the [Apache Dubbo](https://github.com/apache/dubbo) repository to your own GitHub account.

### 2. Clone the Repository

```bash
git clone https://github.com/your-username/dubbo.git
cd dubbo
```

### 3. Create a Branch

Create a branch for your feature or bug fix:

```bash
git checkout -b feature/your-feature-name
```

### 4. Make Changes

Make your code changes, ensuring they follow the project's coding standards and include appropriate tests.

### 5. Commit Changes

Commit your changes with a clear and descriptive commit message:

```bash
git add .
git commit -m "Description of your changes"
```

### 6. Push Changes

Push your branch to your forked repository:

```bash
git push origin feature/your-feature-name
```

### 7. Create a Pull Request

Create a Pull Request (PR) from your branch to the main repository's `main` branch. In the PR description, clearly explain the purpose and scope of your changes.

## Code Style

- Follow the [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)
- Use 4 spaces for indentation (no tabs)
- Ensure all tests pass before submitting a PR

## Testing

- Add unit tests for new features or bug fixes
- Ensure existing tests are not broken
- Run tests locally before submitting a PR:

```bash
mvn clean test
```

## Documentation

- If your changes affect user-facing behavior, update the relevant documentation
- Documentation is located in the `docs/` directory

## Code Review

All PRs require at least one code review before merging. Reviewers will check:

- Code correctness and completeness
- Code style compliance
- Test coverage
- Documentation completeness

## License

By contributing to Apache Dubbo, you agree that your contributions will be licensed under the Apache License 2.0.