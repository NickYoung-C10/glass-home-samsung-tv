# Cutting Samsung's ads and tracking — QN65S84FAEXZA (2025 S84F OLED)

Model-specific supplement to [SAMSUNG-SETTINGS.md](SAMSUNG-SETTINGS.md), which was
written for a 2023 Q70C on Tizen 8. This TV is a 2025 model on Tizen 9/10 (One UI
Tizen), where several menus moved. Paths marked **(moved)** differ from the
Tizen 8 guide; paths marked **(unverified)** are my best read of the One UI
layout and should be eyeballed before you commit to them.

Every change below is reversible except the PSID reset, which is harmless.

## What is NOT possible on this TV

Disabling Samsung's ad and telemetry *packages* over a debug shell — the
`pm disable-user` equivalent — does not work on retail Tizen:

- Tizen uses `sdb`, not `adb`. There is no `adb` on this platform.
- `sdb shell` is restricted on retail firmware; `pkgcmd -D` (disable, reversible)
  requires root.
- `sdb uninstall` on a signed system package fails with error 132 — the system
  partition is read-only and signature-checked.
- Public root exists only for specific older models and firmware. Nothing for a
  2025 S84F.

So the two levers that *do* work are the settings below and DNS filtering.

## 1. Privacy and promotions (remote, on the TV)

| Change | Where | Undo |
|---|---|---|
| Stop content recognition (ACR — identifies what's on screen, HDMI inputs included) | **Settings > All Settings > General & Privacy > Terms & Privacy** — untick **Viewing Information Services**, leave "I agree to all" unticked, move to **OK** | Same screen, tick it |
| Stop personalised ads | Same screen — untick **Interest-Based Advertisements** | Tick it again |
| Reset the advertising ID | Same screen > **Reset PSID** > Reset | By design, not undoable (nothing depends on the old ID) |
| Stop promotional pop-ups | **Settings > Notification** (just above All Settings) > gear icon > untick **Samsung Promotion** | Tick it again |
| Don't open Smart Hub at power-on | **Settings > All Settings > Advanced Features > Start Screen Option**. On 2025 models Advanced Features is a top-level item under Settings, *not* inside General & Privacy as it was on Tizen 8. Set **Start with Smart Hub Home** off | Switch it on |
| Voice / Bixby telemetry **(unverified)** | **General & Privacy > Voice** — turn off **Voice Recognition Services** | Turn back on |
| Mute Home-screen previews | Home > For You > scroll to the bottom > Media Options > **Autoplay Sound** | Press OK again |
| Tidy the Home row | Highlight a tile, hold **OK** > **Remove** (the app stays installed) | Apps > App Settings > the app > Add to Home |

### If "Autorun Last App" isn't in that menu

Several 2025 owners report the option missing from Start Screen Option
entirely, and some TVs still power on into Samsung TV Plus even with the
autorun toggles off. If that's the case here, look in the same menu for
**TV Plus Autorun Start** and turn it off; that is Samsung's own
moderators' workaround. Without Autorun Last App there is no supported way
to have the TV open Glass Home at power-on, which is the one thing Glass
Home relies on a TV setting for.

**Do not touch, they are adjacent to the above:**
- **Factory data reset** sat just below Start Screen Options on Tizen 8.
  Its position in the 2025 Advanced Features menu is unverified, so read
  before pressing either way.
- In App Settings, **Delete** uninstalls an app — use Home-row *Remove* instead.
- **Reset Smart Hub** (Support > Device Care) signs you out of everything.

## 2. DNS filtering — on the TV, not the router

The Netgear Orbi 860 is the wrong place for this. Its **Block Sites** feature is
keyword/HTTP-based and does not stop HTTPS, which is all of Samsung's telemetry;
and stock Orbi firmware cannot host a custom DNS blocklist. Setting the TV's own
DNS is better anyway: it is scoped to this one device and undoable from the
remote in fifteen seconds.

**Settings > All Settings > Connection > Network > Network Status > IP Settings >
DNS Setting > Enter manually**, then DNS Server `76.76.2.37` > Done > OK.
Leave the IP setting on automatic.

- That is Control D's free, no-log "HaGeZi Light" resolver (backup `76.76.10.37`).
- Number keys on the remote may not fill the DNS boxes; the on-screen keypad does.
- **Undo:** same screen > DNS Setting > **Get automatically** > OK.
- The TV accepts only one DNS server, so if Control D is ever down the TV has no
  internet. If the TV loses connectivity and nothing else does, revert this first.

### Verified against this resolver on 2026-11-09

Null-routed (returns 0.0.0.0):
`samsungacr.com`, `ads.samsungads.com`, `config.samsungads.com`,
`samsungtvads.com`, `log-config.samsungacr.com`

Still resolving, so these keep working:
`auth.samsungosp.com` (Samsung sign-in and the app store — needed to install and
update Netflix/Plex), `netflix.com`, `plex.tv`, `youtube.com`,
`time.samsungcloudsolution.com` (clock sync),
`geocoding-api.open-meteo.com` (Glass Home's weather)

`api.samsungcloudsolution.com` returns NXDOMAIN, but it does so on Cloudflare
too — that hostname does not exist and is not being blocked.

It will not remove YouTube's own ads or ads baked into a video stream.

## 3. What stays broken regardless

- The **New & Trending** / **Live** rows and the Smart Hub banner — no setting
  exists; DNS filtering blanks some of the ad tiles.
- The fixed left-hand rail (Daily+, Game).
- What the **Home** button opens. Glass Home works around this by starting at
  power-on instead.
