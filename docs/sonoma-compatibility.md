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
`noah-code.Look.Sonoma` and disables upstream update notifications. Configuration
and database paths remain upstream's defaults, so this does not isolate app data.
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
