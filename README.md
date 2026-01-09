# mise monorepo multiple tasks with dependencies bug

Minimal reproduction for a bug where running multiple monorepo tasks with `:::` fails when tasks have dependencies.

## Setup

```
mise.toml                 # root config with experimental_monorepo_root = true
project-a/mise.toml       # has ci task with depends = [":lint", ":test"]
project-b/mise.toml       # has ci task with depends = [":lint", ":test"]
```

## What works

Running single tasks:
```bash
mise //project-a:ci    # works
mise //project-b:ci    # works
```

Running multiple tasks without dependencies:
```bash
mise //project-a:lint ::: //project-b:lint   # works
```

Running sequentially:
```bash
mise //project-a:ci && mise //project-b:ci   # works
```

## What fails

Running multiple tasks that have dependencies:
```bash
mise //project-a:ci ::: //project-b:ci
# ERROR: task not found: :lint
```

## Expected behavior

Both `ci` tasks should run in parallel, with each task's dependencies resolved within its own project context.

## Environment

- mise version: 2026.1.0
- OS: macOS (also reproducible on Linux)
