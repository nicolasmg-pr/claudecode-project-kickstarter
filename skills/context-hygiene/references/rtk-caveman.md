# RTK + Caveman install

## RTK — compresses tool output by up to 80% ([rtk-ai/rtk](https://github.com/rtk-ai/rtk))

Requires Rust's cargo:

```bash
cargo --version                     # skip next lines if already installed
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh   # choose option 1 (default)
source "$HOME/.cargo/env"
```

Install and wire into Claude Code:

```bash
cargo install --git https://github.com/rtk-ai/rtk
rtk gain      # verify: must show stats, not "command not found"
rtk init -g   # installs PreToolUse hook, patches ~/.claude/settings.json
```

Restart Claude Code, run any git/pnpm/test command, check `rtk gain`.
Uninstall: `rtk init -g --uninstall`.

## Caveman — trims response verbosity ([JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman))

Requires Node ≥ 18. Note: the skill/plugin `caveman`, not `caveman-code` (a separate standalone agent).

```bash
curl -fsSL https://raw.githubusercontent.com/JuliusBrussee/caveman/main/install.sh | bash
```

Auto-detects and installs into Claude Code. Active from message one — toggle with "normal mode" / "talk like caveman".
