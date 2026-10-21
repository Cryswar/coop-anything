# StarCraft II (2010): campaign co-op

| | |
|---|---|
| Engine | Blizzard SC2 engine (Galaxy scripts, data XML, MPQ maps/mods); s2client API in the retail client |
| Versions tested | Battle.net retail 5.0.17.98310 (data version 9C1F3D3D30CA252E33FC67CA05CBE1EA), enUS, Windows 10/11 |
| Projects | SC2Coop launcher (this write-up, not released yet); Heart of the Swarm Co-Op (Arcade map, 2019, untested) |
| QA level | automated (local two-client smoke test of every mission of one campaign) + two-PC owner sessions |
| Last updated | 2026-10-21 |

## Where things stand

Two players can play Wings of Liberty campaign missions together today, in the WoL@Kit fan campaign format, over
Steam (Valve's free Spacewar app id as a relay) or a direct connection. Both share one army (player 1); each walks
around their own copy of the hub (Hyperion), and a mission pick starts the mission for both. Two-PC sessions played
several missions, a victory and bonus objectives without game-breaking problems. The guest's input runs through a
click bridge, so it feels slightly less direct than the host's.

## Existing work

- **Heart of the Swarm Co-Op** (LordDz and team, Team Liquid thread, last post 2019-12): an Arcade remake of HotS
  missions with per-player hero choices. Arcade-only (needs the map from Battle.net), not tested here.
- No non-Arcade co-op for the retail campaigns was found (searched 2026-10-16: GitHub, SC2Mapster, Team Liquid,
  Reddit, Steam via web search). Blizzard's own Co-op Missions mode is a separate PvE mode, not the campaign.
- Archipelago's SC2 client drives the retail game through the same API (single player) and was a useful reference.

## How the game works (for co-op)

- **The API.** The retail client accepts `-listen/-port` and speaks s2client-proto over a websocket. It can create
  multiplayer games of any local map, and players join with ports. Multiplayer only works when every client uses a
  loopback host address, so the launcher tunnels the four game ports between PCs itself (one TCP stream, or Steam's
  P2P relay; game UDP is sent unreliably).
- **Campaign = maps + mods + banks.** Missions and the hub (story map) are `.SC2Map` files depending on mods; the
  campaign library (Kit's `LibWoLC`/`LibWCMI`) moves between them with `GameSetNextMap`/`GameOver`. All campaign
  progress lives in player-1 bank files (`%USERPROFILE%\Documents\StarCraft II\Banks`).
- **Player slots.** Maps declare their players in `MapInfo`. Adding a user slot (a copy of player 1's record) is enough
  for a second human; an alliance with shared control, vision and spending makes them co-own player 1's army.
- **API quirks that shaped everything** (all observed, see NOTES.md):
  - The creator's native world input needs `raw=true` in InterfaceOptions; a joiner's world clicks are dropped, so the
    guest's clicks go through the API as feature-layer actions (the "click bridge").
  - Any objective set to completed or failed makes the API end the game for both clients (status 5).
  - `RequestObservation` crashes for guest player ids above 3 in some setups (the API's `3 - player` 1v1 assumption).
  - `RequestGameInfo` crashes on story maps. A story map's `GameOver` loads the next map unless it is cleared.
  - A running game doesn't re-read a bank file changed on disk (`BankLoad` returns the cached bank).
  - A process that already played a game does not join another multiplayer game (join hangs).
  - SC2 sometimes exits with -2000/-2001 right after start; a retry works.
  - `text` is a Galaxy type name: a local variable called `text` disables the whole map script without a message.

## What we changed

The SC2Coop launcher (C#, .NET Framework; no game files are shipped, everything is built from the player's own install):

- **Mission adapter** (applied to each mission map at build time): guest slot in `MapInfo`; shared-control alliance
  re-applied every second; objective state calls (engine and Kit's) routed so completed/failed objectives are shown
  as a name suffix instead of ending the API game; the 13 camera functions aimed at player 1 mirrored to the guest
  (cutscenes); player-1 pings shown to the guest; skippable cutscenes skip on one player's Esc; next map cleared;
  difficulty and team colors set; test chat commands (`-coopwin`, `-coopwinnow`, `-cooplose`, `-coopobj`).
- **Hub adapter**: mission launches recorded in a bank for the launcher; `-cooplaunch <mission>`.
- **Campaign flow**: both hubs as solo API games; a pick in either closes both and starts the multiplayer mission;
  afterwards banks sync host -> guest and both hubs reopen. Setup check at connect (launcher, maps, every mod by
  SHA-256), bank autosave before each mission and repair after a crash, crash and connection-loss handling, a loading
  curtain between games, player names, and team colors read from the campaign's own data.
- **Click bridge** for the guest: selection, box select, orders, targeted abilities (target-mode events plus a probe
  order swapped for the real ability), build placement, camera keys, pings.
- **Transport**: TCP 6600 with UPnP, or Steam P2P via the Spacewar app id (friend invites, lobby), with a heartbeat.
- **Map checker** (`campaign-check`): lists per map what the launcher handles and what needs a map author. The porting
  checklist is in [PORTING.md](PORTING.md).

## QA

- **Smoke test** (one PC, two real clients, copy of the install, test hooks `SC2_COOP_TEST_LAUNCH`/`SC2_COOP_TEST_WIN`):
  every mission of a 29-mission WoL@Kit campaign launched from the hub by chat command, won with `-coopwinnow` after
  45 s, back to both hubs. Run 2026-10-20: 28/29 passed on the first run; the failing mission (guest JoinGame error 6)
  passed twice after joins used player 1's race. Checks: both players join, API status stays "in game" until the win,
  the mission ends normally, banks sync, both hubs reopen.
- **Targeted tests** (one PC, test hooks): objective completion keeps both clients in game; a crash of either client
  returns both to the hub with banks intact (a truncated bank is restored); Stop and a lost connection end every wait;
  a setup mismatch is reported on both sides; cutscene camera identical on both clients; a guest-only Esc skips.
- **Two-PC sessions** by the owner and a friend over Steam (2026-10-19 and 2026-10-20): missions 1-3 to the Hyperion,
  several later missions, a victory, bonus objectives.
- Not covered: a full campaign playthrough in co-op, long sessions. The guest's input quality is judged by players.

## How to play

Not released yet. The intended flow: both players unzip the launcher, click Prepare (reads their own SC2 install),
tick Campaign and pick the campaign folder, then one hosts (Steam or address) and the other joins.

## Known issues

- Antivirus: Windows Defender's cloud ML has quarantined test builds (Trojan:Win32/Prowloc.A!cl), most likely because of
  the guest's mouse and keyboard hooks in an unsigned exe.
- The guest's commands react about one network round trip later than the host's.
- Mission Archives replays, the Simulator and the Arcade in the hub are not supported.
- Chat between the two hubs is not synced (each hub is a separate game).
- Some mission triggers react only to the host's input (tutorial prompts, mission choice dialogs).

## Next steps

- Separate armies mode (each player their own units and upgrades): see the checker's "Matters for separate armies".
- Vanilla Blizzard campaigns (different campaign library and bank names).
- Hub chat relay via API chat injection.
- [NOTES.md](NOTES.md) is the dated lab notebook.
