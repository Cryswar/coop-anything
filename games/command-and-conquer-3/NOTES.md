# Command & Conquer 3: Tiberium Wars / Kane's Wrath: working notes

Started 2026-10-17. Goal: play the single-player campaigns of both games in co-op (two humans).
Preferred: each player has own side and money. Fallback: both share control of the campaign's
human side.

## 0. Identity (observed)

| | Tiberium Wars | Kane's Wrath |
|---|---|---|
| Store | Steam app 24790, buildid 17447444 | Steam app 24810, buildid 17447564 |
| Launcher | `CNC3.exe` (reads `CNC3_english_1.10.SkuDef`) | `CNC3EP1.exe` (reads `CNC3EP1_english_1.3.SkuDef`) |
| Real game exe | `RetailExe\1.10\cnc3game.dat` (PE32 x86, not packed, PE timestamp 2025-02-19) | `RetailExe\1.3\cnc3ep1.dat` (PE32 x86, not packed, PE timestamp 2025-02-19) |
| SHA-256 game exe | `af568a89b6d0d4f69ec690cf1d443539cb15afe197b96ac3fc5bf0ec0a56d141` | `1716c954ab80cf05752f4a4a467d34313a95a55a408718270f96382e330572f3` |
| Sections | `.text .rdata .data rts.ver .tls .rsrc` (no SteamStub `.bind`, no SecuROM) | same |
| Engine | SAGE (Generals lineage), see section 3 | same |
| OS | Windows 11 | |

Older exes ship next to it (`RetailExe\1.0`..`1.9`); the retail 1.9 `.bak` has `.securom`
sections, the Steam 1.10 / KW 1.3 builds do not. Only the Steam 2025 builds matter here.

## 1. Existing work (searched 2026-10-17)

Searched: this repo (directory, games/, issues, PRs for "Command and Conquer", "Tiberium",
"Kane", "SAGE", "RTS"), web search (several phrasings), GitHub repo search ("cnc3 coop",
"tiberium wars coop", "kanes wrath coop", "cnc3 campaign multiplayer", "sage engine coop").
ModDB pages blocked automated fetches (Cloudflare check); only search-result text was used.
Not checked: CNCNet / C&C Online Discords, Revora forums in depth, archive.org.

Observed:
- No official PC campaign co-op for either game. Source: Revora forum answer,
  https://forums.revora.net/topic/100755-how-to-play-c-c-3-campaign-co-op-please-help/
- Xbox 360 had co-op versus AI, not the story. Source: https://www.co-optimus.com/game/181/xbox-360/command-and-conquer-3-tiberium-wars.html
- Community co-op *custom* missions exist as multiplayer maps (they are not the official
  campaign): COOP Evacuation (needs an AI bot in a slot), https://www.moddb.com/games/cc-tiberium-wars/addons/coop-evacuation-co-operative-mission ;
  Boyle's Legacy co-op pack and others, https://forums.cncnz.com/topic/16232-boyles-legacy-co-op-mission-pack-re-released/ ;
  Co-Op Tiberian Sun Map Pack (needs Tiberium Essence), https://www.moddb.com/mods/co-op-tiberian-sun-map-pack
- A search snippet mentions a ModDB member converting official TW missions to have a
  "Co-Commander"; no release page found. Status unknown.
- Kane's Wrath already has a community `dsound.dll` proxy ("Tacitus", QoL wrapper, found in
  the owner's install). Shows that a proxy DLL loads fine in the Steam build.

Not found by these searches: any engine-level campaign co-op for C&C3 or KW.

## 2. Data formats (observed)

- Campaign maps: `SP\1.0\SPMaps.big` (TW, 39 maps), `Kane\1.0\KaneMaps.big` in the TW folder
  and the KW install for KW. Each map folder has the raw `*.map` plus compiled
  `map.bin/.imp/.manifest/.relo`.
- `.big` is the usual BIGF/BIG4 archive (tools/bigtool.py in the workspace).
- `.map` = RefPack stream wrapping `EAR\0` + u32 size + RefPack stream wrapping a `CkMp`
  chunk file (tools/sagemap.py decodes it; layout ported from OpenSAGE,
  https://github.com/OpenSAGE/OpenSAGE src/OpenSage.Game/Logic/Map/SidesList.cs, Data/Map/Player.cs).
- Campaign map sides (example GDI 1-1 Pentagon): one side with `playerIsHuman=true`
  (`Plyr_GDI_a`), several named AI sides with their own allies/enemies and AI flags
  (`Plyr_Nod_a..d`, allied `Plyr_GDI_b` "Pentagon Defense Force"), plus civilian and neutral.
  Nod 3-1 Slovenia: human `PlyrNOD_A`, eight other sides. Side names differ per map.

## 3. Engine seams (observed in the TW 1.10 exe with REA/Ghidra 12.1.4)

- Reference: EA's GPL release of Generals Zero Hour, https://github.com/electronicarts/CnC_Generals_Zero_Hour
  (`GameLogic::startNewGame`, `SidesList::prepareForMP_or_Skirmish`). In Generals, any
  network or skirmish game calls `prepareForMP_or_Skirmish`, which removes every map side
  except civilian and then builds players `player0..N` from the lobby slots.
- TW 1.10 `FUN_0068a498` is the C&C3 version of that function: it walks the sides, keeps
  only `PlyrCivilian` and `PlyrCreeps`, then adds `SkirmishHuman`, `FactionCivilian`,
  `Multiplayer_Human` templates.
- Multiplayer in the 2025 Steam builds uses Steamworks: interface strings
  `SteamMatchMaking009`, `SteamNetworkingMessages002`, `SteamNetworkingSockets012` in both exes.
- LAN lobby is still present (`AptLanLobby::*`, `LANAPI`), including
  `AptLanLobby::OnLoadGameBttn` (multiplayer save/load).
- Command line switches present: `-file`, `-xres`, `-yres`, `-noaudio`, `-noshellmap`.
- Kane's Wrath 1.3 has the same strings (`PlyrCreeps`, `Multiplayer_Human`, `SkirmishHuman`,
  `SteamNetworkingMessages002`, `LanLobby.apt`) at different addresses.

## 4. Derived (reasoning, not tested)

- Both games are one engine; a single DLL design with per-exe address tables (found by
  byte signatures) should cover both. Start with TW.
- The engine is lockstep (Generals lineage). If both machines load the campaign map with
  its own sides, the map scripts and AI run identically on both, so no state sync is
  needed, only correct player setup and command routing.
- Plan: start a LAN/Steam lobby game on a campaign map, and in a proxy DLL replace the
  `prepareForMP_or_Skirmish` step so the campaign sides stay. Then either
  - shared control: bind both humans to the map's human side (commands from slot 1 run as
    the campaign player), or
  - own faction: add a second human side cloned from the campaign human side, allied with
    it. Scripts that name the campaign player (reinforcements, loss checks, objectives)
    only affect player 1, so this needs per-mission rules.
- Campaign-only systems to handle: mission briefings/movies, objectives UI, mission
  sequence (next map after victory), saves. These live outside lockstep.

## 5. Unknowns

- Whether the game loads the raw `.map` or the compiled `map.bin` for sides and scripts.
- How C&C3 binds lobby slots to players and stamps network commands with a player index.
- Whether a campaign map can be selected in a lobby at all, or the map list filters it.
- How the lockstep checksum is computed and logged (needed as the test oracle).
- Whether two copies of the game can run on one PC (instance checks), for automated tests.

## 6. Workspace and harness (2026-10-17, observed)

- Hardlinked test installs in `<workspace>\testinstall\tw|kw` (big files hardlinked, configs and
  the game exes are real copies). Verified with `fsutil hardlink list`.
- `.dat` files are associated with a hex editor on the owner's PC, so `Start-Process` opened the
  editor. Launch with CreateProcess (`UseShellExecute=false`).
- Launcher runs the game exe as `cnc3game.dat -config "<install>\CNC3_english_1.10.SkuDef"`
  (strings `-config "`, `-modConfig "` in CNC3.exe). Windowed: `-win` (string next to
  `-fullscreen`), `-xres/-yres`.
- Proxy DLL `dinput8.dll` (the exe imports only `DirectInput8Create` from it) loads fine.
  It redirects `SHGetSpecialFolderPathA/W` so each instance has its own AppData (the game then
  writes `AppData\Command and Conquer 3 Tiberium Wars\Profiles`); the owner's profile folders
  are not touched.
- Background mode: the game only renders while it thinks it is the foreground app. Hooking
  `GetForegroundWindow` (returns the game window), `IsIconic` (FALSE), `ShowWindow`
  (SW_SHOWMINNOACTIVE) and no-op `SetForegroundWindow/SetFocus` keeps it minimized and running.
- PrintWindow captures are black (D3D9). In-engine screenshots: inline hook on
  `IDirect3DDevice9::Present`/`EndScene` + `D3DXSaveSurfaceToFileA` from `d3dx9_29.dll`.
  `Present` on this PC is already inline-hooked by another module (starts with `E9`, probably the
  Steam overlay), so the hook chains through that jump; vtable patching was bypassed.
- Commander name prompt is skipped by seeding `Profiles\directory.ini` (each char encoded as
  `<c>_00`, non-alphanumerics as `_<hex>_00`; encoder checked byte for byte against a real file)
  plus `Options.ini` (800x600, low detail, volumes 0).
- Posted `WM_MOUSEMOVE/WM_LBUTTONDOWN/UP` to the minimized window click APT menu buttons
  (main menu at 1024x768: Skirmish = (311,303)).

## 7. startNewGame (TW 1.10, observed in Ghidra dump)

- `FUN_004e0fd3` = map-load half of GameLogic::startNewGame. `DAT_00bf92c4` = TheGameInfo
  (null in campaign). With a GameInfo it removes teams, calls `FUN_0068a498`
  (prepareForMP_or_Skirmish) when `TheGameEngine->isMultiplayer()` or a slot is AI, then
  `FUN_004c1674(hasAI, ...)` which builds `Player_%d` / `Observer_%d` sides from slots.
  `DAT_00bf34a0 + 0x18` = TheGlobalData map name (passed to the map loader).
- `FUN_0068ccee(i)` = GameInfo::getSlot(i); 8 slots.
- Network command classes match Generals (`NetGameCommandMsg`, `ConnectionManager`), but no
  `Player_%d` lookup in the net code: how a command picks its player is still unknown.

## 8. Shared-control prototype, single client (2026-10-17)

DLL hooks (TW 1.10 only, each checked against its expected prologue bytes):
- `0x4e0fd3` startNewGame map part: replace the lobby map with `CNC3COOP_MAP`; for this call only,
  flip the `JZ` at `0x4e11a5` to `JMP` so the lobby-game team-deletion loop keeps the mission teams.
- `0x68a498` prepareForMP_or_Skirmish: skipped. `0x4c1674` slot sides: skipped.
- `0x4bc5aa` (adds ReplayObserver side): afterwards set `multiplayerIsLocal` on the map side with
  `playerIsHuman` (not on ReplayObserver, which is also "human"; marking it made the local
  player an observer: full map reveal, "game over" UI).
- `0x50fff4` VictoryConditions::update: skipped (it checks the Player_N slot players).
- Tried and rejected: nulling TheGameInfo during the map load to force the campaign path.
  The skirmish load screen dereferences it (AV at `0x68cd12` via `0x4e1076`).

Observed results (screenshots via the in-engine capture, runs driven by tests/t_skirmish.py):
- Skirmish start + co-op hooks + `MAP_SP_GDI_0_1_Prologue`: same starting squads, camera area and
  idle state as the real campaign start (tests/t_campaign.py reference run), no defeat after
  ~2 min. The sidebar shows the skirmish top buttons instead of the campaign objective/intel ones.
- Skirmish start + co-op hooks + `map_sp_gdi_1_1_pentagon` (mission 2, no campaign context):
  intro cinematic, objective text and enemy AI run, but a script action ends the game with
  DEFEAT ~70-120 s in (trace: ScriptActions switch `0x5365b7` -> `0x516bfb` -> end-game UI
  `0x50fea6`). Not compared with a real campaign run of that mission yet. Open.
- Real campaign reference: Campaign -> GDI Campaign -> "Autosave Overwrite" prompt (appears even
  with an empty instance folder; cause unknown) -> movie (Esc skips) -> briefing waits on PLAY.
- `-file <map>` is not a reliable shortcut: given a `.map` path it appends the folder name again
  and crashed after load; given the folder it did not start a game.

Not yet tested: two clients, orders from the second client, save/load, mission transitions.

## 9. Two clients on one PC, LAN co-op start (2026-10-17)

Observed:
- The network lobby first shows "Firewall Detected" and refuses to continue on NO. For tests the
  DLL makes the game's own `CoCreateInstance` of NetFwPolicy2/NetFwMgr fail (test env flag only);
  no firewall setting is touched. A real player clicks YES once (that adds the exception).
- LAN lobby protocol is broadcast only: every message goes to 255.255.255.255 on ports 8086-8093.
- Two instances with the same IP: lobby works, guest joins, but at start the guest hangs on
  "Starting the game..." and the host wins by timeout. Matches Generals' LAN code, which identifies
  players by IP (`LANAPI::handleRequestJoin` checks `msg->GameToJoin.gameIP != m_localIP`).
- Fix (test-only network shim, `CNC3COOP_LOCALIP`/`CNC3COOP_PEERS`): each instance reports
  127.0.0.N from `gethostbyname`, binds its sockets to it, broadcasts are re-sent as unicast to the
  other instances, `GameSpyIPAddress = 127.0.0.N` and a distinct `FirewallPortOverride` are seeded
  in Options.ini, and `SIO_UDP_CONNRESET` is switched off. Without the last one, ICMP "port
  unreachable" from the unicast copies made the receiver lag ~1 minute and joins timed out.
- With the shim: vanilla two-client LAN game starts on both (test `t_lan_coop.py mode=`).
- Co-op shared mode over LAN, GDI prologue: both clients load the campaign map with campaign sides,
  both mark `GDI_a` local, logic frames advance on both (~15/s), screenshots of both clients match.
- Slot -> player mapping: C&C3 stores the slot's player name at `GameSlot+0x34` (set to
  `Player_%d` by `0x4c1674`, call at `0x4c1743`) and resolves slots to players by that name. Co-op now
  sets every occupied human slot's name to the campaign human side. Effect on orders: not verified.

Harness input findings:
- Posted mouse messages work for APT menus, unit selection and hover/tooltips in game, but do not
  press in-game sidebar buttons (Power Plant stays unbuilt) and a left click on the ground with
  infantry selected did not move them (prologue; may also be mission input lock, unknown).
- In tactical view the game reads the cursor with `GetCursorPos`, so the DLL now answers it (and
  ScreenToClient/ClientToScreen/GetAsyncKeyState) from a virtual cursor and blocks
  `SetCursorPos`/`ClipCursor` in background mode. The game creates only a DirectInput keyboard; it is
  zeroed in background mode so the owner's keys never reach a test instance.

Open:
- Prove that orders from the guest execute for the shared side. Plan: in-game bot that appends
  GameMessages directly (move, build) instead of clicking; C&C3 has a `MSG_*` name table (227 names)
  to derive message type ids.
- Pentagon mission scripted defeat; mission transitions; saves; separate factions; Kane's Wrath.

## 10. First human test and follow-up (2026-10-17)

Owner report (C&C Online, two PCs, build 0.0.1): game hosted normally, both join, chat works,
both see the same units, map area and cutscenes. Player 2 cannot control player 1's units.
Exiting a running game closes to desktop.

Observed afterwards (LAN, two instances on one PC, TW 1.10):
- Logic dispatcher = `0x4e995b` (thiscall, ret 8). GameMessage +0x0c type, +0x10 player index;
  executed for `ThePlayerList(*0xbee110)->getNthPlayer(idx)` (`0x49714c`). Player +0x44 name key;
  PlayerList +0x1c local player, +0x20 count, +0x24 Player*[]. NameKeyGenerator `*0xc28ccc`,
  nameToKey `0x9ddec0`.
- Message type ids from the name switch at `0x54f2d6` through its byte table at `0x54ffc1`
  (dump/msgtypes_tw110.txt): MSG_CREATE_SELECTED_GROUP 0x1f5, MSG_DESTROY_SELECTED_GROUP 0x1f8,
  MSG_DO_MOVETO 0x23c, MSG_DO_STOP 0x242, MSG_LOGIC_CRC 0x257.
- Both clients: campaign side GDI_a = player 1 = LOCAL player.
- Posted-click selection on either client produced no logic messages (selection is client-side
  until a command), and posted clicks produce no orders, so the harness still cannot test control.
- Exit path on LAN (Esc via injected DirectInput key > Exit > Yes > Skip > Continue): host and
  guest both return to the Network lobby, no crash. The owner's crash is likely in the online-only
  post-game path (results reporting expects Player_N slot players; stats show
  `MISSING: 'TimeLine:PlayerName:1'`). Unconfirmed.
- Build 0.0.2 = logging build (CNC3COOP_MSGLOG=1 by default) for an owner test of control + exit.

## 11. Owner test of 0.0.2 (C&C Online, 2026-10-18)

- Host log: commands arrive as player 1 (campaign side): `MSG_CREATE_SELECTED_GROUP`(0x1f5) then
  `MSG_DO_MOVETO`(0x23c) at frames 248/256 and 523/531. Unknown which pair was the guest's.
- Guest log unusable: it came from an earlier process (PID 3092) that stopped after hook setup;
  the game that ran was PID 21976 and could not open the locked log. 0.0.3 falls back to
  `cnc3coop_<pid>.log`.
- Exit crash, both clients, identical: AV in ntdll heap (+0x4e6e2) on a worker thread running
  0x6ac197 -> 0x69d58e -> 0x6944b0 (builds the online results report: `FUN_007ba0ea(...,5,players,
  teams)` then per-player records) -> 0x7c654c. LAN never runs this. 0.0.3 makes 0x6944b0 return
  false in co-op (the game's own "could not create report" path). Not yet confirmed online.
- tools/minidump.py reads WER minidumps (exception, modules, faulting thread stack).
- 0.0.3 adds Setup.bat (WinForms): Steam auto-detect via registry + libraryfolders.vdf +
  appmanifest_24790.acf, remembers folders in %LOCALAPPDATA%\cnc3coop\settings.json, updates only
  replace files when the install copy is newer.

## 12. Guest orders fixed (2026-10-18)

Owner test of 0.0.3 (C&C Online): exit crash gone on both clients. Guest can select and hears
voice lines but units do not move. Both logs show the same command stream, all as player 1.

Observed (LAN, two clients on one PC, msg logging on both):
- Guest selection: created by its UI (TheMessageStream `*0xbf34b0` vtable+0x48 appendMessage),
  handed to the send step `0x69fa54`, but never added to any frame (copy ctor `0x554928` called from
  `0x6a7329`) on either client. Host commands go through on both.
- Cause: receive handler `0x6a35e3` keeps, per slot (+0xe0 in the slot's 0xd0-byte record), the command
  id `cmd+0x24` from the slot's first command; if another slot already has that id it marks the slot
  -2 and drops all its commands. Both slots act for the campaign side, so the guest slot was the
  duplicate. Fix: in shared mode `JNC` at `0x6a37b0` (`0f 83 04 01 00 00`) -> `90 e9 04 01 00 00`.
  After the fix the guest's commands execute on both clients.
- Shared logic selection (Player+0x208, set only by selection messages) is now kept per client:
  each client clones its own selection messages (copy ctor, alloc `0x402997`) and, when another
  client's selection executed meanwhile, re-sends them through the game's own send sequence
  (`0x690691` ctor(frame, msg) -> `0x69e3b9` queue on `[net+0x1c]` -> `0x68d841` release) right before
  its next order. Test: host selects, guest selects, host presses S: host re-sends its selection at
  frame 642, then DO_STOP at 642, same order on both clients.
- Posted mouse clicks still do not produce move orders in the harness; keyboard orders (S) do, since
  posted keys are now fed into the DirectInput keyboard in background mode.

## 13. Mission end and mission flow (2026-10-18)

Observed (TW 1.10):
- Mission scripts end missions through objectives and by killing the enemy sides; only GDI 3-3 uses
  `VICTORY`/`DEFEAT` and GDI 3-4 `QUICKVICTORY` (script survey, see mission-survey.md). Script actions:
  VICTORY = type 3 -> `[0xbf0440]` (TheVictoryConditions) vtable +0x7c; DEFEAT = 4 -> +0x80;
  PLAYER_KILL = 78 -> `0x518311` -> Player::kill `0x4a67c5`; MARK_MISSION_OBJECTIVE_COMPLETED = 536 ->
  `0x4f79b8` -> objective setStatus(2) `0x4c900c` (EVA only, no win check). TheMissionObjectiveTracker is
  `*0xb81560`.
- VictoryConditions vtable `0xa6456c`: +0x38 hasWon `0x4f671f`, +0x3c hasLost `0x4f67b6`, +0x40
  isDefeated `0x4f67f5` (+0x74 killed, +0x88 elimination rule), +0x7c victory `0x516b29` (records campaign
  progress via `0x4e0766`/`0x4e0614`), +0x80 defeat `0x516bfb`. The update's last block calls
  hasWon(local) -> victory, else hasLost(local) -> defeat. hasWon is last-side-standing.
- With update on in co-op, the campaign side failed the multiplayer elimination rule at the start
  (EndGameUI DEFEAT from `0x510279`). Fix: hook isDefeated so the local (campaign) side is never
  eliminated by that rule; the update then stays on.
- Test (two clients, LAN, prologue): a test-only trigger kills all other sides at frame 600 (inside the
  lockstep dispatch, same frame on both): both clients get EndGameUI(VICTORY) via `0x51049e` at frame 611.
- Mission flow: the current mission is the folder name in `<home>\coop_mission.txt`, read at every game
  start; on victory both clients advance it to the next mission in missions.txt (same campaign group).
- Movies: `Movies\1.0\Movies*.big` hold VP6 files (`g01a`, `g02ba`, ...); the mission -> movie link lives in
  compiled campaign data. Playing them from the DLL is not done yet.

## 14. GDI 2 (Pentagon) defeat traced (2026-10-18)

Observed (two clients, LAN, Pentagon started directly, nobody playing):
- 0.0.5 as first built: DEFEAT at about 15 s on both clients, same frame. Trace path: a DLL hook on the
  script runner `0x532d6f` (thiscall, `ret 8`; arg = Script, vtable `0xa646f4`; +0x30 OrCondition list,
  +0x34 actions, +0x38 false actions, +0x44 name) and on the single-condition check `0x528c13` (thiscall,
  `ret 4`; condition +4 type, +0xc first parameter, AsciiString at parameter+0x10). Condition type numbers
  come from the template table in `0x5082ec` (record stride 0x80; e.g. 4 TIMER_EXPIRED, 8 TEAM_DESTROYED,
  15 NAMED_DESTROYED, 53 UNIT_HEALTH, 56 PLAYER_HAS_OBJECT_COMPARISON). Within an OrCondition the
  conditions are ANDed; the OrConditions are ORed.
- Chain: the map's lose scripts only set the flag `Mission Lost`; a script not in the map (empty name,
  `TIMER_EXPIRED t_ShowDefeated` -> `DEFEAT`) shows the defeat a few seconds later. At frame 75 the
  multiplayer elimination rule judged three AI sides defeated (Plyr_GDI_b, Plyr_Nod_d, Plyr_GDI_d) and the
  victory update killed them (`0x4a67c5`), destroying their objects. Plyr_GDI_b owns the Pentagon's base
  object `_G_B_Penta-building`; the map's "symbiotic health" scripts then set the Pentagon to 0 ->
  NAMED_DESTROYED -> Mission Lost.
- Fix: in co-op a side counts as defeated only after Player::kill (flag `Player+0x20c`, set by the
  scripts' PLAYER_KILL), never by the "has nothing left" rule. After the fix no side is eliminated.
- What remains with nobody playing: the allied power plant 1 (starts at 50%) is destroyed by Nod at
  about 40 s and the mission is lost by its script (`G_A_POwer Plant 1 lose`). Expected for an idle
  player; not compared with a single-player run. Pentagon has no difficulty-specific scripts.
- Prologue power plant (owner report: bombers damaged but did not destroy it): no script destroys it;
  a Nod attack team (`Act1_TEAM_PowerPlantAttack`) targets it, then the MCV. If the team is shot
  before it arrives the plant can survive. Prologue difficulty scripts change Nod money and attack
  forces only.
- Only the first 0.0.5 build had this problem (0.0.4 had the victory update off). Before the fix any AI
  side that failed the rule at the first check would have been killed in any mission (not checked
  mission by mission).
