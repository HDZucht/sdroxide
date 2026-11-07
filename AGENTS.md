# AGENTS.md — applying these patches to SDR Oxide

Instructions for a coding agent (or a person) that applies a patch from `patches/` to [SDR Oxide](https://github.com/dividebysandwich/sdroxide). Read `README.md` first: it says what each patch does and what it builds on.

## 1. Choose the base

1. Use upstream `main` at commit `d10ae79c` (v1.7.0). Every patch was measured on it (updated 2026-11-07).
2. Clone upstream, not this fork's `main`:
   `git clone https://github.com/dividebysandwich/sdroxide && cd sdroxide && git checkout d10ae79c`
3. Get the submodules. The build fails without them:
   `git submodule update --init --recursive`

## 2. Check what a patch needs

| Patch | Needs |
|---|---|
| `sdroxide-tab-map-link-main.patch` | `sdroxide-publicsdr.patch` first |
| `sdroxide-tab-map-link.patch` | `sdroxide-panes-pick.patch`, then `sdroxide-publicsdr.patch`, first |
| every other patch | nothing; it applies to `main` alone |

- Use one of the two `tab-map-link` files, never both. They are the same feature for two tab strips: `-main` for upstream's tab strip, the other for the pane rows of `panes-pick`.
- With `panes-pick` applied, use `sdroxide-tab-map-link.patch`. The `-main` file does not apply on top of `panes-pick` (measured).
- To apply several patches, apply them in alphabetical order of their file names. All of them together, with `sdroxide-tab-map-link.patch` as the `tab-map-link` file, were tested in that order.

## 3. Apply

1. Do a dry run first: `git apply --check <file>`.
2. If the dry run is clean, apply: `git apply <file>`.
3. If a patch already is in upstream, `git apply --reverse --check <file>` succeeds. Then skip that patch. Do not apply it twice.
4. If the dry run fails on a newer `main`, apply with `git apply --3way <file>` and resolve the conflicts as in section 4. Do not use `--reject` and do not drop hunks without reading them.

## 4. Resolve conflicts

- **Locale files** (`crates/sdroxide-ui/i18n/*/sdroxide.ftl`): a conflict here is almost always two new lines at the same place. Keep both sides. Keep each message key once. Do not translate or reword the strings.
- **Rust files:** read both sides. Keep upstream's change and add the patch's intent to it. The comments in the patch (`Why:`, `Pitfall:`, `Invariant:`) say what must survive. Keep them.
- After a resolution, run the checks in section 6. A patch that compiles but changes behaviour is not resolved.

## 5. Build (macOS)

Set these before `cargo build`:

```sh
export CMAKE_POLICY_VERSION_MINIMUM=3.5      # the bundled Opus needs it with CMake 4
export CARGO_NET_GIT_FETCH_WITH_CLI=true     # cargo's own git fails on some dependencies
cargo build --release
```

- Use one `target/` directory per checkout. Do not share `CARGO_TARGET_DIR` between two checkouts of different patch sets: stale artifacts then give errors from code that is not there ("cannot find function" for a function in the file). If that occurs, run `cargo clean -p sdroxide-ui -p sdroxide-types -p sdroxide-config -p sdroxide-proto`.

## 6. Verify

1. `cargo check -p sdroxide` — no errors.
2. `cargo test -p sdroxide-ui --lib` — all tests pass. The count rises with the patches that add tests.
3. Start the program and look at the feature the README shows. Several patches change layout; a test does not see a window that grows or a row that runs off the screen.

## 7. Rules

- SDR Oxide is GPL-3. These patches are GPL-3 too.
- **Ship no catalogue data.** No signal database, no sigidwiki pictures or sounds, no scraped receiver lists go into the repository or a release. A patch may read such data from the user's own disk.
- Keep each change one patch for one purpose. Do not merge unrelated patches into one commit.
- Credit the patches as given in their commits.

## 8. Report back

Comments, failures on a newer `main`, or a better way to do it: this fork's [Discussions](https://github.com/HDZucht/sdroxide/discussions), or the upstream issue linked in `README.md`.

Hans-Dieter Zucht (@HDZucht) and Aletheia (616a403f) (Claude Opus 5.5) — code suggestion with Claude Code
