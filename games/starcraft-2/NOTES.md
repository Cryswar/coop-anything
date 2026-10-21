# StarCraft II feasibility research notes

## Current checkpoint — paused 2026-10-16

The owner requested a documented pause for a usage reset and another agent. Read the private workspace `outputs/SC2-HANDOFF.md` before resuming. All trial games have ended. Native API shared control/production passes, but native world selection in the connected two-participant launcher remains unresolved. Single-client selection and two independent concurrent solo games work. The entries below are chronological: initial assessment-only scope and early host-selection/race hypotheses are superseded by later runtime evidence. Do not publish private workspace material or launch new tests until the owner resumes.

## Chronological history

Date: 2026-10-16
Scope: assessment only; no game changes or runtime tests.
Repository reviewed: adcbfa152184b84e7f61c0f74cef6e04de041a5d.
## Observed
- AGENTS.md and research/triage instructions read.
- rg search for starcraft/sc2 across this checkout returned no matches.
- Blizzard public protocol exposes local-map creation, realtime and remote host_ip.
Sources: https://github.com/Blizzard/s2client-proto/blob/master/s2clientprotocol/sc2api.proto
- Archipelago SC2 client source available for studying mission launching.
Source: https://github.com/ArchipelagoMW/Archipelago/blob/main/worlds/sc2/client.py
## Derived
- Local multiplayer mission launcher is a candidate; does not prove campaign compatibility.
## Hypotheses
- Shared control on extra participant slots can preserve original faction ownership, but UI and triggers may require adapters.
## Unknown
- User game build, executable SHA-256, install and campaign ownership not inspected.
- No game run, network test, save test, or binary analysis performed.
- Public GitHub issue/PR pages failed via web fetch; retry via public API.
## Searched
- Co-op Anything checkout: starcraft, sc2 (no matches).
- Web: SC2 API shared control archon mode campaign multiplayer local maps.
- Web: Starcraft 2 Archipelago client multiplayer shared control campaign github.
- Web: Starcraft 2 coop campaign offline launcher shared control no arcade.
## Additional observed evidence
- Public issues endpoint returned three issues/PRs, none about SC2 (all states, page 1; fewer than 100 results).
- Archipelago starcraft_launch uses a single Bot participant in realtime; adapter sends campaign settings/tech via chat commands. This is a launching/integration precedent, not evidence of same-mission co-op.
- Public SC2 protocol does not expose explicit Archon team/user mapping. client_ports comment says only a single client is supported. Treat >2 human clients as an early unverified gate, not a proven hard limit for current retail builds.
- Galaxy API reference exposes control and spending alliance settings.
- LordDz author announcement confirms separate 2-3-player HotS co-op on Arcade; outside requested delivery route.
- SC2CCM source manages campaign map/mod files; inspected code does not establish multiplayer session creation.
- No REA tool is exposed in this chat, and rea was not found on PATH or in checked standard skill/npm locations. This does not establish that the user's REA installation is absent.
## Derived recommendation
- Prototype local-map native-network session first, preserve one faction and script ownership, then verify shared control/spending and human UI.
- Normal campaign-shell integration is a later independent milestone; save/reload and >2 humans must not be promised.
## Further sources
https://github.com/7thAce/SC2CCM/blob/main/CCM/FormMain.cs
https://mapster.talv.space/galaxy/reference/lib-ntve-gf-make-ally
https://tl.net/forum/sc2-maps/554236-heart-of-the-swarm-co-op
https://archipelago.gg/games/Starcraft%202/info/en
https://github.com/Blizzard/s2client-proto/blob/master/docs/protocol.md
## Search limitations
- Search engine returned noisy/missing hits for Archon API and >2-player issues; no exhaustive absence claim.
- No Discord access, account use, software installation, mod download, or publication.
- No confirmed ready-made non-Arcade shared-faction campaign solution found in reviewed sources.
## Assessment deliverables
- Completed feasibility/design report, copied to games/starcraft-2/README.md.
- Added a conservative existing-project directory entry for LordDz HotS co-op (current status unknown, QA none).
- Directory regeneration completed; validation --check passed.
- Source snapshot SHA-256 manifest stored in local/source-hashes.json.
- No runtime validation or publication performed.
## Follow-up: confirmed party size
- User requirement: exactly two humans (host + buddy); larger parties optional and must not block success.
- Test sequence accepted; initial read-only build identification performed.
- Installed executable metadata identifies build 5.0.17.98310. SHA-256 and private path recorded in local/sc2-identity.json.
- No copied test installation or runtime test yet. The default Python lacks s2clientprotocol, websocket/websockets and mpyq; no packages installed.
- Updated report to remove larger-party support as a required gate.
## Runtime experiments (initial session)
- Created an independent 31.83 GiB test copy; original executable hash matched recorded identity before test-copy changes.
- Loaded existing Archipelago libraries with bundled Python 3.12, without installing packages.
- First sandboxed startup timed out. Unsandboxed API ping succeeded, identifying data version 9C1F3D3D30CA252E33FC67CA05CBE1EA.
- Initial shfolder-only isolation was incomplete: normal Variables.txt, ExecuteInfo.txt and startup logs were written. No campaign saves changed. Exact previous settings contents were not backed up; do not claim exact restoration.
- Corrected isolation covers legacy and known-folder APIs. Native folder API test passed. Corrected SC2 launches and simultaneous two-client API pings passed with normal Documents profile metadata unchanged.
- Windows Security blocked remote-injection launcher; replaced that architecture with ordinary loader dependency in copied executable. No protections disabled or exclusions added.
- Downloaded Blizzard Empty.SC2Map fixture by pinned Git blob; built an MPQ writer with full entry roundtrip verification for the script-modified tiny fixture.
- Local map create_game accepted packed fixture. Loose directory map was rejected by public API as nonexistent.
- Two-client join and shared control/spending tests in progress; do not claim these pass yet.
## Shared-faction runtime result
- Native two-client multiplayer on unmodified Blizzard ladder sample passed (both player IDs 1 and 2, matching simulation loop).
- Tiny Empty fixture rejected guest with NoSpaceForUser; do not use its metadata as the multiplayer template.
- Rebuilt fixture preserving multiplayer map metadata. Two clients joined. Guest moved host-owned marine. Guest trained at host-owned barracks: host minerals 1000 -> 950, guest minerals remained 0, host barracks production order recorded.
- Positive stepped probe passed; normal Documents profile unchanged during this run.
- Negative-control test and production-completion/state-comparison checks pending.
## Negative control
- Disabled both shared-control and shared-spending alliance aspects, leaving shared vision.
- Guest move and production requests were rejected with action result 87.
- Host marine did not move, host minerals stayed 1000, guest minerals stayed 0, host barracks had no order, and no new marine appeared.
- Both clients agreed on selected unit tags/owners/positions. Negative-control scenario passed.
- This establishes that the positive API commands depended on permissions, rather than bypassing ownership checks.
## Realtime completion result
- Realtime positive probe passed on two actual SC2 clients.
- Guest moved host marine and trained from host barracks, host minerals 1000 -> 950, guest minerals 0.
- Production completed: host-owned marine count increased from 1 to 2; no player-2 army appeared.
- Compared selected unit tags/owners/positions across both clients: equal.
- Independent camera positions observed: (30,130) and (95,40).
- Normal Documents profile unchanged during corrected realtime run. Original install executable hash rechecked separately.
- No campaign mission, native campaign hub, GUI selection/build-placement, separate-PC networking, or progression transition tested.
- Scope achieved for this spike: native two-client shared control/spending/camera feasibility demonstrated; campaign integration remains a later milestone.
## Final deliverables and cleanup record
- Saved user-facing runtime report, sanitized JSON outcomes and source snapshot in outputs/.
- Recorded four successful scenario results: unmodified multiplayer baseline, positive stepped control/spending, permissions-disabled negative control, and positive realtime production/camera/state checks.
- Repaired normal ExecuteInfo.txt to point to original installation, preserving other bytes. Initial Variables.txt contents were not backed up, so exact pre-test restoration remains unknown; disclosed in the report.
- All test drivers ended. Per-client exit codes/forced termination were not retained, so no stronger exit-quality claim.
- Source snapshot syntax checks and directory validation passed. No publication or account use.
## Campaign mission spike started
- Read-only CASC extraction from the copied installation using locally built CascLib at commit 38a34665624b8775bb875274b36191b21c38d97b; no software installed.
- Extracted 41 stock The Outlaws files (~7.8 MB) from the original TRaynor02 package, excluding other locales. Private assets stay under local/.
- Stock mission faction slots 1..5 preserved; added unused slot 6 as second human in the co-op copy.
- Baseline and co-op packed-map roundtrip checks passed. Added in-game diagnostic banks and counters; original campaign script retained.
- Set Normal difficulty identically in both probes as test configuration.
- Baseline mission initialization/runtime test in progress; no campaign compatibility claim yet.
## Campaign baseline and guest observation crash
- Single-player stock-derived The Outlaws probe passed 7200 loops (450 game seconds) on Normal. Main objective active, original AI started, and one original attack-wave send returned.
- Two-player copy admits original host slot 1 and added guest slot 6. Both advance through 7200 loops; host diagnostic bank reaches 450 seconds with objective/AI/wave counters intact and guest control/spending enabled.
- Guest crashes with ACCESS_VIOLATION on its first API observation request. Reproduced at 400 and 7200 loops. Removing guest score data, then both raw and score data, did not prevent the crash.
- Do not claim campaign co-op passes based on joins or stepping alone. Host-only observations plus guest commands are being tested to distinguish gameplay from API observation failure.
- User saw a crash reporter; advised unchecking Send to Blizzard before closing. No crash-report submission authorized or performed by agent.
- Normal Documents profile metadata remained unchanged in these corrected-isolation runs. Private logs and original mission source remain under local/.
- Intended hands-on network: separate PCs over the internet. Internet transport remains untested.
## Campaign workaround and realtime checks
- Avoiding guest API observations passed the 7200-loop co-op probe. Both mission banks matched at 450 and final 483 game seconds. Original AI/wave counters remained active. Guest moved a player-1 marine, spent 50 host minerals to train, and produced an additional player-1 marine; no slot-6 army appeared. Both processes exited 0.
- Short realtime campaign probe also passed movement, production and bank matching (final tick 58). It did not reach the first wave. Both processes exited 1 on cleanup; this remains unresolved and is not a clean-exit pass.
- Root cause of guest observation crash remains unknown. Workaround is sufficient for further investigation, not proof of a finished mod or native UI play.
- User asks about Parsec versus direct connection versus virtual LAN. Recommended first trial transport is a private virtual network such as Tailscale, based on official docs. Parsec to one host game would share one mouse/camera. No network software installed or settings changed.
- Saved campaign probe report, sanitized results and own adapter/driver sources under outputs/. No original stock script/assets or crash dumps included.
## Visible launcher implementation and initial checks
- User chose visible local launcher followed by direct internet connection; buddy uses Windows/SC2 without Archipelago.
- Built standalone x64 .NET Framework launcher using Windows' ClientWebSocket and a small protobuf wire encoder/decoder. No Python/Archipelago needed by this launcher.
- Added a read-only CascLib mission extractor and own MPQ writer; package contains no stock game assets. Verified all 42 MPQ entries readable and all untouched stock files byte-identical using the independent Python reader.
- Standalone local two-client launch with a shared relative map name passed. Proper leave/quit sequence yielded exit 0 for both processes; normal Documents profile unchanged. Longer visible run also exited both 0, though leave requests logged Request missing command (unresolved protocol error).
- Visible native host selection works. Guest sees the army and starting camera but cannot select units using the initial metadata record. Human input is therefore not yet a pass.
- Testing a complete copy of stock human MapInfo settings for slot 6, rather than an empty race field. This is a hypothesis for both UI selection and the previous observation crash; not confirmed yet.
- Standalone extraction caught ABI and checksum transcription errors during development; corrected one-byte native boolean marshaling, bounded enumeration, and independently verified checksum constants before map edits.
## Native UI comparison follow-up
- Cloning the complete stock human metadata did not fix the slot-6 observation crash (400-loop reproduction). Keep workaround; empty race was not the root cause.
- Explicit post-intro UI setup can display a selected host command center in the guest, but guest mouse selection still failed.
- Blizzard's full shared-control alliance helper did not fix native guest selection in the campaign.
- Reproduced guest mouse selection failure on ordinary player-2 ladder fixture, so it is not specific to campaign scripts or slot 6.
- Mouse clicks on the guest's Menu button work; this is not a complete mouse-input failure.
- Explicit disposable-profile selectuncontrolable=true did not fix selection on the fixture.
- Disabled .NET ClientWebSocket automatic keepalives and added request IDs. A >120-second fixture run then exited both clients 0 without the previous Request missing command errors. This supports the keepalive diagnosis; no API is exposed off loopback.
- Next diagnostic separates own-unit selection from shared-unit selection and tests raw_affects_selection interface option.
- Native UI testing paused when physical Escape was detected in previous turn; resumed only after user explicitly asked to continue.
- User needs router guidance. Recorded private local adapter/gateway metadata without changing network settings. Direct internet transport is still untested.

### 2026-10-16 — native input isolation follow-up

- Observation (owner): both host and guest cannot select units in the world, but production works from the structure already selected. Treat this as a world-input/selection defect, not total input failure or proof of missing spending permission.
- Observation (automated native UI): with both raw and score interface flags false, guest world clicks still leave the forced command-center selection unchanged. Standard hotkey profile shows Selection = Left Mouse Button and Smart Commands = Right Mouse Button.
- Observation (automated native UI): a per-process HIGHDPIAWARE compatibility environment did not restore click or box selection after closing the campaign tutorial panel. No global Windows compatibility setting was changed.
- Verification: native-interface run 20261016-172536-361-local shut down both game processes with exit 0 and normal Documents profile unchanged.
- Next diagnostic: add script mouse-click and unit-selection counters to disposable banks, rather than inferring event delivery from hover tooltips. This is diagnostic only; no input workaround is claimed.
- Unknown: underlying cause of world input failure. Disabling raw/score, standard alliance/UI permissions, enabling all selection types, and DPI compatibility have not resolved it.

- Observation (script instrumentation, run 20261016-173029-030-local): each client bank shows one initial scripted selection and one selected unit; measured world click produces zero script mouse-click events and no extra selection events. The callback counter is diagnostic evidence, not proof of a particular native cause.
- Observation (spatial-interface comparison, run 20261016-173159-454-local): enabling feature_layer with 84x84 screen / 64x64 minimap / camera width 24 does not restore selection after the tutorial closes. Both clients exit 0, normal profile unchanged.
- Observation (unmodified BelShirBaseline.SC2Map, run 20261016-173308-450-local): clicking the owning player's command center leaves selection panel empty. This reproduces without campaign modifications or the shared-control adapter. Single-client vs two-client distinction remains to test.

### 2026-10-16 — positive native-input control

- Observation (native UI, 20261016-173528 solo run): one API-launched client on stock BelShir accepts a world click selecting its own command center. This uses one participant plus built-in computer and an absolute map path; no multiplayer ports.
- Observation (native UI, overlapping independent solo runs): while a second independent SC2 game is loaded, the first accepts a worker selection, and the second accepts command-center selection. Multiple running instances alone do not disable selection. Both games exit 0 and normal profile metadata remains unchanged.
- Observation (native UI, separate ProgramData network comparison): the two-participant network game still cannot select its own command center with fully separate temporary ProgramData folders. Shared ProgramData is not a sufficient explanation.
- Derived: failure is specific to the tested network-session launch path, rather than campaign scripting, shared-control ownership alone, a universal API input limitation, or merely opening two SC2 processes. Exact cause remains unknown. Solo comparison also differs in map path and opponent type; do not claim a uniquely identified engine defect.
- Next: compare network session initialization / human local-user mapping against the working single-client path; consider replay/initData and native input dispatch investigation. Do not send a buddy package as playable until native selection and movement pass.
- Cleanup verification: final separate-ProgramData run 20261016-173720-438-local exited both game processes with code 0; normal SC2 Documents metadata unchanged. All trial game sessions launched during this investigation have ended. User-facing comparison saved as outputs/sc2-input-investigation.md.

## 2026-10-16 evening: native input isolation (Claude session)

Probe: `work/input_probe.py` (Python 3.12 + Archipelago `library.zip`). Two API participants on stock
`BelShirBaseline.SC2Map` in the isolated copy (patched exe), realtime, game ports on loopback. A real
`SendInput` click at a command center's screen position; verdict from raw `is_selected` plus a
selection-panel pixel check and a screenshot.

Observed (each row is a run that actually happened):

| Run | Result |
|---|---|
| Solo control, raw on or off | Native click selects the CC. Validates the automation. (An earlier probe bug, a wrong `INPUT` struct size giving SendInput error 87, invalidated runs solo1–6.) |
| 2 participants, creator, raw=True | **Native click selects the CC**, and the API reports `action_feature_layer.unit_selection_point` |
| Creator with raw=False (polling on or off) | Selection fails: Codex's host symptom, now explained |
| Creator with raw=True, no polling | Works. Polling is irrelevant |
| Joiner (any of: raw+score, raw only, raw_affects_selection, feature layer) | World click/drag produces **no action at all** and no drag box. Hover ring works. A minimap click works (`camera_move`) |
| Roles swapped (second-started process creates) | The creator works and the joiner fails, so this isn't process start order |
| Joiner joins 4 s before the creator | The creator still gets player 1 |
| Joiner: API `action_feature_layer.unit_selection_point` at screen centre after a raw camera_move | Result Success; the joiner's native UI shows the CC selected |

Derived: the host fix is `InterfaceOptions.raw = true` on the creator. The joiner's world-click handler is gated
inside the client (not permissions or network). Creator vs player-1 can't be separated, since the creator always gets P1.
Unknown: whether the joiner gate also happens with the two clients on separate PCs.

## 2026-10-16 night: host fix applied, guest click bridge prototype (Claude session)

Code (originals kept as `work/TrialLauncher.cs.before-claude`):
- `TrialLauncher.cs`: `Settings.Raw` (default true) sets `InterfaceOptions.raw` on JoinGame. `Settings.Bridge` (default
  true) + `BridgeCalibration` (default `bridge-calibration.json` next to the config). The process is per-monitor DPI
  aware (`SetProcessDpiAwarenessContext(-4)`).
- `ClickBridge.cs` (new): a WH_MOUSE_LL hook on the guest SC2 window. World-area left click maps to
  `unit_selection_point` (shift = toggle, ctrl or double click = all of type), a drag maps to `unit_selection_rect`
  (shift = add), and a right press/release maps to `unit_command` ability 1 (Smart, shift = queue). It sends via the guest's
  own API on a worker thread. Native events pass through, so the console, minimap and menus stay native.
- `calib.py`, `ui.py`, `input_probe.py`: calibration and test drivers. The calibration homography (window pixel ->
  feature-layer coordinate, 1280x720, feature width 48, 384 px) came from 41 native host right-clicks. Mean error
  1.7 feature px (~0.2 world units). Stored in `outputs/sc2-local-trial/bridge-calibration.json`.
- Build: add `.\work\ClickBridge.cs` to the csc command. Output `outputs/sc2-local-trial/SC2CoopTrial-claude.exe`.

Observed on The Outlaws (`trial.json`, local mode, host P1 + guest P6), screenshots in the Claude scratch runs:
- Host (raw=true): native click selects the Barracks, native box selects 6 marines, native right-click moves them.
- Guest without the bridge: native click on the Barracks does nothing (as before).
- Guest with the bridge: click selects the Barracks, box selects 6 marines (allied yellow rings), right-click moves
  them to the clicked spot. Bridge log: `rect 205,111-241,146 -> ok`, `smart 220,183 from client 793,324 -> ok`.
- Every run exited 0 with the normal Documents profile unchanged. No guest RequestObservation was used.

Bugs found and fixed along the way: wrong client size under 125% display scaling (hook points are physical, the .NET
process wasn't DPI aware); shift's `selection_add` encoded outside the rect message; right-button release positions
from the hook sometimes jump (seen as (0,440) and (1798,828)), so commands use the press position.

Known gaps (not yet built or tested): targeted abilities and build placement (attack-move, abilities, buildings: the
bridge can't see the guest's pending targeting mode). The camera model assumes flat ground at the calibrated window size,
so cliffs and other aspect ratios add error. Point-select on empty ground returns result 3 (harmless). Not tested on two PCs.

## 2026-10-16 late: guest targeted commands (Claude session)

Observed first: on the guest, hotkeys and the command card enter native targeting mode (the command card shows
"Attack ... Esc"), but the targeting click in the world is dropped like any other world click. The OS cursor
handle doesn't change in targeting mode (SC2 draws its own), so the cursor can't detect it.

Guest observation crash root cause (crash report code bytes + registers, campaign guest slot 6):
`movzx eax,r14b; mov ecx,3; sub ecx,eax; movzx eax,word [rbx+rcx*2+0xCC]`, i.e. index `3 - player`, which is
out of bounds for player 6. The API assumes 1v1. Any guest slot above 3 crashes on RequestObservation, independent of
raw/score/feature options (also tested with guest raw off).

Design (implemented):
- `work/campaign_targeting.galaxy` (appended by `BuildMap`): `TriggerAddEventTargetModeUpdate(6, null, any)`
  stores the pending abilcmd and saves the guest's local `CoopBridge` bank (`Target/seq,on,abil,cmd`).
  `TriggerAddEventUnitOrder(move 0/1)` collects guest-issued probe orders. One game loop later each probed
  unit either gets the real order (point target, unit target, or the nearest unit within 1.25 when the point
  order is invalid, e.g. Gather on minerals) or has its queue restored without the probe. Build/Morph
  abilities go to one unit. Diagnostics live in `Probe/seen,restored,issued,issue_failed,last`.
- `ClickBridge.cs`: polls the bank (15 ms); a low-level keyboard hook plus command-card clicks mark "targeting may
  start", and the next world click waits up to 600 ms for the map's report. In targeting, a left click sends a queued
  Move probe (Patrol if shift) and then Esc (not with shift). A right click in targeting sends Esc only.

Observed on The Outlaws (guest P6, local mode), all via bridge + native hotkeys:
- A + click on empty ground: 6 marines attack-moved to the point (screenshot). Earlier, a click next to SCVs resolved
  to a unit and the marines force-attacked and killed an SCV (owner saw it). That's consistent with native A-click on a unit.
- SCV, B, S + click: a supply depot was built at the point with P1's minerals. Bank: `seen 7, issued 7, failed 0`.
- SCV, G + click mineral field: the first attempt targeted a point and was dropped (orders 0). After the nearest-unit
  fallback the SCV mines ("Workers 1/30", `SCVHarvest ... orders now 1`).
- Every run exited 0 with the normal Documents profile unchanged.

Not yet tested: the queue-restore path (mixed selections where some units lack the ability), caster spells,
shift-queued targeting, drags while targeting, structure targeting (rally via probe fails since structures
can't Move), and other missions. The 600 ms wait after non-neutral keys can delay a plain selection click.

## 2026-10-16 late night: two-PC networking and buddy package (Claude session)

Finding: SC2 API multiplayer only works on loopback. A guest using `host_ip=127.0.0.2` fails JoinGame
(error 12 / 537001988). pysc2 documents the same limit ("SC2 is limited to only allow LAN games on
localhost", `pysc2/bin/play_vs_agent.py`) and forwards all four game ports between machines (TCP server base
guest->host, TCP client base host->guest, UDP between the game ports).

Implemented (`work/Tunnel.cs`): both games use host_ip 127.0.0.1. The guest launcher opens ONE TCP connection to
the host launcher (port 6600) and the launchers impersonate each other's game ports on loopback, framing UDP
and TCP over that connection. Only the host needs an inbound port, so the guest's NAT doesn't matter. The hello
exchange carries the map SHA-256 (both must match) and the host's base port. `Upnp.cs` maps TCP 6600 via
HNetCfg.NATUPnP (best effort; can be skipped with SC2_COOP_NO_UPNP=1) and prints join instructions (with a carrier-grade NAT warning).

Product mode (`Settings.Product`): runs the player's normal SC2 install (exe hash check only, no copy, no exe
change), builds the map from the player's own CASC into `local/` (`prepare-product`), and the bridge reads the bank
from `Documents\StarCraft II\Banks`. The map built from the owner's real install has the same SHA-256
(a416fc29...) as the one built from the copy: deterministic.

Package: `outputs/sc2-coop-package/` and `outputs/SC2Coop-Outlaws-test.zip` (403 KB): SC2Coop.exe (winexe GUI:
Prepare / Host / Join / Local test), CascLib.dll (static CRT, imports only WS2_32 and KERNEL32) + MIT license, three
adapter .galaxy files, bridge-calibration.json, README.txt. No game files.

Observed on one PC:
- Copy mode, host + separate join launcher via tunnel (guest base 7500, host 6500): both joined (P1/P6), the guest's
  bridged box-select + move was visible in the host's window.
- Product mode (owner's real install, UPnP skipped): same, plus guest A-move via bridge, visible in the host's window.
  The owner's Variables.txt/ExecuteInfo.txt were backed up before and restored after (Variables.txt had changed).
- Caveat: each tunnel carried only ~1 KB per session. With different bases on one PC, the games exchange their
  real UDP ports in-band and talk directly over loopback, so the tunnel's UDP path was NOT exercised. On two PCs
  (same base both sides) the in-band ports land on the tunnel's impersonation sockets, as in pysc2's design.
  Only a real two-PC test can confirm it.
- The real profile ran the game windows at 2402x1351. The bridge scales from the 1280x720 calibration, which is valid
  for the same 16:9 aspect.

Not tested: two real PCs, UPnP on a real router, SmartScreen/Defender reaction to an unsigned exe with
low-level hooks, non-English clients (map extraction expects enUS).

## 2026-10-16: UPnP check on the owner's router

Observed: Windows HNetCfg.NATUPnP found no StaticPortMappingCollection. An SSDP M-SEARCH (IGD v1/v2, rootdevice) got answers only from 4 LAN media devices, none from the gateway 10.0.0.1. A NAT-PMP external-address query to 10.0.0.1:5351 timed out. So UPnP/NAT-PMP is off or unsupported on this router. No mapping was created; TCP 6600 was absent before and after. `upnp-test` aborts if the port is already mapped, and TryMap now only replaces mappings described as 'SC2 campaign co-op'.


## 2026-10-16 late night: Steam transport, tested on a mock (Claude session)

Goal: friends connect without trading IPs or opening ports. Following `playbook/networking-steam-spacewar.md`.

Implemented:
- `work/Tunnel.cs`: the frame format is unchanged, and the byte transport is now any ordered, reliable `Stream`.
  `Accept`/`Connect` (TCP, still the default) and the new `Tunnel.Over(host, stream, ...)` share the same code.
  Each frame now goes out in a single `Write`, so a message transport sends one message per frame.
- `work/Steam.cs` (new): loads `steam_api64.dll` from next to the exe at runtime through the flat API. There are no
  SDK headers, and the export names are the playbook's plus `GetNumLobbyMembers`, `GetLobbyMemberByIndex`,
  `GetLobbyData`, `ClearRichPresence`, `GetFriendCount/ByIndex/GamePlayed/PersonaName`, `GetPersonaName` and
  `SteamAPI_Shutdown`. Callbacks use manual dispatch on one pump thread.
  - Host: friends-only lobby (type 1, max 2) with lobby data `sc2coop=1`, `map=<sha prefix>` and `host=<name>`,
    plus rich presence `connect=+connect_lobby <id>`. It calls `ActivateGameOverlayInviteDialog` and copies the
    lobby id to the clipboard (GUI only). It accepts a SessionRequest only from a current lobby member.
  - Guest: finds friends whose `FriendGameInfo_t` shows AppID 480 and a lobby (GUI "Find" or CLI `friend`), or uses
    a pasted lobby id, an invite (callbacks 333/337) or `+connect_lobby <id>` on the command line. It joins the
    lobby, then checks `sc2coop=1` and leaves if the tag is missing. It talks only to the lobby owner.
  - `SteamLink : Stream`: channel 0 carries reliable+NoNagle messages of at most 60000 bytes. It retries every 5 ms
    on `LimitExceeded` (25). Channel 1 carries a 1 s keep-alive and a goodbye. The link drops after 20 s of
    silence, on LobbyChatUpdate left/disconnected/kicked/banned for the peer, or on SessionFailed. The
    send/receive buffer sizes and RecvMaxMessageSize are set to 32 MB, best effort (config keys 9/47/49).
  - Errors: a missing DLL, Steam not running (`IsSteamRunning` false or `InitFlat` 2), a version mismatch (3) and
    "not signed in" (`BLoggedOn`) each get their own message.
- `TrialLauncher.cs`: modes `steam-host` / `steam-join`. The GUI has "Host via Steam (no IP needed)" and
  "Join a Steam friend" (a friend dropdown with Find; an empty box waits for an invite). It preselects Steam join
  when started with `+connect_lobby`. Role checks now use `hosting`/`guest` flags instead of `mode=="join"`.
- Test-only hooks, off unless an environment variable is set: `SC2_COOP_TEST_MARINES=1` (host logs player-1
  marines from its own observation at +25/+50/+75 s) and `SC2_COOP_TEST_BRIDGE="ax,ay,bx,by;fx,fy"` (guest puts
  a box select and a right-click into the click bridge's queue at +35 s, without moving the mouse).
  `ClickBridge.TestDragAndSmart`.
- `work/steam-mock/steam_mock.c` → mock `steam_api64.dll` (MSVC x64). The Deus Ex repo's mock wasn't available
  locally, so this one was written from scratch to the playbook's description. Peers talk over 127.0.0.1 UDP
  (port 47100+n). Lobby create/join/leave and member lists are real between the processes. Hosting is published
  in `%TEMP%\sc2coop-steam-mock\<n>.lobby` for friend lists. A stranger's first message is held until
  AcceptSession (1251). Env vars: `MOCK_STEAM_ID/NAME/FRIENDS/INVITE/OFFLINE`.
- Build: `work/build-launcher.ps1 <out.exe> [exe|winexe]`. Test folder: `outputs/steam-mock-test/`, which has
  the mock DLL, copy-mode `trial.json`, and launcher logs `*.out`.

Observed (one PC, two launcher processes, copy-mode install, hidden windows, host base 8500, guest base 9500):
- Mock `MOCK_STEAM_OFFLINE=1`: the launcher fails with "Steam isn't running. Start Steam and sign in, then try
  again." No DLL: "Steam connections need steam_api64.dll next to SC2Coop.exe, and it is missing ...". Both exit 1
  with the profile unchanged.
- Run 1 (friend list, pre-fix build): host lobby open → guest "Joining Steam friend MockUser1" → session accepted
  → tunnel hello OK → host joined P1, guest P6. Guest bridge: `test rect 205,111-241,146 -> ok`,
  `test smart 220,183 -> ok`. The host's own observation showed 6 P1 marines with mean position 86.6,34.4 at loop 548,
  then 86.3,28.0 at loops 1120 and 1693. Both SC2 exited 0, the guest's goodbye was seen by the host, and the profile
  was unchanged.
- Run 2 (invite path, `MOCK_STEAM_INVITE`) FAILED: "Joining the Steam lobby timed out". Cause: a race in
  `Steam.cs`. The call result could be dispatched before the waiter was registered. Fixed (the waiter is now
  registered under the same lock as the call). This would have hit real Steam too.
- Run 2 rerun (invite path, fixed build, no guest command = control): joined P1/P6. P1 marines stayed at
  86.6,34.4 in all three samples, so the movement in run 1 came from the guest's command. Clean exit 0 on
  both sides; the host's "buddy closed the connection" and the guest's "left the lobby" paths were both seen.
- Run 3 (friend list, final build): same as run 1. Host P1, guest P6, bridge rect/smart ok. P1 marines went from
  34.4 at loop 544 to 28.0 with 1 unit still holding an order at loop 1117, then 28.0 at loop 1689. Exit 0 both,
  profile unchanged.
- Run 4 (direct TCP regression on the refactored Tunnel, tunnel port 6610, UPnP skipped): joined P1/P6. Bridge
  ok, the same marine movement, exit 0 both, profile unchanged.

Derived:
- The tunnel frames are transport-independent. Steam only replaces the byte stream, so game traffic is framed the
  same way as over TCP.
- As in the earlier TCP tests, with different bases on one PC the games talk game UDP directly over loopback.
  Each tunnel carried about 1 KB (hello + game TCP join), so these runs check the Steam join/connect path and that
  the game-level session works, not sustained game traffic through the Steam relay.
- The reliable channel serialises everything, including UDP game datagrams. That is head-of-line blocking under
  loss. If relay latency hurts, send `Udp` frames unreliable (flag 1) on their own channel.

Unknown, needs two real Steam accounts on two PCs:
- whether `InitFlat` succeeds from SC2Coop.exe with only `SteamAppId=480` in the environment plus
  `steam_appid.txt` next to the exe;
- that the interface versions (User v023, Matchmaking v009, Friends v018, NetworkingMessages v002, Utils v004)
  match the DLL that will ship;
- message identity offset 16 on x64 (derived from the struct layout; the mock uses the same assumption, so it
  can't catch an error here);
- that `GetFriendGamePlayed` shows the lobby while the host's status is "Spacewar";
- invite overlay behaviour from a WinForms app with no overlay, and Join Game while the launcher isn't running
  (it probably starts Spacewar instead);
- relay latency/throughput for real game traffic;
- that config keys 9/47/49 are the right enum values (the return value is ignored).

Steamworks redistribution (checked 2026-10-16, partner.steamgames.com/documentation/sdk_access_agreement):
section 1.1(b) lets a licensee redistribute the files in the SDK's `redistributable_bin` "along with the Licensee
Software in object code form". It doesn't restrict this to software on Steam. The licensee must have accepted the
SDK Access Agreement (Steam login on the partner site), and Valve reserves all other rights. Steamworks.NET's
releases bundle the same Valve DLL; its own MIT license doesn't relicense the DLL. Not bundled; the owner decides.

Real Steam, single account (owner's PC, 2026-10-16 23:55), observed:
- DLL: the owner copied `steamworks_sdk\sdk\redistributable_bin\win64\steam_api64.dll` into the package.
  SHA-256 `e6d9bafb9a41e42fba7b21553db49f8719027af3f0deb23ff86cfc44d60e776d`, 319128 bytes, file version
  10.87.46.89, Authenticode valid (Valve Corp.).
- "Host via Steam": `InitFlat` OK, signed in as the owner's persona (Spacewar, AppID 480). All exports and
  interface getters resolved (any missing one would have aborted the start). The friends-only lobby was created
  (id <lobby id>) and the id was copied to the clipboard. Stop gave "Cancelled" (expected). The owner
  reports that "Join a Steam friend" also showed the correct sign-in.
- Not yet shown: a second account joining, so the friend list, session accept, message offset 16 and relay are
  still unknown (see list above).
- Package zip rebuilt with steam_api64.dll (590 KB). `trial.json`, `local\` and `steam_appid.txt` are left out,
  as before. The previous zip is in `outputs\steam-build\`.

## 2026-10-17: first real two-PC session over Steam, follow-up (Claude session)

Owner report (two PCs, two Steam accounts, Steam transport, The Outlaws, about 19 min): it connected and both
played. Move/attack and other basic commands work for the guest. Issues they reported:
- the guest's drag box isn't drawn;
- the guest doesn't see P1's resources (but can spend them);
- the guest's pings don't reach the host (the host's pings reach the guest);
- the guest feels some lag compared with normal multiplayer; the host feels none;
- Ctrl+Alt+F shows 0 ping on both sides;
- when the yellow ally (rebels) was rescued: a large lag burst, after which the guest could no longer select units.

Observed from the host's files:
- Host launcher.log: Steam friend connected, joined P1, READY, then 00:20:26 SC2 exit 0. The tunnel carried
  402 KB sent / 397 KB received (≈350 B/s). Game UDP does go through the tunnel on two PCs.
- The host's diagnostics bank at the end shows `guest_control=1`, `guest_spending=1`, `rebel_objective_state=2`. The
  P1→P6 control/spend alliance survived the rescue, so the map did not revoke control.
- No `.SC2Save` was written during the session. The stock rescue trigger calls
  `libCamp_gf_CreateMissionAutosavePoint` → `GameSaveCreate`, but only when `libWoLC_gf_CurrentMap()` is set
  (checked in a LibCamp/LibWCMI copy, not the shipped mod), and no save appeared. Unknown whether anything else
  hitched.
- Product mode writes campaign banks (`TCampaign`, `THorner01`, `TRaynor02`) to the real
  `Documents\StarCraft II\Banks` at mission start. Noted; not investigated.

Hypothesis tested and rejected: "a game stall makes a bridge API request time out after 10 s, and cancelling a
ClientWebSocket receive aborts the socket, so all later bridge clicks fail". I froze the host SC2 process for 25 s
(NtSuspendProcess) in local mode; the guest's bridge action mid-freeze still returned ok in 15 ms. The guest's
API acknowledges actions without waiting for the simulation. The guest's launcher.log (bridge results) is needed.

Defender: the rebuilt test exe `SC2Coop-repro.exe` was quarantined as `Trojan:Win32/Prowloc.A!cl` (a cloud
machine-learning verdict) at 00:29, while it ran. Earlier `sc2_isolated_launcher.exe` got `Bearfoos.A!ml`. The
shipped SC2Coop.exe builds weren't flagged on this PC so far. This is a distribution risk (unsigned exe with
low-level hooks). No exclusion was added.

Latency measurement (implemented):
- `SteamLink` sends a 9-byte ping on channel 1 every second, and the peer echoes it. Both launchers log the
  round-trip avg/median/min/max every 15 s and for the whole session, plus Steam's own estimate from
  `GetSessionConnectionInfo` (optional export; status ping at offset 4 and quality at 8/12; offsets derived,
  not checked on real Steam).
- `timeBeginPeriod(1)` at Steam init: the pump polls with Sleep(1), which at the default 15.6 ms timer adds ~8 ms
  per direction.
- Ctrl+Alt+F shows 0 because each SC2 talks to the launcher on 127.0.0.1; the real path is the tunnel.
- Mock checks (`outputs/steam-mock-test/lat*-*.out`): with 40 ms injected one way, the measured round trip was avg
  87 ms (median 87, 89 samples) and the Steam estimate field read back 80 ms. With 0 ms injected: avg 4 ms,
  median 3 ms. So the launchers add ≈3–4 ms round trip. Bridge and game join still pass.
- Derived: the guest's command delay ≈ relay round trip + SC2's own turn delay. The host's commands don't cross
  the network before taking effect locally, which explains host-no-lag / guest-lag. If the measured round trip
  is high or has spikes, send game UDP frames unreliably on their own channel (no head-of-line blocking).
  Not done yet.

Package: SC2Coop.exe rebuilt with the latency log (sha256 57c0cb12...), zip rebuilt (592 KB).

### Root cause of "guest lost control after the rebel rescue" (2026-10-17, observed)

- Guest launcher.log (the friend's, from the session): bridge actions were ok until 03:17:31. From 03:17:43 every action
  got `SC2 API: Game has already ended` (237 accepted, 172 rejected). The Steam link stayed up until the host quit
  at 03:20:25. The guest's SC2 exited -1.
- Local repro, no network (copy install, local two-window mode, hook `SC2_COOP_TEST_RESCUE`: show_map/debug-spawn
  4 P1 marines at the rescue area 48.4,23.6): about 5–10 s after the rescue, the API status of BOTH clients went
  3 → 5 (ended). The simulation kept running (loop 1228 → 2343). `player_result` was all 1 (victory) for P1..P6.
  The mission's own `gv_gameOver` stayed 0.
- Killing all rebels instead (hook `SC2_COOP_TEST_KILL_PLAYER=4`) gave the same, with results all 2 (defeat).
- A hidden placeholder unit for P4 (created on last death or owner change) was created (`keepers_created=1`) but did
  not help → it isn't elimination. Removed again.
- Skipping `ObjectiveSetState(gv_objectiveRebels, completed/failed)` → the API game stays in progress, and 10/10
  bridge actions after the rescue were ok.
- Replacing the objective with `ObjectiveCreate(..., c_objectiveStateCompleted, false)` → ended again.
- Conclusion (observed): in this API game, any objective reaching completed or failed ends the API game for all
  clients. The host doesn't notice (native input); the guest's bridge dies. The "lag burst" is unexplained (it may
  come from the API game-end processing; unknown).

Fix (`campaign_shared_control.galaxy`, `TrialPreparation.BuildMap`): the two `ObjectiveSetState(gv_objectiveRebels, …)`
calls go through `CoopSecondaryObjectiveState`. It keeps the objective active and appends " (Completed)" /
" (Failed)" to its name (`ObjectiveSetName`); diagnostics count the calls. The primary objective is untouched: its
completion is the mission victory, where the game ends anyway (whether the guest notices anything odd during the
victory sequence is untested).
- Verified (local, final map e787d2fc..., no keeper): rescue path 10/10 bridge actions ok after the rescue, API
  status stays 3, no player results. Rebels-killed path: the same. Prepare from the real install (product mode,
  scratch folder) produces the same map SHA-256.
- Not verified: how the renamed objective looks in the objective panel (the windows were hidden); a real two-PC run.
- `Settings.Recipe` + `TrialPreparation.Recipe()` (RecipeVersion 2 + hash of the three adapter .galaxy files): an old
  trial.json now fails Validate with "This launcher version changes the mission map; click Prepare again"
  (checked against the old steam-mock-test config).
- Package: SC2Coop.exe sha256 db89d5a4..., updated adapter files, README "Updating" + known limitation; zip 596 KB.
  The previous files are in outputs/steam-build/*before-objective-fix*.

## 2026-10-17: second two-PC Steam session and follow-up (Claude session)

Owner report: the rescue no longer broke anything. The mission victory gave a huge lag burst and an apparent desync.
Both players felt about 200 ms on unit moves (each hosting once), while builds and selection felt instant. Guest
point-selects sometimes failed (box select always worked). The guest's keyboard camera scrolling doesn't work.
In-game "End Game" froze (Alt+F4 needed). The host had to close a Steam invite window before seeing the game.

Observed in the logs (host + guest): the Steam round trip was steady, avg 74 ms (host: 463 samples, max 88;
guest max 294 once during load). The guest bridge accepted 398 actions and rejected 42. The rejects were point
selects with result 3, plus "Game has already ended" from 04:47:10, which is the victory.

Changes, all checked in local two-window runs on the copy install (map 05507cdd...):
- Objectives: all 19 `ObjectiveGetState/SetState` calls in the stock script are rewritten to
  `CoopObjectiveGetState/SetState` (BuildMap checks the count). Completed/failed are recorded, shown as a name
  suffix, and returned by reads; the engine state stays active. This replaces the rebel-only fix, which had a side
  effect: stock reads (colonist dialogue, line 1695, the victory check at 4295) would have seen "active".
  Rescue run: `rebel_objective_state=2` (the coop read), 10/10 bridge actions ok. Victory run (debug-kill all P2):
  `game_over=1`, main objective 2, API status stayed 3 (no API end). After victory the guest's right-click returns
  2 (the victory sequence). Not seen on screen: the renamed objective text, the victory screen.
- End Game: the main loop checks every game's API status every 2 s. Status other than 3 → close SC2. On the host,
  status 5 with the game loop still advancing is kept open, with a warning. Test: the guest left via API
  RequestLeaveGame (stand-in for the menu) → "has left the match (status 1); closing" within 1 s, both exit 0.
  Not tested: the real menu End Game (needs UI input).
- Point select: result 3 is retried at 12 offsets (±2/±4 feature px). Test: 2 of 5 points hit via an offset
  ("hit at 223,126"); 2 empty points still 3.
- Command delay (local mode, no network, `SC2_COOP_TEST_DELAY`): from request to orders visible in the host
  observation, host raw move 109–160 ms (4 loops), guest feature-layer move 92–104 ms (2–3 loops). Derived: SC2's
  own lockstep delay ≈ 0.1–0.15 s, plus the ~74 ms network round trip ≈ the owner's ~200 ms estimate. The
  launcher's share is ~5 ms (mock). Unknown: whether SC2 exposes a lower command delay for LAN/API games.
- Steam: `ActivateGameOverlayInviteDialog` is no longer called when hosting.
- Mock Steam regression on the final build: lobby, friend join, P1/P6, bridge ok, latency 87 ms at 35 ms injected each way.
- RecipeVersion 3 → both players must Prepare again. Package exe/adapters/README updated; zip 598 KB; the previous
  files are in outputs/steam-build/*before-session3.
Open: guest keyboard camera scrolling (native camera keys are dropped for the guest like world clicks; likely needs a
map trigger on key events plus CameraPan for player 6, untested). Drag box, resources and pings are in a separate
session.
- Correction 02:20: the 02:16 package build picked up another session's in-progress guest-UI edits in work\ (its
  Prepare would have failed: missing campaign_guest_ui.galaxy). The package was rebuilt from that session's
  `*.before-guest-ui` snapshots (02:15:45, which contain all changes above) in a scratch folder. Check: the
  unzipped zip's own SC2Coop.exe ran `prepare-product` → Recipe 3-87c2499b71077e6c, map 05507cdd... (the tested
  map). The broken exe is kept as outputs/steam-build/SC2Coop.exe.broken-0216.

## 2026-10-17: guest drag box, P1 resources and pings for the guest (Claude side session)

Code (snapshots of the previous state: `work/*.before-guest-ui`):
- `work/ClickBridge.cs`
  - Drag box: the hook thread owns a `WS_POPUP` window with
    `WS_EX_LAYERED|WS_EX_TRANSPARENT|WS_EX_NOACTIVATE|WS_EX_TOOLWINDOW|WS_EX_TOPMOST` and colour key magenta
    (`LWA_COLORKEY`). It also answers `WM_NCHITTEST` with HTTRANSPARENT and `WM_MOUSEACTIVATE` with MA_NOACTIVATE. While the left
    button is held after a press in the world area, each mouse move positions it over the dragged rectangle, using the same
    clamp to the world area as the release. It paints a 2 px green frame and is hidden on any left release. It isn't
    shown while the map reports targeting. The hook thread's message loop now dispatches messages. The hook itself still
    returns `CallNextHookEx` for every event.
  - Ping: Alt + left press in the world area → API `action_chat` (Action field 6, channel Broadcast) with the text
    `coop:ping <fx> <fy> <resolution> <feature width>` (feature-layer screen point). It doesn't start a selection.
- `work/campaign_guest_ui.galaxy` (new, appended by `BuildMap`; `CoopGuestUiInit()` runs after
  `CoopSharedControlUiInit()`; in `Recipe()`, `RecipeVersion` 4):
  - Resources: player 6's native resource panel is hidden (`UISetFrameVisible(..., c_syncFrameTypeResourcePanel, false)`)
    and a label dialog anchored top right (visible to player 6 only) mirrors player 1's minerals, vespene and supply
    used / min(made, limit) every 0.25 game s, with the Core icons. It is hidden (and the native panel restored) while
    `gv_inCinematic`. Player 6 gets no resources of its own.
  - Ping: `TriggerAddEventChatMessage(6, "coop:ping", partial)` → world point = `CameraGetTarget(6)` +
    ((fx/res − 0.5)·width, −(fy/res − 0.5)·width). Then `PingCreate(all, "PingSimple")` + `MinimapPing` (minimap),
    a "MinimapPing" model (`Assets\UI\Feedback\Ping\Ping.m3`) on the ground for 4 game s, the `UI_MiniMapPing` sound,
    and `UIClearMessages(all, chat)`.
  - Diagnostics bank section `GuestUi`: `pings`, `last_ping` (point + guest camera), `p1_minerals`, `p6_minerals`.
- `TrialLauncher.cs`: test hook `SC2_COOP_TEST_GUEST_UI="ax,ay,bx,by;fx,fy[;s]"`. After s seconds it shows the drag box
  over that guest-client rectangle for 15 s (via the overlay's own window procedure) and sends the Alt+click ping three
  times, 5 s apart. No mouse input is used.

Why chat for pings (route chosen): the API has no ping action, and the guest can't observe (slot-6 RequestObservation
crash), so the bridge doesn't know the camera. A chat action is the only free-form guest→map channel without side
effects on units (a Move probe needs a selection and touches orders; banks are local and unsynced). The map knows
the guest's camera target, and the feature layer is a top-down square of `FeatureWidth` world units centred on it.
Cost: `UIClearMessages` also clears real chat lines that are still visible when a guest pings.

Observed (local two-window mode, copy install, test folder `outputs/guest-ui-test`, map 6d37965b..., run 6; screenshots
`outputs/guest-ui-test/shots6/`):
- Drag box: green rectangle at guest client 420,180–760,420 (t066-win2.png). Window ex-style read back 0x080800a8.
  The foreground window was never the overlay. Hidden again after 15 s.
- Resources: the guest's top bar shows 100 / 0 / 14/19, the same as the host's native panel (t066-win1/win2). Bank:
  `p1_minerals=100`, `p6_minerals=0`.
- Pings: 3/3 sent and handled by the map (`pings=3` in both clients' banks). The ping marker appears at the expected
  spot on both host and guest (world 89.9,32.2 = the guest client point ~930,189 from the calibration), plus the
  cyan minimap ping on both minimaps (`ping-zoom.png`; not present at t064, present at t066/067/071). No chat line
  was visible in those shots.
- Existing bridge commands in the same run: `test rect 205,111-241,146 -> ok`, `test smart 220,183 -> ok`. Both clients
  exited 0 and the normal Documents profile was unchanged (all 6 runs).
- The stock tutorial Help panel was closed for the screenshots with Esc posted (PostMessage, no SendInput, no focus
  change) to both test SC2 windows (`shots.ps1`). Screenshots use PrintWindow on the SC2 windows only (an early
  full-screen grab caught the owner's desktop and was deleted). The overlay is composited onto the guest shot at its real
  screen position. That is a composite, not one screen grab; the overlay's position, visibility and styles were read
  from Windows.
- Gotcha: a Galaxy local named `marker` (a built-in type) makes the whole map script fail to compile. The game still
  starts with no triggers: the guest sees black, P1 has 0 minerals, no banks are written, bridge smart returns result 2,
  and the chat line stays (run 5). There's no error message; check that `CoopCampaignProbe.SC2Bank` exists after a map change.

Derived:
- The resource display reads player 1's properties on every client (synced state), so no desync risk; only the
  label text differs per player.
- Ping accuracy follows the bridge's flat-ground calibration (~0.2 world units mean error at the calibrated size).
  Ping point = guest camera target at the game loop the chat is processed; a camera moving fast while pinging shifts it.

Unknown / not tested:
- A real mouse drag and a real Alt+click on the guest (needs SendInput; the owner was using the PC). The overlay path
  and the ping path after the hook were exercised by the test hook, not by the hook's own WM_MOUSEMOVE/LBUTTONDOWN.
- Exclusive fullscreen: a topmost overlay can't draw over an exclusive-fullscreen SC2 (windowed / windowed fullscreen
  should work). The friend's display mode is unknown.
- Whether a chat line flashes for a frame before `UIClearMessages` on a slower PC or over Steam latency.
- Whether native Alt+click on the guest's minimap should also ping (not bridged; minimap clicks pass through natively).
- Two PCs over Steam.
- `outputs/sc2-coop-package` was not touched by this session.

### Guest keyboard camera scrolling (arrow keys), 2026-10-17 (same side session)

Owner report: player 2's arrow-key camera scrolling doesn't work; hotkeys do.

Observed, control (run 7, map without the fix): the test posts Right arrow (WM_KEYDOWN repeated every 30 ms for 2 s,
then WM_KEYUP, via PostMessage; no SendInput, no focus change) to each SC2 window. The host camera scrolls far
right; the guest camera doesn't move (`shots7/grid.png`). So posted key messages are a usable test input, and the
guest's native camera keys are dropped, like its world clicks.

Fix (map side, `campaign_guest_ui.galaxy` `CoopCamInit`): `TriggerAddEventKeyPressed(6, Left/Up/Right/Down, down/up,
modifiers ignored)` tracks held arrows. A 0.0625 game-s periodic trigger pans player 6's camera
(`CameraPan(6, p, 0.0625, …)`) at 18 world units per game second, clamped to `RegionPlayableMap()`. The position is
integrated in the map from `CameraGetTarget(6)` read when scrolling starts. It's paused while `gv_inCinematic`. Header var
`coopGuestCamKeys`; diagnostics `GuestUi/cam_keys` and `GuestUi/guest_camera`. Still RecipeVersion 4 (not yet packaged).

Observed with the fix (run 8, map ad326e16...): the same posted Right arrow scrolls the guest camera about as far as the
host's native scroll (`shots8/grid.png`). Bank `cam_keys=1` (key repeats don't re-fire the down event),
`guest_camera` 82.6 → 120.0. In the same run: bridge `test rect -> ok`, `test smart -> ok`, both exit 0, profile unchanged.

Derived: key events are game input, so on two PCs guest scrolling starts and stops one network round trip late
(the host's native scroll is local and instant). If that feels bad, a bridge-side route (keyboard hook → API
camera_move) would avoid the trigger round trip, but the bridge doesn't know the camera position (no guest observation).

Unknown / not tested: a physical keyboard (same WM_KEYDOWN path is expected); mouse edge scrolling and middle-drag
scrolling for the guest (not checked whether they are dropped too); custom camera-key bindings (only the arrow keys are
handled); smoothness over Steam latency.

### Package with guest UI, 2026-10-17 02:50 (main Claude session)
- Built from the merged work\ after the guest-UI session finished (RecipeVersion 4). Checked that the merged sources
  still have the objective layer, End Game watcher, select retry, Recipe check, no invite overlay, and timer resolution.
- Local rescue test on the merged map (ad326e16..., the same map as the guest-UI session's run 8): objective read 2,
  10/10 bridge actions ok after the rescue, API status 3, `GuestUi` bank written (p1_minerals=100).
- Package: SC2Coop.exe sha256 ea3b5b48..., + campaign_guest_ui.galaxy, README (drag box, Alt+click ping, arrow keys,
  resources). Zip 604 KB. Unzipped exe's prepare-product → Recipe 4-1ab58ed6a106ef69, map ad326e16... Previous
  files: outputs/steam-build/*before-guest-ui-package.

## 2026-10-18: where the move delay comes from; unreliable game UDP over Steam (main Claude session)

Owner question: arcade games at the same ping (70–80 ms) feel instant; ours feel ~200 ms on moves.

Observed (copy install, `SC2_COOP_TEST_DELAY`, request → orders visible in the host's own raw observation):
- Single-player API game (solo vs computer; the tutorial Help panel that pauses single player was closed with posted
  Esc, `outputs/keeper-test/post-esc.ps1`): 17–55 ms, 0–1 game loop, 12 samples.
- Two-player LAN API game on one PC (no network): 87–170 ms, 2–4 loops (this run and the 2026-10-17 run).
- Real Steam path: ~74 ms round trip (logs); the launcher adds ~3–5 ms (mock).
Derived: SC2's multiplayer lockstep in API/LAN games adds about 2–3 loops (~90–130 ms) by itself; plus the network
round trip, that is about the ~200 ms the owner felt. Not the tunnel, not Blizzard's servers. Unknown: why
Battle.net/arcade feels faster (different turn scheduling for Battle.net games?), and whether any setting changes
LAN command delay. A web search found none for SC2 (only Brood War/Remastered turn-rate settings, and an informal
TL.net report that the original StarCraft's LAN mode also adds ~200 ms).

Defender quarantined the rebuilt console test exe again (keeper-test/SC2Coop-console.exe, 2026-10-18 03:39:52,
Trojan:Win32/Prowloc.A!cl). That's the second build of the launcher code flagged. The windowed package exe wasn't
flagged on this PC so far. No exclusion was added.

Unreliable game UDP (owner asked to try it):
- `Tunnel.IDatagramLink`; `SteamLink` sends game UDP on channel 2 with Unreliable|NoNagle (flag 1). The tunnel frames
  stay reliable on channel 0, control on 1. Hello field `dgram:1`; it's used only when both sides offer it, otherwise
  the launcher logs "Game UDP packets go reliably over Steam" (older peer). `SC2_COOP_NO_DGRAM=1` turns it off.
- Mock: messages carry the send flags. `MOCK_STEAM_LOSS_PCT` drops unreliable messages, and makes a lost reliable one
  arrive 2×delay+30 ms late, holding up later messages on its channel (head-of-line, like real reliable delivery).
- `steam-udp-test host|join <base> <s>` (no SC2): 50 numbered 200-byte packets/s through the tunnel's game-UDP path,
  echoed. Mock, 35 ms each way (`outputs/udp-test/udp-matrix.ps1`):
  - loss 0%: reliable and unreliable both median 87 ms, max 96.
  - loss 2%, reliable (old): median 89 ms, 95th 241, 99th 276, max 282 (head-of-line stalls).
  - loss 2%, unreliable (new): median 87, 95th 94, max 96; 48 of 972 echoes lost (SC2's own protocol handles loss).
- Full mock-Steam game regression (keeper-test, map ad326e16): both sides "go unreliably", P1/P6, bridge ok,
  marines moved, exit 0. 6/2 game UDP packets crossed the datagram path.
Derived: with ~0% loss (the owner's sessions: quality 99–100%) the steady feel won't change. It removes the spikes on a
lossy connection. Real two-PC effect: unknown.
Package: SC2Coop.exe sha256 aa21a771..., README note, zip 606 KB. No map change (Recipe 4, no new Prepare needed).

## 2026-10-18: Hyperion hub (tstory01) assessment, static only (Claude session)

Sources read: vanilla `Campaigns/Liberty.SC2Campaign/.../TStory01.SC2Map` + `LibertyStory.SC2Campaign` libs from
CASC (`work/casc_hub.py`, read-only, private output), and the owner's WoL@Kit campaign copy
(`<campaign dir>\tstory01.SC2Map`, "WoL@Kit" 1.1.2 by Kit) + `Mods\kit_liberty_story.SC2Mod`.

Observed, vanilla:
- MapScript 41,316 lines; deps Liberty mod/campaign + LibertyStory; the room "StorySections" are editor-only (no
  CASC files), so all rooms live in one map. The campaign logic is `campaignlib.galaxy` (14.6k lines, libCamp).
- Map transitions: hub -> mission = `libCamp_gf_PrepareForMission` -> `TS_SaveAllTransitionData` (TCampaign +
  per-mission banks, player 1) -> `GameSetNextMap("Campaign/<file>")` -> `GameOver(1, victory)`. Mission -> hub is the same with
  next map TStory01. The star map's mission pick is in the library (line ~3403); the chosen mission is NOT written to
  any bank. The hub registers 29 briefing triggers via `libCamp_gf_SetMissionBriefing(mapID, gt_Briefing...)`, and the
  library runs the picked mission's briefing trigger just before launching, so an adapter can wrap them to record the pick.
- State: credits are `ConversationDataStateGetValue("Credits")` (saved into the bank via `ConversationDataSaveStateValues`);
  the TCampaign bank sections include TechStatus, MercStatus, LabResearchState, StoryStage, Mission*Age, ObjectiveProgress,
  Conversation. The owner's real TCampaign.SC2Bank (written 2026-10-17 02:01, likely by the co-op Outlaws runs) has 20 sections.
- Story UI: `libNtve_gf_StoryMode(PlayerGroupAll(), true)`; the native `StoryMode` is declared "Blizzard maps only".
  532 player-1 camera calls, UISetMode letterbox/fullscreen.

Observed, the WoL@Kit campaign and Kit:
- Hub MapScript 40,668 lines; single dep `kit_liberty_story.SC2Mod` (36 MB), which depends on `Void.SC2Campaign`
  (LotV). Libraries: LibWoLC (8k), LibWCMI (6.4k), LibWCUI (3k), LibCamp (1.2k), plus Galaxy UI panels
  (`travel_panel`, `tech_purchase_panel`, `research_panel`, `mercenary_panel`, ...) built from DialogCreate templates.
- Banks WArmy/WCampaign/WStory/WArchive/TStoryTransition01/PStoryTransition etc. Credits = player property
  `c_playerPropCredits` set from `libWoLC_gf_CreditsAvailable()`. Transitions via `libWoLC_gf_PS_SaveAllTransitionData`
  + `GameSetNextMap("Campaign/"+libWoLC_gf_WC_CampaignMapFile(next))`. Also uses `libNtve_gf_StoryMode` (7 calls).

Derived:
- All persistent campaign state crosses maps as player-1 bank files, so a launcher can copy, diff and merge it.
- A "two solo hubs" design needs no lockstep, no click bridge and no guest slot: each player runs a solo API game
  (creator native input works with raw=true). On the hub's GameOver the launcher learns the pick (Kit: one-line edit in
  editable lib; vanilla: wrap the 29 briefing triggers) and both launchers start the shared mission session.
- Purchases: merge the guest's bank delta into the host's bank at departure (easy-medium, schema-specific per framework),
  or live sync via adapter hooks (bank writes out, API chat commands in, as Archipelago does) (harder; Kit easier since
  its UI is its own code).

Unknown (needs a short runtime test):
1. Does a hub load and run as an API game (StoryMode, story UI, campaign libs), unmodified and modified?
2. What does API mode do on `GameOver` after `GameSetNextMap`: just end, or load the next map?
3. In a 2-participant mission, do clients share player 1's banks (host's TCampaign) so the state is deterministic?
4. Does Kit's dependency on Void.SC2Campaign require owning LotV?

## 2026-10-18: Hyperion hub runtime tests (Claude session)

All in Codex's isolated copy (patched exe + disposable profiles, fresh campaign state). Drivers:
`work/hub_test.py` (solo API session of a hub), `work/bankshare_test.py`, `work/pack_folder.py` (component folder -> SC2Map).

Observed:
1. Hubs run as API games.
   - The API refuses component-folder maps ("file doesn't exist"); a packed .SC2Map works.
   - Vanilla TStory01, repacked (i.e. not a Blizzard-signed file): intro cinematic, then Mission Selection
     (Liberation Day, difficulty, Launch) with story UI working. `StoryMode` ("Blizzard maps only") did not block it.
   - WoL@Kit campaign hub (packed from the owner's folder; kit_liberty_story mod identical in the copy): intro, then
     Kit's LotV-style mission screen (Start Mission/Back).
   - `RequestGameInfo` right after join crashed SC2 on the vanilla hub (null read). Avoid game_info on hubs; observation is fine.
   - The WoL@Kit campaign hub opens SC2's Trigger Debugger window via `TriggerDebugWindowOpen(true)` (hub MapScript line
     14867; Kit libs 5070, 7358). An adapter should strip it.
   - On a fresh campaign, Close/Back do nothing: the first mission is forced.
2. Launch / Start Mission: the briefing plays, then the API reports player 1 Victory and status ended (5). The client
   does NOT load the next map; ~7-20 s later it crashes (exit 536936448) with DataSource `Maps/Campaign/TRaynor01.SC2Map`,
   i.e. while following GameSetNextMap. So the launcher should quit the client as soon as a hub game ends.
   - Banks written at launch. Vanilla: `TCampaign` (ScenePlayed, StoryMisc, MissionAvailableAge, StoryStage, ...).
     Kit: `WArchive`, `WArmy`, `WCampaign` with named keys (CampaignLastInfo/LastMap=TStory01, Perks, Scenes,
     MissionAvailableAge/TRaynor01, CampaignSetting/Difficulty, MissionBestTime ...).
   - Neither records the picked mission, so both need the hook (vanilla: briefing-trigger wrappers; Kit: library edit).
3. Bank sharing in a 2-participant mission (probe copy of the co-op Outlaws map; only the host profile had a
   TCampaign bank with CoopTest/who=HOST-SEEDED): the host read HOST-SEEDED, and the guest (P6, empty profile) also read
   HOST-SEEDED (has_key 1, sections 1). Player-1 banks are transmitted to the other client at game start.
4. LotV ownership for Kit's Void.SC2Campaign dependency: not testable here (the owner owns all campaigns).

## 2026-10-18: bare-bones campaign hub flow, WoL@Kit (Claude session)

Code: `work/Campaign.cs` (new) + small changes in `Tunnel.cs` (control frame type 5, `StartReading`, `Open`
only binds once, `BasePort`) and `TrialLauncher.cs` (`Session.EstablishLink` extracted from `Start`,
`PresetTunnel`/`PresetRoots`, CLI `campaign` and `campaign-build`, GUI "Campaign hub" row + folder). Backups:
`Tunnel.cs.before-hub`, `TrialLauncher.cs.before-hub`. Test build: `outputs/sc2-local-trial/SC2CoopTrial-campaign.exe`.
The release package (outputs/sc2-coop-package) is NOT rebuilt yet.

Pieces:
- `MapArchive`: read-only MPQ (zlib + a new C# bzip2 decoder; editor maps are mostly bzip2) or component folder.
  Verified byte-exact against mpq.py on Kit traynor01 (45 untouched files identical).
- `MapInfoPlayers.AddGuest`: generic port of mapinfo_codec; clones player 1's record into the lowest free slot >= 6.
  Kit Liberation Day uses slots 0-8 (6 is a computer), so the guest is player 9.
- `BuildKitHub`: wraps the hub's `libWoLC_gf_LaunchMission(...)` calls (3 in the WoL@Kit campaign's TStory01) so the pick
  (mission, difficulty, seq) lands in a `CoopHub` bank, then launches as before.
- `BuildMission`: generic co-op adapters for any WoL-style mission script: objective reroute, guest slot substituted
  into the adapters, `PlayerSetDifficulty(1, pick)`, alliance mirroring every 1 s, guest UI and camera at tick 2,
  `gv_inCinematic` declared if missing, CoopBridge bank for the guest.
- `CampaignRun`: one tunnel per sitting; each side runs its own solo hub (persistent profile folders under
  local/campaign/profile-host|guest); a pick ends that hub and the launcher quits it at once. The picker's freshly saved
  campaign banks go to the host (the mission runs on player-1 banks); the host sends "go"; both build the mission and
  compare SHA-256; the mission runs via Session with the preset tunnel; when either side's mission ends the other's
  is stopped ("mission-over"); the host's banks go to the guest (the guest's originals are backed up once); the hubs reopen.
  A hub closed without a pick ends the session for both ("quit"). Campaign bank set = every player-1 bank in the
  campaign's BankLists (12 for the WoL@Kit campaign, incl. TCampaign and the campaign's own bank), Coop* excluded.
- The trigger debugger window (opened by Kit's library) is hidden by the launcher.

Observed (one PC, isolated copy, fresh campaign so Liberation Day is forced):
- Local mode: guest-hub pick -> both hubs closed -> mission built (guest P9) -> two-window session, guest bridge box
  select accepted -> test-hook leave -> hubs reopened.
- Two launchers over the tunnel (guest base 7500): host-hub pick -> guest got "go" and closed its hub -> identical
  mission hashes -> P1 + P9 joined -> host left (test hook) -> guest's mission closed by "mission-over" -> host sent 3 banks
  (WArchive/WArmy/WCampaign) -> guest applied them -> both hubs reopened. A guest hub closed without a pick ended both.
- Not tested: playing a mission to victory and the resulting bank changes, a second mission, Steam, two real PCs,
  product mode (it would overwrite the buddy's real banks of the same names, after a backup), the vanilla hub.
- Known gap: Kit missions set objectives through the campaign library, not the map script, so the objective reroute
  can't catch them and the API "game over" lockout of the guest bridge may still occur.
- 2026-10-18 18:40: release rebuilt with campaign mode (`work/build-launcher.ps1` now includes Campaign.cs).
  `outputs/SC2Coop-Outlaws-test.zip` (604 KB; the previous one is kept as `SC2Coop-Outlaws-test.before-campaign.zip`,
  the previous exe as `outputs/SC2Coop.exe.before-campaign`). In the package folder, product-mode `prepare-product` from the
  owner's real install and `campaign-build` against the WoL@Kit campaign folder both succeeded (hub + TRaynor01 built).
  README gained a "Campaign hub" section. Not yet run as a real two-PC session.

## 2026-10-19: campaign + Steam fixes after the first real attempt (Claude session)

Owner report: in campaign mode over Steam, Stop did nothing on either side, and the friend's Join (friend picked
in the list) never reached the host; the friend's log said "Waiting for a Steam invite".
Causes (Campaign.cs): CampaignRun called `Session.EstablishLink` with lobby 0. The lobby-id parsing lives in
`Session.Start`, before the call, so the joiner waited for an invite. The campaign Stop event wasn't connected to the
link session's or the mission sessions' `Stop`.
Fix: parse the lobby id in CampaignRun; register the campaign stop onto `link.Stop` and each mission
`session.Stop`; Stop during the hub stage sends "quit". Test hook `SC2_COOP_TEST_STOP=<s>`.
Verified with the mock Steam DLL (`outputs/campaign-steam-test/`, copy-mode config):
- Steam host waiting alone, Stop at 15 s: ended immediately ("Cancelled").
- Host + joiner given the lobby id (the GUI's friend pick): "Joined MockUser1's Steam lobby", tunnel up, banks synced,
  both hubs open; host Stop at 100 s: both launchers ended without errors, no SC2 processes left.
Release zip rebuilt 2026-10-19 02:10 (SC2Coop.exe 172,544 bytes, real steam_api64.dll).

## 2026-10-19: first real two-PC campaign run over Steam (owner + buddy) and fixes (Claude session)

Observed (owner report, launcher logs, screenshots): Steam lobby join, tunnel (RTT ~72-80 ms), bank sync, both hubs,
pick sync ("go"), identical mission hashes all worked. Problems:
1. Both screens flickered black, and a "second SC2 window" kept opening/closing: the launcher hid the Trigger Debugging
   window every 500 ms and the game showed it again (hide/show fight). It stopped once the hubs closed.
2. Buddy: `JoinGame failed: 10` = CannotOpenMap (ResponseJoinGame.Error). The hub (deps: kit_liberty_story only) worked;
   the mission also needs `<campaign mod>`, which depends on its asset, sound and test mods (dependency tree
   read from DocumentInfo). Most likely missing from the buddy's SC2 Mods folder (not yet confirmed).
3. Then the host sat in its 3-minute JoinGame wait (not cancellable via the API), the buddy gave up waiting for banks
   after 2 minutes, and the session ended with errors.
Fixes: minimize the debugger window once per handle; `CampaignMaps.CheckDependencies` (recursive DocumentInfo
`file:Mods\...` check; Blizzard `bnet:.../999` skipped) runs before building the hub and each mission and names missing
mods; a buddy's mission build error is shown on the host; on mission-over, not-yet-joined SC2 clients are killed; the
bank wait is 5 minutes. Tests: dependency check positive (copy install) and negative (fake install with only the Kit mod:
"Mission TRaynor01 needs this mod in ...\Mods: <campaign mod>"); mock-Steam pair regression (hubs open, debugger
minimized once, Stop ends both with "The host ended the campaign session"). The join-abort path was not exercised.
Release rebuilt 02:27 (SC2Coop.exe 174,592 bytes; the README lists the 5 campaign mods).

## 2026-10-19 (cont.): second real two-PC run, then fixes (Claude session)

Owner report: the full loop worked over Steam (hub -> Start Mission -> Liberation Day for both). Issues: (a) the buddy's
campaign folder wasn't remembered (he unzips each update into a new folder); (b) the mission "loaded twice": the hub
client began loading the next map after its GameOver, before the launcher closed it; (c) after End Game both hubs
opened, then closed again and the mission restarted.
Causes and fixes:
- (c) Both players had pressed Start Mission. The host acted on its own pick, and the buddy's "pick" + banks message
  stayed in the inbox and was taken as a new pick at the next hub stage (it could also have applied stale banks). Fix:
  a hub round counter, advanced together on both sides; pick/go/quit/ready/banks messages carry it; only current-round
  messages are taken; older ones are dropped at each hub stage.
- (b) The hub wrapper now calls `GameSetNextMap("")` after `libWoLC_gf_LaunchMission`. A first attempt emitted
  `GameSetNextMap(");` (C# verbatim-string `""` escaping), which broke the whole hub script (black world, plain
  UI); caught in the solo test before release. With the fix: a solo hub's mission launch ended the game (Victory),
  the client stayed idle 20 s and exited 0, and there was no crash dump (before: a crash while loading Maps/Campaign/TRaynor01).
- (a) The campaign folder is also kept per Windows user (%LOCALAPPDATA%\SC2Coop\campaign-folder.txt).
- Per-side mission map folders (local/campaign/maps/<host|guest|local>) avoid a one-PC file clash between two launchers.
Verified with the mock-Steam pair, both hubs pressing Start Mission: the buddy's pick won, the host closed its hub,
the mission ran (P1/P9), the test-hook leave ended both missions, banks synced, both hubs reopened and stayed 45 s.
Release rebuilt 2026-10-19 03:22 (SC2Coop.exe 177,664 bytes).
Owner feedback still open: drag box is an outline (vanilla has a translucent fill); arrow-key scrolling lags like
commands (it runs through the map trigger, i.e. lockstep); the Alt+click ping clears the chat line; guest commands
feel 200 ms+ while the host's feel instant (RTT 70-80 ms).

## 2026-10-19: guest input latency in the click bridge (main Claude session)

Prompted by an outside review (ChatGPT) of the guest input path. Only `work/ClickBridge.cs` was changed (snapshot
`ClickBridge.cs.before-latency`). Tests ran in `outputs/latency-test/` with a COPY of the work sources (the campaign-hub
session is editing work\); measurement hooks exist only in that copy's TrialLauncher.cs.

Changes:
- Right-click moves are queued on WM_RBUTTONDOWN (they were sent on release, so every guest move waited for the
  button to come up, typically ~80–120 ms of hold, derived, not measured).
- A 22 px green plus is flashed for 400 ms at the right-click point, in a second click-through topmost overlay window
  (same class as the drag box; WM_TIMER hides it). The guest gets the instant local feedback that the native client gives
  the host. Not shown while the map reports targeting.
- A left click waiting for the targeting report (up to 600 ms after a non-neutral key or command-card press; the
  report comes through the game's network turn) stops waiting as soon as a right-click is queued behind it. Order is
  kept (select then move). After such a cut-short wait, the miss-retry at nearby offsets is skipped (the click may have
  been meant as a target).

Observed (local two-window mode, copy install, map ad326e16, hidden windows):
- Marker check: 150 ms after a flash the overlay is "visible 22x22", 750 ms after it is hidden.
- Cut-short check (key + left click + right-click, 3 times): right-click sent "in 0 ms"; the left click is logged
  "(targeting wait cut short by a right-click)". Before the change, that right-click waited up to 600 ms.
- Same session, same marines, send → orders visible in the host observation: host API move median 108 ms (92–139, 6);
  guest bridge right-click median 118 ms (114–157, 6). The bridge queue and send path add ~10 ms.
- Native host input could NOT be measured without the real mouse: posted WM_LBUTTONDOWN/MOUSEMOVE/UP selected 0/6
  marines, and posted right-clicks produced no orders (5 of 6; the one hit at 761 ms is not trusted). SC2 doesn't take
  posted mouse messages as game input (posted keys do work, see the guest-UI notes).
- Not exercised: the real WM_RBUTTONDOWN hook path with a physical mouse; how the marker looks over the game.

Open: a native-vs-API comparison with SendInput (needs the owner's OK, takes the cursor for about a minute).
Not packaged (owner: wait until the campaign-hub work in work\ is settled).
- Native measurement with the owner's permission (SendInput, visible windows, host window brought to front: True),
  2026-10-19 03:33 and 03:36: INCONCLUSIVE. Run 1: a native host drag at the guest's feature rect (inverse
  calibration) selected 0/6 marines; the host camera differs from the guest's. Run 2: F2 (select army, posted key)
  → host observation still 0/6 selected (Unit field 11 `is_selected`). In both runs 5 of 6 native right-clicks gave no
  orders (1 outlier each, 211 ms / 2573 ms, not trusted). Unknown whether F2 failed or the is_selected read is wrong.
  Valid in the same runs: host API move median 128 / 122 ms, guest bridge right-click median 110 / 114 ms.
  Stopped there (owner's mouse time). A better setup: select the host's marines by a native click at their screen
  position, derived from the host camera (needs the host camera target; raw obs has `is_on_screen`).
- Run 3 (03:40, owner's permission again): host selection through its own API (ActionUI.select_army) → host observation
  still 0/6 `is_selected`; 0/6 native SendInput right-clicks produced orders. Host API move median 146 ms (one 8 ms
  outlier), guest bridge median 106 ms. Since even the API selection reads as 0, either the is_selected read (Unit
  field 11) is wrong, or the host's UI selection isn't what this setup assumes. Unknown which. Stopped. A screenshot of the host
  window (PrintWindow, no input) after select_army would separate the two without taking the mouse.
- Owner note: a native SC2 Help screen was open during the SendInput runs. The posted Esc (post-esc.ps1) evidently
  didn't close it in visible-window mode, so it probably intercepted the native clicks. Close it reliably before any
  future native test.
- Packaged 2026-10-19 03:50 (owner: campaign-hub work settled). work\ was identical to the tested snapshot apart from
  test-only hooks. Regression on the exact work\ build: local rescue test (objective read 2, 10/10 bridge ok, GuestUi
  bank written) and mock Steam pair (unreliable game UDP, P1/P6, bridge ok, marines moved). Package SC2Coop.exe
  sha256 59dceefb..., README line about press-time moves and the marker, zip 623 KB. Unzipped prepare-product →
  Recipe 4-1ab58ed6a106ef69, map ad326e16... (unchanged, no new Prepare needed). The build includes the campaign-hub
  GUI as that work left it.

## 2026-10-19 (later): third two-PC session report; repeat-session overlay bug, Help popup, last mode (main Claude session)

Owner report: movement now feels almost like arcade play. Abilities (attack-move) and arrow-key scrolling are still
laggy. Over three sessions in one launcher run (host, guest, guest), the guest saw white boxes on every click-move
in the third, and Stop didn't end that. The tutorial Help panel pops up ~10–20 s in. Request: remember the last mode.

Root cause (observed): the overlay window class ("SC2CoopDragBox") was registered per ClickBridge, but a window class
can be registered once per process. In the second session of a process, RegisterClass failed ("drag box unavailable:
RegisterClass 0"), and CreateMarker created its window on the class registered by the first, disposed bridge (its
WndProc delegate). Repro (outputs/repeat-test, two local sessions in one process, SC2Coop-old.exe = the 03:21 bridge):
session 2 logged that error and the launcher process then died silently, leaving both test SC2s running.
Fix (ClickBridge.cs, snapshot `.before-overlay-fix`): the class is registered once with a static WndProc that routes by
hwnd to the owning bridge (static dictionary); the hook thread destroys its windows, kills the marker timer and deletes
its brushes; Dispose waits for the hook thread. Check (SC2Coop-new.exe): both sessions show "visible 22x22" 150 ms after
a marker flash and hidden at 750 ms, no RegisterClass error, right-clicks ok, all four SC2 exit 0.
Campaign hub (Campaign.cs) runs several sessions per process, so it hit the same bug.

Help popup: the stock `gt_TutorialReviewQ` calls `HelpPanelDisplayPage(PlayerGroupAll(), c_helpPanelPageTutorials)` on
casual/normal when no order was given yet; BuildMap removes that one line (marker-checked). Tutorials stay in the Help
menu. RecipeVersion 5 → Prepare again. Not checked on screen.
Last mode: `local/last-mode.txt` and `%LOCALAPPDATA%\SC2Coop\last-mode.txt` (mode id, host address), written on Start,
read when the window opens; a Steam invite (+connect_lobby) still wins. Not tested in the GUI.
Package: SC2Coop.exe sha256 a5e1abae..., README (help-window line removed, last-mode line), zip 624 KB. The unzipped
exe's prepare-product → Recipe 5-1ab58ed6a106ef69, map 7bbb26c8... (= the repeat-test map).
Open: ability/targeting latency (bank report through the network turn + Move probe) and arrow-key scroll latency
(map-side camera via key events through the network turn).

## 2026-10-19 (later): guest targeting and arrow-scroll latency (Claude side session)

Changed in work\ (snapshots `*.before-input-latency`): ClickBridge.cs, campaign_targeting.galaxy, campaign_guest_ui.galaxy,
TrialPreparation.cs (RecipeVersion 5 → 6: both players must Prepare again). Not packaged; outputs\sc2-coop-package untouched.
Tests in `outputs/input-latency-test/` (copies of the sources; test-only hooks in TestHooks.cs and the copy's TrialLauncher.cs,
env `SC2_COOP_TEST_INPUTLAT`): local two-window mode, hidden windows, copy install. "before" = the work\ sources as they were
(map 7bbb26c8..., Recipe 5), "after" = the new sources (map 9f0ec37a..., Recipe 6, built under a separate name; the shared
OutlawsCoopProbe.SC2Map was restored unchanged). No SendInput, no focus change: keys are posted to the guest window
and fed to the bridge's key handler; window contents read with PrintWindow.

What changed:
- Targeting is guessed locally. A (attack, standard and grid hotkeys) counts as "targeting on" at once. So do input
  pairs (previous key within 3 s + key, or + command-card cell) that started targeting before, learned from the map's
  report. While targeting is known, a world left click is sent on the press as the queued Move/Patrol probe, with no
  wait for the report and no wait for the button release. A red plus marks the spot.
- Safety net in the map: the probe trigger now takes every player-6 Move/Patrol order without the `c_cmdSmartClick` flag
  (=3, from Core TriggerLibs/GameData/unit.galaxy; also `c_cmdQueued`=1, `c_cmdUser`=8). If no targeting mode was on when
  the probe arrived, the queue is restored and `Probe/stray` counts it. The bridge then unlearns the pair. If the probe
  is refused by the API (e.g. only buildings selected), the click is sent as the plain select instead. A right-click
  after a guess is a cancel only once the map confirmed targeting (A at a barracks trains a marine; the right-click then
  still sets the rally). Re-issued orders inside the probe processing are ignored (`coopProbeBusy`).
- Arrow keys: the bridge scrolls the guest camera itself with ActionRaw camera_move (center_world_space) on the guest's
  own API at ~60 Hz, 25 world units per real second, and swallows the key-downs (key-ups pass). Start position from
  the new `Camera` section of the CoopBridge bank (synced CameraGetTarget(6), cinematic flag, playable bounds; written
  when it changes, at least once per game second). The position counts only when nothing moved the camera for 700 ms.
  The bridge's own moves, minimap clicks, Backspace/Space/F5-F8, a double-tapped control group, middle button, the
  cursor at the window edge, and a map-handled arrow all make it stale. While stale, in a cinematic, with the chat line
  open (Enter toggles, Esc closes), or if raw camera moves are refused 3 times, arrow keys go through to the map's
  existing scroll (CoopCamInit, unchanged = fallback). Decision per press; auto-repeats follow it.

Observed (local two-window mode; times from the bridge input to the effect):
- Attack-move, posted A + bridge key, then a bridge world click; host observation polled for a non-Move order (ability 23).
  Before (12 trials, A→click gaps 30/100/250 ms): click→attack order median 170 ms (104–222). The click waited
  78–125 ms for the map's report at 30 ms gap (185–222 ms total), 16–62 ms at 100 ms, none at 250 ms.
  After (12 trials, same gaps): median 116 ms (87–151); at 30 ms gap 87–151 ms. Bridge log "guessed from A ... via probe
  -> ok in 0 ms". Map: 72 probes seen, 72 attack orders issued, 0 stray, 0 failed. An earlier after-run had 10/12
  (2 trials showed no order in the host observation within 3 s although the map counted the attack issued; not repeated
  in the 12/12 rerun; cause unknown).
- Probe flags (map diagnostics `Probe/flags`): the bridge's Move probe = smart 0, queued 1, user 1. Bridge right-clicks
  (API Smart) never reached the probe trigger (`Probe/seen` stayed 0 over 4 rect+smart tests), so they can't be taken
  back as strays.
- Wrong guess (A fed to the bridge only, no native targeting): probe taken back (stray 6, then 12 = 6 marines each),
  marines didn't move (orders 0, position ±0.1). After that the pair (A,A) was unlearned and the third click took
  the old wait-then-select path.
- Arrow scroll, map path (before, and the after build's fallback after Space): key-down → first moved frame median 138 ms
  (103–151, 8 trials); fallback 139 ms (78–151, 8). Motion continued ~100–300 ms after key-up (map lag).
  Bridge path (after): first moved frame median 0 ms (the first ~12 ms capture), max 34 ms, 24/24 over three runs;
  view at +1200 ms after key-up identical to +50 ms in 8/8. A single raw camera_move showed within one capture
  (6/6), and the map's synced camera reported the new position afterwards.
- Regression on the after build (rescue-test.ps1 from keeper-test): rescue objective read 2, bridge test rect/smart 10/10
  ok, GuestUi bank written, both SC2 exit 0, normal profile unchanged. Every test run above exited 0 with the profile unchanged.

Derived:
- On two PCs, the targeting report arrives later than locally (it rides the network turn: 100–600 ms after the key, owner
  sessions). The old wait was "report time − key→click gap", so the saving from the guess there is larger than the
  ~50–100 ms seen locally. A guessed attack-move now costs the same as a right-click move (one turn to the order).
- Local scroll lag (138 ms) was the lockstep turn; on two PCs the map path adds the round trip. The bridge path doesn't
  depend on either (camera moves are client-local and only synced afterwards).

Unknown / not tested:
- Real hardware input: the WH_KEYBOARD_LL swallow of arrow keys and the press-time target click with a physical
  keyboard and mouse (tests fed the bridge's key handler directly), and how the red marker looks over the game.
- Two PCs: no Steam/mock-Steam run of these paths (local mode only).
- Campaign missions: Campaign.cs appends the same two .galaxy files (guest slot substituted); not run with the new code.
- Learned pairs other than A in a real game (build menus, caster abilities, command-card buttons); the command-card
  cell grid (5×3 over the card area) is an estimate.
- Whether the guest's native edge scrolling works at all (the bridge treats the window edge as "camera may move").
- Chat-line detection is Enter/Esc toggling. If it drifts (Enter in a menu), arrows fall back to the map scroll and
  key guesses pause until Esc.
- Camera bounds: the bridge clamps to the playable map; a mission's tighter camera bounds can give a short dead zone when
  reversing at the edge until the next report.

### Package with ability/scroll changes, 2026-10-19 15:25 (main Claude session)
- Built from work\ after the ability/scroll side session finished (RecipeVersion 6; ClickBridge.cs, campaign_targeting.galaxy,
  campaign_guest_ui.galaxy, TrialPreparation.cs changed). The earlier fixes are still present (overlay class, marker,
  press-time moves, select retry, last mode, Help popup removal, objective layer, End Game watcher, unreliable game UDP).
- Reviewed the new keyboard-hook key swallowing: arrow key-downs are swallowed only while the guest SC2 window is in
  front and the bridge scrolls; key-ups are never swallowed.
- Regression on a copy of the exact sources (outputs/pkg6-test, map 9f0ec37a, copy install): rescue test (objective read 2,
  10/10 bridge actions ok, GuestUi bank), two sessions in one process (marker shown/hidden in both, right-clicks not
  held by the targeting wait, 4× SC2 exit 0), mock Steam pair (unreliable game UDP, P1/P6, bridge ok, marines moved).
- Defender: the console test exe was quarantined again (Trojan:Win32/Prowloc.A!cl, 15:17:30/37, after the rescue run).
  Rebuilt once, and the remaining runs completed. Correction to the repeat-session note above: Defender also detected
  SC2Coop-old.exe at 04:25:17 with a running process, so the old repro's "process died silently" may be Defender's
  kill, not the overlay bug. The RegisterClass failure itself was logged and is the white-box cause.
- Package: SC2Coop.exe sha256 659fc2e0..., README (red + for targeted clicks, arrow-key scrolling), zip 629 KB.
  Unzipped prepare-product → Recipe 6-0c91d5b56bf169d4, map 9f0ec37a... Both players must Prepare again.

## 2026-10-19 16:15 - Skip-ahead test tools; first co-op mission victories; Hyperion reached

- Kit mission end path (LibWCMI): mission victory trigger -> libWCMI_gf_RunMissionVictorySequence(cinematic) -> mission's gt_Victory
  -> libWCMI_gf_EndCampaignMission(1, victory) (PrepareForStoryMode: CompleteMission + transition banks, score screen)
  -> Continue (libWCMI_gt_WM_VictoryContinue) -> GameOver. Blizzard-style missions also have gt_VictoryCheat (PlayerLeft victory event).
- Added chat commands: hub "-cooplaunch <mission>" (wraps CoopHubLaunchMission); mission "-coopwin" (gt_VictoryCheat),
  "-coopwinnow" (gt_Victory then VictoryContinue after 2 s), "-cooplose". API chat (RequestAction.action_chat) triggers them.
- Test hooks SC2_COOP_TEST_LAUNCH=TRaynor01,... and SC2_COOP_TEST_WIN=<s>[,now|cheat|lose]; CLI campaign-banks save|load|list.
- Local two-client run (copy mode, fresh banks): TRaynor01, 02, 03 each launched by chat and won with -coopwinnow ~4 s after the
  command; the API game ended normally, banks synced, hubs reopened. WCampaign showed MissionCompletedAge TRaynor01=1 and
  TRaynor02 available after the first. After TRaynor03 the hub played the Horner arrival cinematic and opened on the Hyperion
  bridge (screenshot). Snapshots saved: campaign-after-TRaynor01, campaign-hyperion (outputs/campaign-steam-test/local/campaign/snapshots).
- Not yet tested: -coopwin (cinematic + score screen needing a human Continue), -cooplose, two PCs.

## 2026-10-19 16:40 - Kit objective lockout reproduced and fixed

- Kit missions set objectives through libWCMI_gf_ZS_SetMissionObjectiveState (107 calls across the 31 campaign maps; no library-internal
  completed/failed calls). It sets Kit's own UserData ("MissionObjective" Complete/Available, read by MissionObjectiveStateCheck) and the
  engine ObjectiveSetState. Bank saves (WC_SaveMapToBank) store only non-primary objectives' UserData.
- Repro (SC2_COOP_NO_OBJECTIVE_FIX=1, local, TRaynor01, chat "-coopobj TRaynor01_2"): 1 s later the API reported the game over for both
  clients; the launcher closed the guest, which ended the mission for both and returned to the hubs. Worse than "guest loses mouse".
- Fix: BuildMission rewrites those calls to CoopKitObjectiveState: Kit's UserData + progress as the library does, engine state through
  CoopObjectiveSetState (stays active, name gets " (Completed)"/" (Failed)"); other states pass through. Logged as "N Kit objective state calls routed".
- Verified: TRaynor01 with completed _2, failed _3, then -coopwinnow: API status 3 for both until the win. Second run: completed bonus _3
  then -coopwin: Victory score screen (Liberation Day - Normal) on the host, Continue clicked -> both games ended normally, hubs reopened;
  WCampaign MissionObjective.TRaynor01_3 Complete=1 Progress=6. Guest screen during the score screen showed the mission's help dialog
  (both clients open it at mission start); only the host's Continue matters.
- Not verified: guest bridge actions after an objective (status 3 was the proxy, as in the 2026-10-17 Outlaws fix); two PCs.

## 2026-10-19 16:55 - Setup check at connect (campaign mode)

- After the link is up (before bank sync), both launchers exchange a fingerprint (control message "fingerprint"): SHA-256 of the
  running exe, the adapter Recipe, every *.SC2Map in the campaign folder (folder maps: hash over sorted file names + file hashes),
  and every mod those maps depend on (recursive DocumentInfo walk; absent mods = "missing"). Differences are logged per item in
  plain words on both sides and the session stops before any game starts. Old launchers (no fingerprint) time out after 2 min with a
  version hint. Hash cache: <Workspace>/campaign/hash-cache.json keyed by path|size|mtime; map dependency lists cached by map hash.
- MapArchive.ReadFile reads one archive entry; CollectMissing uses it (it used to unpack whole mods, e.g. the 107 MB asset mod,
  on every mission build).
- Tested on one PC (TCP host + join, copy mode): skewed <campaign mod> -> both logs "<campaign mod>.sc2mod is a different
  version on each PC", exit before hubs; identical -> "Setup check passed (38 items)", banks synced, hubs opened. The campaign: 38 items,
  5 mods, 1.8 s cold, 0.1 s cached.

## 2026-10-19 17:15 - Crash recovery during missions

- Already there: a StarCraft II process exiting breaks Session's loop; MissionStage sends mission-over; the partner's watcher stops its
  mission; banks sync host -> guest; hubs reopen. Silent before (only "SC2 PID x exit -1").
- Added: Session.Crashed + "StarCraft II closed unexpectedly (exit code N); ending the mission."; mission-over carries crash=true, the
  partner logs "Your buddy's StarCraft II closed unexpectedly...", both log "The mission does not count; the campaign continues from where
  it was before the mission." Host/local: banks autosaved before each mission (<Workspace>/campaign/autosaves, newest 10; restorable with
  campaign-banks load autosave:<name>); after the mission any bank that isn't readable XML triggers a full restore of that autosave.
- Tests (one PC, copy mode, TRaynor01, SC2_COOP_TEST_CRASH=<s>[,index[,corrupt]]): TCP host+join, host SC2 killed at 45 s -> joiner ended
  within 1 s with the crash message, hubs back after ~11/20 s; joiner SC2 killed -> host ended, both hubs back; local mode guest window
  killed + WCampaign truncated -> "Damaged campaign bank ... restored", WCampaign hash == autosave, hubs reopened.
- Not covered: connection loss mid-mission (still ends the whole session; restart keeps progress), a frozen-but-running game, hub crashes
  (hub exit without pick = session end, as before).

## 2026-10-19 17:40 - Setup check names the differing mod files (Claude side session)

Owner report (two PCs): "Setup difference: kit_liberty_story.sc2mod is a different version on each PC" persisted after the buddy
extracted the owner's copy over his old folder and both restarted. The check hashes every file in a component-folder mod, so
leftover files from the old copy (or a copy nested one level too deep) keep it different; the message didn't say which files.
Owner's D:\...\Mods\kit_liberty_story.SC2Mod: 150 files, no clutter files (observed). Buddy's folder: not seen (unknown).

Change (work\Campaign.cs, snapshot `Campaign.cs.before-file-diff`): when mods differ, both launchers send a second control message
"fingerprint-files" with those mods' file lists (relative path -> first 16 hex digits of the SHA-256; a packed .SC2Mod is one
entry; ~13 KB for the Kit mod) and log per mod: file counts, files only on each PC, files with different contents (8 named,
"+N more"), a hint when the mod folder appears inside itself, and "delete the folder, then unzip a fresh copy". desktop.ini,
Thumbs.db and .DS_Store are no longer hashed. Older launchers: the detail wait times out after 30 s (they differ in
"launcher" anyway). No map change (Recipe unchanged).

Test (outputs/setup-check-test, TCP host + join on one PC, UPnP off, owner's WoL@Kit campaign folder read-only, test-only hook
SC2_COOP_TEST_MOD_OVERRIDE in the copy pointing the join side at an altered copy of the test install's Kit mod: +1 stale file,
1 edited, 1 deleted, 2 files in a nested kit_liberty_story.SC2Mod\, + desktop.ini). Observed on both sides: "150 files on your PC,
152 on your buddy's" (desktop.ini ignored), the 3 extra files named, documentinfo.version missing, preload.xml different, the nested
hint, then the session stopped before any game. Same setup without the override: "Setup check passed (38 items)", both hubs
opened, Stop ended both, SC2 exit 0. Not tested on two PCs; not packaged.

## 2026-10-19 18:20 - Two-PC test of phase 1 (owner + buddy, Steam), and a start retry

Owner report: setup check, -cooplaunch, -coopwinnow, missions 1-3 to the Hyperion all worked over Steam; TRaynor02 rebel rescue
objective completed with no crash/desync. -cooplaunch TRaynor01 from the Hyperion "only reloaded the Hyperion".
- Logs: the first four connects stopped at "kit_liberty_story.sc2mod is a different version on each PC" (the setup check caught a real
  mismatch; fixed by the players). 18:07: the hub launched TRaynor01 fine, then the host's mission SC2 exited -2001 two seconds after
  start ("SC2 exited before API startup"), so the mission was called off for both and the hubs reopened. The hub start one second later
  with the same arguments worked. Cause of -2001 unknown (GameLogs show only SystemInfo); every other start that evening worked.
- Fix: GameClient retries a start that exits before the API is up, twice, 3 s apart (test hook SC2_COOP_TEST_STARTFAIL=1; verified).
- Hyperion mission select: star map/briefing launches go through the hub script's gf_MissionLaunchWithBriefing -> wrapped. Not wrapped:
  LibWCUI's Mission Archives replay (libWCUI_gt_WU_ArchivesLogicMissionLaunch, simulation=true) -> hub ends without a pick -> session ends.
  Documented as unsupported.

## 2026-10-19 19:15 - Phase 2 transitions: timing, faster hub close, process-reuse experiment

- Timeline (owner's two-PC log, TRaynor03): hub pick -> hub SC2 gone 10 s (after a launch the hub never answers RequestQuit; Dispose
  waited 10 s then killed it: every hub "exit -1"), mission SC2 start ~4 s, join/load ~15 s. Mission end -> hub: close 3-7 s, start +
  load ~11 s.
- Change: hub Dispose waits 1.5 s before killing (GameClient.Dispose(waitMs)). Local: pick -> hubs gone 2-4 s (was 11-16 s).
- Mission adapter: GameSetNextMap("") every tick (Kit's victory sets the story map as next map; the launcher decides what loads).
- Experiment SC2_COOP_REUSE (off by default):
  =1 (hub process reused for the mission): CreateGame ok, multiplayer JoinGame hung 3 min (a process that already played a game
     doesn't join a multiplayer game).
  =hub (mission process reused for the next hub, solo): hub opens in ~6 s instead of ~18 s with no window close; the guest's reused hub
     works (story scene, launch). The host's reused hub (whose mission ended through Kit's victory + score screen) renders the Hyperion
     behind a near-black fade and never handles -cooplaunch (chat result 1, status 3, no pick written), also with GameSetNextMap("")
     in the mission. Cause unknown (victory UI/fade state carried over in the process?). Not pursued further.
- Seen again: SC2 exiting right after start (-2000) when two start close together locally; the retry recovered it.
- Verified default flow (local): TRaynor02 -> hub -> TRaynor03 -> hub, wins via -coopwinnow, no errors.

## 2026-10-19 evening: guest clicks drift when the guest zooms in (Claude side session)

Owner report: targeting works well, but after player 2 zooms in (mouse wheel), commands land off centre.

Cause (derived, then measured): the bridge's click homography was calibrated at SC2's default camera. Core CameraData's ZoomTable
has 5 wheel steps that change distance AND pitch: 34/56°, 30/52°, 26/48°, 22/44°, 18/40° (FOV 27.8°). A zoomed camera
maps screen pixels to different ground points; the homography didn't know.

Fix (work\ClickBridge.cs, snapshot `ClickBridge.cs.before-zoom`; campaign_guest_ui.galaxy `.before-zoom`; RecipeVersion 6 → 7):
- BridgeCalibration fits a pinhole camera to the homography at load (Nelder-Mead over 170 pixels: focal scale, principal point,
  feature offset). Fit: mean 0.10 world units, vertical FOV 28.1° (data: 27.8°). Click = homography point + model(current
  distance, pitch) − model(default). Same mapping for right-click, targeting, select, drag rect and ping.
- Zoom source: the wheel over the world steps the ZoomTable at once (forward = in; assumed, not verified). The map's Camera
  report now also has `dist`/`pitch` (CameraGetDistance/Pitch(6)). A report counts when it changes and no wheel step is
  within 1.5 s (so an unsynced default can't undo a wheel step; mission cameras still come through).

Measured (outputs/zoom-test, local two-window, hidden windows, test map hook "coop:zoom <d> <p>" = CameraSetValue on player 6):
- Posted WM_MOUSEWHEEL does not zoom SC2; posted mouse clicks raise no map mouse-click events. So no wheel-path or native-click
  ground truth without the real mouse. Ground markers (ping model, debug-spawned and map-spawned lowered depots) were swamped
  by CHEAT text, units moving, fog reveal and the command centre at the camera target: not usable.
- Ground texture alignment (`zoomtex`): default-zoom capture vs each step with the camera target fixed; for ~50–85 textured
  ground points the model's predicted pixel is refined by block matching. Click miss, world units, median (max):
  30/52 corrected 0.06 (0.74) vs uncorrected 0.76 (1.80); 26/48 0.12–0.13 (1.25) vs 1.69–1.71 (3.32);
  22/44 0.19–0.22 (1.35) vs 2.63–2.71 (5.76); 18/40 0.28–0.29 (1.35) vs 3.42 (6.79). In 2 of 12 level measurements over three
  runs (22/44 once, 30/52 once) both columns shifted together (corrected 1.24/1.43); read as a stale capture, not repeated.
- The bridge logs and uses each step the map reports ("guest camera zoom: distance 26.0, pitch 48.0").
- Regression on a clean Recipe-7 map (outputs/zoom-test/clean, rescue-test.ps1): rescue objective 2, rect/smart 10/10 ok,
  GuestUi bank written, both SC2 exit 0, profile unchanged.

Unknown / not tested: the real wheel (direction, whether SC2's "mouse wheel zoom" option is on, whether a user zoom shows in
CameraGetDistance(6)); zoom mid-transition (ParamSmooth) clicks; cameras with yaw ≠ 180 (not modelled); other window
aspect ratios (the calibration already assumes 16:9); two PCs. Another session ran SC2 tests at the same time as some runs.

## 2026-10-19 22:40 - Loading curtain

- work/Curtain.cs: a borderless, topmost, non-activating, click-through black WinForms window (own STA thread) placed over the last
  known SC2 window rect, with the next map's display name (DocInfo/Name from enUS GameStrings.txt: "Liberation Day", hub "Wings of
  Liberty") and a status line. Shown at a hub pick ("Starting the mission..."; remote: "Waiting for your buddy..."), Session progress
  "joining" -> "Loading...", hidden at "ready" (all joined; the game is running). Mission end ("ended") -> "Returning to the campaign..."
  until the hub's JoinGame returns. Safety auto-hide after 4 min; SC2_COOP_NO_CURTAIN=1 disables.
- Window tracking: Process.MainWindowHandle was Kit's minimized trigger debugger -> first curtain covered the whole primary screen.
  Now EnumWindows for the PID, largest visible "StarCraft II" window.
- Verified locally (log + SC2_COOP_TEST_CURTAIN_SHOT, which saves the form via DrawToBitmap, not the screen): pick -> curtain 1298x767
  over the hub window, Loading 11 s later, hidden at READY (29 s); mission end -> curtain, hidden when the hub opened (18 s).
  Note: full-screen captures of the desktop must not be used for tests (they capture the owner's own screen); deleted.
- Limits: covers the old window's position (a new SC2 window opening elsewhere isn't covered); hides SC2's own loading screen art;
  can't draw over exclusive fullscreen.

## 2026-10-19 23:40 - Player names and colors

- Launcher row "Your name / Color" (per Windows user: %LOCALAPPDATA%\SC2Coop\profile.txt; CLI/test overrides SC2_COOP_NAME,
  SC2_COOP_COLOR, local-mode guest SC2_COOP_GUEST_COLOR). Name -> RequestJoinGame.player_name (missions and hubs; empty = Windows
  user name; local test's second window stays "Buddy").
- Colors are SC2 color indices (0 White, 1 Red, 2 Blue = Kit's libWCMI_gf_CampaignPlayerColor, ... 15 Pink). After the setup check
  both launchers exchange {"type":"profile",name,color}; missions are built with PlayerSetColorIndex(1, hostColor, true) and
  PlayerSetColorIndex(guest, guestColor, true) at campaign tick 2, so both PCs build identical maps. Host color = the shared army.
- Verified: local TRaynor01 with host color Red -> the marines are red (game window shot); TCP host+join exchange logs
  "You play as Owner (Red); your buddy as Buddy (Green)" on both sides. GUI row checked via a launcher-window capture.
- Not checked: how the guest's name/color look in chat and pings on two PCs.

## 2026-10-20 14:55 - Cutscenes, skips and pings for both players

- Survey (31 campaign maps): camera work targets player 1 only (CameraApplyInfo 1119, CameraShakeStart 123, CameraPan 35, CameraLockInput 33,
  CameraRestore 20, a few others) -> the guest kept its own camera during cutscenes. CinematicMode, TransmissionSend (35) and
  ConversationDataRun (18) use PlayerGroupAll -> the guest already got letterbox, talking heads, conversations. Skippable sections:
  228x TriggerSkippableBegin(PlayerGroupAll(), 0, ...) (0 = every player must press Esc) + 4x PlayerGroupSingle(P1). Pings:
  PingCreate/libNtve_gf_CreatePingFacingAngle/MinimapPing with PlayerGroupSingle(P1) (~50) -> guest never saw objective pings.
- BuildMission.CinematicsForBoth: the 13 camera functions -> Coop wrappers (call for the player, and for the guest when player == 1);
  PlayerGroupSingle on ping/skip lines -> CoopPlayerGroupSingle (adds the guest for player 1); skippable required count 0 -> 1.
  Generated in C# (not an adapter file) so the exe hash in the setup check covers it.
- Verified on TRaynor01 (local, game-window shots): host and guest show the same intro shot; guest-only Esc at +12 s -> both in
  gameplay (timer 0:06) at +16 s, control run without Esc still in the cutscene at +16 s; host Esc also skips for both.
- Test folder: outputs/claude-campaign-test (own copy; outputs/campaign-steam-test was rewritten by another session at 14:37 and had
  Oct 19 02:07 adapter files).

## 2026-10-20 15:50 - Native guest pings (no chat), ping panel, ally view on the minimap (Claude side session)

Owner request: the chat-based guest ping wipes the chat; look at trigger ping functions instead, enable the ping panel
(attack/defend/retreat variants), and add the allied minimap frustum, as in two of the owner's own campaign mods.

Sources read: the two mods are packed MPQs, not components folders. Copies were read with a stdlib MPQ reader in the
scratchpad (no need for the components format). Current natives.galaxy and libertylib.galaxy came from CASC: the
work/sources copies are older and lack TriggerAddEventPing / EventPingOption / PingSetPlayerPingsShown / c_syncFrameTypePingPanel.
- Ping panel = `libLbty_gf_EnableDisablePingPanel`: PingSetPlayerPingsShown(players, false) + UISetFrameVisible(players,
  c_syncFrameTypePingPanel, true) + a trigger on TriggerAddEventPing that redraws every ping (EventPingOption 0 attack,
  1 on my way, 2 retreat, 3 defend; plain pings and unit pings otherwise) with PingCreate + beacon effect + sound.
- Frustum = a "PingViewArea" ping (the mod's own CModel: Liberty asset Assets\UI\Feedback\Hud_FOV\Hud_FOV.m3, no stock
  CModel uses it) per player, moved on TriggerAddEventCameraMove; scale 0.5 × 108 / larger playable side; Alt+V pans to the
  ally's camera.

Observed (local two-window mode, copy install, outputs/ping-test; input = PostMessage to the test SC2 windows only, no
SendInput; the owner's real cursor was elsewhere, and the clicks landed at the posted coordinates; time-ordered map log):
- Guest world click and world Alt+click: no mouse event, no ping event (dropped like all guest world clicks).
- Guest minimap Alt+click: a mouse event and a ping event (opt -1, minimap True) at the same world points as the host's
  (3/3). With the ping panel on, those pings show on both minimaps.
- Minimap geometry at 1280x720 (fitted from 3 clicks, exact to 0.1 world units): square 0.2376 h, left 0.0293 h, bottom
  0.0139 h above the client bottom, showing the playable area (Outlaws 16..120 × 8..120) scaled to fit and centred.
- The guest's minimap ping button puts its client into "Ping Minimap" mode (command card); no target-mode event reaches
  the map. No attack/defend/retreat panel appeared from the button, from posted Alt or Ctrl+Alt holds, at any time.

Changed in work\ (snapshots `*.before-native-ping`):
- ClickBridge.cs: a world Alt+click (or a world click within 15 s after a click on the minimap ping button; Esc,
  right-click, or a minimap click end that mode) is swallowed in the low-level hook and replaced by a click posted to the
  guest window at the minimap pixel of the same world point (guest camera from the map's Camera report + feature-layer
  offset), with a posted hover move back. Chat ping removed (Kind.Chat gone). Alt+V marks the bridge's camera position stale.
- campaign_guest_ui.galaxy: chat trigger removed. CoopPingInit does what the Liberty panel does with core natives only
  (Kit missions may not include LibertyLib; a missing library breaks the whole script): hide host+guest native pings,
  show the panel frame, CoopPingEvent redraws each ping for the pinger and its allies by type (attack/on my way/retreat/
  defend: PingPanelPing* model, PingPanelBeacon* effect, UI_PingAdvanced_* sound, chat line "<name>: attack"; unit pings ->
  attack (enemy) or defend; plain: MinimapPing + 4 s ground marker), 0.25 s per-player guard. CoopAllyViewInit: view
  frames + Alt+V, as in the owner's mod.
- TrialPreparation.cs: AddCoopModels merges CModel PingViewArea into the map's ModelData.xml (Outlaws has one; added if
  missing); called from BuildMap and Campaign.BuildMission. RecipeVersion 7 → 8: both players must Prepare again.
- Gotcha, again: a local named `sound` (a Galaxy type, like `marker`) silently broke the whole map script (run 5). All
  native type names: abilcmd actor actorscope aifilter bank bitmask bool byte camerainfo color datetime doodad
  effecthistory fixed generichandle int marker order playergroup point region revealer sound soundlink string text timer
  transmissionsource trigger unit unitfilter unitgroup unitref void wave waveinfo wavetarget.

Verified (map fa20ecebb... in ping-test with test-only logging in its diagnostics copy):
- Bridge test ping (feature 250,150, Alt posted): "world 89.9,32.2 via minimap pixel 140,673", 3/3. Map log: three P6 ping
  events at 89.9,32.2. Pings on both minimaps; no chat line.
- Host posted Alt+click world (90.0,32.2, minimap False) and minimap: ping events, drawn on both minimaps, ground marker at
  the spot in both views. (In 2 of 4 runs the host's posted clicks produced no events at all, plain clicks included; a
  harness flake, cause unknown; run 8 had all 5 inputs.)
- View frames: each minimap shows the other player's view; after the host scrolled, the guest's minimap showed it moved;
  guest Alt+V (posted) moved the guest view onto the host's.
- Regression: bridge rect/smart ok; drag box overlay shown; resources; both SC2 exit 0. Run 3 ended with "profile
  changed" because the SC2 Editor (open since 15:09) closed at 15:27:18 and rewrote EditorVariables.txt in the real
  Documents folder; not caused by the test.

Unknown / not tested:
- How the owner opens the attack/defend/retreat panel in their mod (not found by button or held modifiers here). The
  map side handles options 0-3 when they arrive; the bridge only converts world clicks in plain ping mode.
- Real mouse and keyboard: the hook swallowing the real Alt+click and SC2 using the physically held Alt for the posted
  minimap click (tests posted Alt).
- Other window sizes and aspect ratios (minimap fit assumes the console scales with height), other maps' bounds, Kit
  missions, two PCs.
- Kit mission check (outputs/ping-campaign-test, `campaign trial.json local - <owner's WoL@Kit campaign folder>`,
  SC2_COOP_TEST_LAUNCH=TRaynor01): the mission built with AddCoopModels and its script compiled (Kit's trigger debugger
  listed CoopCamTick/CoopCamReport/CoopResUpdate running; the bridge got the guest camera). Both minimaps showed the
  ally's view frame. The test ping was posted (world 114.4,84.5); its accuracy wasn't checked there (no event log in the
  campaign build; the Help panel covered the view). The mission window there is 1280x720 too. Stop at +230 s: host SC2
  exit 0, guest SC2 exit -1, profile unchanged, launcher exit 0.

## 2026-10-20 16:00 - Launcher window: logic split from the window, dark redesign (Claude side session)

Owner request: make the launcher look modern without losing function; separate the window code from the session logic.
- work/LauncherCore.cs (new): LauncherController, no WinForms. Modes (same ids and order as last-mode.txt), saved
  choices (last mode + address, campaign folder, name/color, both places as before), readiness (Load + Validate),
  Prepare, Steam friend lookup, invite parsing (+connect_lobby), Start (Session or CampaignRun, as before), Stop and stop
  on close. Status for the window comes from one classifier over the log lines plus Session stages (single missions):
  Waiting for your buddy / Connected / Starting StarCraft II / Loading / In game / Campaign open / errors, and the Steam
  latency line ("42 ms round trip").
- work/LauncherWindow.cs (new): the window only. Dark theme, a Ready card with Prepare, mode tiles (Steam host marked
  Recommended), a context row per mode (address field, Steam friend box + Find, or a hint), Campaign switch + folder,
  name and color (with swatches), status line, Start/Stop, collapsible log ("Show details"; opens on errors). Dark title
  bar and rounded corners via DwmSetWindowAttribute (20, 33). Sizes scale with the system DPI. Plain WinForms + GDI, C# 5.
  Test hook SC2_COOP_TEST_UI=<mode>[,single][,start] (WinForms raises Click only under the real cursor, so posted clicks
  can't drive it).
- TrialLauncher.cs: the old LauncherForm is removed; Main runs LauncherWindow. build-launcher.ps1 adds the two files.
  Snapshots: TrialLauncher.cs.before-ui, build-launcher.ps1.before-ui.
- Verified (outputs/ping-test, GUI build, captures of the launcher window only with PrintWindow): layout at 125% (ui8/ui9);
  SC2_COOP_TEST_UI=local,single,start ran a local session from the window: status "In game", Stop enabled, inputs locked
  (ui5). WM_CLOSE during the game: "Stopping; close the launcher after the game has shut down.", both SC2 exit 0, profile
  unchanged, window back to "Stopped" with Start enabled (ui7). The owner's %LOCALAPPDATA%\SC2Coop choices were backed up
  before and restored after (the test's Start saved "local").
- Not checked: the dark title bar (PrintWindow draws the frame light; unknown whether the real frame is dark), keyboard
  navigation of the tiles (Tab skips them; Enter = Start), Prepare / Find / Browse dialogs from the new window (same
  controller calls as before), a 100% or 150% display, the campaign path from the GUI.

## 2026-10-20 18:45 - Phase 3: all campaign missions in co-op (smoke test), map checker, porting checklist

- All 29 campaign missions build through BuildMission (guest slots 6-14; 3-11 Kit objective calls routed per mission).
- Smoke test (local two-window, copy install, outputs/claude-campaign-test): each mission launched by hub chat, -coopwinnow after 45 s,
  back to both hubs. TRaynor01-03, THanson01-03B, TTosh01-03B, THorner01-05S, TTychus01-05, TZeratul01/03/04, TValerian01-03 (incl. the
  finale) passed: both joined, API status stayed 3 until the win, mission ended normally, banks synced, hubs reopened. TZeratul02 failed
  once: the guest's JoinGame returned 6 "Unable to launch game". Afterwards joins use player 1's race from MapInfo (Zeratul missions are
  Protoss; RequestJoinGame race was always Terran) and TZeratul02 passed twice; TZeratul01 had passed as Terran, so the cause is unproven.
  The start retry caught one more early SC2 exit (-2000).
- Run 1 was killed from outside at 17:08:56 (exe deleted at the same second; another session's cleanup?); restarted under SC2Coop-smoke.exe.
- campaign-check (Campaign.cs CampaignCheck) + work/<campaign>-coop-check.md; work/PORTING.md (what's automatic, what needs the map
  author, how to test).
- Not covered by the smoke test: real play (objectives, mid-mission choices, guest input over time), two PCs.

## 2026-10-20 23:10 - Package for the two-PC test; the "mystery kills" were Windows Defender

- Correction to 18:45: the runs killed at 17:08:56 and 23:03:55 (and outputs/campaign-steam-test "reset" at 14:37) were Windows Defender
  quarantining the console test exe as Trojan:Win32/Prowloc.A!cl (cloud ML; ThreatID 2147727733): process killed, file removed
  (Get-MpThreatDetection). Earlier hits: pkg6-test (10-19 15:17), repeat-test (10-19 04:25). No detection for outputs/sc2-coop-package or
  the owner's Desktop\SC2Coop so far. Likely trigger: low-level mouse/keyboard hooks in an unsigned exe. Not another session.
  Exclusions are the owner's call (security setting); README now tells players how to restore from Protection history.
- Package rebuilt after commit 7765c09 (launcher redesign, native guest pings, ally view): SC2Coop.exe from HEAD, all four adapters copied
  (the package's were from 10-19), README updated for the new window (tiles, "Prepare...", "Campaign") and Alt+V. RecipeVersion 8:
  players must click Prepare again.
- Final local check (combined code): hub -> TRaynor02 -> win -> host hub reopened; then Defender quarantined the test exe while the guest
  hub was starting.

## 2026-10-21 00:15 - Owner's second two-PC session: fixes for Stop/re-host, team colors; hub chat experiment

- Owner report: several missions, a victory, bonus objectives fine. Issues: Dark Green showed purplish, Dark Grey dark red; guest ping panel
  missing and guest pings slightly right of target; guest resource display looks off; Medic scan-move doesn't work; weird bugs when the
  buddy hosted after a Stop (23:18-23:20); "Stopping" forever after the buddy left during connect; wants chat and colors in the hub.
- Stuck Stop (owner log 23:22:48): connected, setup check passed, buddy left the lobby -> tunnel closed; the guest sat in ReceiveBanks
  (5 min, no Stop/closed check). Fix (c91dcfa): Take() polls in 250 ms slices and returns on Stop or a closed tunnel; Gone() gives the
  reason. Tunnel heartbeat: Hello frame every 5 s, close after 30 s silence (a killed launcher's SC2 inherited its socket, so TCP never
  closed). Verified: killed host launcher -> joiner "nothing heard for 30 seconds", session ended. 23:18:40 failure ("nothing heard from
  your buddy for 20 seconds") was on the buddy's side; their logs needed.
- Colors (96c1b24): CGame TeamColors are data; Kit and the campaign mod change 1,7,10-15 (11 = "Rose", 14 = "Dark Red") and append 16 "Mengsk";
  <campaign mod> appends 9 more, but only TRaynor01/02 load <campaign mod> (26 of 29 missions and the hub load only Kit). TeamColors.cs
  rebuilds the table (base 16 + mods in dependency order, most common mission dependency set); launcher list/swatches/log names follow
  the selected campaign; CLI campaign-colors. The dropdown itself was not seen in a screenshot (popup), the list is the CLI's.
- Hub chat experiment (not kept): out via chat trigger -> bank works in principle, but in: a running game doesn't re-read a bank file
  the launcher rewrites (BankLoad returns the cached bank; no seq seen after 3 s). Also: `text` is a Galaxy type name; `string text`
  silently disabled the whole hub script. Next idea: launcher injects the buddy's line with API chat ("-coopmsg ...") and the hub
  trigger redraws the chat area without the raw line; needs a check that story mode shows the chat area.
- Package rebuilt with these fixes.
