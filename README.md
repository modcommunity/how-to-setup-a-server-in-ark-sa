In this guide we will be **downloading**, **configuring** and **running** an [ARK: Survival Ascended](https://store.steampowered.com/app/2399830/ARK_Survival_Ascended/) dedicated server.

ASA servers are heavier than most. The Unreal Engine 5 rewrite pushed the memory requirements up sharply, and there is one platform fact that shapes this entire guide: **ARK: Survival Ascended has no native Linux dedicated server**. Wildcard ship a Windows binary only, and Linux hosts run it through Proton or Wine.

We cover Windows properly, then the Proton route for Linux with `apt` and `dnf` instructions, since that is what most people running ASA on a Linux box are actually doing.

[**View Guide On TMC (Recommended Due To Better Formatting)**](https://moddingcommunity.com/blog/how-to-setup-an-ark-survival-ascended-server/)

## Table Of Contents
* [Requirements](#requirements)
    * [Memory Per Map](#memory-per-map)
    * [The Licence Restriction](#the-licence-restriction)
* [Downloading The Server Files](#downloading-the-server-files)
* [Windows Prerequisites](#windows-prerequisites)
    * [Windows Server And The Certificate Revocation List](#windows-server-and-the-certificate-revocation-list)
* [Map Names](#map-names)
* [Launch Options](#launch-options)
    * [The Three ASA Syntax Traps](#the-three-asa-syntax-traps)
* [Starting The Server On Windows](#starting-the-server-on-windows)
    * [Creating A Startup Script](#creating-a-startup-script)
* [Running On Linux Through Proton](#running-on-linux-through-proton)
    * [Installing Dependencies](#installing-dependencies)
    * [Running It](#running-it)
    * [Keeping It Running](#keeping-it-running)
* [Ports And Port Forwarding](#ports-and-port-forwarding)
    * [Multiple Servers On One Machine](#multiple-servers-on-one-machine)
* [Configuration Files](#configuration-files)
* [RCON](#rcon)
* [Adding Mods](#adding-mods)
* [Updating The Server](#updating-the-server)
* [Troubleshooting](#troubleshooting)
* [Conclusion](#conclusion)
* [See Also](#see-also)

## Requirements
* **Windows 10 22H2** or later, or **Windows Server 2016** or later. Both Microsoft and Steam need to support the version you pick.
* Alternatively, a modern **Linux** distribution running the server under **Proton** or **Wine**.
* A **64-bit** system. The server binary is 64-bit only.
* Around **11 GiB** of disk space per server install, plus room for saves, profiles, logs, updates and mods. Budget 20 GB to be comfortable.
* **4 logical cores** per server instance is the recommendation. Single-thread performance still matters more than core count, since gameplay outside of physics runs on one thread.
* A lot of RAM. See below.

### Memory Per Map
This is the number that surprises people coming from ARK: Survival Evolved. These are for an **empty** map with default settings and nobody connected:

| Map | Level name | Memory on an empty map |
| --- | ---------- | ---------------------- |
| The Island | `TheIsland_WP` | 8 - 10 GiB |
| The Center | `TheCenter_WP` | 10.5 - 12 GiB |
| Scorched Earth | `ScorchedEarth_WP` | 7.5 - 9 GiB |
| Aberration | `Aberration_WP` | 8 - 10 GiB |
| Extinction | `Extinction_WP` | 7 - 8.5 GiB |

Memory climbs from there with connected players, with the age of the world as structures and tamed creatures accumulate, and with mods. Usage also spikes during the creature respawn phase, which lasts several minutes.

For comparison, the same maps on ASE sit around 3 - 5 GiB. Plan for **16 GB minimum** on an ASA box and more if you want a busy server on The Center.

### The Licence Restriction
Worth knowing before you spend money on hosting. Under ARK's end-user licence agreement, dedicated servers may only be hosted for personal, non-commercial purposes unless you obtain a dedicated server licence from Nitrado.

Donations are fine as long as the donor gets nothing of value in return, and so is recovering your operational costs. Making money beyond that is not.

## Downloading The Server Files
The ASA dedicated server is Steam app **2430930**, and it is free with an anonymous login.

You will want [SteamCMD](https://developer.valvesoftware.com/wiki/SteamCMD) for this. We have a separate guide on it:

https://moddingcommunity.com/blog/how-to-download-run-steamcmd/

Once SteamCMD is installed:

```
login anonymous
force_install_dir C:\ark-asa
app_update 2430930 validate
quit
```

Or as a single command:

```batch
steamcmd.exe +login anonymous +force_install_dir C:\ark-asa +app_update 2430930 validate +quit
```

**NOTE** - Steam's CDN sometimes deprioritises or outright rejects anonymous logins when it is busy, which is especially common when a server tries to pull several mods at once. If the download keeps stalling, logging in with a real Steam account instead usually clears it up.

The server executable ends up at:

```
<install folder>\ShooterGame\Binaries\Win64\ArkAscendedServer.exe
```

That is also where your startup script should live.

## Windows Prerequisites
The server needs two runtimes that are not bundled:

* [Microsoft Visual C++ 2013 Redistributable, x64](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist)
* [DirectX End-User Runtimes, June 2010](https://www.microsoft.com/en-us/download/details.aspx?id=8109)

Install both before your first launch. A missing VC++ redistributable is the single most common reason a fresh ASA server exits instantly with no useful error.

### Windows Server And The Certificate Revocation List
If you are on a Windows Server edition rather than desktop Windows, there is an extra step, and skipping it means your server never appears in the browser.

Windows Server handles certificates and revocation lists differently, and TLS 1.2 has to be enabled and configured correctly. You also need to install a certificate revocation list manually:

1. Download `r2m02.crl` from [dev.epicgames.com](https://dev.epicgames.com/), or directly from [crl.r2m02.amazontrust.com/r2m02.crl](http://crl.r2m02.amazontrust.com/r2m02.crl).
2. Right-click the downloaded file and choose to install it.
3. Select **Place all certificates in the following store** and pick **Trusted Root Certification Authorities**.
4. Restart the machine.

**WARNING** - If you set a server up before 18 November 2023 and installed the `r2m02.cer` **certificate**, that certificate has since been revoked and has to be removed or your server stays invisible. Run `certmgr.msc` and `certlm.msc`, search both for `Amazon RSA 2048 M02` under Trusted Root Certification Authorities, remove it, and restart.

## Map Names
ASA uses World Partition level names, which end in `_WP` and differ from the ASE names. Getting this wrong means the server refuses to start.

| Map | ASA level name |
| --- | -------------- |
| The Island | `TheIsland_WP` |
| The Center | `TheCenter_WP` |
| Scorched Earth | `ScorchedEarth_WP` |
| Ragnarok | `Ragnarok_WP` |
| Aberration | `Aberration_WP` |
| Extinction | `Extinction_WP` |
| Astraeos | `Astraeos_WP` |
| Lost Colony | `LostColony_WP` |
| Club ARK | `BobsMissions_WP` |

The map name comes immediately after the executable, before any other option.

## Launch Options
ASA mixes two syntaxes on one command line, which is the root of most configuration problems.

**Question mark options** come straight after the map name, chained with `?`:

| Option | Description |
| ------ | ----------- |
| `?SessionName=<name>` | The server name shown in the browser. |
| `?ServerPassword=<password>` | Password players need to join. Omit the whole option for an open server. |
| `?ServerAdminPassword=<password>` | The admin password, also used by RCON. |
| `?RCONEnabled=True` | Turns RCON on. |
| `?RCONPort=27020` | The RCON TCP port. |

**Hyphen options** come after those, separated by spaces:

| Option | Default | Description |
| ------ | ------- | ----------- |
| `-port=<port>` | `7777` | The UDP game port. |
| `-WinLiveMaxPlayers=<n>` | `70` | Maximum players. |
| `-mods=<id1,id2>` | *N/A* | CurseForge Project IDs, updated automatically on startup. |
| `-NoBattlEye` | off | Runs without BattlEye. |
| `-log` | off | Writes a console log. |
| `-MULTIHOME` | off | Binds to a specific IP. Needs `MULTIHOME=<ip>` set too. |
| `-NoTransferFromFiltering` | off | Disables ARK Data transfers between single player and non-clustered servers. |

### The Three ASA Syntax Traps
These catch nearly everybody once.

1. **`?ServerAdminPassword=` must be the last `?` option.** ASA parses everything after it as part of the password. Put it at the end of the `?` chain, before your hyphen options.
2. **ASA needs `-port=<port>`, not `?Port=<port>`.** Use the `?` form and it silently ignores you and stays on 7777. This is a change from ASE.
3. **`ActiveMods` in `GameUserSettings.ini` does nothing.** It is an ASE setting. ASA uses `-mods=` on the command line and does not use Steam Workshop at all.

## Starting The Server On Windows
Open a terminal in `ShooterGame\Binaries\Win64` and run:

```batch
.\ArkAscendedServer.exe TheIsland_WP?SessionName="My ASA Server"?ServerPassword=joinme?ServerAdminPassword=changeme -port=7777 -WinLiveMaxPlayers=20 -log
```

First start takes a while, since the server generates the world. Watch for it appearing in the in-game unofficial server browser to confirm it is up.

### Creating A Startup Script
Typing that out every time gets old. Create `start.bat` in the same folder as `ArkAscendedServer.exe`:

```batch
@echo off
setlocal

set SESSIONNAME=My ASA Server
set MAP=TheIsland_WP
set PORT=7777
set MAXPLAYERS=20
set ADMINPASS=changeme
set RESTART_DELAY=10

:restart
cls
echo Starting ARK: Survival Ascended server...
echo.

.\ArkAscendedServer.exe %MAP%?SessionName="%SESSIONNAME%"?RCONEnabled=True?RCONPort=27020?ServerAdminPassword=%ADMINPASS% ^
    -port=%PORT% ^
    -WinLiveMaxPlayers=%MAXPLAYERS% ^
    -log

echo.
echo Server stopped or crashed. Restarting in %RESTART_DELAY% seconds...
timeout /t %RESTART_DELAY% /nobreak > nul
goto restart
```

Edit the variables at the top, save, and double-click it. The script restarts the server automatically if it crashes or is shut down.

**TIP** - Note where `?ServerAdminPassword=` sits: last in the `?` chain, immediately before the hyphen options. That ordering is deliberate.

## Running On Linux Through Proton
There is no native Linux build of the ASA server. What follows is the community approach, and it works well, but it is not something Wildcard supports.

### Installing Dependencies
You need SteamCMD and a Proton or Wine runtime.

**Debian and Ubuntu:**

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository -y multiverse
sudo apt install -y steamcmd lib32gcc-s1 wine winetricks screen curl tar
```

**Fedora, RHEL and Rocky:**

```bash
sudo dnf install -y glibc.i686 libstdc++.i686 wine winetricks screen curl tar
# SteamCMD is not packaged on all RHEL-family distros, so fetch it directly:
mkdir -p ~/steamcmd && cd ~/steamcmd
curl -sqL "https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz" | tar zxvf -
```

Then download the server files exactly as on Windows, using the Windows platform override so SteamCMD fetches the Windows binaries:

```bash
./steamcmd.sh +@sSteamCmdForcePlatformType windows +login anonymous \
    +force_install_dir /home/ark/asa +app_update 2430930 validate +quit
```

That `+@sSteamCmdForcePlatformType windows` line is essential. Without it SteamCMD looks for a Linux depot that does not exist.

**NOTE** - Most people running ASA on Linux use [GE-Proton](https://github.com/GloriousEggroll/proton-ge-custom) rather than plain Wine, since it handles the Epic Online Services bits more reliably. Several community wrapper projects exist that set the whole prefix up for you, and they are worth looking at before you build one yourself.

### Running It
With a Wine prefix set up, the launch command mirrors the Windows one:

```bash
export WINEPREFIX=/home/ark/.wine-asa
export WINEARCH=win64

wine /home/ark/asa/ShooterGame/Binaries/Win64/ArkAscendedServer.exe \
    "TheIsland_WP?SessionName=My ASA Server?RCONEnabled=True?RCONPort=27020?ServerAdminPassword=changeme" \
    -port=7777 -WinLiveMaxPlayers=20 -log
```

Expect this to be rougher than a native server. Startup is slower, memory use is higher, and diagnosing problems means working out whether the fault is ASA's or the Wine prefix's.

### Keeping It Running
Wrap the above in a script and run it under `screen`:

```bash
screen -S asa ./start_asa.sh
```

Detach with `CTRL` + `A` then `D`, and reattach with `screen -r asa`.

For something more permanent, a systemd unit at `/etc/systemd/system/asa.service`:

```ini
[Unit]
Description=ARK Survival Ascended Dedicated Server
After=network.target

[Service]
Type=simple
User=ark
WorkingDirectory=/home/ark/asa/ShooterGame/Binaries/Win64
ExecStart=/home/ark/start_asa.sh
Restart=on-failure
RestartSec=30
TimeoutStopSec=180

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now asa
```

The generous `TimeoutStopSec` is deliberate. ASA takes a while to save and shut down, and cutting it short risks losing progress.

## Ports And Port Forwarding
The server listens on these local ports. Forward them on your router and allow them through the OS firewall.

| Protocol | Port | Purpose |
| -------- | ---- | ------- |
| UDP | 7777 | Game port |
| UDP | 7778 | Peer port, always game port + 1 |
| UDP | 27015 | Query port, for the Steam server browser only |
| TCP | 27020 | RCON, if enabled |

**WARNING** - ARK's server does not support UPnP or any other automatic port forwarding, so you have to do this by hand on your router.

The peer port is used by Steam's P2P protocol for in-game browser advertising. Epic clients use standard ports 80 and 443 with TLS 1.2 instead.

On Linux:

```bash
# Debian/Ubuntu with ufw
sudo ufw allow 7777:7778/udp
sudo ufw allow 27015/udp
sudo ufw allow 27020/tcp

# Fedora/RHEL with firewalld
sudo firewall-cmd --permanent --add-port=7777-7778/udp
sudo firewall-cmd --permanent --add-port=27015/udp
sudo firewall-cmd --permanent --add-port=27020/tcp
sudo firewall-cmd --reload
```

### Multiple Servers On One Machine
Supported, and the usual approach is to step the ports up in blocks:

| Instance | Game (UDP) | Peer (UDP) | Query (UDP) | RCON (TCP) |
| -------- | ---------- | ---------- | ----------- | ---------- |
| 1 | 7777 | 7778 | 27015 | 27020 |
| 2 | 7779 | 7780 | 27016 | 27021 |
| 3 | 7781 | 7782 | 27017 | 27022 |

Completely different port numbers work too. The one constraint is that the Steam server browser cannot find servers with a query port above 27020 unless the query port is provided explicitly.

Remember the memory table when you do this. Three ASA instances on The Island is 30 GiB before anybody joins.

## Configuration Files
Beyond launch options, the bulk of server configuration lives in two `.ini` files:

```
<install folder>\ShooterGame\Saved\Config\WindowsServer\GameUserSettings.ini
<install folder>\ShooterGame\Saved\Config\WindowsServer\Game.ini
```

They are only created after the first launch, so start the server once before editing them.

`GameUserSettings.ini` holds most day-to-day settings under `[ServerSettings]`: harvest and XP multipliers, taming speed, day and night length, structure limits, PvP toggles. `Game.ini` holds the deeper tuning such as per-level engram points and individual creature stat multipliers.

The [ARK Wiki server configuration page](https://ark.wiki.gg/wiki/Server_configuration) is the complete reference for both and is genuinely exhaustive. It is the page to keep open while you tune.

**TIP** - Stop the server before editing the `.ini` files. ASA rewrites them on shutdown and will happily overwrite your changes if it is running while you edit.

## RCON
RCON lets you run admin commands against the server remotely, which is much nicer than alt-tabbing to the console window.

Turn it on with these `?` options:

```
?RCONEnabled=True?RCONPort=27020
```

`ServerAdminPassword` doubles as the RCON password, so it has to be set for RCON to work. Make it a real password, and only give it to people you trust, since it grants full admin control.

Forward TCP 27020 to use RCON from outside your network, and only do that if you understand the exposure.

**NOTE** - Our own [TMC App](https://moddingcommunity.com/tmc-app) has RCON built in, with full command history and logging, commands sent to several servers at once, and commands scheduled for later. It also has a server browser with real-time latency and status graphs. If you are running more than one ARK server it is worth a look, with one caveat: **it is in very early development** and its README says as much, so treat it as partially tested. Trying it and telling us what broke is the most useful thing anyone can do for it at this stage. It is open source under GPL-3.0 at [github.com/modcommunity/tmc-app](https://github.com/modcommunity/tmc-app), and bug reports and feature requests are welcome in [the issue tracker](https://github.com/modcommunity/tmc-app/issues).

## Adding Mods
ASA mods come from [CurseForge](https://www.curseforge.com/ark-survival-ascended), not Steam Workshop, and they are added with `-mods=` using CurseForge **Project IDs**:

```
-mods=1163881
```

Several are comma separated with no spaces:

```
-mods=1163881,927131,893657
```

The server downloads and updates them itself on startup. Load order runs left to right, with the leftmost ID taking priority over those after it. A custom map mod normally goes first.

Players do not need to install anything in advance. Joining a modded ASA server downloads the mods to the client automatically, which is also how it works on PS5 and Xbox Series consoles.

Our separate [ARK: Survival Ascended mod guide](https://moddingcommunity.com/blog/how-to-install-mods-in-ark-survival-ascended/) covers the client side and how to find Project IDs.

## Updating The Server
ASA clients refuse to connect to a server running an older build, so you update after every game patch.

Re-run the same SteamCMD command you used to install:

```batch
steamcmd.exe +login anonymous +force_install_dir C:\ark-asa +app_update 2430930 validate +quit
```

Stop the server first. Updating files underneath a running server is a good way to corrupt something.

Mods update themselves on the next server start, so there is nothing extra to do for those.

**TIP** - Back up `ShooterGame\Saved` before any update. It holds your world, player profiles and tribe data, and it is the only part that cannot be re-downloaded.

## Troubleshooting
**The server exits immediately with no message.** Missing prerequisites. Install the VC++ 2013 x64 redistributable and the June 2010 DirectX runtimes.

**The server runs but never appears in the browser.** On Windows Server, this is almost always the certificate revocation list. Follow the steps above. On desktop Windows, check the query port is forwarded.

**It always uses port 7777 no matter what I set.** You used `?Port=` instead of `-port=`. ASA needs the hyphen form.

**The admin password does not work and nothing after it applies.** `?ServerAdminPassword=` is not last in your `?` chain, so everything after it got swallowed into the password.

**Mods do not load.** Check you used `-mods=` rather than `ActiveMods`, that you used Project IDs and not file IDs, and that there are no spaces after the commas.

**The server is killed by the OOM killer on Linux.** Look at the memory table again. ASA needs a lot, and The Center needs the most.

**Steam downloads keep stalling.** Anonymous logins get deprioritised when Steam is busy. Log in with a real account instead.

**Everything is slower on Linux.** Expected. You are running a Windows binary through a translation layer. If performance matters, Windows is the supported platform.

**Can I run the ASE server on Linux natively?** Yes, and that is a genuine difference between the two games. ASE supports Linux natively and needs glibc 2.17 or newer. ASA does not.

## Conclusion
An ASA server is mostly a memory and syntax problem. Pull app **2430930** with SteamCMD, install the VC++ and DirectX runtimes, and build a command line that puts `?ServerAdminPassword=` last in the `?` chain and uses `-port=` with a hyphen.

The two things to check before committing: you have the RAM, since an empty Island map already wants 8 - 10 GiB, and you are on Windows or prepared to deal with Proton, because there is no native Linux server.

## See Also
* [ARK Wiki: Dedicated server setup](https://ark.wiki.gg/wiki/Dedicated_server_setup) - The most thorough reference there is, and the source for much of this guide.
* [ARK Wiki: Server configuration](https://ark.wiki.gg/wiki/Server_configuration) - Every `.ini` setting and launch option.
* [ARK: Survival Ascended on CurseForge](https://www.curseforge.com/ark-survival-ascended)
* [Official ARK Discord](https://discord.com/invite/playascended)
* [SteamCMD documentation](https://developer.valvesoftware.com/wiki/SteamCMD)
* [GE-Proton](https://github.com/GloriousEggroll/proton-ge-custom) - The Proton build most Linux ASA hosts use.
* [TMC App](https://github.com/modcommunity/tmc-app) - Our own server browser with live latency graphs, plus RCON with history, multi-server commands and scheduling. Open source under GPL-3.0 and in very early development, so feedback is welcome.

We keep this guide as current as we can, but ARK changes regularly and the Linux situation in particular moves with the community tooling. If an instruction here no longer matches what you are seeing, please report it or open a [pull request](https://github.com/modcommunity/how-to-setup-a-server-in-ark-sa/pulls) on this guide's GitHub repository.

Join our [Discord server](https://discord.moddingcommunity.com) if you have any questions or want help with anything server or modding related!
