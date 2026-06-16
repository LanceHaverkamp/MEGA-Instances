# MEGA-Instances

Run multiple MEGA (megasync) instances on Linux — one per account, each with its own isolated configuration.

## Dependencies

- [megasync](https://mega.nz/linux/repo/)
- zenity

## Installation

```bash
sudo wget -O /usr/bin/megasync-instances https://raw.githubusercontent.com/NicoVarg99/MEGA-Instances/master/mega_instances.sh
sudo chmod 755 /usr/bin/megasync-instances
megasync-instances
```

Arch Linux users can install from AUR: [megasync-instances](https://aur.archlinux.org/packages/megasync-instances)

### First run (setup)

The script will ask:

1. **How many instances** you need
2. **A name for each** (e.g. "Work", "Personal", "Family")
3. **Log in to each** — a MEGAsync window opens; sign in, then quit

After that, your instances are configured for autostart.

## How it works

Each instance runs under its own `HOME` directory at `~/MEGA/<name>/`, with separate `XDG_DATA_HOME`, `XDG_CONFIG_HOME`, and `XDG_CACHE_HOME`. This keeps config files, sessions, and sync state completely isolated.

The script **auto-detects your init system** and uses the best autostart method:

| Init system                | Autostart method                                             | Desktop environments                                  |
| -------------------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
| **systemd** (detected)     | Per-instance systemd user services at `~/.config/systemd/user/megasync-<name>.service` | KDE 6, GNOME, modern distros                          |
| **non-systemd** (fallback) | Wrapper script + `.desktop` entry in `~/.config/autostart/`  | Devuan, Alpine, Gentoo (OpenRC), Artix, Void (runit)… |

### systemd method (v2.0+)

Each service sets the isolated environment via `Environment=` directives, starts after `graphical-session.target` and `network-online.target`, and is enabled for `graphical-session.target`. Instances start one at a time and survive cold boots reliably — no race conditions, no lost sessions.

This was the critical fix for KDE 6 / Wayland, where the old wrapper-script approach caused sessions to be invalidated after every cold boot.

### Non-systemd method (legacy)

A wrapper script launches each instance with `&` and a brief delay. A `.desktop` file in `~/.config/autostart/` ensures they start with your desktop session.

## Uninstallation

#### Stop all instances

```bash
# systemd
systemctl --user stop megasync-*.service

# non-systemd
killall megasync
```

#### Disable autostart and remove scripts

**systemd:**

```bash
for svc in ~/.config/systemd/user/megasync-*.service; do
    name=$(basename "$svc" .service)
    systemctl --user disable "$name"
done
rm -f ~/.config/systemd/user/megasync-*.service
systemctl --user daemon-reload
```

**non-systemd:**

```bash
rm ~/.config/autostart/mega_instances.desktop
rm ~/.local/bin/megasync-instances-launcher.sh
```

**Both:**

```bash
rm ~/.config/megasync-instances/status
sudo rm /usr/bin/megasync-instances
```

#### Remove all instance data

```bash
rm -rf ~/MEGA/*
```

## Images

![System tray](img/tray.png?raw=true "System tray")
![File manager](img/file-manager.png?raw=true "File manager")

## Changelog

### v2.0 — KDE 6 / Wayland + non-systemd support

- **systemd**: Generate per-instance systemd user services; cold-boot safe
- **non-systemd**: Legacy wrapper + .desktop autostart preserved
- Auto-detects init system at runtime
- Clean up old-style autostart on upgrade
- KDE globals symlink for correct theming in isolated environments

### v1.0 — Original release

- Interactive setup with zenity dialogs
- Wrapper script launches all instances at boot
- Works on KDE 5 / X11 and non-systemd distros
