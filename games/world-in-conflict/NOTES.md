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
