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
