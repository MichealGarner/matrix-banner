# Matrix Login Banner

```
 __  __        _        _      _   _
|  \/  |  __ _| |_ _ __(_)_  _| | | | ___ _ __ __ _| |__| |
| |\/| | / _` | __| '__| \ \/ / |_| |/ _ \ '__/ _` | |/ _` |
| |  | || (_| | |_| |  | |>  <|  _  |  __/ | | (_| | | (_| |
|_|  |_| \__,_|\__|_|  |_/_/\_\_| |_|\___|_|  \__,_|_|\__,_|
```

> *"I check logs. I hold a grudge. You have been warned."*

A Matrix-themed, animated, colourised login experience for Debian Linux.
Three layers of login goodness — SSH pre-auth warning, dynamic MOTD with live system stats,
and an animated typewriter shell banner with random personalised greetings.

---

## Screenshots

```
Call trans opt: received. 12-04-26 14:32:07 REC:Loc
     Trace program: running...

           wake up, anon...

                        (`.         ,-,
                        ` `.    ,;' /
                         `.  ,'/ .'
                          `. X /.'
                .-;--''--.._` ` (
              .'            /   `
             ,           ` '   Q '
             ,         ,   `._    \
          ,.|         '     `-.;_'
          :  . `  ;    `  ` --,.._;
           ' `    ,   )   .'
              `._ ,  '   /_
                 ; ,''-,;' ``-
                  ``-..__``--`

  ██╗  ██╗███████╗██████╗  █████╗ ██╗     ██████╗
  ██║  ██║██╔════╝██╔══██╗██╔══██╗██║     ██╔══██╗
  ███████║█████╗  ██████╔╝███████║██║     ██║  ██║
  ██╔══██║██╔══╝  ██╔══██╗██╔══██║██║     ██║  ██║
  ██║  ██║███████╗██║  ██║██║  ██║███████╗██████╔╝
  ╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝╚═╝  ╚═╝╚══════╝╚═════╝

  ╔══════════════════════════════════════════════════════════════╗
  ║               !! AUTHORISED ACCESS ONLY !!                  ║
  ╠══════════════════════════════════════════════════════════════╣
  ║  This system is privately owned and monitored continuously.  ║
  ║  All connections, commands, and file activity are logged.    ║
  ║  Unauthorised access will be investigated and prosecuted     ║
  ║  under the Computer Misuse Act 1990 (UK).                   ║
  ║                                                              ║
  ║       I check logs.  I hold a grudge.  You have been        ║
  ║                    w a r n e d .                            ║
  ╚══════════════════════════════════════════════════════════════╝

  ┌─[ SYSTEM STATUS ]──────────────────────────────────────────┐
  │  Host:               Herald (Debian 13)                    │
  │  Kernel:             6.12.74+deb13-amd64                   │
  │  IP Address:         192.168.11.82                         │
  │  Uptime:             3 days, 7 hours, 42 minutes           │
  │  Load (1m):          0.12                                  │
  │  Memory:             1842MB used / 7892MB total            │
  │  Disk (/):           28G/117G (24% used)                   │
  │  Login time:         2026-04-12 14:32:07 BST               │
  └────────────────────────────────────────────────────────────┘

  There is no place like 127.0.0.1.

anon@Herald:~$
```

---

## Features

- **Animated typewriter greetings** — picks a random Matrix-style message each login, personalised with `$USER`
- **Live system stats** — kernel, IP, uptime, load average, memory, disk usage and pending `apt` updates pulled fresh at every login
- **Colourised ANSI output** — green Matrix palette throughout, red/amber warning box, cyan status panel headers
- **Three-layer architecture** — SSH pre-auth banner, PAM dynamic MOTD, and animated interactive shell banner work independently and stack together
- **Legal warning** — Computer Misuse Act 1990 (UK) compliant pre-auth and in-shell notices
- **Debian-native** — uses `/etc/update-motd.d/` PAM hooks, `/etc/profile.d/`, and `sshd_config` — no daemons, no cron jobs, no extra services
- **Multi-host deployment** — single installer script, deploy to a fleet with a simple rsync loop or Ansible playbook
- **Safe and reversible** — backs up every file it touches with a timestamp suffix, validates `sshd_config` before reloading, includes a full uninstaller
- **Graceful degradation** — works without `figlet` or `lastb` installed; plain fallbacks kick in automatically

---

## How It Works

The banner runs in three independent layers that stack on top of each other:

```
SSH Connection Attempt
        │
        ▼
┌───────────────────────────────────┐
│  LAYER 1 — Pre-auth SSH Banner    │  /etc/ssh/banner.txt
│  Shown BEFORE authentication.     │  Controlled by sshd_config
│  Plain text. Seen by everyone,    │  Banner directive.
│  including uninvited guests.      │
└───────────────────────────────────┘
        │  (login succeeds)
        ▼
┌───────────────────────────────────┐
│  LAYER 2 — Dynamic MOTD           │  /etc/update-motd.d/
│  PAM runs all executable scripts  │  10-matrix-motd
│  in update-motd.d/ and joins      │
│  their stdout into the live MOTD. │
│  Warning box + system stats.      │
└───────────────────────────────────┘
        │
        ▼
┌───────────────────────────────────┐
│  LAYER 3 — Animated Shell Banner  │  /etc/profile.d/
│  Runs for all users in            │  00-matrix-banner.sh
│  interactive login shells.        │
│  Typewriter animation, ASCII art, │
│  random greeting, stats panel.    │
└───────────────────────────────────┘
        │
        ▼
   anon@Herald:~$
```

---

## Requirements

| Package | Required | Purpose | Install |
|---|---|---|---|
| `bash` | ✅ Required | Script runtime | Pre-installed on Debian |
| `figlet` | ⚪ Optional | Hostname ASCII art | `apt install figlet` |
| `openssh-server` | ⚪ Optional | Pre-auth banner | `apt install openssh-server` |
| `util-linux` (`lastb`) | ⚪ Optional | Failed login count | Pre-installed on Debian |

> Scripts degrade gracefully if optional packages are absent — a plain hostname fallback is used if `figlet` is missing, and the failed login count is simply omitted if `lastb` is unavailable.

---

## Installation

### Single host

```bash
wget https://github.com/MichealGarner/matrix-banner/releases/latest/download/matrix-banner.tar.gz
tar -xzf matrix-banner.tar.gz
sudo bash matrix-banner/install.sh
```

The installer will:

1. Offer to install `figlet` if not present
2. Disable any clashing default Debian MOTD scripts
3. Back up any existing files it replaces (timestamped `.bak` files)
4. Validate `sshd_config` before reloading the SSH daemon
5. Clear `/etc/motd` so the dynamic MOTD is used instead

### Multiple hosts — rsync loop

```bash
HOSTS="server1 server2 server3"
for HOST in $HOSTS; do
    rsync -av matrix-banner/ root@${HOST}:/tmp/matrix-banner/
    ssh root@${HOST} "bash /tmp/matrix-banner/install.sh"
done
```

### Multiple hosts — Ansible

```yaml
- name: Deploy Matrix Login Banner
  hosts: all
  become: true
  tasks:
    - name: Copy banner files
      copy:
        src: matrix-banner/
        dest: /tmp/matrix-banner/
        mode: preserve

    - name: Run installer
      command: bash /tmp/matrix-banner/install.sh
      args:
        creates: /etc/profile.d/00-matrix-banner.sh
```

### Test without logging out

```bash
bash /etc/profile.d/00-matrix-banner.sh
```

---

## File Layout

```
matrix-banner/
├── install.sh                            ← Run as root to install
├── uninstall.sh                          ← Removes all installed files
├── README.md
└── etc/
    ├── profile.d/
    │   └── 00-matrix-banner.sh           ← Animated shell banner
    ├── update-motd.d/
    │   └── 10-matrix-motd                ← Dynamic MOTD fragment
    └── ssh/
        └── banner.txt                    ← Pre-auth SSH warning (plain text)
```

---

## Customisation

### Change the greeting messages

Edit the `MATRIX_MESSAGES` array in `00-matrix-banner.sh`. Each entry can contain `%s`, which is replaced with `$USER` at runtime:

```bash
MATRIX_MESSAGES=(
    "wake up, %s..."
    "the Matrix has you, %s..."
    "knock, knock, %s."
    "follow the white rabbit, %s."
    "your own message here, %s."
)
```

### Change the hostname font

Install any figlet font and pass the `-f` flag:

```bash
apt install figlet

figlet -f big    "$HOSTNAME"    # larger
figlet -f small  "$HOSTNAME"    # compact
figlet -f banner "$HOSTNAME"    # classic wide
figlet -f slant  "$HOSTNAME"    # default used here
```

Extra fonts are available in the `figlet-fonts` and `figlet-fonts-extra` packages, or from [figlet.org](http://www.figlet.org/fontdb.cgi).

### Adjust animation speed

Change the `delay` argument in `typewrite` calls. Default is `0.04` seconds per character:

```bash
typewrite "your text" 0.02   # faster
typewrite "your text" 0.08   # slower / more dramatic
typewrite "your text" 0      # instant, no animation
```

### Change the legal jurisdiction

The warning references the **Computer Misuse Act 1990 (UK)**. Edit both `00-matrix-banner.sh` and `10-matrix-motd` to update for your jurisdiction.

### Skip the banner for a specific user

Add this to the user's `~/.bashrc`:

```bash
MATRIX_BANNER_SKIP=1
```

Then add this near the top of `00-matrix-banner.sh`, after the `[ -t 1 ]` check:

```bash
[ "${MATRIX_BANNER_SKIP:-0}" = "1" ] && return
```

---

## Uninstall

```bash
sudo bash matrix-banner/uninstall.sh
```

This removes all installed files, re-enables any default Debian MOTD scripts that were disabled, removes the `Banner` directive from `sshd_config`, and reloads the SSH daemon. Timestamped backup files are left in place and must be removed manually if no longer needed.

---

## Project Page

A full interactive demo and project page is available at:

**[michealgarner.github.io/matrix-banner](https://michealgarner.github.io/matrix-banner/)**

---

## Licence

MIT — do whatever you like with it. A mention or a star is always appreciated.

---

*There is no place like 127.0.0.1.*
