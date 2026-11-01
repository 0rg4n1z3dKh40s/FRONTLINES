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

## Phase 2: AI behaviour and difficulty (done)
Built as planned. Notes from the build: all numbers are first guesses, to tune in play. Not yet seen in play: AI bombs, the invade preview and the 90% win screen.

- **Less bomb spam.** In `aiStrike`, at most one AI strike per month across the whole world (`lastAiStrike`). Each AI waits a cooldown after firing, set per difficulty by `strikeCd` in `DIFFS` (Easy 30 months, Medium 20, Hard 12, Extreme 6, each randomised by ±20%). Each shot adds "heat" to the AI, raising its bomb prices by 45% per heat (`AI_HEAT_COST`). Heat is capped at 4 and fades at 1 per 12 months (`aiHeat`). Only AIs pay it; the player's prices are unchanged.
- **Easy mode barely retaliates.** The roll lives in `startAttack` (which `aiHitBack` reads), not in `aiHitBack` itself. On Easy an invaded AI answers back 10% of the time, shrinking to 0 as it nears 30% of the land. Other difficulties keep the old odds.
- **Optional win at 90%.** At 90% of the land (`WIN_EARLY`) `checkEnd` shows "You win!" with **Continue** or **Finish** (the Finish button is the main-menu button, relabelled). Continue plays on toward `WIN_SHARE` (98%) or until no rivals are left, then the full victory screen shows. The win medal and quest trigger fire once. `won90` is saved with the game.
- **Survival chance on attacks.** The invade panel shows "N troops · X% chance to win" and both sides' population, always, with or without the assistant. It uses a new `survivalChance`, not `estimateSectors`: your troops against the people in the 12 enemy sectors nearest the target. Even strength is 50%, double is about 90%.

## Phase 3: More upgrades and bombs (done)

Built, with these notes. Levels now start at 0 and every stat is a formula in the constants block (`FACTORY_INCOME(l)` etc.), with `maxLevel` (100 for Factory, City, Spy HQ, Silo, Research Lab, otherwise 50) and `MILESTONES`. Real milestone abilities so far: silo bomb unlocks (toxic 2, firestorm 4, dirty 6, nuke 8, MIRV 15), factory income steps, radar chance steps; the rest are plain scaling and need milestone abilities in a later pass. Upgrade cost is `cost × 0.8 × (level+1)^1.35`, with an "Upgrade ×10" button. Saves before v4 are shifted down one level on load. MIRV count is `tiles / 6` (1 to 100) and lands over about two seconds; the AI doesn't fire MIRVs yet. Not yet played by feel: all costs, bomb stats and milestone levels are first guesses. News: removed AI-only alliances, ceasefires, joins, quakes, plagues, booms and shot-down lines; added "lost a silo/factory" lines.
- **Many levels, few real unlocks.** A newly built building is level 0. The first upgrade makes it level 1, and so on. Every level gives a small steady bump (a bit more money, a bit more population). Only milestone levels add real new abilities, for example 2–4 of them by level 10 for something like a Spy HQ, with the big ones further out. Costs rise steeply, so the per-level arrays (`FACTORY_INCOME`, `RADAR_RANGE`, etc.) become formulas plus a short milestone table per building. `upgradeBuilding` and the level cap (currently 3) change accordingly.
- **Level caps.**
  - Up to about 100 (late-game, the ones worth pouring money into): Factory, City, Spy HQ, Silo, Research Lab.
  - Up to 50: every other building (Barracks, Radar, Port, Power Plant, Farm, Bank, Fort, Bunker, Airbase, Hospital, Monument).
- **New bombs** in the `BOMBS` table, gated by silo level.
  - **MIRV (cluster warhead):** the strongest bomb. It splits near the target into submunitions, each as strong as the second-strongest bomb. The count scales with the target nation's size (1 for tiny, about 10 for medium, up to about 100 for large). It detonates once it is close enough to the target area.
  - A couple more mid-tier bombs between toxic and nuke.
- **More news items** in the side feed ("X bombed Y", "Y's silo destroyed", and similar). Remove the low-value ones that currently clutter it.

## Phase 4: Spies and fog of war (done)

Built as planned, with these notes. Hidden kinds are `HIDDEN_ORDER` (radar, bunker, silo, airbase, spy HQ, spy radar); the discovered ones live in `found` (saved). Spy ship ($6,000, needs a Spy HQ, built at a port) scans 6 sectors around it. Spy plane is sent from the enemy-land panel from any airbase within 3x its range + 20 and reveals 7 sectors around where it lands. Spy radar needs a Spy HQ at level 100 (`maxLevel('spy')`), so it is a very late unlock; its level sets how many hidden kinds it finds (one more every 7 levels) and level 50 sees stealth weapons, as does a level-50 radar. Spy bomb (silo 30) and spy warhead (silo 40) are hidden in flight and radars below level 50 can't shoot them down. The AI doesn't use spy weapons, ships or planes yet. Spy HQ perks: army +0.2% per level, city income and people +0.3% per level. Intel tiers are in `intel()` (1, 5, 10, 25, 50); at 10 an HQ also reveals hidden buildings in its range, and at 50 every hidden building of the tapped nation. Stats and two medals (Eyes Open, Spymaster) cover the spy agency; no quest yet. All numbers are first guesses and only checked by script, not by play.
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
