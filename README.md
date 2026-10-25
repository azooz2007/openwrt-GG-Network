<h1 align="center">GG Network for OpenWrt</h1>

<p align="center">
  <b>Route your games through GGamers servers and combos — from your router, for every device on your network.</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/version-1.4.6-19E3C0?style=for-the-badge" alt="version">
  &nbsp;
  <img src="https://img.shields.io/badge/OpenWrt-24.10%20%7C%2025.12%20%7C%20snapshot-00B5E2?style=for-the-badge" alt="OpenWrt">
</p>

<p align="center">
  <sub>Banana Pi BPI-R4 and any OpenWrt router · LuCI app · TCP &amp; UDP · Aggressive mode · updates itself</sub>
</p>

GG Network is the **GGamers Network tab of the Windows app, on your router**. Sign in with your GGamers
account, pick a server or a combo, press **Start** — your PC, console and phone play through GGamers,
with nothing to install on them. Everything is done in the router's web page (LuCI).

---

## Contents

1. [Features](#features)
2. [Requirements](#requirements)
3. [Download](#download)
4. [Install](#install)
5. [First start](#first-start)
6. [Updating](#updating)
7. [Uninstall](#uninstall)
8. [Troubleshooting](#troubleshooting)
9. [What's new](#whats-new)
10. [Credits](#credits)

---

## Features

- **Every device or only the ones you pick** — by name, MAC address or IP address.
- **Servers and combos** with flags and live ping; combos show your **ping in game**
  (your ping + the links + the last node to the game server), like the Windows app.
- **Aggressive mode** — GGamers' encrypted tunnel that sends every game packet more than once over your
  combo's route, so lost packets and ping spikes disappear. Staff choose Normal / single path /
  multipath, with a live picture of the route.
- **Only games go through GGamers**: web browsing, Discord and TeamSpeak voice, ZeroTier, Tailscale,
  WireGuard, AnyDesk, TeamViewer, Parsec, Google push and time sync go direct (switchable).
- **Live connections** — every tunneled connection with its game (icon + name, beta), route, traffic and
  start time; sort by any column; copy the address or build a combo from it.
- **Live chat**, account and subscription, combo marketplace, bandwidth graph, log.
- **Updates itself** from this page (on by default, only while no game is running).

## Requirements

| | |
|---|---|
| Router | OpenWrt **24.10** (opkg, `.ipk`) or **25.12 / snapshot / ImmortalWrt** (apk, `.apk`) |
| Firewall | `firewall4` (nftables) — the default since OpenWrt 22.03 |
| Space | about **10 MB** free flash |
| Account | a GGamers account with an active subscription |

The packages pull `kmod-nft-tproxy`, `ca-bundle` and `luci-base` by themselves.

## Download

Pick the file for your router's CPU and OpenWrt version:

| CPU (package architecture) | Routers | OpenWrt 24.10 | OpenWrt 25.12 / snapshot | Any version (installer) |
|---|---|---|---|---|
| `aarch64_cortex-a53` | Banana Pi BPI-R4 and most MediaTek Filogic / ARM64 routers | [`ggnet_1.4.6-r1_aarch64_cortex-a53.ipk`](ggnet_1.4.6-r1_aarch64_cortex-a53.ipk) | [`ggnet-1.4.6-r1-aarch64_cortex-a53.apk`](ggnet-1.4.6-r1-aarch64_cortex-a53.apk) | [`ggnet-1.4.6-r1-aarch64_cortex-a53-installer.tar.gz`](ggnet-1.4.6-r1-aarch64_cortex-a53-installer.tar.gz) |
| `x86_64` | x86 PCs, mini-PCs and virtual machines | [`ggnet_1.4.6-r1_x86_64.ipk`](ggnet_1.4.6-r1_x86_64.ipk) | [`ggnet-1.4.6-r1-x86_64.apk`](ggnet-1.4.6-r1-x86_64.apk) | [`ggnet-1.4.6-r1-x86_64-installer.tar.gz`](ggnet-1.4.6-r1-x86_64-installer.tar.gz) |

Not sure which CPU you have? On the router run `apk --print-arch` (25.12 / snapshot) or
`opkg print-architecture` (24.10).

## Install

**Easiest:** LuCI → **System → Software → Upload Package…** and pick the file
(for `.apk` add the key first, see below).

**OpenWrt 24.10** (`.ipk`) — copy the file to the router (e.g. with WinSCP to `/tmp`), then:

```sh
opkg update
opkg install /tmp/ggnet_1.4.6-r1_aarch64_cortex-a53.ipk
```

**OpenWrt 25.12 / snapshot / ImmortalWrt** (`.apk`) — trust the GG Network key once, then install:

```sh
wget -O /etc/apk/keys/ggnet.pem https://raw.githubusercontent.com/azooz2007/openwrt-GG-Network/main/ggnet-apk-key.pem
apk update
apk add /tmp/ggnet-1.4.6-r1-aarch64_cortex-a53.apk
```

(The file names above are for the BPI-R4; use the one for your CPU from the table.)

## First start

1. Open your router's page → **Services → GG Network**.
2. **Sign in** with your GGamers account.
3. Pick a **server or combo** — the ping shows next to each one.
4. Press **Start**.

Settings worth a look (**GG Network → Settings**): which devices use GG Network, the bypass switches,
start at boot, and automatic updates.

## Updating

GG Network checks this page every 6 hours and shows a banner when a new version is out.
With **Install updates automatically** on (the default), it installs it by itself while no game is
running, then reconnects. You can also press **About → Check for updates → Update**.
Your settings and sign-in are kept.

## Uninstall

```sh
opkg remove ggnet      # OpenWrt 24.10
apk del ggnet          # OpenWrt 25.12 / snapshot
```

Everything GG Network set up on the router is removed with it.

## Troubleshooting

| Problem | What to do |
|---|---|
| `UNTRUSTED signature` when installing the `.apk` | Add the key (the `wget` line in [Install](#install)), then try again. |
| "Choose your devices" when pressing Start | Settings → Devices: pick your PC/console, or switch to "All LAN devices". |
| "Panel unreachable, reconnecting…" | GGamers' server is busy or your internet dropped. Your games keep running; it reconnects by itself. |
| A game doesn't go through | Settings → Protocol & bypass: make sure its ports aren't bypassed (e.g. switch off "Bypass voice chat" for a game that uses STUN). |
| Nothing works after an update | Reboot the router once. Still stuck? Use **Report a problem** (bottom right) or contact us. |

## What's new

### 1.4.6

- BETA "Program" column: 36 more games from a second research round (346 with ports), e.g. SoulWorker,
  Lineage, Honkai Impact 3rd, HoN Reborn, FIFA 14-20, KartRider, Battlerite, BGMI, War Rock, World War Z,
  Rusty Hearts, Myth of Soma, Nexus. 16×16 icons (from each game's official site) for 40 of the games that
  had none (`tools/gamelist/extra_icons.py`), stored in the app like the others.
- **Connections fits the screen**: icon buttons (IP, IP:P, ⛓, ✕) with the full text as tooltips, a
  shorter route badge (🔥 AGGR), shorter titles, smaller text; long device and program names are cut
  with "…" (full name on hover).

Older versions: [CHANGELOG.md](CHANGELOG.md).

## Credits

Created by **Yakuza** · Telegram [@Yakuza4](https://t.me/Yakuza4) · Snapchat [@yakuuo](https://www.snapchat.com/add/yakuuo)
· Special thanks to RebootIQ · RebootSA · [GGamers.net](https://GGamers.net)

Country flags: flagcdn.com (public domain). World map: Natural Earth (public domain).

Free for personal use. The author is not responsible for any ToS violations resulting from proxy use in
online services. *Made by gamers for gamers ❤️*
