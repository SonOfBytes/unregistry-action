# Contributing to Unregistry Action

Thank you for your interest in contributing to the Unregistry GitHub Action! This document provides guidelines for contributing to this project.

## 🚀 Getting Started

### Prerequisites

- Docker installed and running
- Git
- GitHub account
- Basic understanding of GitHub Actions and YAML

### Local Development Setup

1. **Clone the repository**:
   ```bash
   git clone https://github.com/sonofbytes/unregistry-action.git
   cd unregistry-action
   ```

2. **Run local tests**:
   ```bash
   ./test-local.sh
   ```

3. **Test with real Docker images**:
   ```bash
   # Build a test image
   docker build -t test:latest .
   
   # Test the action (will fail at SSH - expected)
   docker pussh test:latest user@example.com
   ```

## 🧪 Testing

### Local Testing

- Use `./test-local.sh` for comprehensive local validation
- Tests include Docker image building, docker-pussh installation, and parameter validation
- All tests should pass before submitting PRs

### CI Testing

- GitHub Actions workflow (`.github/workflows/test.yml`) runs automatically on PRs
- Tests run on both Ubuntu and macOS runners
- Integration tests require server secrets (configured by maintainers)

### Test Requirements

- ✅ All existing tests must pass
- ✅ New features should include corresponding tests
- ✅ Test both success and failure scenarios
- ✅ Validate security practices (SSH key handling, cleanup)

## 📝 Pull Request Process

### Before Submitting

1. **Run local tests**: `./test-local.sh`
2. **Lint your changes**: Ensure YAML syntax is valid
3. **Update documentation**: Update README.md if needed
4. **Test edge cases**: Verify error handling works correctly

### PR Guidelines

- **Clear title**: Describe what the PR does in one line
- **Detailed description**: Explain the changes and why they're needed
- **Reference issues**: Link to related issues if applicable
- **Screenshots/output**: Include test output or examples if relevant

### Example PR Description

```markdown
## Summary
Add support for custom SSH configuration files

## Changes
- New input parameter `ssh_config_file`
- Updated action.yml to handle custom SSH configs
- Added tests for SSH config functionality
- Updated README with new parameter documentation

## Testing
- [x] Local tests pass (`./test-local.sh`)
- [x] Manual testing with custom SSH config
- [x] Integration tests pass

## Breaking Changes
None - new feature is optional and backward compatible
```

## 🐛 Bug Reports

### Before Reporting

1. **Check existing issues**: Search for similar problems
2. **Test with latest version**: Ensure you're using the latest release
3. **Reproduce locally**: Try to reproduce the issue locally

### Bug Report Template

```markdown
**Describe the bug**
A clear description of what the bug is.

**To Reproduce**
Steps to reproduce the behavior:
1. Use action with these inputs: ...
2. Run on runner: ...
3. See error: ...

**Expected behavior**
What you expected to happen.

**Environment**
- Runner OS: [ubuntu-latest, macos-latest, etc.]
- Action version: [v0.1.0, main, etc.]
- Docker version: [if relevant]

**Logs**
```
Include relevant action logs or error messages
```

**Additional context**
Any other context about the problem.
```

## 💡 Feature Requests

### Suggesting Features

- **Check roadmap**: Review existing issues and project roadmap
- **Provide context**: Explain the use case and why it's needed
- **Consider alternatives**: Mention if you've tried other approaches
- **Offer to help**: Indicate if you're willing to implement it

### Feature Request Template

```markdown
**Feature Description**
A clear description of the feature you'd like to see.

**Use Case**
Describe the problem this feature would solve or the workflow it would enable.

**Proposed Solution**
Describe how you envision this feature working.

**Alternatives Considered**
Any alternative solutions or workarounds you've considered.

**Additional Context**
Any other context, examples, or screenshots about the feature.
```

## 🔧 Development Guidelines

### Code Style

- **YAML formatting**: Use 2-space indentation, consistent formatting
- **Shell scripts**: Follow bash best practices, include error handling
- **Comments**: Add comments for complex logic or security-sensitive code
- **Documentation**: Keep README.md and comments up to date

### Security Considerations

- **SSH key handling**: Always use secure permissions (600) for key files
- **Cleanup**: Ensure temporary files are removed with `if: always()`
- **Input validation**: Validate user inputs where possible
- **Secrets**: Never log or expose SSH keys or sensitive data

### Version Management

- **Upstream sync**: Keep action version aligned with unregistry releases
- **Breaking changes**: Major version bump for breaking changes
- **Backwards compatibility**: Maintain compatibility when possible

## 🤝 Code of Conduct

### Our Standards

- **Be respectful**: Treat all contributors with respect and kindness
- **Be inclusive**: Welcome contributions from everyone
- **Be constructive**: Provide helpful feedback and suggestions
- **Be patient**: Remember that everyone is learning

### Unacceptable Behavior

- Harassment, discrimination, or personal attacks
- Spam, trolling, or off-topic discussions
- Publishing private information without permission
- Any conduct that would be inappropriate in a professional setting

## 📞 Getting Help

### Channels

- **GitHub Issues**: For bugs, feature requests, and general questions
- **Discussions**: For open-ended questions and community discussions
- **Security**: For security-related issues, please see SECURITY.md

### Response Times

- **Bug reports**: We aim to respond within 48 hours
- **Feature requests**: Response within 1 week
- **Security issues**: Response within 24 hours

## 🎯 Project Roadmap

### Upcoming Features

- [ ] Support for Docker contexts
- [ ] Improved error messages and debugging
- [ ] Performance optimizations
- [ ] Additional authentication methods

### Long-term Goals

- [ ] Windows support (when upstream supports it)
- [ ] Integration with other container tools
- [ ] Advanced deployment strategies

## 📄 License

By contributing to this project, you agree that your contributions will be licensed under the same [Apache License 2.0](LICENSE) that covers the project.

---

**Thank you for contributing!** 🙏

Your contributions help make container deployments easier for everyone. Whether it's a bug fix, feature addition, documentation improvement, or just a question, every contribution is valuable.

*Happy coding!* 🚀