# Contributing to Learn-Hyperscale-rs

Thank you for your interest in contributing to this project! This document provides guidelines and instructions for contributing.

## Code of Conduct

Be respectful, inclusive, and professional in all interactions.

## How to Contribute

### Reporting Issues

If you find a bug or have a suggestion:

1. Check if the issue already exists
2. Provide a clear description
3. Include relevant examples or screenshots
4. Specify your environment (OS, browser, etc.)

### Improving Documentation

Documentation improvements are always welcome:

1. Fix typos and grammar errors
2. Clarify confusing sections
3. Add missing information
4. Improve examples

### Adding New Content

To add new guides or flowcharts:

1. Follow the existing structure and naming conventions
2. Use clear, professional language
3. Include both English and Portuguese versions when applicable
4. Test all links and references
5. Follow the documentation standards

## Documentation Standards

### Markdown Files

- Use GitHub-flavored Markdown (GFM)
- Use clear headings hierarchy (H1, H2, H3)
- Include a table of contents for long documents
- Add links to related content
- Update the "Last updated" date

### Flowcharts

- Use Mermaid format for diagrams
- Save as `.mmd` files in `mermaid/` directory
- Render to PNG and save in `images/` directory
- Create documentation in `docs/` directory
- Update INDEX.md and SUMMARY.md

### File Naming

- Use lowercase with underscores for file names
- Use consistent numbering (01_, 02_, etc.)
- Avoid special characters and spaces
- Use descriptive names

### Bilingual Content

When adding content:

1. Create English version first
2. Create Portuguese translation
3. Use consistent naming between versions
4. Link between language versions

## Directory Structure

```
pt/fluxogramas/
├── INDEX.md                 # Main index
├── SUMMARY.md              # Detailed summary
├── mermaid/                # Source diagrams
│   └── 01_name.mmd
├── images/                 # Rendered images
│   └── 01_name.png
└── docs/                   # Documentation
    └── 01_name.md

en/flowcharts/
├── INDEX.md
├── SUMMARY.md
├── mermaid/
├── images/
└── docs/
```

## Commit Guidelines

- Use clear, descriptive commit messages
- Reference issues when applicable
- Keep commits focused and atomic
- Use present tense ("Add feature" not "Added feature")

### Commit Message Format

```
[type]: Brief description

Longer explanation if needed.

Fixes #123
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

## Pull Request Process

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-feature`)
3. Make your changes
4. Commit with clear messages
5. Push to your fork
6. Create a Pull Request with a clear description
7. Address review comments

## Testing

Before submitting:

1. Check all links work correctly
2. Verify markdown renders properly
3. Test on different platforms if applicable
4. Proofread for typos and grammar

## Style Guide

### Writing Style

- Use active voice
- Be concise and clear
- Avoid jargon or explain it
- Use examples when helpful
- Keep paragraphs short

### Technical Writing

- Explain concepts before using them
- Use consistent terminology
- Include code examples when relevant
- Link to related concepts
- Provide context

### Markdown Formatting

```markdown
# Heading 1
## Heading 2
### Heading 3

**Bold text** for emphasis
*Italic text* for emphasis

- Bullet points
- For lists

1. Numbered lists
2. For sequences

[Link text](url)

> Blockquotes for important notes

`inline code`

\`\`\`
code blocks
\`\`\`
```

## Review Process

- Maintainers will review contributions
- Feedback will be provided constructively
- Changes may be requested
- Once approved, your contribution will be merged

## Recognition

Contributors will be recognized in:
- Commit history
- Project documentation
- GitHub contributors page

## Questions?

- Open an issue for questions
- Check existing documentation first
- Be patient and respectful

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

Thank you for contributing to Learn-Hyperscale-rs! 🚀
