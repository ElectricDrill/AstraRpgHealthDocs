---
_layout: landing
---

> [!NOTE]
> Join the Astra RPG Discord server!  
> There is now a dedicated **Discord server** for Astra RPG Framework and its extensions.
> Join to **receive notifications** about new extension releases and important updates, to **ask for new features**, **report bugs**, **share ideas**, and **showcase your Astra creations** with other developers.  
> <span style="font-size:1.18em; font-weight:600;">💬 Join the Discord Server: https://discord.gg/nJVRMkGrZg</span>

## 💚 Astra RPG Health

### 👉 Introduction

**A complete health and damage system built on Astra RPG Framework's data-driven foundation.**

Astra RPG Health adds robust health management to your entities. Handle damage, healing, death, and resurrection with a flexible pipeline that adapts to your game's mechanics.

Simple to set up, powerful in action. Customize damage types, reduction formulas, and calculation pipelines without touching code.

---

### ✨ Key Features

- ❤️ **Health Management:** Complete system for HP, damage, healing, death, and resurrection
- 🗡️ **Damage Types & Sources:** Define custom damage types with defensive stats and armor penetration. Define Damage Sources for damage tracking, or advanced game mechanics
- 🛡️ **Damage Reduction:** Multiple reduction functions—flat, percentage, logarithmic, or custom
- ⚙️ **Damage Pipeline:** Fully customizable calculation pipeline with reorderable steps
- 🩹 **Healing System:** Heal sources and mechanics that integrate seamlessly with your game
- 🧛 **Lifesteal:** Configurable lifesteal that works with any damage type
- 💀 **Death & Resurrection:** Flexible strategies for death behavior and bringing entities back
- 📈 **Health Scaling:** New scaling component for formulas based on current, max, or missing HP
- ⚡ **Health Events:** Extended game events for damage, healing, death, and resurrection

---

### ⚙️ Implemented on top of `ScriptableObjects` architecture

- 🎨 **Inspector-driven:** Configure damage types, sources, and strategies directly in the Unity Inspector
- 🧩 **Modular and reusable:** Share damage calculation strategies across entities or customize per-entity
- 🧪 **Test in play mode:** Tweak damage formulas, reduction functions, and pipeline steps without recompiling and without restarting play mode

---

### 🔍 Features Breakdown

#### ❤️ Health Management

A new MonoBehaviour component adds health management to any entity. Entities can take damage, heal, die, and resurrect.

Set maximum health, track current health, and configure death thresholds. Everything integrates with the core framework's stats and attributes.

#### 🗡️ Damage Types

Damage types represent categories of damage in your game—Physical, Magic, Fire, Poison, etc. Define your own types as `ScriptableObjects`.

Optionally link each damage type to a defensive statistic. For example, "Physical Damage" reduced by "Armor", or "Magic Damage" reduced by "Magic Resistance".
True damage can be implemented as a damage type with no associated defensive stat.

Assign piercing statistics to ignore portions of defense. "Armor Pierce" could reduce the effectiveness of "Armor" when calculating net damage.

#### 🛡️ Damage Reduction Functions

Model how defensive stats reduce incoming damage using mathematical functions. The package includes three common functions:

- **Flat reduction:** Each point of defense reduces damage by a fixed amount. Predictable and easy to balance.
- **Percentage reduction:** Defense directly expresses damage reduction percentage. Effective across varying damage ranges.
- **Logarithmic reduction:** Diminishing returns as defense increases. Prevents complete damage negation.

Implement custom reduction functions if the defaults don't fit your needs.

#### 🛡️ Defense Reduction Functions

Piercing stats use reduction functions too, determining how they bypass defense.

The same flat, percentage, and logarithmic functions are available out of the box.

#### ⚙️ Damage Calculation Pipeline

The damage calculation pipeline transforms raw damage into net damage through customizable steps. Configure the order of operations to match your game's mechanics.

Pre-defined steps include:

- **Defense mitigation:** Apply defensive stats and piercing calculations
- **Damage modifiers:** Apply percentage-based buffs/debuffs for weaknesses, resistances, immunities
- **Barrier absorption:** Temporary hit points absorb damage before actual HP
- **Critical multiplier:** Multiply damage for critical hits

Set a default pipeline for all entities, or assign custom pipelines to specific entities. Override pipelines at runtime for temporary effects or boss mechanics.

For example, add a damage limiter step to boss entities to cap incoming damage at 10% of max HP. Or implement a debuff that doubles electric damage by swapping in a custom pipeline with an extra electric damage multiplier step.

This flexibility makes implementing complex damage mechanics straightforward.

#### 🩹 Heal Sources

Heal sources identify where healing comes from—Potion, Skill, Regeneration, Lifesteal, Ally, Self, etc.

Use heal sources to implement mechanics like bonus healing from specific sources or reduced healing from enemies.

#### 🧛 Lifesteal

Lifesteal converts dealt damage into health for the attacker. The system is highly configurable.

Link lifesteal stats to specific damage types and choose which pipeline step it applies to—raw damage, net damage, pre-mitigation, post-mitigation, before barrier, after barrier, etc.

You can fine-tune lifesteal application timing to fit your game's design.

#### 💀 Death

By default, when an entity's health reaches zero, it is considered dead. The package also allows setting a custom death threshold, which can be useful for implementing mechanics such as "down but not out" or "last stand" where an entity can survive until a certain negative health value before dying.

When an entity dies, a Game Action can be executed, allowing for custom behavior upon death, such as triggering events, playing animations, or dropping loot. Beyond game mechanics, Game Actions can tailor the death behavior to your architecture. For example, if your game uses object pooling, you can implement a Game Action that returns the entity to the pool instead of destroying it. Alternatively, if an entity can resurrect, you could opt for the pre-defined "Toggle Active Game Object" Game Action, which simply disables the entity upon death, allowing for easy resurrection later on.

A default on-death Game Action for all entities can be set, and custom on-death Game Actions can be assigned to specific entities as needed.

The base package, Astra RPG Framework, comes with the following pre-defined Game Actions useful for reacting to death events:

- **Toggle Active Game Object:** Enables or disables the entity's game object upon death. Useful for disabling entities that can resurrect.
- **Destroy Game Object:** Destroys the entity's game object upon death. Suitable for entities that do not need to be resurrected or reused.
- **Toggle Renderers (and colliders):** Recursively enables/disables all renderers on the GameObject and its children. Useful to switch off the visuals of a dead entity awaiting possible resurrection.
- **No-op action:** Does nothing on death.
- **Composite action:** Combines multiple on-death Game Actions to be executed in sequence upon death.

#### 😇 Resurrection

Resurrection is the process of bringing a dead entity back to life. An entity can be resurrected via API, providing the amount of health to restore, either as a flat value or as a percentage of its maximum health.

Similarly to on-death Game Actions, on-resurrection Game Actions can be used to define custom behavior when an entity is resurrected. The same Game Actions mentioned in the previous Death section can be used in response to resurrections. You'll likely use the Toggle Active Game Action to enable a previously disabled object, and the Toggle Renderers to re-enable previously disabled visuals and (optionally) colliders.

#### 📈 Health Scaling Component

Extend scaling formulas to use health values. Scale based on maximum health, current health, or missing health.

Build abilities that grow stronger as health decreases. Create healing that scales with max HP. Design berserker mechanics that reward risk. And more.

#### ⚡ Health Game Events

New game events for health-related actions:

- Damage taken
- Healing received and applied
- Death and resurrection
- And many more...

Wire these events to your game logic for abilities, UI updates, sound effects, particles, etc.

---

### 🛒 Where to Buy

Unity Asset Store: [Astra RPG Health](#) *(coming soon)*

---

### 📬 Information & Contact

Questions, feedback, or bugs? Email us at [electricdrill.info@gmail.com](mailto:electricdrill.info@gmail.com).

Unity Forum: [Astra RPG Health Discussion](#) *(coming soon)*

Discord: [Astra RPG Discord Server](https://discord.gg/nJVRMkGrZg)
