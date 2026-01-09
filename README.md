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

Verbose output:
```
$ mise -v //project-a:ci ::: //project-b:ci
DEBUG ARGS: /opt/homebrew/bin/mise -v //project-a:ci ::: //project-b:ci
DEBUG config: /private/tmp/mise-monorepo-multiple-deps-repro/mise.toml
DEBUG config: ~/.config/mise/config.toml
DEBUG opened gitignore file: /Users/chad/.gitignore_global
DEBUG glob `Glob("**/*~")` converted to regex: `"(?-u)^(?:/?|.*/)[^/]*\\~$"`
DEBUG glob `Glob("**/.aider*")` converted to regex: `"(?-u)^(?:/?|.*/)\\.aider[^/]*$"`
DEBUG built glob set; 0 literals, 5 basenames, 0 extensions, 0 prefixes, 0 suffixes, 0 required extensions, 2 regexes
DEBUG opened gitignore file: /private/tmp/mise-monorepo-multiple-deps-repro/.git/info/exclude
DEBUG ignoring /private/tmp/mise-monorepo-multiple-deps-repro/.git: Ignore(IgnoreMatch(Hidden))
DEBUG opened gitignore file: /Users/chad/.gitignore_global
DEBUG glob `Glob("**/*~")` converted to regex: `"(?-u)^(?:/?|.*/)[^/]*\\~$"`
DEBUG glob `Glob("**/.aider*")` converted to regex: `"(?-u)^(?:/?|.*/)\\.aider[^/]*$"`
DEBUG built glob set; 0 literals, 5 basenames, 0 extensions, 0 prefixes, 0 suffixes, 0 required extensions, 2 regexes
DEBUG opened gitignore file: /private/tmp/mise-monorepo-multiple-deps-repro/.git/info/exclude
DEBUG ignoring /private/tmp/mise-monorepo-multiple-deps-repro/.git: Ignore(IgnoreMatch(Hidden))
Error:
   0: task not found: :lint

Location:
   src/task/mod.rs:1046

Version:
   2026.1.0 macos-arm64 (2026-01-07)

Backtrace omitted. Run with RUST_BACKTRACE=1 environment variable to display it.
Run with RUST_BACKTRACE=full to include source snippets.
```

## Expected behavior

Both `ci` tasks should run in parallel, with each task's dependencies resolved within its own project context.

## Environment

- mise version: 2026.1.0
- OS: macOS (also reproducible on Linux)
