# Tiberium Wars mission script survey (co-op planning)

Generated 2026-10-18 by `tools/mapscripts.py` from the 39 campaign maps in `SP\1.0\SPMaps.big`
(Steam 1.10). It decodes each map's script lists (format from OpenSAGE) and counts every script
condition and action whose arguments name the campaign's human side, one of its teams, or a
`<Local Player...>` placeholder. Counts are of script items, not of how often they run.

## Summary

- Shared control (both humans on the campaign side): **no map edits are needed**. The scripts see
  exactly the side they were written for. The prologue played through its tutorial steps unmodified.
- How missions end: every mission ends through its objectives and scripts, mostly by killing the
  enemy sides (`PLAYER_KILL`) so the engine's last-side-standing check declares the win. Only GDI 3-3
  uses explicit `VICTORY`/`DEFEAT` and GDI 3-4 `QUICKVICTORY`. Co-op has to keep that engine check
  running (done in 0.0.5, see NOTES).
- Own faction per player: every mission refers to the human side 14-174 times. Almost all references
  fall into seven kinds (table below), so a **generic engine rule per kind** looks feasible instead of
  editing 39 maps by hand. What each rule needs is in "Own factions" below.
- `<Local Player>` is used in 18 of 39 missions (412 references). Inside lockstep logic it must mean the
  campaign side on both PCs, or the two games would evaluate different players and desync. With shared
  control both PCs' local player already is the campaign side.

## Kinds of references

| Category | references | missions |
|---|---|---|
| Checks on the human side (any unit/structure, in area, owned, counts) | 647 | 39 |
| Checks on contact with the human side (attacked by, discovered, sighted) | 334 | 32 |
| Units/structures handed to the human side (transfer, reinforcement teams, spawns) | 276 | 29 |
| Money (set, give, cap) | 128 | 37 |
| Tech/build permissions, powers, upgrades | 279 | 33 |
| Map reveal | 199 | 32 |
| Diplomacy and AI targeting | 139 | 26 |
| (of all the above) written as `<Local Player>` instead of the side name | 412 | 18 |

Not counted above (smaller, mostly presentation): sounds at a team, kill recording counters, object
references, selecting objects, radar events, panel flags (about 110 references).

## Own factions: what each kind would need

Plan sketch: player 2 gets a second human side cloned from the campaign side and allied to it; the
campaign side stays player 1. Then, engine-side and without map edits:

| Kind | Rule for two human sides | Risk |
|---|---|---|
| Checks on the human side | existence and "in area" checks: true if **either** side qualifies; count comparisons (`PLAYER_HAS_OBJECT_COMPARISON`, used for defeat checks such as "has 0 structures"): **sum** of both sides | which comparisons mean "lost everything" vs "has at least N" must be read from the operator; sum covers both |
| Contact checks (attacked by, discovered, sighted) | either side | low |
| Units handed to the human side | default to player 1; optional split (alternate teams, or by where the reinforcement arrives) | per mission feel: reinforcements are the main thing a second player wants |
| Money | split or duplicate grants (option); `SET_MONEY` sets both | balance, not correctness |
| Tech, powers, upgrades | apply to both sides | low |
| Map reveal | both sides (shroud is shared between allies anyway) | low |
| Diplomacy, AI targeting | apply to both; AI "favored enemy" picks either | low |
| `<Local Player>` | always the campaign side (player 1) in logic; presentation (camera, EVA) stays per PC | must be done before anything else or the game desyncs |

Starting units: most maps pre-place 1-117 objects for the human side (table below); those could be
split between the two sides at load (for example alternate squads), or kept on player 1.

## Per mission

| Mission | human side | scripts | refs to human side | via `<Local Player>` | explicit VICTORY/DEFEAT | preplaced human objects |
|---|---|---|---|---|---|---|
| aliens_1_1_london | Plyr_Alien_A | 106 | 28 | 27 | 0 | 17 |
| aliens_1_2_munich | Plyr_Alien_A | 97 | 50 | 48 | 0 | 4 |
| aliens_1_3_croatia | Plyr_Alien_A | 30 | 15 | 14 | 0 | 11 |
| aliens_1_5_italy | Plyr_Alien_A | 102 | 28 | 7 | 0 | 49 |
| gdi_0_1_prologue | GDI_a | 262 | 102 | 17 | 0 | 1 |
| gdi_1_1_pentagon | Plyr_GDI_a | 166 | 54 | 0 | 0 | 7 |
| gdi_1_2_langley | Plyr_Gdi_a | 103 | 62 | 0 | 0 | 1 |
| gdi_1_3_hampton | Plyr_GDI_a | 66 | 17 | 0 | 0 | 25 |
| gdi_1_4_white_house | Plyr_GDI_A | 136 | 64 | 62 | 0 | 6 |
| gdi_2_1_weapons_factory | Player_GDI | 96 | 51 | 0 | 0 | 5 |
| gdi_2_2_nod_port | PlyrGDI_A | 234 | 49 | 0 | 0 | 5 |
| gdi_2_4_nuke_facility | Plyrfactiongdi | 135 | 46 | 0 | 0 | 11 |
| gdi_3_1_albania | PlyrGDI_A | 219 | 27 | 0 | 0 | 117 |
| gdi_3_3_croatia | PlyrGDI_A | 313 | 89 | 0 | 2 | 5 |
| gdi_3_4_sarajevo | PlyrGDI_A | 251 | 53 | 0 | 1 | 14 |
| gdi_4_1_munich_outskirts | PlyrGDI_A | 310 | 14 | 0 | 0 | 8 |
| gdi_4_2_stuttgart | PlyrGDI_A | 217 | 118 | 0 | 0 | 2 |
| gdi_4_3_cologne | PlyrGDI_A | 187 | 57 | 0 | 0 | 1 |
| gdi_4_4_berne | PlyrGDI_A | 138 | 40 | 0 | 0 | 18 |
| gdi_5_1_tower | PlyrGDI_A | 174 | 52 | 6 | 0 | 2 |
| gdi_5_2_relay_node | PlyrGDI_A | 179 | 77 | 7 | 0 | 8 |
| nod_0_1_prologue | Nod_a | 145 | 45 | 8 | 0 | 3 |
| nod_1_1_white_house | Plyr_NOD_A | 226 | 40 | 40 | 0 | 16 |
| nod_1_2_andrews_afb | NOD_A | 229 | 133 | 108 | 0 | 10 |
| nod_1_3_hampton | Plyr_Nod_a | 72 | 37 | 1 | 0 | 20 |
| nod_1_4_dccity | Plyr_N_a | 88 | 34 | 0 | 0 | 14 |
| nod_2_1_brazilianjungle | Nod_a | 130 | 35 | 5 | 0 | 34 |
| nod_2_3_coastal_brazil | Nod_a | 189 | 37 | 2 | 0 | 29 |
| nod_3_1_slovenia | PlyrNOD_A | 212 | 73 | 2 | 0 | 2 |
| nod_3_2_sarajevo | PlyrNOD_A | 305 | 85 | 0 | 0 | 13 |
| nod_4_1_broken_hill | PlyrNOD_A | 231 | 66 | 0 | 0 | 14 |
| nod_4_2_sydney_west | NOD_A | 119 | 58 | 27 | 0 | 7 |
| nod_4_4_sydney_interior | NOD_A | 111 | 25 | 0 | 0 | 37 |
| nod_4_5_ayers_rock | NOD_A | 139 | 85 | 17 | 0 | 6 |
| nod_5_1_nuke_recovery | Plyrfactionnod | 95 | 28 | 0 | 0 | 11 |
| nod_5_2_mapkey | Plyr_Nod_a | 62 | 25 | 0 | 0 | 9 |
| nod_5_4_operation_stiletto | PlyrNOD_A | 173 | 174 | 0 | 0 | 9 |
| nod_5_5_the_tower | Plyr_N_a | 84 | 18 | 0 | 0 | 8 |
| tutorial_pc | PlyrGDI_Blue | 85 | 47 | 14 | 0 | 1 |
