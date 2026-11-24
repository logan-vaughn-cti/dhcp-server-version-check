# DHCP Server

**[⬇ Download CTI DHCP Server Setup v1_7_0.exe](https://dev.azure.com/CTI-AV/Skunkworks/_apis/git/repositories/DHCP-Server/items?path=/dist/CTI%20DHCP%20Server%20Setup%20v1_7_0.exe&versionDescriptor.version=main&versionDescriptor.versionType=branch&download=true&%24format=octetStream&api-version=7.1)**
(run it once; it installs for all users, and a newer setup upgrades in place)

A DHCP server for bench and commissioning networks. It hands out
addresses to the devices on one network adapter and tells you when each
one gets an address. It tells you when a newer version is out.

Up to 1.1.3 the DHCP engine was a heavily modified version of
[Tftpd64](https://github.com/PJO2/tftpd64)'s (its TFTP, DNS, SNTP and Syslog
servers, and everything else, were gone). From 1.2.0 the engine is CTI's
own, written clean-room to `docs/ENGINE_SPEC.md`; `docs/CLEAN_ROOM.md`
records how.

The download link above is the `dist\` setup on `main`. Changes to `main`
come in as pull requests, which the maintainer approves.

## Installing

1. Download the setup with the link above and run it. Windows asks for
   administrator rights once.
2. The setup isn't code-signed, so SmartScreen may say *"Windows protected
   your PC"*: choose *More info > Run anyway*. That happens once for each
   new setup, and never for the installed program.
3. **DHCP Server** is then in the Start menu (type "DHCP"), and in
   *Settings > Apps*, where it can be uninstalled.

The setup allows the program through Windows Firewall (UDP 67 and 68, for
this program only), so Windows doesn't ask. Uninstalling removes that rule
and the Start-menu entry, and keeps your settings and leases.

### Updates

At start-up the program checks whether a newer version exists. If one
does, a banner in the Windows accent colour says so. **From 1.3.0, click it
to install the update**: the program downloads the new setup, checks it
against the version file's SHA-256, and runs it (Windows asks for
administrator approval once). The setup closes the program, upgrades it and
opens it again, keeping the settings and leases. Serving stops while it
updates; press Start again afterwards. The download uses git with your own
Azure DevOps sign-in (git may show its sign-in window the first time). On a
PC without git, or if anything goes wrong, the banner opens the download in
your browser instead, as before: run the setup it saves. Versions before
1.3.0 always use the browser, so 1.3.0 itself is installed that way. To check on demand, click the **···** button (*More
options*) at the top right, next to Start, and choose **Check for updates…**.
The same menu has **Taskbar badge**, **Theme** (**System**, the default, follows
Windows' light or dark mode; **Light** or **Dark** keeps that one; applied at once and
remembered), **Change log…** (what each version
brought, newest first, built into the program from [CHANGELOG.md](CHANGELOG.md))
and **About**; Check for updates, Change log and About are also in the
window's system menu (Alt+Space). The check reads one
small file from `raw.githubusercontent.com`. It sends nothing about the PC
and downloads nothing until you click. Offline, it says so in the log and
carries on.

The last portable single-exe version (1.0.0, no installer) is git tag
`dhcp-portable`.

## Using it

**···** > **Help…** shows how to use the program, in the program itself.

1. Give the laptop's wired adapter a **static IP** (for example
   192.168.1.10 / 255.255.255.0) and plug it into the devices or their
   switch.
2. Open **DHCP Server** from the Start menu. Choose that adapter under **Network adapter**. The
   first time, a range on that adapter's network is proposed (for example
   .100 to .199). Choosing another adapter proposes a new range and mask
   when the current range isn't on that adapter's network; a range that
   fits is kept.
3. Press **Start**.

Each device that gets an address appears under **Leases**, and a Windows
notification says *"Assigned 192.168.1.100 to 00:05:A6:12:34:56"*. When
several devices boot at once, they're grouped into one notification.
Renewals don't notify.

Right-click a lease (or select it and press Shift+F10 or the menu key) to:

- copy its IP or MAC address;
- **Renew**: ask the device to renew now;
- **Revoke**: make the device take a new address. Its next request for the
  old one is refused, and the old address shows "Revoked" and is given to
  nobody until you delete it;
- **Block** / **Unblock**: ignore every request from the device. A blocked
  device with no lease still gets a row, so it can be unblocked;
- **always give that device that address** (a reservation);
- delete the lease.

Renew and Revoke send a FORCERENEW (RFC 3203) to the device. Many devices
ignore it; a revoked device then moves when it next renews on its own.

**Clear lease rules** is for aborting, or for a fresh start at a new site.
It removes every reservation, blocked device and revoked address, and forgets
every saved lease, except those of devices that are on their address right
now (checked by ARP). Those are kept, so their addresses are never handed to
another device, and while serving each one is asked to renew.

**Export CSV…** saves the lease list. For the log, **Copy all** copies the
whole log (to copy some lines, select them and press Ctrl+C or right-click >
Copy) and **Export…** saves a `.log` file. Both
save dialogs open in the settings file's folder. Log lines can be selected, with
Ctrl+C and Ctrl+A.

### The window and the tray

**Closing the window does not stop the server.** The close button hides the
window in the system tray, and a notification says so every time ("Still
serving 192.168.1.100 - 192.168.1.199" while serving). Click the tray icon,
or open DHCP Server again, to bring the window back. To quit, right-click the
tray icon and choose **Exit** (**Stop serving and exit** while serving).
Windows 11 puts new tray icons in the overflow (^); to keep this one in view,
turn **DHCP Server** on in *Settings > Personalization > Taskbar > Other system
tray icons*.

The **taskbar button carries a number**: the clients being served, counted
from the leases of the running server. **···** > **Taskbar badge** chooses
what it counts: **Online clients** (the default: active leases whose device
answers an ARP check, made every 20 seconds per device), **Clients with a
lease** (every active lease), or **Off**. While the window is hidden the
number moves to the tray icon. Stopped, there is no number.

The **Settings** and **Log** headers collapse their cards (click them;
the chevron shows which way). Collapsed, Settings shows only the network
adapter and the address range, as text, and Leases takes the room; a setting that Start rejects
opens the card again. Collapsed, Log shows a counter of what was logged
since ("12 new lines, 1 warning"); opening it resets the count. Click a lease column's header to sort by it, again to
reverse. The sort, the column widths, which cards are collapsed, the badge
choice and a maximized window are all remembered.

### Leases

While serving, the table lists the server's leases, then any saved leases
from an earlier range that have not expired ("Outside the range"). While
stopped it lists everything saved. Rows that are not a live lease are shown
in grey. For those, and for every active lease, the program checks with ARP
(every 20 seconds) whether the device is still on the address: "device
online", "device offline" or "another device has this address", so a device
unplugged while serving reads "Active - device offline". Windows' ARP cache can report a device that has just left
as online for a minute or so.

The **Vendor** column names the maker registered for the MAC's first three
bytes (for example `00:05:A6` = Extron Electronics). Names are looked up
online the first time a vendor prefix is seen (api.macvendors.com; only
those three bytes are sent, never the whole MAC) and kept in the settings
file, so each vendor is asked for once and works offline afterwards. With
no internet, as on most bench networks, the column stays blank until a later
lookup succeeds. A MAC whose second-lowest bit of the first byte is set
(a phone's random address, a virtual machine) reads "Locally administered"
and is never looked up. The exported CSV has the column too.

### Range, mask and DNS

- **The range can span several `x.x.x.0-255` blocks** (for example
  172.30.0.100 to 172.30.1.50 with a /23 mask). Addresses ending in **.0
  or .255 are never handed out**. The count beside the range excludes them.
- The range must stay inside one subnet for the mask. It can't include the
  subnet's network or broadcast address, the gateway, or this PC's own
  address. The limit is 1,024 addresses.
- **Subnet mask:** `255.255.254.0`, `/23` or `23`.
- **DNS primary / DNS secondary** are both optional. Choosing an adapter
  fills them with its own DNS servers when they are on its subnet (so
  devices can reach them), otherwise with **8.8.8.8** (Google) and
  **1.1.1.1** (Cloudflare, a second provider). A first run fills them the
  same way. Leave both empty to send no DNS server; that is kept across
  restarts. A secondary on its own is sent as the only DNS server.
- **"Check each address is free"** asks the network (ARP) for up to 1 s
  before offering a new address. It is skipped when a device is re-offered
  its own lease.

### It will not fight another DHCP server

DHCP Server never competes with a real DHCP server. **Before you press
Start** it already checks: when it opens (if an adapter is chosen), when you
choose another adapter or press Refresh, and every minute after that. While
another server answers, or another program holds UDP port 67, **Start is
greyed out** and a red banner says who it is. The banner shows the server's
address, its MAC address when ARP can find it (on the adapter's own subnet),
and the address it offered. Start comes back when the next check finds
nothing (port 67 is checked every 2 seconds). When the window opens, and
after choosing an adapter or pressing Refresh, Start also stays greyed out
for the few seconds the check takes ("Checking for other DHCP servers..."),
and never more than 10 seconds.

While serving, it stops, shows a red warning in the window and sends a
Windows notification when:

- **Another DHCP server is on the network.** When starting, and every
  minute while running, it sends one test request on the chosen adapter.
  Any answer means another server is there.
- **A device asks another server.** A client request naming a different
  server means one is live on the network.
- **Another DHCP program runs on this PC.** For example Tftpd64, or
  Windows Internet Connection Sharing / Mobile Hotspot, which appear as
  `svchost.exe`. The warning names the program.

It also never answers on any adapter except the one you chose, so being on
corporate Wi-Fi at the same time is safe.

### Duplicate IP addresses

While the program is open (in the tray too, serving or not), it watches the
chosen adapter's whole subnet for an address that two devices answer:
static addresses set twice, a device that kept an old address, a second
server's lease. Windows' own ARP check only ever reports the first device
that answers, so this needs **Npcap**, which comes with Wireshark (most of
us have it). Without Npcap the check is off and the log says so once.

- It listens to the ARP traffic on the adapter and, every 2 minutes, asks
  every address on the subnet once, a few at a time (subnets up to /22;
  larger ones are only listened to).
- When a second device answers an address the first one still answers, it
  asks that address again at once. Only when both answer again is it a
  duplicate: a **yellow banner** names the address and the devices' MAC
  addresses with their vendors where known (see Leases; it waits up to 3
  seconds for a name being looked up), the log has a WARNING line, and
  Windows shows a notification.
  Each address is reported once (again only if another device joins).
- A device replaced by another on the same address, a device checking an
  address before taking it, and one device answering for several addresses
  (a router) are not duplicates and are never reported.
- The banner's x dismisses it until the next duplicate; choosing another
  adapter clears it. A red banner (another DHCP server) takes its place
  while it shows, and the yellow one comes back when it goes.
- If Npcap was installed for administrators only, the log says so: run the
  program as administrator, or reinstall Npcap without that option.

## Settings file

Settings live in `CTI-DHCP.ini` in `%LOCALAPPDATA%\CTI-DHCP\`, one per
Windows user. (An exe run from outside Program Files, such as a development
build, keeps the file next to itself instead, unless that folder is
read-only.) The log's first line shows which file is in use. The program
itself writes nothing to the registry; the setup writes only its
uninstall entry.

- `[DHCP]` uses Tftpd64's key names (`IP_Pool`, `PoolSize`, `Mask`,
  `Gateway`, `DNS`, `DNS2`, `Lease (minutes)`). It also holds reservations
  as `MAC=IP` lines, for example `00:05:A6:12:34:56=192.168.1.50`, and the
  saved leases (`Lease_N_...`).
- `[EarlierLeases]` holds saved leases that were outside the range at the
  last Start and have not expired (`IP=MAC,renewed,offered`). Tftpd64 used
  to drop those, so one Start with a different range lost them. Now they
  move back into `[DHCP]` when the range covers them again.
- `[Blocked]` holds blocked devices (`MAC=1`) and `[Revoked]` revoked
  addresses (`IP=MAC`). Each section disappears when its last entry goes.
- `[Settings]` holds `Interface` (the chosen adapter's address),
  `CheckAddressFree`, `PersistentLeases`, `Notify`, `CheckUpdates`
  (default 1; 0 skips the start-up check, for example on a PC that is never
  online; the menu item still works), `Badge` (`online`, `leases` or `off`)
  and `LogLevel` (default 5; 15 logs every packet).
- `[Window]` holds the window's position and `Maximized`, the lease table's
  `LeaseColumns` widths, `SortColumn` (0 IP address, 1 MAC address,
  2 Vendor ... 5 Lease ends) and `SortDescending`, and
  `SettingsCollapsed` / `LogCollapsed`. A file from before 1.5.1 (five
  widths) is read in that layout: the Vendor column is added with its
  default width and a saved sort keeps its column.
- `[Vendors]` caches MAC vendor names by prefix (`0005A6=Extron
  Electronics`; `-` = the registry has no such prefix). Delete a line to
  have it looked up again.

The engine has no WINS, NTP, SIP, domain name, boot file or extra-option
support at all, so nothing is sent that the window doesn't show. Those
Tftpd64 keys are deleted from the file on every Start.

## Building

Requires MSYS2 with the UCRT64 gcc toolchain
(`pacman -S mingw-w64-ucrt-x86_64-gcc`) and, for the setup, Inno Setup 6
(`winget install JRSoftware.InnoSetup`). Then run:

```powershell
.\build.ps1
```

This produces `build\DHCP Server.exe` and, from it,
`dist\CTI DHCP Server Setup v<version>.exe`, e.g. `v1_1_3` (`installer\DHCP Server.iss`; the
setup's name carries the version, with underscores). `dist\` holds only the
setup, which is what is committed; the previous version's is removed. Without Inno Setup the
exe is still built, so the tests can run. `.\build.ps1 -Out <path>` writes
the exe elsewhere and skips the setup. The exe is x64, statically linked,
and imports only Windows system DLLs (the C runtime ships with Windows 10
and 11).

**The version** lives in `src\resource.h`: `APP_VERSION_STR` and
`APP_VERSION_NUM` change together. The installed exe's name never carries
it (`DHCP Server.exe`, so the firewall rule and shortcut stay valid); the
setup's name does. The settings file keeps its name (`CTI-DHCP.ini`)
across versions, so a new version finds the old settings.

**Releasing a version:**

1. Bump both version defines and update the download link at the top of
   this README to the new setup's name (it appears twice in that line).
   Add the version's entry at the top of [CHANGELOG.md](CHANGELOG.md): `## <version> -- <date>`,
   then a `- ` line for each thing a user gains. The program shows that file
   rendered, and knows only `#`, `##`, `- ` and `**bold**`.
   Run `.\build.ps1`; it warns if the link does not name the setup it just
   built, if the change log's first heading is not this version, or if it
   uses other markdown. Commit the setup with the change and merge the pull request into
   `main`.
2. **Right after that**, replace `version.json` in the public repo
   [logan-vaughn-cti/dhcp-server-version-check](https://github.com/logan-vaughn-cti/dhcp-server-version-check)
   with the line `build.ps1` printed (the GitHub web editor is enough): the
   new `version`, the new download link as `url`, and the setup's `sha256`
   (from 1.3.0; without it, installed copies fall back to the browser
   download). Not before the merge (users would be sent to a setup that is
   not there yet), and not much after (the merge removes the previous setup,
   so the old `url` stops working).

```json
{"version": "1.3.0", "url": "<the download link above>", "sha256": "<the setup's SHA-256>", "notes": "One line about what changed."}
```

The program downloads the setup from this repo itself (`APP_DOWNLOAD_URL`), never from the
version file's `url`: the public file can only say which version, and which hash to insist on.
`version` must be digits and dots. `url` is what the banner opens when the
program cannot install the update itself, and
must start with `https://`; without one, the program opens this repo's
page instead (`APP_DOWNLOAD_URL`). `notes` is optional. GitHub
caches the file for about five minutes. The repo holds only that file and
a README: no source and no exe. The address is `UPDATE_URL` in
`src\resource.h`.

### Icon

`assets\make_icon.ps1` builds `assets\app.ico`. It places the CTI triangle
mark (`assets\cti_mark.png`) bottom-right, and a DHCP badge
top-left, drawn under the mark where the two overlap. The badge is top-left
(from 1.3.0) because Windows draws the taskbar number bottom-right; the mark
is moved, never flipped. The badge is the Windows DHCP console icon (`dhcpsnap.dll`,
icon 0), saved per size as `assets\badge_<n>.png`. It is Microsoft artwork, so
replace it with original art if the program is ever given out beyond CTI.

Both CTI images come from CTI's brand logo files.
`cti_mark.png` is `CTI Triangle Logo.png` cropped to its alpha bounds
(608x555). `cti_logo.png`, the window header, is `CTI Color Logo.png`
scaled to 400 wide with its framing unchanged. The window fits it to the
header area, so a replacement of any size needs no code change.

### Look

The window follows the Windows 11 design and switches between light and
dark with the Windows app setting, live. Buttons, toggles and highlights use
the user's Windows accent colour; the font is Segoe UI Variable (Segoe UI on
Windows 10). All of it lives in `src\theme.c`: the real Windows controls are
kept and restyled around, so typing, tab order and screen readers behave as
before. In dark mode the header card is a light grey (`#EBEBEB`) because the
CTI logo's navy disappears on dark backgrounds.

The last version with the classic Windows look is git tag `dhcp-classic-ui`.

## Testing

```powershell
python tests\loopback_test.py
```

This runs the real exe against a scripted DHCP client over **loopback**,
so nothing is ever answered on a real network. Both test scripts close any
running copy of the program first (it allows one instance). It covers:

- the full request/offer/ack exchange and every option in the reply
- renewals not notifying
- leases surviving a restart, including an offered-but-unconfirmed one
  keeping its offer time
- reservations
- release
- all three "another DHCP server" cases, plus a stand-in server answering
  the test request (`--probe-to`): Start refused, and the answer named,
  relayed or not
- saved leases surviving a range change, and orphaned ones being recovered
- blocked devices, revoked addresses (the NAK) and Clear lease rules
  (the FORCERENEWs and what is left in the INI)
- the update check against a local version file: newer, current, older,
  a file with no version, 404 and no server; `--no-update-check` and
  `CheckUpdates=0`
- the window (W1-W5): the close button hides it to the tray and serving goes
  on, a second launch brings it back, Exit really ends it, and the sort,
  column widths, collapsed cards and badge choice are saved and read back

Since the close button only hides the window, the tests end the program
the way the tray menu's Exit does: a `WM_COMMAND` with `IDM_TRAY_EXIT`
posted to its window (`post_exit` in `loopback_test.py`).

```powershell
python tests\self_update_test.py
```

Installing an update from the banner, end to end, without installing
anything: a bare git repo on disk plays the Azure DevOps repo and a local web
server the version file; `--self-update-test` stops at logging the setup's
command. It checks the right hash, a wrong one (refused, deleted), none, and
no newer version. Needs git.

```powershell
python tests\change_log_test.py
```

"..." > Change log, opened through the system menu: the title, the built
version first, every version heading in order, a bullet for every `- ` line
and no markdown marker showing; the text follows a resize, the window never
shrinks below its first size, and Close closes it. It never presses Start.
The windows appear for a few seconds (no mouse); it needs any running copy
Exited.

```powershell
python tests\focus_ring_test.py
```

A click on a switch or a push button hides the keyboard-focus ring, and Tab
brings it back. Messages are sent to the window: no mouse, no keyboard. It
never presses Start; it needs any running copy Exited.

```powershell
python tests\duplicate_test.py [--live <adapter IPv4>]
```

The duplicate IP check (needs Npcap). It writes a capture file of crafted ARP
traffic and replays it through the program (`--duplicate-replay`): two
duplicates are reported, once each, with the yellow banner, and a replaced
device, a device back long after another, probes, a multicast sender and a
router answering for several addresses are not. `--live` also opens the
program on a real adapter (never Start) and injects ARP replies for two
unused addresses through Npcap, sent to this PC's own MAC only: a replaced
device stays quiet, two at once are reported, and a second capture sees the
program sweep the subnet and re-ask the suspect address. No mouse or keyboard.

```powershell
python tests\vendor_test.py
```

MAC vendor names, against a local stand-in for the lookup service
(`--vendor-url`; nothing reaches the internet): a duplicate names both
vendors and only the first three bytes of each MAC are asked for, once; the
names are cached in `[Vendors]` and a second run asks nothing; offline, the
duplicate still comes within a few seconds with the MACs alone; the lease
table's Vendor column shows a real vendor and "Locally administered" (never
sent); a pre-1.5.1 settings file keeps its widths and sort. Needs Npcap (the
duplicate part replays a capture). No mouse or keyboard.

```powershell
python tests\ui_test.py
```

This drives Renew, Block, Unblock, Revoke and Delete through the real lease
menu, opened from the keyboard, also over loopback. It then checks the
update banner in both themes and **Check for updates…** from the **···**
menu, and that Start is greyed out, with the banner saying why, while
another program holds port 67 or the stand-in server answers, then comes
back once it is gone (Refresh re-checks). The window and its menu
appear on screen while it runs, so leave the mouse and keyboard alone for the
half minute it takes.

Command-line switches used by the tests:

| Switch | Effect |
|---|---|
| `--ini <file>` | use this settings file |
| `--log <file>` | also write the log to a file |
| `--start` | press Start on launch |
| `--clear-rules` | with `--start`: Clear lease rules right after Start, without asking |
| `--allow-loopback` | list the loopback adapter (tests only) |
| `--theme light` / `--theme dark` | force the light or dark look instead of following Windows (screenshots, tests) |
| `--probe <adapter-ip>` | only check that adapter for another DHCP server, then exit (code 1 = found, 0 = none). Never serves anything. |
| `--probe-to <ip:port>` | send the test request for another DHCP server to this address, unicast, even on loopback (the tests' stand-in server) |
| `--update-url <url>` | read the version file from this address instead (tests use a local `http://127.0.0.1` server) |
| `--no-update-check` | skip the start-up update check (the tests pass it unless they are testing the check) |
| `--update-repo <url>` | download an update's setup from this git repo instead (tests use a bare repo on disk) |
| `--self-update-test` | take an update without asking, as an installed copy would, but only log the setup's command, then exit |
| `--duplicate-replay <file.pcap>` | feed this capture file through the duplicate IP check instead of the adapter (no subnet limit, nothing sent) |
| `--vendor-url <url>` | look MAC vendors up at `<url>XX-XX-XX` instead (tests use a local stand-in); `off` = the cache only, no lookups |

The tests default to `build\DHCP Server.exe`; pass another exe as the first
argument, for example the previous build as a negative control.

## Source layout

| Path | What |
|---|---|
| `src/dhcp_message.c`, `src/address_range.c`, `src/text_formats.c`, `src/address_probe.c`, `src/windows_error_text.c` | the engine's foundations (`docs/ENGINE_SPEC.md` round 1): the wire format and reply builder, the range, every text form, the address probe (ARP on the subnet, ICMP off it), Windows and ICMP error text |
| `src/lease_store.c`, `src/dhcp_responder.c`, `src/probe_cache.c`, `src/dhcp_service.c` | the engine's core (round 2): the lease table and its INI form, the decision for each request, the background probe cache, the receive loop and the sender |
| `src/ini_writer.c`, `src/event_log.c`, `src/competitor_check.c` | the server side the engine calls: queued INI writes with queue-aware reads, the log, the competitor latch |
| `src/server.c` | start/stop, competitor detection, adapters, lease access, lease rules, saved leases |
| `src/arp_probe.c` | ARP with a time cap: the address probe, the lease list's online check, the competitor's MAC |
| `src/duplicate_watch.c` | the duplicate IP check: Npcap loaded at run time, the ARP listener and sweep, the decision (`RecordAddressClaim`) |
| `src/vendor_lookup.c` | MAC vendor names: the `[Vendors]` cache, the paced online lookup of a prefix (first three bytes only) |
| `docs/ENGINE_SPEC.md`, `docs/CLEAN_ROOM.md` | the specification of the engine and the clean-room record of how it was written (the previous engine's files, as taken from Tftpd64 plus CTI's edits, are in the history up to 1.1.3's `src/` and, as reference, in `src/upstream/` until tag `dhcp-engine-round-2`) |
| `tests/unit/engine_unit_test.c` | the engine's unit tests, written from the specification (`build\engine_unit_test.exe`) |
| `src/config.c` | INI load/save |
| `src/main_window.c`, `src/app.rc` | the window, tray icon, notifications and the update check |
| `src/taskbar_badge.c` | the number on the taskbar button and the tray icon (an overlay icon drawn at run time; Repo Check's) |
| `src/self_update.c` | installing an update from the banner: git download, SHA-256 check, the silent setup (Repo Check's design) |
| `installer/DHCP Server.iss` | the setup (Inno Setup 6): install, Start menu, firewall rule, uninstall |
| `src/theme.c` | the Windows 11 look: light/dark, accent colour, cards, owner-drawn buttons and toggles |

## License

Copyright (c) 2026 Conference Technologies, Inc. All rights reserved
(`LICENSE.md`): a CTI tool, for use within CTI. Versions up to 1.1.3 were
GNU GPL v2 because their engine was taken from Tftpd64; they and their
source stay under that licence in the history (tag `dhcp-engine-round-2`
marks the last `main` that carried the old engine's files).
