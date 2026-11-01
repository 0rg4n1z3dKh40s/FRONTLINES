# FRONTLINES design doc

Everything lives in `index.html`. Phases are ordered; each should be one commit or PR.

## Phase 1: Sound and feel (done)
Built as planned. Notes from the build: music defaults to "Generated" (switch to "Tracks" in settings), the coin sound plays quietly each month, and the nuke slow-motion only plays when the blast is on screen. Not yet play-tested by ear: tune the song and effect levels in `SONGS` and `SFX`.
- **Generated audio, no downloads.** Songs and sound effects are built with Web Audio, so nothing needs to be found or added to `audio/`. The three existing mp3s stay as an option, with a "Generated / Tracks" switch in music settings.
  - Songs: lobby, peace and war loops, using the existing `wantedTrack()` switching.
  - Sound effects: missile, toxic and nuke explosions, interception, build, upgrade, coin income, war declared on you, alliance, ceasefire.
  - The effects play from the existing events (`spawnBoom`, `intercept`, `news`, `placeBuilding`) and have their own on/off setting.
- **Win and lose effects** in `showEnd`: a stinger plus a screen effect (gold burst on a win, red fade on a loss).
- **Nukes** keep the existing flash and shake and get a rumble sound, a stronger flash and a short slow-motion beat.
- A short pulse on the sector when you build or upgrade.

## Phase 2: AI behaviour and difficulty
- **Less bomb spam.** In `aiStrike`, at most one AI strike per month across the whole world. Each AI has a long cooldown. Bombs cost an AI more each time it fires (price climbs over a few shots, then recovers slowly). Applies to every difficulty, including extreme, which just gets a shorter cooldown.
- **Easy mode barely retaliates.** In `aiHitBack`, easy-mode AIs do nothing when attacked, except a 10% chance to respond. The chance drops as the AI gets bigger.
- **Optional win at 90%.** At 90% of the land, show "You win" with **Continue** or **Finish**. Continue keeps playing toward `WIN_SHARE` (98%). Change is in `checkEnd`.
- **Survival chance on attacks.** Every attack preview shows a live 0–100% chance of winning, with both sides' population. Shown always, with or without the assistant. Built on `estimateSectors`.

## Phase 3: More upgrades and bombs
- **Many levels, few real unlocks.** A newly built building is level 0. The first upgrade makes it level 1, and so on. Every level gives a small steady bump (a bit more money, a bit more population). Only milestone levels add real new abilities, for example 2–4 of them by level 10 for something like a Spy HQ, with the big ones further out. Costs rise steeply, so the per-level arrays (`FACTORY_INCOME`, `RADAR_RANGE`, etc.) become formulas plus a short milestone table per building. `upgradeBuilding` and the level cap (currently 3) change accordingly.
- **Level caps.**
  - Up to about 100 (late-game, the ones worth pouring money into): Factory, City, Spy HQ, Silo, Research Lab.
  - Up to 50: every other building (Barracks, Radar, Port, Power Plant, Farm, Bank, Fort, Bunker, Airbase, Hospital, Monument).
- **New bombs** in the `BOMBS` table, gated by silo level.
  - **MIRV (cluster warhead):** the strongest bomb. It splits near the target into submunitions, each as strong as the second-strongest bomb. The count scales with the target nation's size (1 for tiny, about 10 for medium, up to about 100 for large). It detonates once it is close enough to the target area.
  - A couple more mid-tier bombs between toxic and nuke.
- **More news items** in the side feed ("X bombed Y", "Y's silo destroyed", and similar). Remove the low-value ones that currently clutter it.

## Phase 4: Spies and fog of war
- **Hidden buildings.** Enemy AI buildings are hidden by default. Factories, cities and ports stay visible. Radar, silo, bunker, spy HQ and airbase are hidden until found. Allies' buildings are always visible.
- **Finding hidden buildings:** a spy ship, and the spy radar. The spy ship needs a Spy HQ and a Port together, is very expensive, and auto-scans the area it sails through.
- **Spy radar.** Unlocked once a Spy HQ is at max level. Each spy radar level detects more kinds of hidden things. Its top level also detects spy weapons in flight.
- **Stealth weapons.** Spy bombs, spy planes and the spy warhead are late-game and very expensive. They are undetectable in flight to everyone except the sender, and only show up when they land. A max-level radar or a top-level spy radar can detect them.
  - **Spy plane:** a new unit launched from an airbase. It flies to a target area unseen, lands, and reveals every hidden building within a radius. It does no damage.
  - **Spy bomb / spy warhead:** stealth versions of the missile and the MIRV that use the same rules but are hidden in flight.
- **Spy HQ perks** (scale with level, expensive): a few percent attack-strength bonus for your army, and a permanent population and income boost for cities. Start modest, since cities already double base income, and tune in play.
- **Spy intel on nations.** Tapping a nation shows more the higher your best Spy HQ level is. Roughly:
  - Level 0: name, size and rough strength.
  - Level 1+: exact population and gold.
  - Level 5+: building counts by type.
  - Level 10+: hidden buildings inside the HQ's range, and who they are at war with.
  - Higher milestones: what they are planning to build or fire, and the live "map" of their land with hidden buildings marked.
- A **spy agency stat** (scans done, buildings found) in the stats and quests.

## Phase 5: Assistant, onboarding and polish
- **Background assistant.** On by default, with a toggle next to the sound button and in settings. It has no idle timer and no auto-attacks. It only places basic buildings and makes small upgrades in the background, so a player who is busy invading or has no idea what to build isn't sidetracked. It never touches big purchases and keeps a gold reserve.
- **Tutorial / beginner round.** A button at the bottom of How to Play starts a round easier than easy. Corner prompts replace the news feed and walk the player through placing buildings, expanding and defending a scripted first invasion. The AIs follow a fixed script. It can be replayed any time. The first real round gets a small money bonus after the tutorial.
- **Update button.** The game is hosted on GitHub Pages. On load it fetches its own `index.html` (cache-busted) and remembers a hash of it. It re-fetches every few minutes, and when the hash changes, the update is live on Pages and not just committed. This avoids reloading into the old version during Pages' deploy delay. When found, an update icon slides down at top center. Clicking it asks to confirm, reloads, and drops the player into their saved game, paused. The existing autosave handles the save.
- **Custom flag.** In settings, upload an image or video as your flag. It is stored locally (cloud save only if small enough) and cleared by "reset all data".
- **Chests.** Chests come from "firsts" and milestones (not from quests): first nuke, first alliance, first city, first intercept, first win on each difficulty, first spy scan, 10 nations conquered, and so on. Each can be earned once, and the list should be long so there is always a next one. The player clicks the chest to open it with an animation.
  - **Permanent rewards:** badges, flag frames, and a tiny starting-gold bump.
  - **Next-game-only rewards:** bonus gold, or a free bomb (including a golden bomb that looks special).
- **Badges.** A new Badges screen lists every badge, with earned ones lit and locked ones greyed out with a hint. Badges come only from chests. The player can pick one to show next to their name. Stored with the save.

## Not planned yet
- **Multiplayer.** Needs a server or realtime backend and is a separate project.
