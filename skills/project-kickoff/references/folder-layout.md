# Folder layout

Source (Python): https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/ — fetched 2026-08-05.

Pick the layout at scaffold time. Moving a package after imports, tooling config, and CI paths exist is a day of churn for zero features.

## Python — default to src-layout

```
.
├── README.md
├── pyproject.toml
├── src/
│   └── awesome_package/
│       ├── __init__.py
│       └── module.py
├── tests/
├── docs/
└── tools/
```

Flat layout for comparison — the package sits at the repo root:

```
.
├── README.md
├── pyproject.toml
├── awesome_package/
│   ├── __init__.py
│   └── module.py
└── tools/
```

### Why src-layout

- **No accidental import of the uncompiled local copy.** Python puts the current working directory first on `sys.path`. With flat layout, `import awesome_package` from the repo root silently picks up the source tree instead of the installed distribution — so tests pass against code that was never packaged, and a broken `pyproject.toml` goes unnoticed until a user installs it.
- **Editable installs expose only the package.** Flat layout can leak `setup.py`, `noxfile.py`, and other root files onto the import path, so dev and production behave differently.
- **Tests exercise the installed artifact.** `src/` forces `pip install -e .` before the test run, which is exactly the thing you want verified.

### The cost, stated honestly

- Nothing runs from a bare checkout: `pip install -e .` (or `uv sync`) is a required setup step.
- CLI entry points cannot be run straight from the source tree — install in development mode, or add a `__main__.py` that adjusts `sys.path`.

Accept that cost by default. Choose flat layout only for a single-module script or a throwaway, and say so in CLAUDE.md.

### Wiring

- `pyproject.toml`: `[tool.setuptools.packages.find] where = ["src"]`, or Hatch `[tool.hatch.build.targets.wheel] packages = ["src/awesome_package"]`.
- `tests/` stays outside `src/` and is not packaged.
- First command in the README and in CI: `pip install -e ".[dev]"` (or `uv sync --dev`).

## Node / TypeScript

```
.
├── package.json
├── tsconfig.json
├── src/            # all source, the only compiled input
├── tests/          # or *.test.ts colocated in src/ — pick one, never both
├── dist/           # build output, gitignored
└── docs/
```

Same principle: source in `src/`, build output in `dist/` and gitignored, config at the root. `"files": ["dist"]` in `package.json` so a publish ships the build, not the tree.

## Framework projects

Framework conventions win — Next.js `app/`, Django app directories, Rails. Do not fight them into `src/`. Keep the framework's layout and put project-owned shared code in the location the framework documents (`src/app` for Next.js when `src/` is used at all, `lib/` for shared modules).

## Universal rules

- One obvious home per file type: source, tests, docs, scripts/tooling. No fifth top-level category without a reason.
- Build output and virtualenvs are gitignored, never committed.
- Config lives at the repo root, not scattered inside the package.
- Record the layout choice in CLAUDE.md under Conventions in one line, so later sessions do not re-litigate it.
