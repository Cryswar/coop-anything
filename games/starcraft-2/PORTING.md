# Making a StarCraft II campaign co-op-ready

What the SC2Coop launcher does to a campaign by itself, what you (the map author) still have to decide or edit, and how to
check a new campaign. Based on porting a WoL@Kit campaign (2026-10). Run the checker first:

```
SC2Coop-console.exe campaign-check "<campaign folder>" report.md
```

It lists, per map, every script call aimed at player 1 that the launcher does **not** already handle.

## How co-op works (one paragraph)

The host is player 1 as usual. The buddy joins as an extra player (a free slot, e.g. 6-14) that shares control, vision and
spending of player 1's army ("shared army" mode). Each player walks around their own copy of the hub (story map); a mission
pick starts the mission for both through StarCraft II's API in multiplayer. Campaign progress is the host's banks; the
launcher copies them to the buddy and back around every mission.

## Done automatically by the launcher (no map edits)

| Area | What the launcher does | Why |
|---|---|---|
| Guest slot | Copies player 1's MapInfo record into a free slot | The map needs a second human player |
| Shared army | Alliance with shared control, vision and spending; re-applied every second | Missions change alliances as they go |
| Guest input | Click bridge (selection, orders, targeting, build placement) and camera keys | The API drops a joining player's world clicks |
| Objectives | `ObjectiveSetState` / Kit's `libWCMI_gf_ZS_SetMissionObjectiveState`: completed/failed are shown as a name suffix; the engine state stays active | A completed or failed objective makes the API end the game for both players |
| Cutscene camera | 13 camera functions aimed at player 1 also run for the guest | Both players see the same shots and input lock |
| Pings | Pings for player 1 also go to the guest | Objective pings were host-only |
| Skipping | Skippable scenes need one player's Esc instead of everyone's | Required count 0 = every player |
| Mission end | Victory/defeat through the campaign library ends both games; banks synced; both hubs reopen | |
| Next map | `GameSetNextMap` cleared | The launcher decides what loads |
| Hub | Mission launches (`libWoLC_gf_LaunchMission`) recorded for the launcher; debugger window kept closed | |
| Difficulty, colors, names | Set per mission from the hub pick and launcher settings | |
| Test commands | `-cooplaunch <mission>` (hub), `-coopwin`, `-coopwinnow`, `-cooplose`, `-coopobj <objective> [fail]` (missions) | Skip ahead while testing |

Also automatic: setup check (same launcher, maps, mods on both PCs), crash recovery (bank autosave before each mission),
loading curtain between games.

## Needs a decision or an edit by you

### Shared army mode (current): nothing required, but know these

- **Host-only input events.** Triggers on player 1's input (`TriggerAddEventButtonPressed(t, 1, ...)`, key presses, chat,
  unit selection) only react to the host. Tutorial prompts like "select the marines" wait for the host. Mission choice
  buttons (e.g. a dialog to pick a side) are the host's to press. If the buddy should be able to do these, add the guest
  player to those events (or use `c_playerAny` and check `EventPlayer()`).
- **Forced selection** (`UnitSelect(u, 1, ...)`) only changes the host's selection.
- **Difficulty.** Two players on one army make missions easier. Consider bumping enemy difficulty or wave sizes.
- **Player-left triggers.** Most missions have victory/defeat cheat triggers on `TriggerAddEventPlayerLeft(..., c_playerAny, ...)`.
  The launcher ends both games when either leaves, so these don't matter today. Check new ones.

### Separate armies (planned mode): real map work

The checker's "Matters for separate armies" section is the to-do list:

- **Units of player 1** (counts, ownership, spawns). Objectives like "train 10 marines" or "bring units to X" count only
  player 1. Decide per objective: count both players, or keep it the host's job.
- **Resources and supply of player 1** (starting minerals, rewards).
- **Tech of player 1** (unlocks, upgrades, research from the hub). The guest needs the same calls for their slot, or their own
  tech from their own banks (separate-upgrades mode).
- **Spatial content** you have to build in the editor: a second start location, base space and starting units for the guest.
  This can't be generated.

### Other campaigns

- **WoL@Kit-style campaigns** (story map TStory01 + `libWoLC_gf_LaunchMission`): should work as is; run the checker and the
  smoke test (below).
- **Blizzard's own campaigns and other mods:** the hub launch hook and bank list depend on the campaign library. Progress lives
  in banks too, but under different names and possibly signed. Expect to add a hub adapter for that library.

## Testing a ported campaign

1. `campaign-check` and read the per-map list.
2. Smoke test every mission on one PC (local mode): `SC2_COOP_TEST_LAUNCH=<mission,mission,...>` launches each from the hub,
   `SC2_COOP_TEST_WIN=45` wins after 45 s. Look for missions that don't end, errors, or the guest's API status leaving 3.
3. Objectives: `-coopobj <objective>` in a mission; both players must stay in the game.
4. Play a few missions properly on two PCs.
