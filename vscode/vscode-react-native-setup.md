# VS Code React Native Setup

A simple VS Code setup for working with React Native projects.

## Install the `code` command on macOS

Open the Command Palette with `Shift+Command+P`, then run:

```text
Shell Command: Install 'code' command in PATH
```

Verify it:

```bash
# Show VS Code CLI help and confirm that the command is available.
code --help
```

## Open a project

```bash
# Open the current directory as a VS Code workspace.
code .
```

## Useful built-in features

- Integrated Terminal for Metro, builds, and Git commands.
- Source Control view for reviewing changes before commits.
- Problems panel for TypeScript, ESLint, and build diagnostics.
- Command Palette for searchable editor actions.
- Workspace search for tracing identifiers and error messages.

## Extension safety

Install only extensions that solve a clear project need. Review the publisher, permissions, update history, and repository trust before installation.

```bash
# List installed extension identifiers.
code --list-extensions
```

Avoid committing personal `.vscode/` settings unless the team intentionally shares and maintains them.
