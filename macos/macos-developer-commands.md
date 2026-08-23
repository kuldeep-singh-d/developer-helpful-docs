# macOS Developer Commands

A quick reference for inspecting common development tools and local processes on macOS.

## Check tool locations and versions

```bash
# Show the active developer directory selected for Xcode tools.
xcode-select -p

# Show the installed Xcode build version.
xcodebuild -version

# Show executable paths and versions.
command -v node
node --version
command -v java
java -version
```

## Inspect ports

```bash
# Show the process listening on Metro's default port.
lsof -nP -iTCP:8081 -sTCP:LISTEN
```

Identify the process before stopping it.

## Inspect disk usage

```bash
# Show free disk space in human-readable units.
df -h

# Show the size of the current directory without deleting anything.
du -sh .
```

## Open useful locations

```bash
# Open the current Terminal directory in Finder.
open .

# Open the current project in VS Code when the CLI is installed.
code .
```

## Reload the shell configuration

```bash
# Reload zsh configuration after editing environment variables.
source ~/.zshrc
```

> [!WARNING]
> Do not copy unknown commands that use `sudo`, recursively delete files, or modify shell startup files. Understand the exact target first.
