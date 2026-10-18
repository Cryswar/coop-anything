# Command & Conquer 3: Tiberium Wars (2007) and Kane's Wrath (2008)

| | |
|---|---|
| Engine | SAGE (Generals lineage; C&C3 era), x86 |
| Versions tested | Steam Tiberium Wars 1.10 (buildid 17447444, `RetailExe\1.10\cnc3game.dat` SHA-256 `af568a89...0d141`; the C&C Online variant `ade9f5b6...a3543` differs only in the PE header: large-address-aware flag and checksum). Kane's Wrath 1.3 identified, not worked on yet. |
| Projects | Campaign co-op prototype (proxy `dinput8.dll`), unpublished, owner's workspace; notes in [NOTES.md](NOTES.md) |
| QA level | community-reports (owner + friend played most of GDI mission 1 online); automated two-client smoke tests on one PC |
| Last updated | 2026-10-18 |

## Where things stand

Two players can play a Tiberium Wars campaign mission together: one hosts a normal multiplayer
game (C&C Online or LAN), the other joins, and both get the campaign mission with its own sides,
scripts, AI, objectives and in-mission cutscenes. Both control the same campaign side (shared units,
shared money). Tested by the owner and a friend over C&C Online on GDI mission 1 (prologue): both
had full control of units, building and placement. Missions can be won and an experimental
mission flow advances both PCs to the next mission (not yet tested by humans). Briefing movies and
own factions per player are not done. Kane's Wrath has the same engine structure but no port yet.

## Existing work

Searched 2026-10-17: this repo, web search, GitHub repo search; ModDB pages were behind a
Cloudflare check (search-result text only).

- No official PC campaign co-op for either game
  ([Revora answer](https://forums.revora.net/topic/100755-how-to-play-c-c-3-campaign-co-op-please-help/)).
  The Xbox 360 version had co-op versus AI, not the story
  ([Co-Optimus](https://www.co-optimus.com/game/181/xbox-360/command-and-conquer-3-tiberium-wars.html)).
- Community co-op missions exist as custom multiplayer maps, not the official campaign:
  [COOP Evacuation](https://www.moddb.com/games/cc-tiberium-wars/addons/coop-evacuation-co-operative-mission),
  [Boyle's Legacy pack](https://forums.cncnz.com/topic/16232-boyles-legacy-co-op-mission-pack-re-released/),
  [Co-Op Tiberian Sun Map Pack](https://www.moddb.com/mods/co-op-tiberian-sun-map-pack) (needs Tiberium Essence).
- Not found by those searches: any engine-level campaign co-op for C&C3 or Kane's Wrath.
- Related engine source: EA's GPL release of Generals Zero Hour
  ([electronicarts/CnC_Generals_Zero_Hour](https://github.com/electronicarts/CnC_Generals_Zero_Hour))
  and [OpenSAGE](https://github.com/OpenSAGE/OpenSAGE) (map format) were the main references.

## How the game works (for co-op)

- **Install:** `CNC3.exe` is a launcher; the real game is `RetailExe\1.10\cnc3game.dat`, started as
  `cnc3game.dat -config "<install>\CNC3_english_1.10.SkuDef"`. `-win`, `-xres`, `-yres` work. The 2025
  Steam exes are not packed. C&C Online ("Tacitus") is a `dsound.dll` proxy and needs no launcher.
- **Maps:** campaign maps are in `SP\1.0\SPMaps.big` as raw `.map` (RefPack, `EAR\0`, RefPack, `CkMp`
  chunks; layout as in OpenSAGE) plus compiled `map.bin`. Each campaign map has one human side and
  several named AI sides with their own scripts.
- **Netcode:** lockstep, Generals lineage. The LAN lobby is broadcast only (ports 8086-8093) and
  identifies players by IP. Commands carry a player index; a slot's commands are accepted only if no
  other slot already acts for the same player (anti-spoofing, see NOTES section 12).
- **Game start:** `startNewGame` with a lobby GameInfo deletes flagged teams, strips every map side
  except civilian/creeps (`prepareForMP_or_Skirmish`) and builds `Player_N` sides from the slots. The
  slot -> Player mapping is the player name stored in the slot.
- **Victory:** a skirmish victory check looks at the slot players; campaign maps decide victory and
  defeat in their scripts. After online games a results report is sent to the ladder service.

## What we changed

All in a `dinput8.dll` proxy; every patch site is checked against its expected bytes and the DLL
does nothing to an unknown build. Addresses are in NOTES.md.

- **Campaign map in a lobby game:** the lobby map is swapped for the chosen mission on both clients;
  side stripping, slot sides and the team deletion step are skipped; the campaign's human side is
  marked local; every human slot is named after it; the skirmish victory check is off.
- **Guest orders were dropped:** the receive-side anti-spoofing check flagged the second slot acting
  for the campaign side and discarded all its commands, even on the guest's own PC (found by logging
  UI-created, sent and per-frame commands on both clients). The check is relaxed for co-op games.
- **Missions could not be won:** missions end the classic SAGE way (scripts kill the enemy sides,
  the victory check declares the last side standing). That check was off in co-op because the
  campaign side failed the multiplayer elimination rule at the start. Now the check runs, but in co-op
  a side counts as defeated only when the mission scripts kill it: the "nothing left" rule had also
  eliminated allied AI sides (and destroyed their buildings) at the first check.
  Test: enemy sides killed at a fixed frame on both clients -> VICTORY on both.
- **Mission flow (experimental):** the current mission is kept in a file in the co-op profile; after a
  victory both clients advance it to the next mission, and the next lobby game loads it.
- **Shared selection:** an order applies to the player's logic-side selection, which both humans
  share. Each client keeps its own selection messages and re-sends them before its next order when the
  partner's selection changed in between.
- **Crash to desktop when leaving an online game:** the results report built one record per player
  and corrupted the heap with co-op's player layout (WER minidumps from both players). Co-op games are
  not reported.
- **Test harness:** two game instances on one PC over LAN (per-instance loopback IP, broadcasts
  re-sent as unicast, `SIO_UDP_CONNRESET` off), each instance with its own AppData, kept minimized
  but "active", driven by posted window messages, with in-engine screenshots (D3D9 `Present` hook).

## QA

- Automated (two real clients on one PC, LAN): mission start in sync on both clients (screenshots
  match, logic frames advance, identical command stream), guest commands executed on both clients,
  shared-selection restore (host selects, guest selects, host orders: host's selection is re-sent
  first), exit from the game back to the lobby on both clients. These are smoke tests with logged
  evidence, not a pass/fail suite with negative controls yet; mouse move orders cannot be produced by
  the harness (keyboard orders can).
- Human test, C&C Online, two PCs, GDI mission 1: full shared control for both players, no crashes,
  no mismatch. One oddity: Nod bombers damaged but did not destroy the starting power plant, so a
  tutorial step had to be finished by hand. Unknown whether this is random or caused by co-op (for
  example campaign difficulty not applied in lobby games).

## How to play

Not published. The owner's package: run `Setup.bat` (finds the Steam install, makes a separate
hard-linked copy of the game with the DLL, remembers folders), pick the same mission on both PCs,
click Play, host a private unranked 2-player game on any map in C&C Online (or LAN).

## Known issues

- Mission flow: after a victory both players return to the lobby and host a new game for the next
  mission; no briefings or between-mission movies; no saves.
- GDI mission 2 (Pentagon), nobody playing: lost after about 40 s when Nod destroys an allied power
  plant (the mission's own lose script). Probably normal for an idle player; not compared with a
  single-player run. (An earlier co-op bug that lost it after 3 s is fixed, see NOTES section 14.)
- Control groups are shared; post-game stats show odd player names.

## Next steps

- Briefing movies between missions; saves.
- Own faction per player: see [mission-survey.md](mission-survey.md) for the script survey of all 39
  missions and the generic rules it suggests (and the `<Local Player>` desync trap).
- Check which campaign difficulty lobby games run (prologue: Nod money and attack forces differ).
- Kane's Wrath port (same seams, different addresses).
