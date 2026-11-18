---
name: ros-lobby-method-encoding
description: RoS lobby entry solved server-side (base-only Athlete + 0x80 method ids); only emulator load-time/inactivity blocks it (2026-11-17)
metadata:
  node_type: memory
  type: project
  originSessionId: db1c3af5-d840-4375-80eb-c999d999ee05
  modified: 2026-11-18T02:01:55.927Z
---

Server->client entity METHOD calls use msg id 0x80|idx (var16 length); 0xC0|idx is the PROPERTY-update range. Idx 61 = extended (efficient = 62-(n+192)//255, then sub-byte). Env: METHOD_BASE=0x80 METHOD_EXT=byte in ros_server.py.

Working lobby route (iteration 15): SWITCH_TO=athlete ATHLETE_CELL=off TRANSITION_RESET=off LEAVE_OLD_AVATAR=off (base-only Athlete, no Avatar/tutorial). enterHall runs, scene\hall\hall.scn (level 20) loads. The prerequisites gate (Application.scene + Globals.player) only applies to cell/world entities, not the base-only lobby player.

Remaining emulator-only issue: hall.scn load blocks the client 68-83s; native InactivityTimeout is 10s until GameInit.updateInactivityTimeout (HALL 30s) -> REASON_INACTIVITY drop + relogin loop. Server keepalives don't help. Try the physical arm64 phone (fast load) or get user approval for a client-side setInactivityTimeout bump on the emulator.

Update 2026-11-18: lobby UI reached on LDPlayer after ~6 load/drop cycles (hall.scn 70s -> 40s, then scene\hall2\hall.scn loads in <1s). Character: saved_accounts.json char_type 30001=bot_base ROBOT, 10001=no model (empty), 20001=human male (works). START button tap produced no server message yet (match start not implemented).

**Why:** Hours were wasted on the scene/Globals.player gate and tutorial-map routes; a base-only Athlete avoids all of it.
**How to apply:** Do NOT attach frida/trace_entity_msgs.js for base-only runs: it crashes the client (SIGSEGV ~3s after createBasePlayer). Use frida/trace_method_decode.js or nothing. Related: [[antigravity-agent-conflict]]

Lobby UI bug (2026-11-18): first lobby button press (e.g. char tap -> Supply) then back throws AttributeError in UIMain.setModelVisible because UIMain.models[gid]['loader'] is None (self hero has empty dts*AppearanceList props). After that every enter_ui throws (UISupplyController.on_leave) so ALL lobby buttons are dead until relogin. Appearance data tables on client are not loaded (dts_appearance_utils.getBodyAppearanceData() raises in preprocessBodyAppearanceData), so valid appearance ids can't be listed from the client; pushing onSetDtsAppearanceGender/onUpdateDtsShow*List (ros_server._push_appearance, trigger via harness/push_now.flag) did not create the model/loader. Hero model only appears after leaving a sub-UI (displayAll).

Breakthrough 2026-11-18: UIMain.on_enter was aborting on AttributeError 'PlayerAthlete' has no attribute 'iLuckyCarnivalIsSuper' (set only by client method Athlete.onUpdateLuckyCarnivalData, a PYTHON pickle dict, op 10000). Fixed by ros_server._lobby_data_calls() sent before enterHall -> full lobby UI + hero + top menu appear. Remaining: UIMain.on_enter next aborts at UIMain.py:1651 -> showRedPoint (line 2708) 'NoneType' not iterable; left-menu buttons (Store/Supply/Depot/Top-up) and Lucky Club do nothing (probably unbound because on_enter aborted), but top-bar profile + Outfit page open. Outfit page empty: dts_appearance_utils data tables fail (KeyError 8 / legacyProperties). Pattern: look for first SCRIPT traceback after hall2 load, supply the missing Athlete attribute via the client method that sets it.
