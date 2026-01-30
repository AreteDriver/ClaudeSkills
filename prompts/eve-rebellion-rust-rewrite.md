# EVE Rebellion Rust Rewrite

**Pattern:** generation  
**Domain:** development (game)  
**Complexity:** complex  
**Tool Requirements:** code execution

### Intent

A comprehensive 10-week development guide for rewriting the EVE_Rebellion Python/Pygame arcade shooter to Rust/Bevy. Covers multi-campaign architecture, ECS patterns, EVE-authentic HUD systems, and cross-platform deployment.

### Prompt

```
# EVE REBELLION - Complete Rust Rewrite Guide

## Project Overview

**Current State:** Python/Pygame arcade shooter
**Target State:** Rust/Bevy multi-campaign EVE arcade game suite
**Timeline:** 10-12 weeks
**Project Path:** ~/projects/EVE_Rebellion_Rust

---

## Technology Stack

| Component | Choice | Reason |
|-----------|--------|--------|
| Language | Rust 1.75+ | Performance, safety, WASM support |
| Engine | Bevy 0.13+ | ECS, cross-platform, active community |
| Audio | bevy_kira_audio | Low-latency, procedural |
| UI | bevy_egui | Immediate mode for menus |
| Input | leafwing-input-manager | Action-based mapping |
| Async | bevy_tokio_tasks | ESI API calls |

---

## Cargo.toml

```toml
[package]
name = "eve_rebellion"
version = "0.1.0"
edition = "2021"
authors = ["ARETE"]
description = "EVE Online arcade shooter suite"
license = "MIT"
repository = "https://github.com/AreteDriver/EVE_Rebellion"

[dependencies]
bevy = { version = "0.13", features = ["dynamic_linking"] }
bevy_kira_audio = "0.19"
bevy_egui = "0.25"
leafwing-input-manager = "0.13"
bevy_tokio_tasks = "0.13"

tokio = { version = "1", features = ["rt-multi-thread", "macros"] }
reqwest = { version = "0.11", features = ["json"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
rand = "0.8"

[profile.dev]
opt-level = 1

[profile.dev.package."*"]
opt-level = 3

[profile.release]
lto = "thin"
codegen-units = 1
```

---

## Project Structure

```
eve_rebellion/
├── Cargo.toml
├── src/
│   ├── main.rs                 # Entry point, app builder
│   ├── lib.rs                  # Library exports
│   │
│   ├── core/
│   │   ├── mod.rs
│   │   ├── game_state.rs       # GameState enum, transitions
│   │   ├── resources.rs        # Shared resources (score, currency)
│   │   ├── events.rs           # Custom events
│   │   └── constants.rs        # Game constants
│   │
│   ├── campaigns/
│   │   ├── mod.rs
│   │   ├── minmatar_rebellion/ # Campaign 1: Minmatar vs Amarr
│   │   ├── last_stand/         # Campaign 2: Caldari vs Gallente
│   │   ├── abyssal_depths/     # Campaign 3: Triglavian roguelike
│   │   ├── sansha_incursion/   # Campaign 4: Wave defense
│   │   └── elder_fleet/        # Campaign 5: Capital ships
│   │
│   ├── entities/
│   │   ├── mod.rs
│   │   ├── player.rs           # Player ship components
│   │   ├── enemies.rs          # Enemy types per faction
│   │   ├── projectiles.rs      # Bullets, missiles, lasers
│   │   └── pickups.rs          # Powerups, modules
│   │
│   ├── systems/
│   │   ├── mod.rs
│   │   ├── movement.rs         # Physics, steering
│   │   ├── combat.rs           # Damage, shields, armor
│   │   ├── ai.rs               # Enemy behavior
│   │   ├── spawning.rs         # Wave management
│   │   └── scoring.rs          # Combo, style grades
│   │
│   ├── ui/
│   │   ├── mod.rs
│   │   ├── hud.rs              # EVE-style capacitor, health
│   │   ├── menus.rs            # Main menu, campaign select
│   │   ├── upgrade_shop.rs     # Between-stage shop
│   │   └── overview.rs         # Targeting brackets
│   │
│   └── assets/
│       ├── mod.rs
│       ├── ship_loader.rs      # Load ships from Image Server
│       ├── sprite_cache.rs     # Runtime sprite cache
│       └── type_ids.rs         # EVE type IDs
│
├── assets/
│   ├── sprites/                # Cached ship renders
│   ├── fonts/
│   └── audio/
│
└── platforms/
    ├── web/                    # WASM build config
    ├── windows/
    ├── linux/
    └── macos/
```

---

## EVE-Style HUD System

Replace heat system with authentic EVE HUD:

### Capacitor (replaces Heat)
```rust
#[derive(Component)]
pub struct Capacitor {
    pub current: f32,      // 0.0 to max
    pub max: f32,
    pub recharge_rate: f32, // per second
    pub recharge_delay: f32, // delay after ability use
}

// Visual: Circular ring that drains/fills like EVE
// Weapons/abilities consume capacitor
// Runs out = can't fire/boost
```

### Health (Shield/Armor/Hull)
```rust
#[derive(Component)]
pub struct ShipHealth {
    pub shield: f32,
    pub shield_max: f32,
    pub shield_recharge: f32,  // passive regen
    pub armor: f32,
    pub armor_max: f32,
    pub hull: f32,
    pub hull_max: f32,
}

// Visual: Three nested arcs like EVE
// Shield (blue) recharges passively
// Armor (orange) requires repair
// Hull (red) = critical, no regen
```

### Speed Gauge (simplified thrust)
```rust
#[derive(Component)]
pub struct SpeedGauge {
    pub current_speed: f32,
    pub max_speed: f32,
    pub afterburner_active: bool,
}

// Visual: Single horizontal bar
// Afterburner pulses the bar
```

### Active Modules (powerup display)
```rust
#[derive(Component)]
pub struct ActiveModules {
    pub slots: Vec<Option<Module>>,
    pub max_slots: usize,
}

// Visual: Row of module icons (like EVE module rack)
// Cooldown overlay sweeps across each icon
// Click/hotkey to activate
```

---

## Campaign Architecture

### Campaign 1: Minmatar Rebellion
- Empire → Tribe → Ship selection flow
- Tribes: Brutor (armor), Sebiestor (drones), Vherokior (speed), Krusual (crits)
- Bosses: Amarr battleships, Avatar titan final

### Campaign 2: The Last Stand
- Faction choice: Caldari OR Gallente (mirrored campaigns)
- Different ship rosters per side
- PvP-style combat mechanics

### Campaign 3: Abyssal Depths
- Roguelike structure: Filament → Tier → 3 rooms
- Triglavian enemies
- Timer mechanic (escape before collapse)
- Permadeath option
- Mutaplasmid drops modify weapons

### Campaign 4: Sansha Incursion
- Wave defense
- Protect station objective
- Escalating difficulty
- Influence meter

### Campaign 5: Elder Fleet
- Capital ship gameplay (unlocks after Rebellion)
- Command multiple ships
- Strategic system selection
- Final boss: Entire enemy fleet

---

## Arcade Systems

### Scoring (Devil May Cry style)
```rust
#[derive(Resource)]
pub struct ScoringSystem {
    pub combo_count: u32,
    pub combo_timer: f32,
    pub style_points: u32,
    pub style_grade: StyleGrade,
}

#[derive(Clone, Copy)]
pub enum StyleGrade {
    D, C, B, A, S, SS, SSS
}

// Style kills:
// - Point blank (close range)
// - Graze (near miss then kill)
// - Environmental (asteroid collision)
// - Overkill (massive damage)
// - Chain (rapid sequential kills)
```

### Berserk System
```rust
#[derive(Component)]
pub struct BerserkMeter {
    pub current: f32,
    pub max: f32,
    pub active: bool,
    pub duration: f32,
}

// Fills from kills and style
// Activate: invulnerable + 2x damage
// Visual: Screen edge glow, ship trails
```

---

## Implementation Phases

### Phase 1: Core (Week 1-2)
- [ ] Bevy project setup
- [ ] Game state machine
- [ ] Basic player movement
- [ ] Ship sprite loading from Image Server
- [ ] EVE HUD framework

### Phase 2: Combat (Week 2-3)
- [ ] Projectile system
- [ ] Shield/Armor/Hull damage model
- [ ] Capacitor consumption
- [ ] Basic enemies
- [ ] Collision detection

### Phase 3: Minmatar Campaign (Week 3-4)
- [ ] Tribe selection UI
- [ ] 4 tribe ship variants
- [ ] Stage progression
- [ ] Boss encounters
- [ ] Campaign save/load

### Phase 4: Arcade Systems (Week 4-5)
- [ ] Combo/chain system
- [ ] Style grades
- [ ] Berserk meter
- [ ] Screen effects
- [ ] Score multipliers

### Phase 5: Additional Campaigns (Week 5-8)
- [ ] The Last Stand (faction mirror)
- [ ] Abyssal Depths (roguelike)
- [ ] Sansha Incursion (wave defense)
- [ ] Elder Fleet (capitals)

### Phase 6: Polish (Week 8-10)
- [ ] Procedural audio
- [ ] Particle effects
- [ ] Screen transitions
- [ ] High score tables
- [ ] WASM web build
- [ ] Platform releases

---

## CCP Attribution

Required in all builds:

```
EVE Online and the EVE logo are registered trademarks of CCP hf.
All ship images and EVE-related content are property of CCP.
Used in accordance with the EVE Online Content Creation Terms of Use.
https://community.eveonline.com/support/policies/content-creation-terms-of-use/

This game is a fan project and is not affiliated with or endorsed by CCP hf.
```

---

## Performance Budgets

| Metric | Target |
|--------|--------|
| Frame time | <16ms (60 FPS) |
| Entity count | 500+ simultaneous |
| Load time | <3s cold start |
| WASM size | <10MB compressed |
| Memory | <512MB peak |

---

Report completion of each phase. Test before proceeding.
```

### Variables

| Variable | Type | Required | Description |
|----------|------|----------|-------------|
| `{project_path}` | string | no | Override default project location |
| `{target_campaign}` | string | no | Focus on specific campaign |

### Model Assumptions

- **Best with:** Claude with code execution (for scaffolding)
- **Context needed:** None, self-contained
- **Tool dependencies:** Rust toolchain, cargo

### Example Input

```
Start Phase 1 of the EVE Rebellion Rust rewrite. Create the project structure and basic Bevy app.
```

### Example Output

```
## Phase 1: Core Setup

Created project at ~/projects/EVE_Rebellion_Rust

**Files created:**
- Cargo.toml (with all dependencies)
- src/main.rs (Bevy app entry point)
- src/core/game_state.rs (state machine)
- src/core/constants.rs (game constants)

**Verified:**
- `cargo check` passes
- `cargo run` shows blank window with correct title

**Next:** Implement player ship movement and sprite loading.

Ready to proceed to player movement?
```

### Failure Modes

| Symptom | Cause | Fix |
|---------|-------|-----|
| Dependency conflicts | Bevy version mismatch | Check crates.io for compatible versions |
| WASM build fails | Missing wasm32 target | `rustup target add wasm32-unknown-unknown` |
| Image Server 404 | Invalid type_id | Verify against EVE SDE |

### Iteration Notes

**v1 (Dec 2025):** Initial Python → Rust concept  
**v2 (Dec 2025):** Added EVE-style HUD (capacitor replacing heat system)  
**v3 (Jan 2026):** Added multi-campaign architecture with 5 campaigns

### Related Prompts

- `eve-esi-integration` — API integration for live data
- `bevy-ecs-patterns` — Common Bevy architectural patterns
