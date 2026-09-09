# Sonoma compatibility experiment

This fork targets macOS 14 on Apple Silicon. Compatibility is experimental;
a successful CI build does not replace testing the app on Sonoma.

## Build

In the fork's Actions tab, select **Sonoma compatibility build**, then **Run
workflow** on `codex/sonoma-compatibility`. Download the `Look-Sonoma-arm64`
artifact from the completed run and extract the enclosed app ZIP.

The workflow uses a standard public macOS runner, Xcode 26.3, and the nightly
Rust toolchain requested by upstream's build phase. It builds both the Swift
app and Rust library for macOS 14. No local compiler installation is required.

The app is ad-hoc signed, not notarized. It uses the bundle identifier
`noah-code.Look.Sonoma` and disables upstream update notifications. The separate identifier selects
upstream's development settings file (`~/.look/config.dev`), but the database
is still shared with other Look builds unless `LOOK_DB_PATH` is overridden.
Do not run it alongside another Look build against the same database.

## Scope

- Lower the deployment target from macOS 15 to 14.
- Use the existing AppKit cursor helper instead of macOS 15's pointer modifier.
- Preserve upstream's compiler, concurrency settings, and newer-OS features.
- Keep personal build infrastructure separate from the compatibility patch.

Apple Intelligence is unavailable on Sonoma. The backdrop uses upstream's
ordinary blur fallback instead of Liquid Glass. Intel and earlier macOS versions
have not been tested.

## Local smoke test

Check cold launch, repeated hotkey open/hide, application and file search,
opening results, clipboard history, settings, and quit/relaunch. Observe idle CPU
and indexing activity with representative folders before calling support stable.

## First build result (2026-09-09)

[CI run 34360874429](https://github.com/hxseven/look/actions/runs/34360874429)
built commit `c9d4d26` successfully on the first attempt:

- 197 launcher logic tests passed.
- Release build, ad-hoc signature verification, and packaging passed.
- The downloaded executable's Mach-O minimum OS is 14.0 (built with SDK 26.2).
- On macOS 14.8.4 / Apple Silicon, `Look --version` exited 0 and printed
  `look 1.0 (1)` without stderr. A temporary config disabled launch-at-login,
  and `LOOK_DB_PATH` pointed at a temporary location for this check.

The full UI and performance smoke test is still pending. No Rust dependencies
or Swift concurrency settings needed changes. The actual compatibility changes
are isolated in commit `4e3ee30`; the other commits support this fork's builds.

When trying the UI, note that upstream defaults launch-at-login to enabled;
disable it in settings if you only want a manual trial. First launch may request
permissions for optional features. The CLI version check does not exercise those
features or prove search performance.

## Everyday use

- Quit: activate Look, then press **Command+Option+Q**. Command+Q only hides it.
- Reload configuration: **Command+Shift+;**.
- Settings: **Command+Shift+,**.
- Hide the development badge: run
  `defaults write noah-code.Look.Sonoma look.showTestHint -bool false`, then
  restart Look. Use `defaults delete noah-code.Look.Sonoma look.showTestHint`
  to restore automatic detection.

The SQLite database is at `~/Library/Application Support/look/look.db`.
For structural inspection while the app runs, use
`sqlite3 -readonly "$HOME/Library/Application Support/look/look.db"`, then
`.tables` or `.schema`. Avoid dumping rows if you only want metadata: the
database includes indexed paths, clipboard text, URLs, and usage history.
It uses WAL mode, so copying just `look.db` while it is open may miss recent
changes; use SQLite's backup command if you need a consistent snapshot.
