# StarCraft II campaign co-op: lab notes

Dated notebook. Each entry separates what was observed in a run from what we derived and what is still unknown.
"Host" is the player who creates the game (player 1), "guest" the one who joins. "One-PC" tests run two real game
clients on one machine from a copied install with separate temporary profiles; "two-PC" sessions were played by
testers over the internet. Game build 5.0.17.98310, enUS, Windows.

## 2026-10-16: assessment and first API spikes

Sources: [s2client-proto](https://github.com/Blizzard/s2client-proto) (`sc2api.proto`, `docs/protocol.md`),
Archipelago's SC2 client (single player, launches campaign missions through the same API), the LordDz Heart of the
Swarm Co-Op Arcade thread, SC2CCM (campaign file manager, no multiplayer).

Observed:
- The retail client's API creates games of any local map. A loose map folder is refused ("map doesn't exist"); a
  packed `.SC2Map` works.
- Two API clients on a stock ladder map join as players 1 and 2. With an alliance that shares control and spending,
  the second client moves player 1's marine and trains at player 1's barracks with player 1's minerals; production
  completes as a player-1 unit. With control/spending removed (vision kept), the same commands fail with action
  result 87: ownership is enforced, the commands only work through the alliance.
- A stock campaign mission (The Outlaws) with an added user slot 6 for the guest runs 450 game seconds with its
  objectives, AI and attack waves intact on both clients.
- The guest's client crashes on its first `RequestObservation` when its player id is above 3. Crash code reads a
  table at index `3 - player` (a 1v1 assumption). Guests never request observations; the host's does the reading.

## 2026-10-16: native input and the click bridge

Observed (stock map and campaign mission, real `SendInput` clicks):
- The creating client's native world clicks only select units with `InterfaceOptions.raw = true`.
- The joining client's world clicks and drags produce no action at all, whatever the interface options. Hover,
  minimap clicks and menus work. Swapping start order doesn't change it: the creator works, the joiner doesn't.
- The joiner's own API feature-layer actions (`unit_selection_point`, `unit_command`) succeed and show in its UI.

Built: the guest's "click bridge". Low-level mouse/keyboard hooks on the guest's game window translate world clicks
into feature-layer API actions (select, box select, smart command), using a window-to-feature-layer calibration.
Targeted commands (attack-move, abilities, build placement, gathering) work through a small map script: the map
reports the guest's pending targeting ability in a bank, the bridge sends a probe order, and the script swaps it for
the real ability one game loop later (nearest unit within 1.25 when a point target is invalid, e.g. gather).

## 2026-10-16/17: networking

Observed:
- API multiplayer only works when every client connects to a loopback address (another address fails JoinGame with
  error 12; pysc2 documents the same limit). The launcher therefore relays the four game ports between the PCs: one
  TCP connection (port 6600, UPnP best effort) or Steam's P2P networking under Valve's Spacewar app id (480). Only
  the host needs to be reachable.
- One-PC runs with different port bases don't exercise the relay's UDP path (the games talk directly over loopback);
  only two-PC sessions do.
- Steam: lobby, friend list, invites and the relay work between two real accounts. Everything first went over the
  reliable channel; game UDP datagrams now go unreliable when both sides support it. Game traffic is small (a few
  hundred KB per 10 minutes), but most of it is TCP game traffic on the reliable channel, so relay packet loss shows
  up as stutters (one session: 100% delivery for 4 minutes, then 91-97% on one side with RTT spikes up to 8 s).
- The move delay the guest feels is mostly SC2's own command delay (0.1-0.15 s on one PC) plus one network round trip.

## 2026-10-17 to 10-19: first two-PC sessions, guest UI

Two-PC sessions over Steam (The Outlaws, then campaign missions) worked: both players play, the guest's moves,
attacks and builds go through. Fixed from tester reports: the guest's drag box (drawn by the launcher as an overlay),
the guest's resource bar (shows player 1's shared minerals/gas/supply), pings for both players, the guest's camera
keys, click drift when the guest zooms (the calibration now follows the guest's camera distance and pitch), a
debugger window that some campaign maps open (minimized once), and a missing-mod join error (the launcher now checks
every map's mod dependencies first and names missing mods).

## 2026-10-18: campaign hub (story map)

Observed:
- Blizzard's own hub map (TStory01, repacked) and a WoL@Kit hub both run as solo API games: intro, mission screen,
  launch. `RequestGameInfo` on the vanilla hub crashed the client.
- Launching a mission ends the hub game for the API (player 1 victory, status 5). The client doesn't load the next
  map; 7-20 s later it crashes while following `GameSetNextMap`. The launcher quits the hub client right away.
- Neither library writes the picked mission to a bank, so the hub's launch call is wrapped to record it.
- Player-1 banks are sent to the other client at game start: in a two-participant mission the guest's empty profile
  read the host's bank value.

Built: the campaign flow. Each player runs their own hub as a solo API game; a pick in either hub closes both and the
two launchers start the mission as a multiplayer game; afterwards the host's campaign banks go to the guest and both
hubs reopen.

## 2026-10-19: making the campaign loop robust

- **Objectives:** any objective set to completed or failed makes the API end the game for both clients (the guest's
  click bridge is then refused). Objective state calls in mission scripts and in Kit's library are routed so the
  engine state stays active and the objective's name shows " (Completed)" / " (Failed)"; reads return the real state.
- **Setup check at connect:** launcher, campaign maps and every mod are hashed (SHA-256) on both PCs; differences are
  listed by file before anything starts.
- **Crash recovery:** banks are autosaved before each mission; if either client crashes or closes, both return to the
  hub with the campaign as before the mission, and a damaged bank is restored.
- **Start retry:** SC2 sometimes exits with -2000/-2001 right after start; a second start works.
- **Transitions:** the hub closes 1.5 s after a pick. Reusing a running SC2 process for the next game doesn't work: a
  process that already played a game hangs on its next multiplayer join. A loading curtain covers the gap between game
  windows.
- **Names and colors:** player names and team colors are chosen in the launcher and exchanged at connect.

## 2026-10-20: cutscenes, pings, all missions

- Cutscenes: 13 camera functions aimed at player 1 also run for the guest, so both see the same shots and talking
  heads; a skippable scene needs one player's Esc instead of everyone's.
- Native pings for the guest (Alt+click), a ping panel, and each player's camera shown on the other's minimap.
- Smoke test of every mission of a 29-mission WoL@Kit campaign (hub, chat-command launch, `-coopwinnow` after 45 s, both
  hubs back): 28/29 passed; the last failed with JoinGame error 6 until the guest joined with player 1's race
  (Protoss missions), then passed twice.
- `campaign-check`: a static map checker that lists, per map, what the launcher handles and what needs the map author
  (see [PORTING.md](PORTING.md)).
- Windows Defender's cloud detection quarantined the unsigned test exe several times (Trojan:Win32/Prowloc.A!cl), each
  time as the guest's input hooks started. Earlier unexplained "killed" test runs were this. No exclusions were added.

## 2026-10-21: hub features and team colors

- Team colors are game data (CGame TeamColors); campaign mods rename, recolor and append entries. The launcher rebuilds
  the color table from a campaign's mods, and custom RGB colors are appended to each mission's table.
- Hub chat: each hub is a separate game, and a running game doesn't re-read a bank file changed on disk. So lines go
  out through a bank and come back in as API chat; the hub hides SC2's chat display and draws the lines itself.
- Mission Archives replays (WoL@Kit) are recorded like star map picks.
- A local Galaxy variable named `text` (a type name) silently disables the whole map script.

## 2026-10-21: separate armies

Experiment on The Outlaws (WoL@Kit), then the real thing on a tester-made map with a second user slot:
- Mission script rewrites: player-1 unit groups and ownership tests count both armies; money and tech setters for
  player 1 are mirrored; units given to player 1 once the mission is under way (`UnitSetOwner`,
  `libNtve_gf_RescueUnit`, unit creation) go to the player whose units are nearest, alternate, or go to both.
- Resource pickups: some are scripted (a campaign-library call that pays the picker), others are pure game data (the
  Liberty pallets' auto-cast ability runs a player-modify effect on the picker). The data ones are caught with the
  pickup ability's execute event; the other player gets the same amount or half.
- Observed in one-PC tests: the guest joins in the map's slot with its race; both armies count for objectives; defeat
  comes only when both are gone; rescued units go to the player standing next to them; pickups pay both.
- Pitfall: helper functions placed before the mission script can't read the mission's globals (Galaxy needs a
  declaration first). Referencing one made the whole map script fail silently (no briefing, 0 minerals). Wrappers now
  sit after the mission script with prototypes before it.
- Alliances: the guest copies player 1's alliance with every other player except control and spending. Copying only
  hostility left AI allies' shared vision with player 1 alone (the guest didn't see an allied AI's units).

## 2026-10-21/22: campaigns on Blizzard's own scripts

Survey: four vanilla-based WoL mods keep Blizzard's hub logic, each with the same five `libCamp_gf_LaunchMission`
calls. Kit's libraries are a renamed copy of Blizzard's `libCamp`, so most hooks map one to one (see PORTING.md).
Banks are plain player-1 banks listed in each map's BankList.xml.

Built: the hub's `libCamp_gf_LaunchMission` calls go to a copy of the library function (A/B mission choice, briefing,
`PrepareForMission`, game over) that records the mission it finally launches; test commands use libCamp's names.

Observed (one PC): hub, then four missions (including a Protoss one), each won and back to both hubs; a WoL@Kit
campaign regression run passed. Full smoke runs of three vanilla-based mods are in progress.

Pitfall found on the way: a joining client that is killed keeps its copy of the shared mission map. When the next
session saved a different map under the same name, the joiner stored it as "~00" but loaded the stale copy, and every
join failed with ChecksumError (11) after the 3-minute wait. Map names now include a hash of the content.

## Open questions

- More than two human players: the protocol takes a list of client ports, so 3-4 may work; untested.
- Heart of the Swarm campaigns: several hub maps, the launch call lives in the campaign library's UI, Kerrigan's
  progress in banks; not started.
- Long sessions and a full co-op campaign playthrough.
