# Fury Tank Configuration Setup

## Project Context

This is a customization of the TankSim Classic WoW simulator, forked from https://github.com/Quadzet/Quadzet.github.io, focused on fury warrior tanking optimization. The original simulator was missing many leveling/pre-raid weapons and had generic defaults. This fork is being customized specifically for fury tank playstyle with pre-configured defaults.

## Fury Tanking Philosophy

- **Spec Balance**: Fury spec is balanced (not too aggressive, not too slow) and teaches survival techniques
- **Agility Focus**: Critically important for dodge chance + crit synergy with fury talents
- **Fast Weapons Preferred**: For Unbridled Wrath procs (generates extra rage)
- **Consistent Experience**: Tanking approach works from RFC at level 13 to 60 raids
- **Progression**: Start with shield + 1H until level 20, then convert to dual wield
- **Mitigation Strategies**: Piercing Howl for kiting, active threat/positioning management

## Configuration Changes Made

All default settings are configured in `/TankSim/main.js` in the `loadInput()` function (lines 177-299). The pattern used is:
```javascript
localStorage.getItem("settingName") ? localStorage.getItem("settingName") : DEFAULT_VALUE
```

If a value exists in localStorage (from previous use), it loads that. Otherwise, it uses the default value we've configured.

### Race Configuration
**File**: `/TankSim/main.js` line 180
- **Race**: Troll (selectedIndex = 6)
- **Reason**: Matches user's character race

### Talent Configuration
**File**: `/TankSim/main.js` lines 220-234

**Fury Tree (32 points total):**
- Cruelty: 5/5 (already set correctly)
- Unbridled Wrath: 5/5 (key talent for fast weapons - *not tracked in simulator*)
- Improved Battle Shout: 5/5 (*not tracked in simulator*)
- Dual Wield Specialization: 5/5 (dwspec = 5)
- Enrage: 5/5 (enrage = 5)
- Death Wish: 1/1 (deathwish = checked/true)
- Flurry: 5/5 (flurry = 5)
- Bloodthirst: 1/1 (bloodthirst = checked/true)

**Protection Tree (19 points total):**
- Shield Specialization: 5/5 (shieldspec = 5)
- Improved Bloodrage: 2/2 (*not tracked in simulator*)
- Toughness: 5/5 (toughness = 5)
- Improved Shield Block: 1/3 (*not tracked in simulator*)
- Last Stand: 1/1 (*not tracked in simulator*)
- Defiance: 5/5 (defiance = 5)

**Note**: The simulator only tracks talents that directly affect threat calculations. Talents like Unbridled Wrath, Improved Battle Shout, Improved Bloodrage, Improved Shield Block, and Last Stand are not simulated but are part of the actual build.

### Buff Configuration
**File**: `/TankSim/main.js` lines 242-277

**Consumables (Dropdown Selections):**
- OH Weapon Stone: Elemental Sharpening Stone (ohstone = 1)
- Strength Buff: Juju Power (strbuff = 2)
- Attack Power Buff: Winterfall Firewater (apbuff = 1)
- Agility Buff: Elixir of the Mongoose (agibuff = 1)
- Stat Buff: Spirit of Zanza (statbuff = 1)
- Food Buff: Desert Dumpling (foodbuff = 1)
- MH Weapon Stone: None (mhstone = 0)
- Alcohol: None (alcohol = 0)
- Potion: None (potion = 0)

**World Buffs (Checkboxes - all set to TRUE/enabled):**
- Gift of Arthas (goa = true)
- Elixir of Superior Defense (armorelixir = true)
- Dragonslayer (dragonslayer = true)
- Spirit of Zandalar (zandalar = true)
- Warchief's Blessing (wcb = true)
- DM Stamina (dmstamina = true)
- DM Attack Power (dmAP = true)
- DM Spell Crit (dmspell = true)
- Songflower (songflower = true)

**Raid Buffs (Checkboxes - all set to TRUE/enabled):**
- Battle Shout (bshout = true)
- Trueshot Aura (trueshot = true)
- Mark of the Wild (mark = true)
- Power Word: Fortitude (fortitude = true)
- Blood Pact (bloodpact = true)
- Windfury Totem (windfury = true)

**Other Settings:**
- Berserking (Troll racial): FALSE/disabled (berserking = false) - User does not use this
- Blessing of Kings: FALSE (kings = false)
- Blessing of Might: FALSE (might = false)
- Gift of the Wild: FALSE (pack = false)
- Flask of the Titans: FALSE (titans = false)
- HP Elixir: FALSE (hpelixir = false)
- Darkmoon Faire: FALSE (dmf = false)
- Inspiration: FALSE (inspiration = false)
- Devotion Aura: FALSE (devo = false)
- Improved Leader of the Pack: FALSE (imploh = false)
- Strength of Earth Totem: FALSE (strofearth = false)
- Grace of Air Totem: FALSE (graceofair = false)
- Improved Weapon Totems: FALSE (impweptotems = false)

## Testing the Configuration

To see the new defaults (after modifying main.js):

1. Open `/TankSim/index.html` in Safari
2. Open JavaScript Console (Cmd+Option+C)
3. Clear localStorage: `localStorage.clear()`
4. Refresh page (Cmd+R)
5. All settings should load with fury tank configuration

After first load, the simulator will remember whatever you last selected via localStorage.

## Next Steps

### Phase 1: Weapons (NOT STARTED - Previous weapon list was outdated)
- User needs to provide updated list of weapons to add
- Add weapons to `/TankSim/config.js` (dropdown HTML options)
- Add weapon stats to `/TankSim/stats.js` (stat objects with min/max damage, speed, stats)
- Test that simulations run correctly with new weapons

### Phase 2: UI Enhancements (NOT STARTED)
Potential improvements to make fury tanking use case clearer:
- Add weapon speed color coding (green=fast 1.5-2.0s, yellow=medium 2.1-2.6s, red=slow 2.7+s)
- Improve weapon display in dropdowns with inline stats preview
- Add weapon comparison mode
- Better mobile responsiveness
- Modernize dark theme aesthetics

### Phase 3: Advanced Features (OPTIONAL)
- Fury-specific optimization suggestions
- Explicit Unbridled Wrath rage generation tracking
- Weapon speed impact visualizations
- Save/load configuration presets

## Technical Details

### File Locations
- Main HTML: `/TankSim/index.html`
- Default configuration: `/TankSim/main.js` (loadInput function)
- Weapon lists: `/TankSim/config.js` (weaponlists object)
- Weapon stats: `/TankSim/stats.js` (large objects with item data)
- Simulation engine: `/TankSim/workers/worker.js`

### How Defaults Work
1. Page loads → calls `onLoadPage()` (line 301)
2. `onLoadPage()` calls `loadInput()` (line 303)
3. `loadInput()` checks localStorage for each setting
4. If not in localStorage, uses our configured default values
5. After first load, localStorage remembers user's selections
6. To reset to defaults: `localStorage.clear()` and refresh

### Important Notes
- The simulator uses localStorage to persist settings between sessions
- Clearing localStorage will reset everything to our configured defaults
- Changes to main.js only affect NEW sessions (or after clearing localStorage)
- Not all talents are tracked by the simulator - only ones affecting threat math
- The simulator is built with pure JavaScript, no build system required

## User's Character Context
- **Three Troll Warriors** (all level 60): Ðred, Ðire, and Ðed
- **Identical Setup**: All use the same 32 Fury / 19 Prot spec and fury tank philosophy (intended as clones)
- **Gear Progression Differences**:
  - **Ðred**: Raid geared from ZG, AQ20, MC, Ony
  - **Ðire**: Raid geared from ZG, AQ20, MC, Ony
  - **Ðed**: Fresh 60 with dungeon/UBRS gear
- Analyzes combat logs for threat generation, DPS output, damage taken
- Focus on Unbridled Wrath synergy with fast weapons
- Values agility for dodge + crit scaling with fury talents
