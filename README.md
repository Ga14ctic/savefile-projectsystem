# Project Codename: PROJECT://SYSTEM - Deep Dive Design & Architecture Brief  
### Roblox Next-Gen Fighting RPG

---

## I. Project Vision

### 1. What’s This Game About?

Codename **PROJECT://SYSTEM** is a truly player-skill-based, anime-inspired, PvPvE fighting RPG for Roblox, rooted in the “high skill ceiling” tradition but with all the modern usability and depth of the best action games.  
Drawing from games and media like **Deepwoken**, **Arcane Odyssey**, “Solo Leveling,” “Bleach,” and “Hunter x Hunter,” PROJECT://SYSTEM aims to blend:  
- Fast, reaction-based, parry-centric PvP combat  
- A rich progression system with stats, skill trees, and unlockable abilities  
- Deep weapon variety (every weapon is viable, nothing is strictly “the best”)  
- Player-run “official” and “unofficial” guilds, RP ranks, and a world that responds to social actions  
- Animations and VFX worthy of community Twitter clips  
- All while being **100% readable, modular, and open to expansion**

**Target Audience:** Players who want to “earn” victories through reactions, tactics, and good reads, not stats or rarity luck—seasoned PvPers, completionists, and “be the MC” roleplayers.  

---

## II. Why This Game, and Why Now?

Most existing Roblox fighters either:
- “Fake” skill with broken hitboxes or stat bloat
- Reward ultra-rare mythics with overwhelming numbers, draining long-term fun and tier diversity
- Frustrate with clunky UI or unclear progression
- Or, gate good combat behind randomness or long grinds

**PROJECT://SYSTEM** is a love letter to the young fighting game joy that hooked us (and our inspirations). It is:  
- Practical refinement of proven PvP/PvE core design  
- A modular codebase, not spaghetti  
- Designed to be content-expandable, even by community contributors later  
- Not dependent on any single dev, with a clear technical “prompt” for future AI assistants

---

## III. Structure: Systems, File Layout and Approach

### 1. Core Game Mechanics - Summary

#### a. Player Creation & Data
- Custom character creation, unique name (with validation), gender, starting faction  
- Persistent player data (saves across sessions), designed with versioning for easy future resets/migrations  
- Fields for guilds, skill trees, weapons, ranks, stats, kills, deaths, gold, exp, etc.  

#### b. Visual Hierarchy/UI
- Clean, mobile-supported custom UI for everything (no default Roblox elements)  
- **Menu (M/tab/tap):** Top-level, skill tree popup  
- **Skill tree:** All three trees shown at once, vertical moves, upgrade buttons, point indicators  
- **Hotbar:** 12-slot, moves only, no tools or inventory  
- **Leaderboard:** White text only, shows real username on hover, and colored guild tags below names  
- **Overhead**: Shows rank, name, official/unofficial guilds

#### c. Combat System (Current Build)
- All combat hit detection & state tracking is **server-authoritative**  
- Modular weapons system supports per-weapon M1 strings (multi-hit combos), crits, Z/X/C special moves  
- Skill tree moves are universal; weapon moves are unique per weapon  
- Weapon rarity impacts flair, not power. A common can beat a legendary—legendary = cooler effects, not higher DPS  
- True combos are nearly impossible; parries, blocking, dodging (directional), uptilt launcher, and universal air crit  
- **No mercy mechanic:** At 0 HP, players are KO’d—others can carry or grip (execute) them; grip is interruptible, carry is droppable if hit

#### d. Admin/Staff Tools
- F2 brings up admin panel (textbox with full auto-complete for every command/field in your schema)
- Commands include: setting stats, giving/removing weapons, manipulating skill points, new guild/rank assignment  
- Admin-only tools to manage guilds (including the invite-only “Monarch” staff guild)

#### e. Guilds (Social System)
- **Official guilds:** Unique colored titles and tags (e.g. "Hunter"), strict invite/promotion/kick functionality, Monarch uses a purple>pink gradient  
- **Unofficial guilds:** Player-created, data-only for now (to be expanded); yellow tag in white brackets under names  
- Full support for future expansion (dynamic ranks, guild bases, wages, etc.)

#### f. Progressive Plan
- Everything is built for **extensibility**, with clearly tagged sections for “MVP” and “to expand.”  
- VFX/animation system has placeholder calls so artists/animators can hook in later without code rewrite.

---

### 2. File Structure (Module/Script Overview)

**ReplicatedStorage**
- Platform.lua (PC/mobile detect)
- SkillTreeConfig.lua (*All* skill tree trees, moves, unlock logic)
- PlayerDataConfig.lua (Defines persistent player data default/fields)
- WeaponConfig.lua (All weapon stats, moves, hit logic)
- AdminConfig.lua (Who is admin, what commands are there, autocomplete meta)  
- GuildConfig.lua (Official + unofficial guild static data, color/rank lookup)  
- CombatConfig.lua (System-wide constants for combat, keybinds, cooldowns, stun durations)

**ServerScriptService**
- DataManager.lua (Handles all player data load/save, display replication)
- CharacterCreationHandler.lua (Handles name validation, gender, faction pick, skin assignment)
- OverheadDisplay.lua (Handles all overhead GUI — name, ranks, clan, guild, tags, colors)
- SkillTreeHandler.lua (Server-side validation for tree point spending, upgrades, move equip)
- AdminHandler.lua (Receives/validates admin commands, syncs data/display, supports dynamic command expansion)
- GuildHandler.lua (Run official/unofficial guild membership/state, incl. Monarch; supports admin-only Monarch invites)
- CombatHandler.lua (All combat: M1 state, moves, block, parry, dodge, damage application, connects to KOHandler)
- KOHandler.lua (Handles KO state, timed auto-death, carry, grip/execute + all related events)
- DummyHandler.lua (Creates nigh-invulnerable combat dummies for constant testing)

**StarterGui, StarterPlayerScripts**
- CustomLeaderboard/LeaderboardClient.lua (All leaderboard logic; hover = real username)
- MenuGui/MenuClient.lua (Handles menu/skill tree/hotbar UI and mobile action remapping)
- AdminGui/AdminClient.lua (F2 admin, search/autocomplete, full schema-aware interpreter)
- CharacterCreationGui/CharacterCreationClient.lua (Stylish, mobile-resilient character creator)
- MobileControls/MobileControlsClient.lua (Creates mobile action buttons for everything “huge button”)
- UIScaler/UIScalerClient.lua (Applies UIScale per device/screen — PC, phone, small tablet all readable)
- CombatGui/CombatClient.lua (Combat input, full key remapping, handles carry/grip input too)
- HideOwnOverhead/DisableDefaultUI (On player, removes all Roblox CoreGui/hud, disables built-in hotbar/backpack)

---

## IV. Design Philosophy/Key Rules

**1. Skill-first, never stat-first.**
   - No true combos. M1 stun is “escapeable” with good timing
   - Parries, dodges (directional, with i-frames/cooldown), uptilts, air crit
   - Universal moves for fairness, weapon moves only add flavor and options

**2. Weapons Are Movesets, Not Power Spikes**
   - Rarity is for style, not power (5% max stat difference between legendary/common)
   - Legendary = unique effects/VFX, not “You Must Use This To Win”
   - All weapons have full custom M1 strings, crits, Z/X/C
   - Per-weapon config supports easy addition/iteration

**3. Data Structures Are Object-Oriented & Modular**
   - Major systems (player, combat, guilds) are non-interlocking, replaceable in isolation
   - All interfaces (admin, trees, guilds) pass through robust remotes/events, server-authoritative always

**4. Clean UI, Mobile-first**
   - Everything is snap-scaled, mobile controls never block core actions
   - All UI disables Roblox stock interfaces (clean, immersive)

**5. True extensibility**
   - Commands/admin tools are data-driven; add new guilds/weapons/ranks, autocomplete “just works”
   - Guild system is futureproofed for major expansion (bases, alliances, wars, taxes, events)
   - Combat handler can handle Mob/Dummy NPCs as easily as players, no rework needed

**6. All VFX are pluggable**
   - Core calls (CombatVFX.FireAllClients) around every move, hit, block/parry
   - Placeholder basic VFX are included, but easily upgradeable per asset/animation team

---

## V. Current Progress — What Works, What’s In-Progress, What’s Planned

### A. **Systems that are FULLY COMPLETE:**
- Character creation + name validation, gender/faction/guild choices, persistent
- Data management and migration (DataStore auto-version bump = total player wipe for quick iteration)
- Complete skill tree system (config, UI, server validation, per-point bonuses, equip UI)
- Custom hotbar, totally respects 12 key slots, works on mobile and PC
- Custom leaderboard, modern look, full guild/guild rank/hover support
- Guilds/Unofficial guilds: official color/rank/title tags, Monarch gradient guild, staff-only invite, data system supporting both
- Admin panel (F2): Dynamic, autocomplete, context-aware; add new fields/commands and autocomplete understands them
- Weapon/skill tree/crit/air crit system: all moves, hitboxes, damage, hitstun, cooldowns defined in config

### B. **Systems IN-PROGRESS:**  
- Combat/KO: Players get KO’d at 0 HP; carry/grip system (with drop/cancel-on-hit/reflex timeouts) in place but VFX/animations are currently placeholders; grip breaks properly if gripper is hit
- Per-weapon M1s: Most variants working; easy to expand for unique weapons
- Dummy handler: Admin command supports spawning dummies anywhere for test (all weapons/damage logic applies to dummy too)
- Mobile support: Every action and menu has a touch counterpart; mobile-friendly scaling
- Status/KO indicators: “Knocked out” overhead label, grip/execute progress bar; minimal VFX

### C. **PLANNED / UPCOMING:**
- Deep VFX pass: Replace placeholder hit/block flashes with S-tier slash, element, and impact VFX per move/weapon
- Full animation pass: Every M1, move, block, dodge, parry, KO state anim per weapon (motion-matched + procedural if possible)
- Mob/AI Enemies: Dummies are a start; PvE challenge arenas, boss fights, guild bases
- Custom guild bases / worldbuilding
- Marketplace/loot: Drop tables, mob drops, unlockable gear, but all balanced so skill>rarity
- Questline/Story/NPCs: Start with basic XP/reward loop, then something actually good
- Cosmetic unlockables: Swaps for VFX, particle trails, skins — not pay-to-win, style only
- Voiceover/SFX system
- Replays/tutorial view mode
- Cross-server guild management and worldwide ranking (for competitive players)
- Rich API for contributors/community coders to extend game safely

---

## VI. DEVELOPMENT PLAN (Immediate Next Steps)

1. **Finalize KO System:** Make sure carry/grip fails properly when gripper or carrier gets hit; clean up ragdoll visuals; add more robust interruptions
2. **Combat Polish:** Tune hitboxes so every move is crisp; fix any “phantom whiff” bugs; add multi-hit support for Twin Fangs/Lacerate, etc.
3. **Combat VFX:** Replace placeholder “damage ball” effects with real slash, spark, shock, and element visuals
4. **Animation Integration:** Build or commission an initial “Basic” animation set per weapon, plug into `WeaponConfig` animation fields, update server/client to play correct anims per combo/move
5. **Mob Expansion:** Add simple pattern-based mob AI using same combat handler structure
6. **Progression Loop:** Add XP/gold gain per fight (even vs. dummies); NPC shop for basic weapon swaps; unlock cosmetic swaps for achievements
7. **Events/Marketing:** GM-led hosted events, global leaderboard races

---

## VII. HOW TO EXTEND THE GAME

- To add a **new weapon**: Add it to WeaponConfig (with all fields), model to ReplicatedStorage/WeaponModels, optionally new M1/move anims/VFX.
- To add an **official guild**: Add to GuildConfig, pick color/title, tweak any title gradient, then assign via admin panel.
- To add **new ranks/moves**: Edit SkillTreeConfig; UI/autocomplete/admin already supports extra moves/fields
- To expand VFX: Insert new hooks in CombatVFX event, don't touch combat logic

---

## VIII. MASTER PROMPT FOR CLAUDE OPUS 4.6

**Copy/paste the entire above brief as context. Then, add the following master prompt:**

---

**MASTER PROMPT FOR CLAUDE OPUS 4.6**

```
You are now the technical and design co-lead on a Roblox skill-based fighting RPG codenamed "PROJECT://SYSTEM."  
You have a complete, up-to-date description of the game's systems, code architecture, current files, and creative vision.

Your goals:
- Understand all game systems, why they exist, and how they connect
- Maintain/extremely improve code clarity, modularity, and extensibility at every step (treat all files as a persistent codebase, not just single scripts)
- When building new features or refactoring, never break working systems; always account for how changes affect downstream gameplay
- When making new systems (combat moves, VFX hooks, classes, guild features), follow the project’s extensibility conventions (data-driven configs, remotes, server-authority)
- Reference any of the prior technical context in this file for design, variable naming, code conventions, UX, admin tools, etc
- When unsure, explain your design decisions in comments as you implement

Task example:
"Build a new weapon called 'Blood Rose', Epic tier, with a unique multi-part crit move and a red-black rose petal VFX. Integrate fully into existing WeaponConfig, admin tools, and ensure dummies and player targets are handled without duplicating code. Provide all code and design notes as if ready to merge into project."

When you respond, assume you're talking to the main developer or a skilled AI assistant after you.  
All new code or suggestions should reference this project's brief/standards.

You are cleared to drive and evolve this game to completion.
```
