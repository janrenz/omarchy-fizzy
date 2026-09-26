# Fizzy on GNOME

Notes for building a GNOME version, written on the Omarchy machine so the work can start
on the GNOME one without starting from zero. **Nothing here is built yet.**

## Why it does not simply run there

GNOME is Wayland too, but that is not the question. Omarchy's bar is a
Quickshell `PanelWindow`, and a panel needs the **`wlr-layer-shell`** protocol:
the one that lets a client say "I am a bar, pin me to the top edge, reserve my
height, keep me above everything". Hyprland, Sway, KDE and COSMIC implement it.
**Mutter does not**, and has declined to for years
([mutter#973](https://gitlab.gnome.org/GNOME/mutter/-/issues/973),
[gnome-shell#1141](https://gitlab.gnome.org/GNOME/gnome-shell/-/issues/1141)).
On GNOME only the shell draws the top bar, and anything that wants a place in
it is a **GNOME Shell extension** — GJS and St, not QML.

A Quickshell `FloatingWindow`, on the other hand, is an ordinary xdg toplevel,
and runs on GNOME like any other window.

## Whose repo this is

This is Jan's **fork** of `ryanyogan/omarchy-fizzy`; the plugin id is still
Ryan's. A GNOME version belongs either upstream as a pull request (ask first —
it is his design), or kept clearly in the fork. Either way it should not change
the Omarchy plugin itself.

## What there is to port

- **`fizzy-fetch`** — bash + curl against `app.fizzy.do`; runs anywhere. It reads
  the token from the widget's settings in `~/.config/omarchy/shell.json`, which
  does not exist on GNOME: give it a config file of its own (`--config` already
  exists) and keep the mode-0600 handling.
- **The panel** (`Panel.qml` with the `*Page.qml` files) — a mobile-style popup
  hung off the bar widget. On GNOME the two choices are:
  1. host the same QML in a Quickshell `FloatingWindow` and let a small Shell
     extension toggle it from the top bar — little new code, but a window, not
     a popup; or
  2. rewrite the pages in St inside the extension's own popup — a real popup,
     but everything is written twice.
  Start with 1.
- **`Model.js`** — plain JS, reusable from GJS.

## What has to be replaced

| Omarchy | GNOME |
|---|---|
| `qs.Commons`, `qs.Ui` (`Style.space/font/corner/selected/hover/normal/focus`, `Color.background/accent/foreground/popups`) | A small `Commons` of our own, fed from `org.gnome.desktop.interface` |
| `omarchy-fizzy` entry points | A `.desktop` entry and a GNOME custom keybinding |
| Settings in `shell.json` | A config file of its own |

## Check first, on the GNOME machine

- `gnome-shell --version` — the extension API moved to ES modules in 45 and
  keeps shifting; target the version that is there, and say so in `metadata.json`.
- Is Quickshell available? AUR on Arch; on Fedora/Ubuntu it has to be built.
- `python3` — the helpers need nothing beyond the standard library.
