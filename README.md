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
- **Live system stats** — kernel, every IPv4 address (one row per interface), uptime, load average, memory, disk usage and pending `apt` updates (the update count is cached and refreshed by an apt hook, so login stays fast)
- **Colourised ANSI output** — green Matrix palette throughout, a white ASCII figure, cyan status panel headers (and a red/amber warning box in the MOTD)
- **Three-layer architecture** — SSH pre-auth banner, PAM dynamic MOTD, and animated interactive shell banner work independently and stack together
- **Legal warning** — Computer Misuse Act 1990 (UK) compliant pre-auth SSH banner and MOTD notice; the in-shell warning box in the animated banner is included but commented out by default
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
3. Back up any existing files it replaces (timestamped `.bak` files in `/var/backups/matrix-banner/`)
   — never next to the originals, because `run-parts` would execute a `.bak` copy left in `/etc/update-motd.d/`
4. Install a small update-count cache (`matrix-update-cache` plus an apt hook) so login never has to run `apt`
5. Validate `sshd_config` before reloading the SSH daemon
6. Clear `/etc/motd` so the dynamic MOTD is used instead

#### Animated banner only (`--no-motd`)

The dynamic MOTD (layer 2) and the animated banner (layer 3) show the same content. On an interactive login the MOTD is printed first, then the animated banner clears the screen and draws it all again, so you briefly see the hostname, warning box and status panel twice. If you'd rather only see the animation, install with:

```bash
sudo bash matrix-banner/install.sh --no-motd
```

This skips the MOTD fragment and the update-count cache, and disables an existing `10-matrix-motd` if one is already installed. The pre-auth SSH banner and the animated banner are unaffected. Run the installer again without the flag to switch the MOTD back on.

Prefer to keep the MOTD but want it readable before the animation clears it? Add `sleep 1.5` on the line before `clear` in `00-matrix-banner.sh`.

Note: the animated banner's own legal warning box is commented out by default, so with `--no-motd` the pre-auth SSH banner (`/etc/ssh/banner.txt`) is the only legal notice shown on SSH logins. Uncomment the box in `00-matrix-banner.sh` (see *Show the in-shell warning box* below) if you want it in the animation too.

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
├── usr/
│   └── local/
│       └── sbin/
│           └── matrix-update-cache       ← Refreshes the pending-update count (run by apt hook)
└── etc/
    ├── apt/
    │   └── apt.conf.d/
    │       └── 99matrix-banner           ← apt hook that runs the cache refresher
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

### Change which IP addresses are shown

The status panel lists every global IPv4 address on interfaces that are up, one row per interface, for example:

```
  │  IP Address:        10.10.0.2/24 (enp1s0)                   │
  │                     192.168.11.20/24 (ens1)                 │
```

Loopback, link-local and IPv6 addresses are not shown. Docker, bridge, veth, libvirt and Kubernetes CNI interfaces (`docker*`, `br-*`, `veth*`, `virbr*`, `cni*`, `flannel*`, `cali*`, `kube*`) are skipped, and the list is capped at six rows with a `(+N more)` line after that. To change this, edit `MAX_IP_ROWS` or the `case` list of skipped interface names near the top of `00-matrix-banner.sh`. The `none` row appears when no address is found.

### Adjust animation speed

Change the `delay` argument in `typewrite` calls. Default is `0.04` seconds per character:

```bash
typewrite "your text" 0.02   # faster
typewrite "your text" 0.08   # slower / more dramatic
typewrite "your text" 0      # instant, no animation
```

### Change the legal jurisdiction

The warning references the **Computer Misuse Act 1990 (UK)**. Edit both `00-matrix-banner.sh` and `10-matrix-motd` to update for your jurisdiction.

### Show the in-shell warning box

The "Legal / warning block" in `00-matrix-banner.sh` is commented out by default. To show it in the animation, remove the leading `# ` from each `printf` line in that block, and from the blank `printf '\n'` that follows it.

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

## Changelog

### Per-interface IP addresses

- The IP Address row now lists every global IPv4 address on interfaces that are up, one row per interface with its prefix length and interface name, instead of only the first address from `hostname -I`. This suits multi-homed hosts such as a lab server with separate management and lab interfaces.
- Docker, bridge, veth, libvirt and Kubernetes CNI interfaces are skipped, the list is capped at six rows, and `none` is shown when there is no address.

### Updated animated banner

- The ASCII figure is now bright white instead of dim green.
- The in-shell legal warning box is commented out by default; the pre-auth SSH banner and the dynamic MOTD still carry the notice. Uncomment it in `00-matrix-banner.sh` to show it in the animation.
- Fixed the SYSTEM STATUS panel's top and bottom borders, which were one character narrower than its rows, using continuous `─` lines so no stray `-` shows in the border.

### Optional animated-banner-only install

- New `install.sh --no-motd` option for hosts where the dynamic MOTD and the animated banner duplicate each other on interactive logins. It skips the MOTD fragment and the update-count cache, and disables an already-installed `10-matrix-motd`. The default behaviour is unchanged.

### Faster login (update-count cache)

- **Fixed a slow login (~12 s on a Raspberry Pi 3).** `10-matrix-motd` used to run `apt-get --just-print upgrade` twice on every login. The counts are now read from `/var/cache/matrix-banner-updates`, which `matrix-update-cache` refreshes from an apt hook after every `apt update` / package change.
- **Fixed backups being executed.** The installer used to leave `10-matrix-motd.bak.*` next to the real script in `/etc/update-motd.d/`; because it kept its executable bit, `run-parts` ran the old slow script as well. Backups now go to `/var/backups/matrix-banner/` (non-executable), and re-running the installer moves any stale copies out of `update-motd.d`.
- Fixed `integer expression expected` in the MOTD when the failed-login count was `0` (`grep -c ... || echo 0` printed `0` twice).

---

## Licence

MIT — do whatever you like with it. A mention or a star is always appreciated.

---

*There is no place like 127.0.0.1.*
