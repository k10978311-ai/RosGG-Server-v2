# Hosting the server for other players

Goal: run `start.cmd` on your PC and let other people's phones/emulators connect with the patched APK.

## 1. What players need
- The patched APK (`client/apk/ros_phone_trustall_debugSigned.apk`, attach it to a GitHub Release; it trusts the server certificate itself, so no CA install).
- Original OBB data files for 1.610637.617289.
- **DNS**: the patched APK uses `.lan` host names. Players must resolve them to YOUR address. Easiest: they set the phone's Wi-Fi DNS (static) to your public IP, where `dns_server.py` answers. (Changing DNS is a phone setting the player does.)

## 2. Server config (`config.cmd`)
- `ROS_PC_IP` = the address players use to reach you. LAN-only test: LAN IP. Internet: your public IP (or a DNS name resolving to it). It must be the same in `server/real/server_list_ad.local.txt`; run `client\scripts\sync_server_list.ps1` after changing it.
- Keep `HALL_DELAY=2`, `METHOD_BASE=0x80`, `FRAG_BODY=700`.

## 3. Ports to open (Windows Firewall inbound + router port-forward to this PC)
| Port | Proto | Use |
|------|-------|-----|
| 25000 | UDP + TCP | game (Mercury) |
| 53 | UDP + TCP | DNS for the `.lan` names |
| 80 / 443 | TCP | patch/SDK HTTP + HTTPS |
| 8080 | TCP | SDK stub |

Opening firewall/router ports is your action (not done by the scripts). Cloudflare Tunnel does not carry UDP, so it cannot replace the port forward for 25000.

## 4. Check
1. `start.cmd`, then `client\scripts\verify_play_path.ps1`.
2. From another network: DNS lookup of the `.lan` name returns your IP; UDP 25000 reachable.
3. Watch `logs\server.log` for `BASEAPP LOGIN REPLY`.

## 5. Security notes
- `.gitignore` keeps CA/server private keys, `loginapp_priv.pem`, logs and `saved_accounts.json` out of Git. Do not publish them: the test CA is trusted by patched clients.
- Each repo clone must generate its own certs/keys (see server/certs) before hosting.
- Only open the ports above; the server has no authentication beyond the game login, so treat it as a public test server.
