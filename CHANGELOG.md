# Changelog

## 1.7.2

Still client spec **1.43.0** (the panel has nothing newer for the app); brings the router level with
the Windows app 6.2.0 – 6.3.1.

- **I have an invitation code**: a new link on the sign-in card (next to *Sign up*) opens the panel's
  invitation sign-up page (`<server>/invite`). `status` has `invite_url`.
- **NanoPi R3S / R3S LTS** (and other `rockchip/armv8` routers): packages for `aarch64_generic` in
  every release (`.ipk`, `.apk`, installer). The router's updater picks them by itself.
- Docs brought up to date: README (version, spec 1.43.0, install file names, features),
  ARCHITECTURE (Aggressive Direct path and the three modes, Smart route), USAGE (Smart route, editing
  combos). The SDK feed Makefile still said `PKG_VERSION` 1.6.0.

## 1.7.1

Synced to client spec **1.43.0** (beta Smart route: 1.42.1 + 1.43.0).

- Smart route signs in as `<user>+smart` only: no device or game name any more. Aggressive mode
  sessions ask with `"smart": true` only.
- The connection card shows **one line per game server** of the last 10 minutes, newest first, each with
  its own exit: `You (23 ms) → Riyadh-1 → France-1 (exit) → 52.28.27.129:38090 · 84 ms · saves 13 ms`,
  or `… → Riyadh-1 (exit) → 13.234.10.5:443 · direct is fastest`. No ms is shown while the server has no
  number yet (`ms` 0). "Routes of this connection" has no game column; `how` is shown as sent
  (`measured: last hop 9 (…)`, `measured: ping (traced)`, `real game traffic …`).
- Each new route is logged once as `smartroute: 52.28.27.129:38090 via Riyadh-1 → France-1 (exit) 84 ms`.
- You still pick Smart route yourself (Normal route / Smart route).

## 1.7.0

Synced to client spec **1.42.0** (beta: Smart route). Built as one separate, removable piece
(`core/smartroute.go`, `api/smartroute.go`, `ggnet/smartroute.js`, one-line hooks marked
"beta (smartroute.go)").

- **🧭 Smart route (BETA)**, shown on the connection card when GGamers offers it
  (`client_config.smart_route`, re-read on every heartbeat; it disappears when withdrawn).
  **You pick it**, like Normal and Aggressive mode: *Normal route* or *🧭 Smart route*. Your pick is
  saved and applies to new game connections at once; running games keep their path.
- With Smart route picked, on a server that can do it (`smart: true`), GG Network signs in as
  `<user>+smart.<device>`: the LAN device's name is the game's name (`router` when the device has
  none), so each device keeps one exit for its game. GGamers takes every game server out through the
  fastest of up to 3 servers. A refused `+smart` sign-in is retried once without it and logged; Smart
  route then stays off until the next start. Combos are never re-routed.
- Aggressive mode: while `smart_route.aggressive` is on, the session asks for Smart route too
  (`"smart": true, "game": "router"`).
- While connected, the routes GGamers chose (`GET /api/client/smartroute`, every `poll_sec`) are shown
  from the first server to the exit: `You (23 ms) → Riyadh-1 → France-1 (exit) → game server · 84 ms`,
  "choosing the best path…" before the first one, "(direct is fastest)" when the entry is the exit.
  "Routes of this connection" lists game, game server, path, exit, ms, saved ms and how it was measured.
  Each new route is logged once: `smartroute: gaming-pc 52.28.27.129:38090 via Riyadh-1 → France-1 (exit) 84 ms`.

## 1.6.0

Synced to client spec **1.40.0** (editing combos).

- **Edit your combos**: an Edit button on your own combos opens the combo builder filled in (route,
  target country, game server, name, provider, connection type) as "Edit combo «name»"; Save changes
  sends the whole combo (`PATCH /api/client/combos/{id}`), Cancel closes it. When players attached it,
  you're told first that they keep it as it is now, as their own copy. After saving: "New connections
  use the new route. Games that are running keep theirs until they reconnect." "Nothing changed" just
  closes the editor; the panel's own messages are shown as they are.
- **Changes made elsewhere are picked up**: the heartbeat's `combos_rev` reloads the combo list when
  anything in it changed (an edit on the panel or another device, an attach, an admin switching a combo).
- **The combo you picked was changed**:
  - by its owner (you had attached it): GG Network switches silently to your private copy of the version
    you were using and says once "<owner> changed «name». You keep the version you were using as «copy».";
  - your own one, or an official one: "This combo was changed: the new route is used from your next
    connection."
  While connected, running game connections keep their route; new ones use the new route (SOCKS5 and a
  new Aggressive mode session). The old session finishes with the connections it carries, then closes.
- Combo cards show "Edited 27 Oct, 14:20" on your combos and "Changed by its owner …" on attached ones.

## 1.5.0

Synced to client spec **1.39.0** (Aggressive mode: three named modes, Direct path, separate ports).

- **Aggressive Direct path: Aggressive mode on a single server.** The old rule "Aggressive mode runs only
  on combos" is gone: on a server with Aggressive mode, GG Network asks for a `node_id` session and sends
  the copies straight to that server. The panel switches it with `aggressive_mode.direct` (missing = on);
  when it's off the log says "Aggressive Direct path is off on the panel" and the server uses the normal
  connection. A `403 feature_disabled` for a server session falls back to SOCKS5 the same way.
- **Three named modes** everywhere: Aggressive Direct path, Aggressive Combo Single-path, Aggressive Combo
  Multi-path. The session's `mode_name` is shown as is (Details, log); the staff connection choice uses the
  same names and offers Direct path on a server.
- **Every copy from its own local port** (`separate_ports`). GG Network asks for it; when the session says
  yes, path 0 gets one UDP socket per copy and the shortcut one, each on its own port, so the internet
  provider spreads the copies over different links. Every frame carries its socket's lane (HELLO and PING
  too), every socket gets its own WELCOME and PONGs, recovery reopens every socket, CLOSE goes from every
  socket. When the session says no (panel switch off, or an older node on the route) nothing changes.
- One replay window for all sockets, one ordered queue to the game: racing sockets can't reorder a stream.
  New tests (spec "required tests"): 3 sockets + shortcut with 0-30 ms random delay and 30% of copies lost
  both ways carry 1 MB exactly with matching fingerprints; with every copy but copy 1 dropped, UDP and TCP
  still work.
- Details: the mode name, "separate ports ✔/✘", `spacing_ms` when set, and one line per socket (path, copy,
  local port, WELCOME, round trip). Log at session start, e.g. `aggressive: Aggressive Combo Multi-path to
  frankfurt-1 · separate ports on (route copy 1 :51234, route copy 2 :51235, shortcut :51236)`. Tunnel
  counters are per socket (`p0c0 rx … ooo … dup …; p1c0 …`), in `tunnel_stream_end` too.
- The route picture also shows Direct path (you → the server → the game).

## 1.4.9

- **Fix: "This release has no .apk package for aarch64" on OpenWrt snapshot** (BPI-R4). The updater asked
  `apk --print-arch`, which on snapshot answers only the CPU family (`aarch64`). It now reads the package
  architecture from `/etc/apk/arch` (then `DISTRIB_ARCH`), e.g. `aarch64_cortex-a53`.
- The public folder also has a copy of the aarch64 `.apk` under the short name 1.4.7 and 1.4.8 look for,
  so those routers can update from the GG Network tab. A GitHub release of the same version gets the
  files only listed in `latest.json` added.

## 1.4.8

- A version to test updating from the GG Network tab (About → Check for updates, or the banner) on
  routers that run 1.4.7. No changes to how GG Network works.
- README: what to do when the `.apk` install says `UNTRUSTED signature` (add the key once and check
  with `apk verify`, download the real file, a different feed's warning, `--allow-untrusted` as a last
  resort).

## 1.4.7

- **Updates come from the public [openwrt-GG-Network](https://github.com/azooz2007/openwrt-GG-Network)
  repository** (install files only). The router reads its GitHub releases, picks the file for its CPU and
  package type (`.apk` on 25.12 / snapshot, `.ipk` on 24.10) and checks it against the SHA-256 GitHub
  publishes. With no release yet, or when GitHub's API limit is hit, it uses the repository's `latest.json`.
  Routers on 1.4.6 or older look in the old private repository, so install 1.4.7 by hand once.
- `build.sh` refreshes the public folder after every build (`tools/publish/publish.py`): this version's
  files, README, CHANGELOG, `latest.json`, the apk key, and the release notes to paste into GitHub.

## 1.4.6

- BETA "Program" column: 36 more games from a second research round (346 with ports), e.g. SoulWorker,
  Lineage, Honkai Impact 3rd, HoN Reborn, FIFA 14-20, KartRider, Battlerite, BGMI, War Rock, World War Z,
  Rusty Hearts, Myth of Soma, Nexus. 16×16 icons (from each game's official site) for 40 of the games that
  had none (`tools/gamelist/extra_icons.py`), stored in the app like the others.
- **Connections fits the screen**: icon buttons (IP, IP:P, ⛓, ✕) with the full text as tooltips, a
  shorter route badge (🔥 AGGR), shorter titles, smaller text; long device and program names are cut
  with "…" (full name on hover).

## 1.4.5

- BETA "Program" column: ports for 78 more games from web research (310 in all), e.g. PSO2/NGS,
  EverQuest II, Star Trek Online, Street Fighter V, Crossout, Warface, PlanetSide 2, Hero Siege, Trove,
  Darktide, Ghost Recon, Supreme Commander, Lineage, Dark Souls III, FFXI. Each result keeps its source in
  `tools/gamelist/research/*.out.tsv`; web ports and spans over 2000 ports are dropped when merging.

## 1.4.4

- **"Bypass web" also covers** Google push (TCP 5228–5230: Chrome, Android) and Cloudflare's alternative
  HTTPS ports (TCP 2053, 2083, 2087, 2096; Discord's gateway was on 2053). **NTP** (UDP 123) is never
  tunneled.
- **VPN sockets are learned**: a LAN socket that talks to a ZeroTier (9993), Tailscale (41641) or WireGuard
  (51820) port is that VPN, and all of its UDP goes direct for 10 minutes after, also to peers on random
  ports (ZeroTier's secondary ports sent to dozens of peers that way). Part of "Bypass voice chat &
  remote tools".
- **BETA: "Program" column on Connections** — the game a connection most likely belongs to, guessed from
  its port: icon + short name, "?" when several games share the port (the others in the tooltip). Display
  only, it never changes routing. About 900 games with their icons stored in the app (`ggnet/games/`),
  ports for about 230 of them plus popular games without an icon (`tools/gamelist/build.py`).

## 1.4.3

- **Bypass voice chat & remote tools** (new, on by default; Settings and "⚙ Protocol & options"): Discord
  (UDP 19294–19344, TCP 8443), TeamSpeak (UDP 9987, TCP 30033/10011), Mumble (64738), voice and video
  calls (STUN/TURN UDP 3478–3481, SIP 5060–5061), ZeroTier (UDP 9993 to and from), Tailscale (41641),
  WireGuard (51820), AnyDesk (TCP 7070), TeamViewer (5938) and Parsec (UDP 8000–8011) go direct instead of
  using GG Network's bandwidth (and Aggressive mode's copies). A router can't see the program, so they
  are recognised by port; checked on a real PC (Discord.exe used 104.29.153.x:19302/19340 and TCP 8443).
  Discord's other range (UDP 50000–65535) is left alone because game servers use it too. With "TCP only
  + block UDP" these keep working directly.

## 1.4.2

- **Fix: packets that already left the LAN were relayed again.** On a BPI-R4 behind a mobile CGNAT, a
  PC's own UDP (ZeroTier) came back to the router with the router's WAN address as source and showed up
  as a second device ("lan-10.18.130.57"), sent through the tunnel twice. Only datagrams whose source is
  inside a LAN interface's subnet are relayed now; the rest are dropped (logged once).
- The router's own public IPv4 (learned with STUN when the proxy starts) is never captured: a LAN device
  talking to it reaches the router, not a game server.
- **"Choose your devices"**: with "Only the devices below" and no device that can be found, Start
  refuses with a popup that leads to Settings → Devices, and Settings won't save an empty list.
- **Combos show their latency like the panel and the Windows client**: in the server menu every combo
  has its total (your live ping to the first node + the measured links, + the last node → game server
  when the combo has one: "in game", else "leave with"; "≈" when a part isn't measured yet), and the
  selected combo shows the total with its parts instead of the first node's ping. Every combo's first
  node is pinged after sign-in and every 30 s while LuCI is open.
- **Connections**: click a column title to sort by it (again to reverse; remembered), and a new
  **Started** column with each connection's start time.
- LuCI calls that time out in the browser now say "busy" instead of "the service isn't running".

## 1.4.1

- **Fix: "Panel unreachable" for good after starting the proxy** (seen on a BPI-R4). The daemon kept
  one HTTP/2 connection to the panel; when it silently died, every call waited for its timeout on it
  until ggnetd was restarted, and LuCI then showed "The GG Network service isn't running". Panel calls
  now use HTTP/1.1 only (ALPN offers `http/1.1`), a failed call closes its connection and the next one
  dials a fresh one; answers must start within 15 s.
- A LuCI call that takes too long now says "GG Network is busy: the GGamers panel is slow to answer"
  instead of "the service isn't running" (that message is kept for a daemon that really is down).
- Refresh asks for the subscription, servers and combos at the same time.

## 1.4.0

For GGamers server **1.38.0** (client spec 1.37.0, 1.37.1, 1.38.0). Everything new in the logs is
sent only while `investigation.tunnel_details` is on.

- **1.37: node sequence numbers from the clock.** A node's answers may start at ~1.79 × 10^15: the
  replay window slides forward (tested with a fake node numbering from its clock).
- **1.37.1: a restarted node → a new session.** Only frames older than the replay window count (copies
  of frames already used never do; any newer frame resets the count). 20 in a row or 1 s of them end the
  session locally (`tunnel_answer_rejected` `replay` "node restarted: new session"), the old game
  connections close at once (the game reconnects) and a new session is opened right away.
- **1.37: per-path counters** (`rx`, `ooo` out of order, `dup` copies) for the whole session, also in
  `tunnel_path_silent` and `tunnel_recovered`.
- **1.37: `tunnel_stream_end`** for every Aggressive mode TCP stream: who ended it (`game`, `app_game`,
  `app_reset`, `server_reset` with the RESET status, `tunnel_lost`), how long it lasted, bytes each way,
  chunks resent and the per-path counters.
- **1.38: stream fingerprints.** CRC-32 of the bytes given to the game (`down`) and read from it
  (`up`), with a mark every 16 KiB (at most 24), in `tunnel_stream_end` as `fp down=… up=…` (the spec's
  test vector passes in any chunk size). The router already hands a stream's chunks to the game through
  one ordered queue with a single reader, so it doesn't have the out-of-order bug 1.38 found.
- **1.37: crashes.** A panic in a tunnel or game-connection goroutine is caught: its stack goes to the
  log, `app_crash` is reported and the tunnel falls back instead of the daemon dying. A fatal error's
  full output is kept in `/etc/ggnet/crash.txt`; at the next start it goes into the log (so "Report a
  problem" carries the stack and the lines before it) and into `app_crash`.

## 1.3.0

For GGamers server **1.36.0** (client spec 1.30.1 → 1.36), and what the Windows client 5.8 has for
GG Network.

- **1.36: minimum app version.** Every call and the chat upgrade send `X-GG-Client: openwrt/1.3.0`.
  Below `min_app_version.openwrt` of `/status`, or on `426 update_required`: the message is shown with
  an Update button, the proxy and the tunnel stop (the account stays signed in), only `/status` is
  polled, and the proxy reconnects by itself once the version is accepted.
- **Updates from GitHub** (Settings → Updates, About → Updates): check every 6 h, install by hand or
  automatically (only while no game uses GG Network). The router's own package manager and architecture
  pick the file (`.apk` for apk, `.ipk` for opkg, e.g. `aarch64_cortex-a53`); size and SHA-256 are
  checked, apk also checks the GG Network signature. A version is auto-installed at most once.
- **1.35 Aggressive mode:** the tunnel's receive loop never stops on a socket error; a tunnel whose
  every path is silent for `recovery.silent_ms` is recovered on new local ports (`tries`, then SOCKS5
  for new connections when `fallback`); `investigation.tunnel_details` incidents (`tunnel_socket_error`,
  `tunnel_answer_rejected`, `tunnel_path_silent`, `tunnel_recovered`, `tunnel_recovery_failed`,
  `tunnel_fallback`) with the same lines in the log.
- **1.32:** `aggressive_mode.allowed`; `ipv4_only` (panel calls and chat forced to IPv4); `panel_url`
  followed after a verified check; Cloudflare's own error pages are "panel unreachable", never
  "tampered"; status polls back off 2/5/10/20/30 s ±20%; "Panel unreachable, reconnecting…" after 20 s;
  one `api_error` per outage of 20 s+ with the real reason (`dns`, `connect_timeout`, `refused`, `reset`,
  `tls`, `timeout`, `no_ipv4`, `522`…) and its duration.
- **1.33 tuning:** the panel can force or lock each router tuning switch (keys in docs/RPC.md); locked
  switches show the developer's note, the player's choice comes back when unlocked. New switches: TCP no
  delay, UDP silence rescue, UDP don't fragment.
- **1.30.1:** a PONG proves a path too; a path with PONGs is never "not reachable".
- **Chat (1.34):** click a name to reply (`@name`), messages that mention you are highlighted, staff
  edit messages ("(edited)"), staff menu (Reply, Edit, Delete, Pin, Mute, Ban, Unmute, Unban), unpin ✕
  and "Clear chat…" for admins, emoji picker, new-message tone with a mute box, 40 users online.
- **Combo builder:** "Your ping in game" while building (the last node tests the game server,
  `combos/gameprobe`), compared with your direct ping ("▼ You lowered your ping by …"), like the
  Windows builder; the target country comes from the panel's `combos/geoip` first, then HTTPS
  services (no more "Can't look up the location right now" on networks that block ip-api.com), only
  once the address is complete. **Use** reconnects through the combo while connected.
- **Connection choice like the Windows client** (staff): Normal connection / 🔥 Aggressive — single
  path / 🔥 Aggressive — multipath (only while GGamers has the backup path on). The mode ticks its
  boxes and the boxes move the mode; options GGamers has off are locked; frozen while connected;
  Aggressive mode needs a combo. A **route picture** shows the lines from you to the exit (copies knotted
  on one route, the backup path as its own line, nodes with flags, the exit with its latency, moving
  lights while traffic flows, unreachable lines red) — the plan before Start, the live session after.
- **Emojis in the chat** are drawn as Twemoji images (Windows browsers can't draw flags and many joined
  emojis), in messages, pins and the picker; without internet they stay text. The same emojis as the
  Windows client are left out.
- **Server menu:** every server is pinged right after sign-in and shows its ping and flag; latency
  pings run only while LuCI is open.
- New `.apk` signing key (the old private key was lost): routers that trusted the old key need the new
  one once, or one `apk add --allow-untrusted` upgrade.
- `mkapk.sh`: CRLF stripped from the install scripts (a Windows checkout made the `.apk` post-install
  fail with exit 127).

## 1.2.0

For GGamers server **1.30.0** (client spec §13, Investigation mode). Nothing changes while GGamers
has it switched off.

- **Incidents**: while `investigation.incidents` is on, the router records tunnel stalls / losses /
  refusals, failed and dropped proxy connections, proxy sign-in failures, game freezes, panel errors
  (5xx incl. 522, timeout, TLS, network), signature failures, key-renewal failures, DNS failures,
  reconnects, WAN changes, suspends and crashes, and sends them with `POST /api/client/incidents`
  every `incidents_every_sec` (buffer of 200 kept on disk for 24 h).
- **Report a problem** button (bottom right, every page) while `investigation.problem_reports` is
  on: note + when + the last `log_minutes` of the log (≤ `max_log_kb`), `POST
  /api/client/problem-report`, one report per 2 minutes.
- Secrets are redacted from every incident and report (`ggc_`, `gpk_`, `Bearer`, `password=`,
  `X-GG-Signature`).
- The `investigation` object is read from login, `/me`, the status poll and the heartbeat.
- Contract test for the incident and report bodies against the server's field set.

## 1.0.1

- **Block IPv6 for tunneled devices** (new, default on): their internet IPv6 is refused so apps use
  IPv4, which goes through GG Network. Fixes "what is my IP" sites showing the home address over IPv6.
- **Auto-stop when idle** (new, default on, 60 min, 5–1440): the proxy stops by itself after a while
  with no traffic and no open connection.
- Silence rescue only for a UDP relay that was answering and went quiet, at most 3 times per flow,
  logged once (no more log spam for one-way UDP or ports GGamers blocks).
- Signed `.apk` packages (GG Network key) for OpenWrt 25.12 / snapshot / ImmortalWrt; the package
  installs the public key for later upgrades.
- Version shown in About; sign-in card with logo, *Sign up* and *Forgot password?*.
- Connection page: Connection and Chat frames side by side; account details moved to the **Account**
  tab; **About** tab; **Test connection**; server/combo locked while connected; flags and live
  latency (3 s) for the selected node.
- Combo builder: world map, target country located from the game server IP, router direct ping to
  the game server, "you lowered your ping by" on saved combos.
- Country flags everywhere (images, not emoji).
- Chat: the 10 highest-ranked online users.
- `build.sh`: more CPU families (i386, ARMv5/v6/v7, MIPS64, RISC-V 64, LoongArch64); Go ≥ 1.23.

## 1.0.0

- First version: the GGamers Network tab of the Windows client as an OpenWrt package (`ggnetd` +
  `luci-app-ggnet`): signed sign-in, servers, combos (list, marketplace, builder), Aggressive mode
  (§12), maintenance, key renewal, heartbeat, usage, chat.
- LAN capture with nftables TPROXY; device selection by MAC / IP / CIDR / name; protocol & bypass
  options; tunnel DNS; router gaming preset; Connections page; Log with bandwidth graph.
- `.ipk` for OpenWrt 24.10, installer bundle, SDK feed Makefile.
