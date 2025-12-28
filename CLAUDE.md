# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TankSim is a discrete-event Monte Carlo simulation engine for analyzing World of Warcraft warrior tank threat generation and pull variance. The project simulates thousands of combat iterations to provide statistical analysis of threat per second (TPS), damage output, and threat variance metrics.

## Repository Structure

This repository contains three separate simulation implementations for different WoW versions:

- **TankSim/** - Classic WoW (Vanilla/1.60) - Fury/Protection hybrid (dual-wield) tanking focus
- **tanksimsod/** - Season of Discovery (SoD) - Most feature-rich version with data-driven architecture using CSV files, rune system, and configurable rotations
- **TankSimTBC/** - The Burning Crusade - Protection spec focus with manual stat input

Each directory is a standalone web application with its own `index.html` entry point. The three versions share similar architecture but have different mechanics, items, and features.

## Core Architecture

### Simulation Engine

The simulator is an **event-driven, time-stepped simulation** that uses Web Workers for parallel processing:

1. **Main Thread** (`main.js`): UI management, spawns Web Workers (one per CPU core), aggregates results, generates statistical visualizations using Plotly.js
2. **Worker Threads** (`workers/worker.js`): Execute parallel simulation iterations, process events in discrete timesteps (default 10ms), return aggregated results
3. **Event Processing**: Each timestep processes ability usage, auto-attacks, aura ticks, proc triggers, and generates event objects that are analyzed for threat/damage

### Key Module Responsibilities

- **config.js**: Massive data dictionaries for items/enchants/buffs (TankSim/TBC) or configuration logic (SoD uses CSV files). The `updateStats()` function transforms UI selections into the global config object containing tankStats, bossStats, and simulation parameters
- **actor.js**: Actor class representing Tank or Boss entities. Manages dynamic stat calculation (armor, AP, crit), rage generation/consumption, and contains arrays of abilities/auras/procs
- **abilities.js**: Ability class hierarchy. Each ability extends base Ability class and implements `use()` (damage/threat calculation), `isUsable()` (rage/cooldown/GCD checks), and `threatCalculator()`
- **attacktable.js**: Classic WoW combat formulas implementing attack table calculations (miss/dodge/parry/block/glance/crit), weapon skill mechanics, armor reduction, and parry haste
- **auras.js**: Buff/debuff system using observer pattern. Auras implement `handleEvent()` to react to combat events and `handleGameTick()` for duration tracking. Includes stacking mechanics (e.g., Sunder Armor)
- **procs.js**: On-hit effect system for weapon procs (Thunderfury, Perdition's Blade, etc.). Uses event listener pattern to trigger on damage events
- **rotation.js**: Priority-based ability rotation logic. TankSim/TBC use hardcoded priorities in worker.js, SoD has configurable rotation with user-defined rage thresholds
- **stats.js**: Item stat database (TankSim/TBC: ~150KB hardcoded JavaScript objects; SoD: CSV loader using PapaParse)
- **attacktable.js**: Implements WoW combat mechanics including two-roll vs one-roll systems, glancing blow calculations, and separate attack tables for Tank→Boss vs Boss→Tank

### Event System

Events are the fundamental data structure:
```javascript
{
  type: "damage" | "buff gained" | "buff lost" | "spell cast",
  timestamp: number,
  source: "Tank" | "Boss",
  ability: string,
  hit: "miss" | "dodge" | "parry" | "block" | "glance" | "hit" | "crit",
  damage: number,
  threat: number
}
```

### Configuration Object

Workers receive a global config object:
```javascript
{
  tankStats: {
    // Base stats
    level, startRage, dualWield,
    // Computed stats from gear/buffs/talents
    AP, crit, hit, armor, dodge, parry,
    // Weapon stats
    MHMin, MHMax, MHSwing, OHMin, OHMax, OHSwing,
    // Modifiers
    threatMod, damageMod, critMod,
    // Talents and set bonuses
    talents: { impHS, impSA, defiance, ... },
    bonuses: { ... }
  },
  bossStats: {
    armor, defense, dodge,
    MHMin, MHMax, MHSwing
  },
  config: {
    iterations: 10000,
    simDuration: 12, // seconds
    timeStep: 10, // ms
    breakpointValue: 3000, // threat goal
    breakpointTime: 3000, // at time (ms)
    // Debuff application delays
    debuffDelay, ieadelay, faerieFire, CoR, IEA
  }
}
```

## Important Coding Conventions

- **No build system**: Plain ES6 JavaScript loaded directly in browser, no transpilation or bundling
- **Class-based OOP**: Uses ES6 classes (Ability, Aura, Proc, Actor) with inheritance
- **Rage costs are negative**: `actor.addRage(-cost)` to consume rage
- **Timestamps in milliseconds**: All time calculations use ms
- **Percentages as whole numbers**: 5% crit = 5 (not 0.05)
- **Two-roll vs one-roll attack tables**: Yellow abilities (special attacks) use two-roll, white attacks (auto-attacks) use one-roll
- **Global state in workers**: SoD uses `Actors` global object for Tank/Boss references
- **Threat multipliers built into abilities**: Each ability has threatScaling and staticThreat properties

## Common Development Tasks

### Adding a New Ability

1. Create a new class in `abilities.js` extending the `Ability` base class
2. Implement required methods:
   - `use(attacker, defender)` - Execute ability, return event object
   - `isUsable(attacker, defender)` - Check rage/cooldown/conditions (optional)
   - `threatCalculator(dmg_event, attacker)` - Calculate threat from damage (optional override)
3. Add the ability instance to Tank or Boss abilities array in `workers/worker.js`
4. Add to rotation priority logic if needed (worker.js for TankSim/TBC, rotation.js for SoD)

### Adding a New Aura/Buff

1. Create a new class in `auras.js` extending the `Aura` base class
2. Implement `handleEvent(owner, event, events, config)` for trigger logic (on-gain, on-crit, etc.)
3. Implement `handleGameTick(ms, owner, events, config)` for duration decay (optional)
4. Add to `defaultTankAuras` or `defaultBossAuras` array, or conditionally via `addOptionalAuras()`

### Adding New Gear/Items

- **TankSim/TankSimTBC**: Add item data to the appropriate object in `stats.js` (e.g., `head`, `neck`, `shoulder`, etc.)
- **tanksimsod**: Add item data to the appropriate CSV file in `data/` directory

### Modifying Rotation Priority

- **TankSim/TankSimTBC**: Edit the `performAction()` function in `workers/worker.js`
- **tanksimsod**: Edit `performAction()` in `rotation.js` and add corresponding UI controls in `index.html`/`main.js`

### Testing/Debugging

- Reduce iterations (e.g., 100 instead of 10000) and fight length (e.g., 5 seconds) for faster iteration
- Uncomment `formatEvent()` calls in `workers/worker.js` to enable combat log output to browser console
- Use browser DevTools to set breakpoints in worker code
- Check the results table and Plotly graphs for statistical output

## Recent Changes

See git log. Recent commits have focused on:
- Season of Discovery rune implementations (Sword and Board, Deep Wounds)
- Combat mechanic fixes (crit damage calculation, rage costs)
- Bug fixes for specific abilities and procs

## Technical Notes

- **localStorage persistence**: UI selections are automatically saved/loaded via `saveInput()`/`loadInput()` in main.js
- **Web Worker message passing**: Config sent via `postMessage()`, results collected via `onmessage` handler
- **Statistical analysis**: Results include mean, standard deviation, quantiles (1st/5th percentile), and threat breakpoint success rate
- **Performance**: Parallelization across CPU cores allows 10,000+ iterations in seconds
- **SoD-specific**: Uses PapaParse library for CSV parsing, actual WoW spell IDs for talents/runes, dynamic UI generation based on equipped runes/talents

## Current Work in Progress: UI Modernization (TankSim/Classic)

### Completed
- ✅ Created external stylesheet (`TankSim/style.css`) with mobile-first responsive design
- ✅ Converted `index.html` from UTF-16 to UTF-8 encoding
- ✅ Added viewport meta tag and semantic HTML improvements
- ✅ Implemented GOV.UK design principles (WCAG AAA contrast, system fonts, relative units)
- ✅ Responsive breakpoints: Mobile (single column), Tablet (768px, 2 columns), Desktop (1024px, 3 columns)
- ✅ Accessibility features: 3px focus outlines, prefers-reduced-motion, high-contrast mode support
- ✅ Fixed-width labels for dropdown alignment (Gear: 100px, Buffs: 120px, Stats: 100px)
- ✅ Text-overflow ellipsis for long item names in dropdowns
- ✅ Gear/Enchants horizontal alignment (31px fixed row height for both columns)

### Outstanding Issues to Fix

#### 1. Box Size and Spacing Inconsistencies
- **Dropdown heights**: Different sections have inconsistent select element heights
- **Input field sizing**: Number inputs and text inputs don't align consistently
- **Padding variations**: Some sections have different padding causing misalignment
- **Need**: Standardize all form element heights and padding across entire UI

#### 2. Remaining Alignment Issues
- **Main Hand/Off Hand section**: Weapon type selects and weapon dropdowns may not align properly
- **Talents section**: Input boxes for talent points need consistent sizing with labels
- **Stats Deltas section**: Input fields on right side may have alignment issues
- **Need**: Verify and fix alignment in all form sections beyond Gear/Enchants

#### 3. Responsive Behavior Improvements Needed
- **Mobile view**: Form inputs stack but may need better spacing/sizing on small screens
- **Tablet view**: Some sections might overflow or wrap incorrectly at 768px breakpoint
- **Touch targets**: Ensure all interactive elements meet 44x44px minimum on mobile
- **Need**: Test on actual mobile devices and refine breakpoints

#### 4. Visual Polish
- **Disabled elements**: Some disabled selects (empty enchant slots) need better visual treatment
- **Hover states**: Not all interactive elements have consistent hover feedback
- **Focus states**: Yellow focus outline may need adjustment for better visibility in some contexts
- **Need**: Comprehensive pass on all interactive states

### Layout Structure Notes
- `#Grid`: Main container, 3-column grid at desktop (450px + 300px + 300px)
- `#leftCol`: Nested grid containing #gear (300px) and #enchants (150px) side-by-side
  - `#bottomleft`: Below gear/enchants, contains #talents and #bonuses in 2-column grid
- `#middleCol`: Contains #buffs
- `#rightCol`: Contains #playerstats (stat deltas inputs)

### Key CSS Classes
- `.hInput`: Flex container for all form rows (label + input/select)
  - General: `padding: 0 0 3px 0`
  - Gear/Enchants only: `height: 31px` for horizontal alignment
- Label widths vary by section to align dropdowns vertically within each column
- Select elements have `text-overflow: ellipsis` for long item names

### Next Steps for Claude Web
1. Audit all form sections for consistent box sizing and alignment
2. Create comprehensive spacing/sizing system (perhaps add more CSS custom properties)
3. Test responsive behavior at all breakpoints in Safari
4. Fix any remaining overlaps or misalignments in Talents, Stats, Main Hand sections
5. Polish visual states (hover, focus, disabled) across all elements
