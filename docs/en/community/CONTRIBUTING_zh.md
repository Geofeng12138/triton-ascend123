# Contributing to Apache DolphinScheduler

We are glad that you are interested in contributing to Apache DolphinScheduler. Before submitting your contribution, please refer to the following guidelines.

## Reporting Bugs

- Ensure that the bug has not been reported yet. You can search for existing issues in the [Issues](https://github.com/apache/dolphinscheduler/issues) section.
- If you are unable to find an existing issue addressing the problem, please open a new issue. Be sure to include a clear title and description, as much relevant information as possible, and a code sample or an executable test case demonstrating the expected behavior that is not occurring.

## Suggesting Enhancements

- Open a new issue with a clear title and description.
- Explain why this enhancement would be useful to most users.

## Your First Code Contribution

### Setting up development environment

Please refer to the [Development Environment Setup Guide](https://dolphinscheduler.apache.org/en-us/docs/latest/user_doc/development-environment-setup.html) to set up your local development environment.

### Fork the repository

- Fork the repository on GitHub.
- Clone your fork locally:
  ```bash
  git clone https://github.com/your-username/dolphinscheduler.git
  cd dolphinscheduler
  ```
- Add the original repository as an upstream remote:
  ```bash
  git remote add upstream https://github.com/apache/dolphinscheduler.git
  ```

### Create a branch

- Create a new branch for your work:
  ```bash
  git checkout -b my-fix-branch master
  ```

### Make changes

- Make your changes in the new branch.
- Ensure your code follows the project's coding style.
- Write or update tests as needed.
- Run tests to ensure your changes do not break existing functionality.

### Commit your changes

- Commit your changes with a clear and descriptive commit message:
  ```bash
  git commit -m "Brief description of your changes"
  ```

### Push to your fork

- Push your branch to your fork:
  ```bash
  git push origin my-fix-branch
  ```

### Submit a Pull Request

- Go to the original repository on GitHub and click "New Pull Request".
- Select your branch and provide a clear description of your changes.
- Reference any related issues in the pull request description.

## Code Review Process

- The project maintainers will review your pull request.
- They may request changes or ask questions. Please be responsive to feedback.
- Once approved, a maintainer will merge your pull request.

## Code of Conduct

Please note that this project is released with a [Contributor Code of Conduct](https://www.apache.org/foundation/policies/conduct.html). By participating in this project, you agree to abide by its terms.

## Additional Resources

- [Apache DolphinScheduler Official Website](https://dolphinscheduler.apache.org)
- [Apache DolphinScheduler Documentation](https://dolphinscheduler.apache.org/en-us/docs/latest)
- [Issue Tracker](https://github.com/apache/dolphinscheduler/issues)
- [Pull Requests](https://github.com/apache/dolphinscheduler/pulls)

Thank you for contributing to Apache DolphinScheduler!