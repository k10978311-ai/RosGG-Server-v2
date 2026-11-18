# RoS Open Play Server (v2, clean layout)

Preservation/research server for Rules of Survival 1.610637.617289 (BigWorld Mercury, UDP/TCP :25000).
This folder is the self-contained server: `server/ros_server.py` is the main game server (login, lobby, HTTP/HTTPS/DNS targets, UDP/TCP gates), not a test mock. The working project outside this folder is untouched.

```
ros_server_v2/
  start.cmd / stop.cmd     launcher (DNS + game server), reads config.cmd
  config.cmd               PC IP, test account, lobby route settings
  requirements.txt
  server/                  everything the server needs (paths are relative to these files)
    ros_server.py          main server: HTTP/HTTPS targets, UDP/TCP gates, Athlete lobby entity
    bwdefs.py              BigWorld .def parser (plain import, no sys.path hacks)
    loginapp.py            Mercury login reply
    dns_server.py
    certs/  real/  server_keys/  saved_accounts.json
    extract/               data only: entity defs, hall_props.json, supplement_info.json
  client/
    apk/                   put the XAPK here (see README)
    scripts/               adb install/setup/DNS scripts (point at ../../server/...)
    frida/                 read-only inspection payloads
  tools/                   probes, test cycles (bash, need adb + frida)
  docs/                    findings (LOBBY_FINDINGS.md), original README, decompile notes
  logs/                    server/dns logs
```

## Run
1. Edit `config.cmd` (`ROS_PC_IP` must be this PC's LAN IP; same value in the device's DNS).
2. `pip install -r requirements.txt` (or reuse the parent `.venv`).
3. `start.cmd`, then `client\scripts\sync_server_list.ps1` and `client\scripts\verify_play_path.ps1`.
4. Install the game: `client\scripts\install_ros.ps1`. Launch it and press PLAY.

## Status (2026-11-18)
Login -> lobby works (base-only Athlete). Lobby UI loads; remaining work and the debugging
pattern are in `docs/LOBBY_FINDINGS.md`. Character type is set in `server/saved_accounts.json`
(`20001` = human male; `30001` = robot placeholder).

## Next: real launcher
A single GUI/CLI launcher can wrap `start.cmd` + `install_ros.ps1`: detect adb device,
write the PC IP into config + server list, start the server, install/launch the APK.
