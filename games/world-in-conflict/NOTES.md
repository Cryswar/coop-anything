# World in Conflict (+ Soviet Assault): co-op campaign notes

Session 1, 2026-10-17. Goal: play the WiC / Soviet Assault campaigns in co-op, ideally with each
player commanding their own forces; shared control of player 1 is the fallback.

## Identity (rule 5)

| | |
|---|---|
| Store | Steam: app 21760 (WiC, buildid 11300) + app 21910 (Soviet Assault, buildid 15036), same folder |
| Version | 1.0.1.1 (b35), PE version info of all three exes |
| Engine | Masstech (Massive Entertainment), Python 2.5 embedded statically (pyc magic `6d f2 0d 0a`) |
| OS / arch | Windows, x86 (PE32) |
| wic.exe SHA-256 | `7e15107e5cec5d66ca2f0a90da94a19a8a893308f5377adee97db1467ff0f5e9` |
| wic_online.exe SHA-256 | `433b406ecc3b92def045c3a8af6cd873b86592ad0937fdbbf33bb5b8de74823b` |
| wic_ds.exe SHA-256 | `c150494dcd3b59c4afe3cf7580619b4864cd0238a892f350f1efcedfe6ac10cf` |
| DRM | wic.exe and wic_online.exe: SteamStub (`.bind` section, entry point in it, `.text` entropy 8.00). wic_ds.exe: no wrapper (`.text` entropy 6.61) |

## Prior work (phase 1)

- coop-anything directory and `games/`: no entry (checked 2026-10-17).
- Web searches ("World in Conflict campaign co-op mod", "... coop campaign mod moddb", "... play campaign with friend"): no co-op campaign mod found. Not found by search, not proven absent. Not yet searched: Massgate.org forums, WiC Discords, Russian/German/Polish phrasings, archive.org.
- BriscoeMod lists skirmish with bots and non-dedicated servers (moddb page 403 to fetch, from search snippet only).
- Ubisoft released the Massgate backend as GPL-2.0 (https://github.com/ubisoft/massgate, last push 2018-10-30). It includes `src/MNetwork` (MN_*, incl. `MN_LoopbackConnection.h`) and `src/MCommon2` (MC_*), the same libraries linked into the game. Useful for naming structures in the binary.
- Mod Kit / WicEd existed (Sierra, 2007): https://worthplaying.com/article/2007/12/18/news/47533-world-in-conflict-v1005-level-editormod-sdk-available-now/ . Fandom map wiki says Server.py drives SP map logic, Client.py does objective cameras (https://worldinconflict.fandom.com/wiki/Map_and_Mod_Wiki/Before_we_start, search snippet only, fetch blocked).

## Observations (things seen in files / decompiler)

1. `.sdf` archives start with `RYS\n`, hold raw zlib streams; scanning for `78 da` streams recovers files. Index format not decoded yet (tail of file; names not recovered).
2. 480 pyc files extracted from wic1/2/3/30/40/45/60/65.sdf (Python 2.5). Mission folders seen: `maps/europe1`, `europe5`, `ustown1`, `usdesert1`, plus MP maps. Each SP map has `server.py`, `server_autogen.py`, `client.py`, `mapvars.py`, `allunits.py`.
3. `serverimports.py` forwards every engine event (`OnUnitDestroyed`, `OnZoneTaken`, `OnPlayerConnected`, `OnPlayerLeave`, `OnPlayerAction`, ...) into a reaction system, and every handler first checks `wicg.IsSinglePlayer()`.
4. Mission scripts reference a single `PLAYER_HUMAN` id (107 script files use it; e.g. `thePlayers[PLAYER_HUMAN]`, `GetCameraPosition(PLAYER_HUMAN)`, `APLimitTest`, `NrOfPlayerUnitsTest`).
5. 19 SP `server.py` files contain both `'OnGameStarted:: Singel'` and `'OnGameStarted:: Multi'` branches, and call `AddDeploymentZoneToTeam`. The `wicg` docstring for `EnableAutoMoods` says "Default is on in multiplayer, off in single player".
6. Script API strings in wic.exe: `SendPlayerEvent` / `SendPlayerAction` "Sends the given player ... to the server"; `IsSinglePlayer` "Check if this game is a singleplayer game"; a separate `wicp` module is "available to client-side scripts".
7. Class/file names in wic.exe: `EX_SinglePlayerMode`, `EX_MultiPlayerHostMode`, `EX_MultiPlayerJoinMode`, `EXR_Server`, `EXR_Client`, `WICR_ServerFactory`, `WICR_ClientFactory`, `EX_AIPlayer(Container)`, `EX_LANCreateBots_*Handler`, `WICP_SaveGameHandler`, `EXCO_Persistent*`. Both exes contain `MN_LoopbackConnection` and `MN_FakeUdpStream` RTTI.
8. wic_ds.exe contains SP mission support: `mySinglePlayerData`, `myGameModeSinglePlayer`, `myIsSinglePlayerGameFlag`, `SINGLEPLAYER_ROLE`, `SINGLEPLAYER_ROLE_USSR`, `myCommanderAIData->myIsSinglePlayerGame==true`.
9. wic_ds.exe `FUN_004aefd0` (Ghidra, via REA) is the server's role-selection handler. It rejects "joining multiplayer match with SP role", and in single-player matches rejects a player whose role is not the SP role ("joining singleplayer match with non-SP role!") unless a global flag at `+4` of the object from `FUN_0045c3b0` is set. Role caps per team come from a server field (`NumAllowedPlayersPerRole` in wic_ds.ini, default 4).
10. LAN host supports AI bots (`MultiplayerLANCreateBots_{None,Even,PvsB,Advanced}`, `[BotMode_*]` in wic_ds).

## Derived (reasoning from observations, not yet tested)

- Single player most likely runs the normal server and client code in one process over a loopback connection (from 3, 6, 7, 8, 9). If so, the campaign already "is" a networked game with one client, and co-op mainly means letting a second remote client join that server.
- Mission logic is server-side Python; client cameras/cutscenes are client-side Python triggered by `ClientCommand`. Scripts can be replaced by repacking `.sdf` or mod folders, so most mission fixes (multi-player `PLAYER_HUMAN`, objectives, AP) are Python work, not native.
- The `Multi` branches in campaign scripts suggest Massive ran campaign maps in MP during development.

## Unknowns

- Whether the client's SP mode really uses `EXR_Server` + loopback (client exe is DRM-encrypted; verify at runtime with an injected DLL or logs, not by unpacking).
- Whether wic_ds.exe (or a listen host) can load an SP map at all, and what game mode / role assignment it uses.
- Whether `ClientCommand` targets one client or all (cutscenes, message boxes, camera moves).
- Save games mid-mission in co-op (`WICP_SaveGameHandler` is client-only; probably unsupported at first).
- Whether the sim is deterministic or server-authoritative state sync (WiC MP is server-authoritative per its DS model; not confirmed in code).
- Campaign progression between missions (`EXCO_Campaign`, unlocks, cinematics) for the guest.

## Session 2 (2026-10-17): first experiments

Workspace: `<workspace>/coop-anything` (branch `game/world-in-conflict`, private files in `local/`),
test install copied to `<workspace>/testinstall` (exe hashes verified equal to the originals).
Only the copy's `wic_ds.ini` (5 values: GameName, MapCycleFile, RandomMapCycle, MinPlayers=1,
MaxPlayers=4; original kept as `wic_ds.ini.orig`) and a new `coop_cycle.txt` were changed.
Runner: `local/tools/run_ds.ps1` (starts the copied wic_ds.exe minimized, kills it after N seconds,
moves its logs into `local/<tag>/`).

Side effect to know: wic_ds.exe writes its logs to `%USERPROFILE%\Documents\World in Conflict\debug\`
and created that folder (it did not exist before). The runner moves every new log out of it.

### Observed

- E1, cycle `europe2`: the server started, listened on TCP 48000 + 52999 (VOIP) and UDP 48000,
  printed 31 loadable maps (`LoadMissions()`), ignored `europe2` and loaded the first map,
  `maps/usdesert2/usdesert2.ice`, then ran InitMission, created AI players and set team deployment zones.
- E2, cycle `do_Riviera`: loaded `maps/europe4/europe4.ice`.
- E3, cycle `maps/europe2/europe2.ice` + `Europe2`: ignored, loaded `usdesert2` again.
- Campaign list (from `wicloc61.sdf` text table `myCampaigns.<cmp>.myItems.MissionN-<Map>`):
  WiC `cmp1`: Seattle2, Seattle3, Ustown4, Usfarmland3, Ustown3, Europe4, Europe2, Russia2, Russia4,
  Newyork1, Usfarmland2, Ustown1, Seattle4, Seattle1.
  Soviet Assault `WIC_SA`: Berlin1, Seattle2, Seattle3, UsFarmland1, Ustown4, Usfarmland3, UsFarmland5,
  Ustown3, Europe4, Europe2, Norway1, Russia2, Russia4, Russia3, Newyork1, Usfarmland2, Ustown1,
  UsFarmland4, Seattle4, Seattle1.
- wic_ds `FUN_004a7f30` asserts `myIsSinglePlayerGameFlag` (EXG_Game.cpp line 0xda2) on the byte at
  `EXG_Game+4`; the role handler `FUN_004aefd0` reads the same byte. The server game class is shared
  between SP and MP; the mode is one flag on it.

- E4, one 12 s run per stock cycle name (`local/mp_map_folders.tsv`), MP name → map folder:

  | MP name | folder | campaign mission(s) |
  |---|---|---|
  | do_Hometown | ustown1 | WiC 12, SA 17 |
  | do_Xmas | ustown3 | WiC 5, SA 8 |
  | do_Seaside | ustown4 | WiC 3, SA 5 |
  | do_SpaceNeedle | seattle1 | WiC 14, SA 20 |
  | as_Dome | seattle2 | WiC 1, SA 2 |
  | as_Bridge | seattle3 | WiC 2, SA 3 |
  | do_Island | seattle4 | WiC 13, SA 19 |
  | do_Liberty | newyork1 | WiC 10, SA 15 |
  | do_Farmland | usfarmland1 | SA 4 |
  | tw_Wasteland | usfarmland2 | WiC 11, SA 16 |
  | do_Riverbed | usfarmland3 | WiC 4, SA 6 |
  | tw_Highway | usfarmland4 | SA 18 |
  | do_Silo | usdesert1 | - |
  | as_AirBase | usdesert2 | - |
  | do_Ruins | europe1 | - |
  | as_Hillside | europe2 | WiC 7, SA 10 |
  | do_Riviera | europe4 | WiC 6, SA 9 |
  | tw_Radar | russia1 | - |
  | do_Powerplant | russia2 | WiC 8, SA 12 |
  | do_Quarry | russia3 | SA 14 |
  | as_Typhoon | russia4 | WiC 9, SA 13 |
  | do_Apocalypse / do_Tequila / do_Studio | own folders | - |

  Campaign maps with no MP twin: usfarmland5 (SA 7), berlin1 (SA 1), norway1 (SA 11). Their
  folders still appear in `LoadMissions()`.

### Derived

- Campaign missions and MP maps share map folders: `do_Riviera` is `europe4`, which is also
  campaign mission 6. One `.ice` holds both the MP modes and the `mySinglePlayerData`. This is why
  mission scripts carry `OnGameStarted:: Singel` and `:: Multi` branches.
- The DS map cycle takes MP display names, not folder names. It cannot select the SP mode of a map
  from config; that needs the SP flag set on `EXG_Game` (native hook) or the client's host mode.

- E5, owner in WiC campaign mission 1 (Steam build, launched from Steam), sockets of `wic.exe`
  read from outside with `Get-NetTCPConnection` / `Get-NetUDPEndpoint`: no TCP sockets at all,
  one UDP socket on `0.0.0.0:<ephemeral>`. The dedicated server, for comparison, listens on
  TCP and UDP 48000.

### Derived

- Single player does not run its server behind a real socket. Together with the
  `MN_LoopbackConnection` RTTI, the likely shape is server and client in one process over an
  in-memory loopback connection. Co-op therefore needs native work: make the SP host also accept
  a real network client (as `EX_MultiPlayerHostMode` does), or start the SP mission inside the
  MP host mode with the SP flag set. Config alone will not do it.
- What the single UDP socket is for is unknown (LAN finder or VOIP are guesses).

### Still open

- SDF index format (file names for every stream) not decoded; streams are found by scanning.
- GOG copy vs Steam copy: unknown whether they can play together. Check exe version and hash on
  the GOG side, then try a stock LAN game over a VPN.

## Networking plan (owner preference)

The owner used Steam "Spacewar" (AppID 480) networking in a previous project
(see `playbook/networking-steam-spacewar.md`). For WiC the relay pattern (separate process, like
`HXCoop.exe`) fits better than loading `steam_api.dll` into the game: `wic.exe` is itself a
SteamStub-wrapped Steam app (AppID 21760), so a second Steam app id inside that process is
risky, while a relay process can be Spacewar on its own and forward the game's UDP/TCP to
localhost. The relay also works unchanged if one player has the GOG build.

## Sources

- https://github.com/cranebeaker/coop-anything (playbook)
- https://github.com/ubisoft/massgate
- https://en.wikipedia.org/wiki/World_in_Conflict
- https://worldinconflict.fandom.com/wiki/Map_and_Mod_Wiki/Before_we_start

## Session 3 (2026-10-17): GOG Complete Edition vs Steam

GOG Galaxy install of "World in Conflict: Complete Edition" (GOG game id 1438332414, Galaxy reports
version 2.0.0.3).

| File | SHA-256 |
|---|---|
| GOG wic.exe | `41017f352cabd5fb3dc2b4457727c6e32678fdbdd89e881b11a8868edba18ebc` |
| GOG wic_online.exe | `7bb41478f3070641b7639707d72a976a2ae5c8f9ad9356ff5733610979c0b7d0` |
| GOG wic_ds.exe | `c150494dcd3b59c4afe3cf7580619b4864cd0238a892f350f1efcedfe6ac10cf` (same as Steam) |

### Observed

- GOG `wic.exe` / `wic_online.exe` have no `.bind` / `.extra` sections and an unencrypted `.text`
  (entropy 6.6). Section addresses and sizes are identical to the Steam exes.
- Section-by-section compare of GOG vs Steam `wic.exe`: same PE timestamp (`0x4a2f7509`);
  `.rdata`, `.data`, `.tls`, `CONST`, `.rsrc` byte-identical; only `.text` differs (Steam's is
  encrypted) plus Steam's extra `.extra` / `.bind` sections and entry point.
- Every shared DLL and every `.sdf` archive hashes identical, except `wicloc46.sdf` and
  `wicloc61.sdf` (localization / GUI text). `wic_ds.ini` differs only in defaults.
- RTTI in GOG `wic.exe` (`local/tools/rtti.py`):
  - `EX_MultiPlayerHostMode` derives from `EX_SinglePlayerMode` (bases: EX_SinglePlayerMode,
    EX_IPlayMode, EXCO_Persistent<EX_SinglePlayerMode>). `EX_DemoReplayMode` too.
  - `EX_MultiPlayerJoinMode` is separate: EX_IPlayMode, `EXR_Server::IClient`.
  - `EXG_Game` bases: WICO_IGameInfo, EXCO_IGame (subobject at +8), EXCO_Persistent<EXG_Game>.
- `0xb405d0` (log label `EX_SinglePlayerMode::InitGame`) takes one bool; when it is 1 it applies
  single-player setup (`singleplayer_craters`). Its only direct caller is
  `EX_MultiPlayerHostMode::InitGame` (`0xb3f2e0`), at `0xb3f386`: `6a 00 / 8b cb / e8 ..`
  (`push 0; mov ecx,ebx; call 0xb405d0`).
- `0xb405d0` creates the game through `0x756990`, which allocates `0x1320b8` bytes and runs the
  `EXG_Game` constructor `0x754f40`. The constructor stores a register argument (Ghidra: `in_AL`)
  at `+0xC`, the byte the server's role check reads through the `EXCO_IGame` subobject
  (`this+8`, field `+4`). The other `0x756990` caller is `WICR_GameController::InitGame`
  (`0x768a10`), the controller used by `WICR_GameController::MapCycle` (`0x768e50`).

### Derived

- The Steam exe is the GOG exe plus SteamStub. Addresses found in the GOG exe should match the
  Steam exe in memory once its stub has run (inferred: identical data sections and section layout;
  not yet checked in memory). A runtime hook keyed on byte signatures can serve both builds.
- Same code and same data means the two builds should talk to each other over LAN (inferred; not
  tested).
- Multiplayer hosting is single-player mode plus a network server. The cheapest co-op prototype:
  make the MP host call `InitGame(1)` (the byte at `0xb3f387`, `00` → `01`) on a map with a
  campaign twin, then join with a second client. Expected next obstacles: the role check
  (`"joining singleplayer match with non-SP role"`), the lobby's map and role selection, and
  scripts written for one `PLAYER_HUMAN`.

### Recommendation

Develop against the GOG build: static analysis works directly on the shipped exe, there is no
DRM in the way of hooking, and it is the build new players can buy. Keep the Steam build as a
second target through signature-checked runtime hooks (no unpacking).

## Session 4 (2026-10-17): host-mode prototype set up

- Client switches found by listing every string passed to the switch checker `0x9fa4c0`, among
  them `window`, `debug`, `pythondebug`, `nointro`, `multi`, `forceteam1`, `forceteam2`, `noai`,
  `dev`, `startingap`. `-multi` skips the single-instance mutex
  `WorldInConflict_Duplicate_App_Detection` in WinMain (`0xb2ef40`), so two copies can run on one PC.
- The retail client writes no debug log by default; `-debug` is expected to enable
  `Documents\World in Conflict\Debug\wic_<id>_debug.txt` (not yet confirmed).
- Prototype in a GOG test copy: `wic.exe` file offset `0x73f387` changed `00` → `01`
  (`push 0` → `push 1` before `call 0xb405d0` in `EX_MultiPlayerHostMode::InitGame`). Result: pending
  owner test.

### Test 1 (owner, v1 one-byte patch, GOG test copy, LAN host on as_Dome)

Observed (client log with `-debug`):
- Lobby shows the campaign description for as_Dome (Soviets invading Seattle). Unknown whether
  the unpatched lobby shows the same.
- On Create: `EX_PlayState::Init(): MULTI_PLAYER_HOST mode`, server listening on 48000, mission
  `maps/seattle2/seattle2.ice` loaded with `units/unittypes_wic_singleplayer.ice`,
  `ex_singleplayermode.cpp InitGame(): creating game...`, then
  `ex_multiplayerhostmode.cpp(788): InitGame(): Error Could not init Python` and back to the menu.

Code (GOG exe):
- Python init `0xb41200(this, spScriptName, flag)` (`ret 0xc`, path in EAX). The MP host pushes
  `0, 0`, computes the path with `0x985dd0` (reads `myPythonServerScript` from mission-info
  `+0x29dc`), then calls it.
- `EX_SinglePlayerMode::PlayerLoadingThread` (`0xb40c90`) calls the virtual `InitGame(1)`, then
  pushes `0`, gets the single-player script name with `0x984310` (reads
  `mySinglePlayerData.mySinglePlayer...` from mission-info `+0x29c0`), pushes it as the second
  argument, and calls `0x985dd0` and `0xb41200`.

v2 patch (GOG test copy, on top of v1): `0xb3f393`–`0xb3f3d7` rewritten to do what single player
does (`push 0; push ecx; mov eax,esi; call 0x984310; add esp,4; push eax; call 0x985dd0;
push ebx; call 0xb41200; test al,al; jnz 0xb3f3e3; jmp 0xb3f3d8`). This overwrites the
"Could not init Python" log call. The exe has no relocations. Patched sha256 `18765094...`.

### Test 2 (owner, v2)

Observed: same symptom (back to menu after ~1 s). Log: no "Could not init Python" any more; Python
init passed. Right after it `EX_PlayState::Update() returning EVENT_GAME_EXIT`, and during teardown
`wico_pythonc.cpp CheckPyError(): exceptions.ImportError: No module named client`.

Code: the MP host's loading thread `0xb3f840` runs `InitGame` (`0xb3f2e0`), then `0xb3f640`,
which creates the local player with `EX_IPlayMode::InitPlayer` (`0xb41e20`). InitPlayer takes the
role in EAX and `(slot, a2, a3, isMultiplayer)` on the stack. It rejects role/match mismatches
with messages logged at a level `-debug` does not write ("Player joining singleplayer match with
non-SP role!", "...multiplayer match with SP role!", FPM variants). Single player calls it with
`EAX=[0xe9ff6c]` (SP role) and `(0,0,1,0)`; the host calls it with `EAX = 0 or [0xe9ff74]` and
`(0,1,0,1)`.

v3 patch (on top of v2): `0xb3f7cc`–`0xb3f7e3` → `mov eax,[0xe9ff6c]; push 0; push 1; push 0;
push 0; nop×11`, so the host's own player is created like a single-player player. sha256
`63e7976f...`. Host script now also passes `-debugspam` (expected to show the low-level messages;
not yet confirmed). Expected next obstacle: remote players will still join with an MP role.

### Test 3 (owner, v3) and switch to runtime tracing

Observed: same symptom. `-debugspam` added nothing. The logger (`0xa02800`) writes level 0 and 2
always, level 1 only when `[0xdffe06]` is set, so the missing "creating player on slot %d"
(level 0, `EX_IPlayMode::InitPlayer`) means the host never reached player creation. Derived: the
failure is still inside or right after the Python init; v2 had overwritten that step's error log.

Tool added (`wic-coop/trace/`, own code): `tracehook.dll` redirects 12 `call` sites on the host
loading path to logging thunks (args, return value), written to `coop_trace.log`;
`coop_launch.exe` starts wic.exe suspended and loads the DLL with CreateRemoteThread +
LoadLibraryA. Built with MSVC x86 (VS 2022). Smoke test in the GOG test copy: all 12 sites hooked.

### Trace 1 (owner, v3 + tracer)

Observed (`coop_trace.log`): host InitGame → `SP::InitGame(1)` returns 1 → InitPython (cave) →
"python setup paths" (`0x972500`) returns 1 → "run server script" (`0x925a80`) returns 0 → InitPython
0 → InitGame 0. `0x925a80` creates `WICP_Python` and its init (`0x925760`, `wicp_python.cpp`) does
`import client`, matching the logged `ImportError: No module named client`. So the client-side
Python cannot find the map's `client.py` on the host.

Code read: `0x984310` returns `mySinglePlayerData.mySinglePlayerScriptFiles`; `0x985dd0` builds
`<map dir>/<myWicedMap.myPythonServerScript>` (mission-info `+0x29dc` is `myWicedMap`, not the game
mode). `0x972500` sets the Python search path from those (via `0x4aab10`). `EXCO_MissionInfo::Init`
(`0x987540`) loads the single-player data (`0x987290`) in every mode.

Next: tracer v2 (21 sites, prints string arguments and results) to compare the host attempt with
a real single-player mission start in one session.

### Trace 2 (owner): host attempt, then Resume Campaign in the same session

Observed: Resume Campaign (berlin1) also failed in the test copy with the same
`import client` failure, through the unpatched single-player path
(`ex_singleplayermode.cpp(163): PlayerLoadingThread(): Error Could not init Python`). Strings from
the trace: single-player script files = `cutscenes/maps/<map>/python/serverplay.py`, server script
= `maps/<map>/python/server.py`, Python path = `cutscenes/maps/<map>/python;maps/<map>/python`;
the host builds identical strings for seattle2.

Derived: the failure is not caused by the host patches. Every test script had passed
`-pythondebug`. In `0x972500`, without `pythondebug` the game increments `[0xe0f418]` before
starting Python. Guess (unconfirmed): that is `Py_OptimizeFlag`, so normal runs import the
optimized `.pyo` files and `-pythondebug` makes Python look for `.py`/`.pyc`, which the archives
do not hold. `-pythondebug` removed from all test scripts. Lesson: assert a single-player
baseline in the exact test setup before testing a patch (playbook lesson 3).

### Test 6 (owner): v3 host without `-pythondebug` — campaign runs from the LAN lobby

Observed (owner + trace): LAN Create Game on as_Dome with the v3 exe loads campaign mission
seattle2. For a few seconds only the starting APCs show, then the opening cutscene plays in full
(sound and animation) and the normal campaign mission starts; the owner reports it plays normally.
Trace: MPHost::InitGame = 1, get player slot = 1, voip connect = 1,
`EX_IPlayMode::InitPlayer(SP role, 0,0,1,0)` = 1, loading step 3 = 1.

Derived: the minimum to run a campaign mission as a network host is three edits in the GOG exe:
`InitGame(1)` (0xb3f387), single-player script path for Python init (0xb3f393..), single-player role
for the local player (0xb3f7cc..). The server keeps listening on 48000 while the mission runs
(from the `enabled reliable listening on port 48000` line in earlier logs; not re-checked during play).

`coop_join.cmd` only opens a second copy at the main menu; joining has not been tried yet.

### Test 7 (owner): second copy joins the v3 host over LAN

Observed: player 2 sees the game in the LAN list, joins, gets the normal MP faction and role
screens, picks a role and is dropped to the menu with "connection to server lost". Host log:
`Client ('Player') accepted on slot '1'`, `SendPlayerStartingData(): Player connected`,
`PlayerJoinedTeam(): Player 1 joined team 1`, then
`exg_game.cpp(4974): PlayerSetRole(): Player 1 joining singleplayer match with non-SP role!`,
`exr_client.cpp(661): ParsePlayerSetRole() failed`, `RemovePlayer(): removed player 1`.

Code (GOG exe, `EXG_Game::PlayerSetRole` `0x752880`): at `0x7529c7` `cmp byte [game+4],0` picks the
SP branch; `0x7529d8 jnz 0x752b92` accepts if a global "allow any role" field is set,
`0x7529e4 je 0x752b92` accepts the SP role, otherwise it logs and fails.

v4 patch (on top of v3): `0x7529d8` `0f 85 b4 01 00 00` → `e9 b5 01 00 00 90` (always take the
accept path in SP matches). sha256 `91e1f854...`. The joiner keeps the MP role it picks.

### Test 8 (owner): v4, player 2 joins the running campaign mission

Observed (owner): player 2 joins, still goes through the MP faction and role screens, gets no
units, but sees the whole map, including every unit on it with no fog of war, and watches player 1's
(Parker's) units move in real time when player 1 gives orders. Host log:
`Player 1 joined team 1`, `PlayerSetRole(): Player 1 has got role INFANTRY (234881739)`, no
rejection.

Derived: a second real client is connected to the campaign server and receives the synchronized
simulation. What is missing is per-player setup: AP/reinforcements, drop zones, fog of war (team or
vision assignment), and anything the mission scripts do for `PLAYER_HUMAN`.

## Status after session 4 (2026-10-17)

Working in a GOG test copy, with four byte-level patches (v1–v4) to wic.exe:
1. A LAN-hosted game on a campaign map runs the real campaign mission (cutscene, scripts, AI).
2. A second client can join it and sees the live mission.

Open, in rough order:
- Player 2 control. Two options: (a) shared control: let the server accept player 2's orders for
  player 1's units, plus a client-side selection change; (b) own forces: give player 2 AP and a drop
  zone, set their team and vision, and teach the scripts about a second human.
- The MP faction and role screens for the joiner.
- The few seconds of "APCs only" before the cutscene on the host.
- Move the patches from exe edits into a runtime proxy DLL that works on GOG and Steam.
- Networking over the internet (LAN tested only; Spacewar relay planned).
