# joelgaff/shurectl

A fork of [Humblemonk/shurectl](https://github.com/Humblemonk/shurectl) (GPL-3.0) that adds
what [omarchy-shurectl](https://github.com/joelgaff/omarchy-shurectl), an Omarchy bar plugin,
needs: every setting readable and writable from a script, and one long-running process that
follows the device. Upstream keeps its CLI to one-shot commands (#126, #127), so this lives
here. Everything else is upstream's; see README.md for the TUI.

## What the fork adds

All of it is in `src/json_cli.rs`, plus the three flags and one dispatch in `src/main.rs`:

```bash
shurectl --state                         # every setting, device info, presets and model
                                         #   features as one line of JSON
shurectl --set gain=12 --set hpf=75      # change settings (applied in order), then print --state
shurectl --set mute=toggle               # booleans take on|off|true|false|yes|no|1|0|toggle
shurectl --set preset_save=2:Call        # save slot 2 as "Call"; preset_load=2, preset_delete=2
shurectl --set help                      # list every key and its values
shurectl --follow 500                    # keep running: print --state again whenever a setting
                                         #   changes, and apply KEY=VALUE lines read from stdin
```

`--state` is a different shape from upstream's `--json` (which is kept as is): it's the one
the plugin reads. `--follow` is meant to be the one process a widget keeps open: it polls the
device (every 500 ms by default), notices the mic's own mute button and changes made from
MOTIV or the TUI, and takes writes on stdin. A disconnected device prints
`{"connected": false, ...}` and is picked up again when it's plugged back in. `--device`
takes a HID path or a serial, as upstream's flags do.

## Taking changes from upstream

`main` here is upstream plus the fork's commits. Merge, don't rebase: omarchy-shurectl pins
commits of this repo as a submodule, and a rebase would orphan them.

```bash
git remote add upstream https://github.com/Humblemonk/shurectl.git   # once
git fetch upstream
git merge upstream/main          # or cherry-pick just the fixes you want
cargo build --release && scripts/check.sh
git push
```

Conflicts, if any, are in `src/main.rs`'s `Cli` struct. When upstream changes `DeviceState`
or `DeviceAction`, `cargo build` points at the lines in `json_cli.rs` to follow.
