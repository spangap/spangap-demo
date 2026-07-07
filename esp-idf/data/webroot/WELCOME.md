# Welcome!

This device runs **Spangap Demo** — the reference image of **Spangap**
(github.com/spangap), an open-source application framework for the ESP32.
It turns a small, cheap, low-power module into a self-contained networked
computer: a genuinely multitasking firmware with a full web interface, a
unix-like command line, secure remote access, and — on boards with a
screen — a phone-style touch UI.

Everything you see is served by the device itself. There is no cloud
account, no app to install, and nothing leaves your network unless you ask
it to.

This page is the built-in manual. Read it here, on the screen, or from any
browser on the same network.

---

## Getting online

Out of the box the device looks for a known WiFi network for about 20
seconds, then starts its **own access point**: an open network named
`spangap_` plus four hex characters. Connect to it and browse to
**https://192.168.1.1**. Your browser will warn about the self-signed
certificate — that's expected on first contact; accept it.

The first visit lands on the **setup page**: choose the device password.
That password guards the web interface and SSH from then on.

Then teach the device your own WiFi: **Settings → Internet → WiFi**, scan,
pick your network, enter its password. The device joins it and is from
then on reachable at **https://spangap.local**. You can store several
networks; it joins whichever it finds.

Prefer the command line? On the USB serial console (or later over SSH):

```
net add MyNetwork mypassword
net                        # status: SSID, IP, traffic
set s.net.hostname=myname  # optional: rename the device
```

## The web interface

The web app is a small desktop: a **Dock** of apps, each opening a
floating window — **Settings** (every knob on the device, in one tree),
a **Terminal** running the device CLI, the live system **Log**, and this
**Viewer**.

The interface is not a snapshot: it is live-synced with the device. Change
a setting on the LCD, in the CLI, or in another browser tab, and you watch
it move here. A login stays valid for 60 days per browser.

## The command line

The same CLI is available everywhere: on the USB serial port, in the web
Terminal, in the LCD's CLI app, and over SSH. Start with `help`.

All configuration is one hierarchical store shared by firmware, browser
and LCD:

```
show s.net                    # list a subtree
set s.ntp.tz=Europe/Amsterdam
set s.lcd.backlight=80
save                          # flush to flash now
```

Three kinds of keys:

- `s.*` — settings: saved to flash and synced to all the UIs.
- `secrets.*` — saved to flash, but never leave the device.
- everything else — live status, gone at reboot.

A `set` takes effect immediately wherever the value is used — try the
backlight example on a board with a screen and watch the slider move in
the web UI at the same time. That one store, synchronized everywhere, is
the heart of the platform.

## SSH

An SSH server is built in (port 22). It admits nobody until you set the
device password (setup page, or `passwd` in the CLI) or authorize a key:

```
sshd add ssh-ed25519 AAAA... laptop
```

Then `ssh admin@spangap.local` gives you the CLI, and
`ssh admin@spangap.local 'net'` runs a single command. The device also
carries an SSH *client* (`ssh`, `ssh-keygen`) for reaching out.

## Reaching it from the internet

All optional, all off by default, all under **Settings → Internet**:

- **WireGuard** — the simplest safe route in: the device dials your
  WireGuard server and is reachable through the tunnel. `wg keygen`
  prints its public key; set `s.wg.endpoint`, `s.wg.address` and
  `s.wg.peer_pubkey`, then `wg up`.
- **UPnP** — asks your router to forward the HTTPS and WebRTC ports.
- **DuckDNS** — keeps a public hostname pointed at your changing IP.
- **ACME** — fetches a real, browser-trusted TLS certificate (Let's
  Encrypt) for that hostname, and renews it automatically.

UPnP + DuckDNS + ACME together make the device a proper public HTTPS
site; WireGuard replaces all three when you'd rather keep it private.

## Time and scheduled jobs

The clock syncs over NTP once the device is online. Set your timezone
with `set s.ntp.tz=Europe/Amsterdam` (any IANA name) — the browser
suggests yours automatically on first connect. `date` shows the result.

A deep-sleep-aware cron runs `/state/crontab`
(`min hour dom month dow flags command`); services like DuckDNS and ACME
schedule their own upkeep there.

## If your board has a screen

Boards like the LilyGo T-Deck get the full phone-style UI: an app
launcher with a status bar (clock, WiFi, battery) and the same Settings,
Log, CLI and Info apps you have in the browser. Swipe up from the bottom
edge to go Home — stop halfway for the recents switcher. Escape goes
back; holding the home key returns to the launcher.

## Under the hood — and yours to build on

Spangap is built from **straddles**: self-contained modules that each
bundle their firmware, browser UI and on-screen UI, and declare what they
need. This image is one selection of them; [BUILD.md](BUILD.md) lists the
exact commit of every straddle it was built from.

For firmware authors the platform means starting at the interesting part:
real multitasking without polling loops, the synchronized config store,
the web and LCD shells, remote access and this viewer are already there.

It is open source, Apache-2.0. Take it apart, build your own image, write
a straddle: **github.com/spangap**. And for a full-scale demonstration of
what a straddle family can do, see **Reticulous** (github.com/reticulous)
— a complete Reticulum mesh networking device built on this platform.

---

*About this viewer: press **Space** for the address bar, use **Back** to
retrace links. [How it works](ABOUT.md) — [Build manifest](BUILD.md)*
