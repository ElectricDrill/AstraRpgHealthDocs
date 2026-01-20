# Package Configuration

Astra RPG Framework needed no configuration. The health package, however, needs to be configured to have the system work around the specific instances of your game. For example, if you defined a "Super Duper All-damage resistance" `Stat` in your game, you need to tell Astra RPG Health to use it when calculating damage.

The package uses a flexible configuration system that balances convenience with explicit control. This page explains how to set up and configure the health system for your game.

## Configuration Overview

The Astra RPG Health system uses a **two-tier configuration architecture**:

1. **Global Settings** (`AstraRpgHealthGlobalSettings`) - A lightweight pointer stored in `Resources` that references your active configuration
2. **Gameplay Configuration** (`AstraRpgHealthConfig`) - The actual configuration containing all gameplay parameters

This separation allows you to:
- Switch between different configuration profiles easily (e.g., for testing, different game modes)
- Keep configuration data separate from the loading mechanism
- Support convention-based fallbacks for quick prototyping

---

## Global Settings

### Automatic Setup

The package automatically creates the Global Settings asset on first import or when the editor loads. You don't need to do anything manually.

**What happens automatically:**
1. The `AstraRpgHealthGlobalSettings.asset` is created in `Assets/Resources/`
2. If a default configuration exists (e.g., from imported samples), it's automatically assigned
3. The system is immediately ready to use

> **📁 Default Location:** `Assets/Resources/AstraRpgHealthGlobalSettings.asset`

### Project Settings

You can manage the health system configuration through Unity's Project Settings window:

1. Open **Edit → Project Settings**
2. Navigate to **Astra RPG Health**
3. Assign your desired **Active Config Profile**

**Status Indicators:**
- ✅ **Gray "Using Explicit Configuration"** - A configuration is explicitly assigned
- ⚠️ **Yellow "Using Fallback"** - No explicit configuration; using convention-based fallback
- 🛑 **Red "No Configuration Found"** - Critical: No configuration available

**Quick Actions:**
- **Create New Config Asset** - Opens a save dialog to create a new configuration

### Convention Over Configuration

The package follows a **convention-over-configuration** philosophy to reduce setup friction:

#### Fallback Resolution

If no explicit configuration is assigned in Project Settings, the system automatically searches for a configuration named:

> **`Astra Rpg Health Config`**

located in any **`Resources`** folder in your project.

#### Search Order

The configuration provider uses a **three-step loading strategy**:

1. **Explicit Configuration** (Project Settings)
   - Loads `AstraRpgHealthGlobalSettings` from `Resources/AstraRpgHealthGlobalSettings`
   - If it has an `ActiveConfig` assigned, use it
   
2. **Convention-Based Fallback**
   - Searches for `Astra Rpg Health Config` in any `Resources` folder
   - Logs a warning indicating fallback usage
   
3. **Error State**
   - If neither is found, logs an error with instructions
   - System will not function until a configuration is provided

> **💡 Tip:** For production projects, always use **explicit configuration** via Project Settings for clarity and control.

---

## Configuration Loading Strategy

The health configuration is loaded lazily on first access and cached for performance:

```csharp
// Automatically loads configuration on first access
var config = AstraRpgHealthConfigProvider.Instance;

// Pre-load during initialization to avoid runtime overhead
AstraRpgHealthConfigProvider.WarmUp();

// Force reload (useful for testing)
AstraRpgHealthConfigProvider.Reset();
```

**When is the configuration loaded?**
- Automatically before the first scene loads (via `RuntimeInitializeOnLoadMethod`)
- Lazily when first accessed via `AstraRpgHealthConfigProvider.Instance`
- Explicitly when calling `WarmUp()`

---

## Creating Configuration Assets
If for any reason you need to create a new `AstraRpgHealthConfig` asset, you can do so via two methods:

### Via Project Settings

1. Open **Edit → Project Settings → Astra RPG Health**
2. Unassign any existing configuration, if any
3. Click **Create New Config Asset**
4. Choose a save location
5. The new configuration is automatically assigned

### Via Asset Menu

1. Right-click in the **Project Window**
2. Select **Create → Astra RPG Health → Configuration**
3. Name your configuration
4. Assign it in Project Settings or in the Global Settings asset

---

## Health Configuration Reference
> [!INFO]
> In the `AstraRpgHealthConfig` asset, you can hover over each field to see a tooltip with a brief description.

The `AstraRpgHealthConfig` asset contains all gameplay parameters for the health system. Below is a detailed explanation of each field.

### Health

#### Health Attributes Scaling
**Type:** `AttributesScalingComponent`  
**Required:** No  
**Description:** Defines how entities' maximum health scales based on character attributes (e.g., Vitality, Endurance).

> **See Also:** [Astra RPG Framework Scaling documentation](https://electricdrill.github.io/AstraRpgFrameworkDocs/MD/workflows.html#create-scaling-formulas)

#### Generic Heal Amount Modifier Stat
**Type:** `Stat`  
**Required:** No  
**Description:** The stat that modifies **all** healing received by an entity.

**Stacking Behavior:**
- Combines **additively** with Source modifications

**How it works:**
- The stat value represents a **percentage modifier**
- Positive values increase healing, negative values decrease it
- Example: A value of `25` means +25% healing received

**Example:**
- Base Heal: 100 HP
- Generic Heal Amount Modifier: 25 (means +25%)
- Final Heal: 100 * 1.25 = **125 HP**

---

#### Heal Source Modifications
**Type:** `Dictionary<HealSource, Stat>`  
**Required:** No  
**Description:** Maps each healing source (e.g., Skill, Trap, Environment) to a modifier stat.

**Use cases:**
- Create specialized healing boosts or reductions (e.g., "Resurrection healing is 50% more effective", "Healing from potions is reduced by 20%")

**Stacking Behavior:**
- Stats stack additively with Generic modifications

**Example:**

- Incoming Heal: 100 from resurrection
- Generic Heal Amount Modifier: +30% (means +30% healing received)
- Resurrection Heal Amount Modifier: +20 (means +20% healing received)
- **Total Heal Modification:** +30% +20% = +50% → Final Heal: **150**

---

### Damage

#### Default Damage Calculation Strategy
**Type:** `DamageCalculationStrategy`  
**Required:** No  
**Description:** The default strategy used to calculate net damage when an entity doesn't specify its own strategy.

**Purpose:**
- Provides a default damage calculation strategy to use for "regular" entities. In most cases, this is sufficient
- Can be overridden per-entity for custom damage pipelines in their `EntityHealth` component

> **See Also:** Damage Calculation Pipeline documentation

#### Generic Damage Modification Stat
**Type:** `Stat`  
**Required:** No  
**Description:** A universal damage modifier that applies to **all damage received**, regardless of type or source.

**Usage:**
- Applied in the damage calculation pipeline if `ApplyDmgModifiersStep` is included in the used strategy

**Stacking Behavior:**
- Combines **additively** with Type and Source modifications

**Example:**
- Incoming Damage: 100
- Generic Damage Modification: -20 (means -20% damage taken)
- **Damage Reduction:** -20% → Final Damage: **80**

#### Damage Type Modifications
**Type:** `Dictionary<DamageType, Stat>`  
**Required:** No  
**Description:** Maps each damage type (e.g., Fire, Ice, Physical) to a modifier stat.

**Usage:**
- Applied in the damage calculation pipeline if `ApplyDmgModifiersStep` is included in the used strategy

**Use cases:**
- Configure different resistances/vulnerabilities per damage type
  **Stacking Behavior:**
- Stats stack additively with Generic and Source modifications

**Example:**

- Incoming Damage: 100 Fire damage
- Generic Damage Modification: +10 (means +10% damage taken)
- Fire Damage Modification: -30 (means -30% damage taken)
- **Total Damage Modification:** +10% - 20% = -10% → Final Damage: **90**


#### Damage Source Modifications
**Type:** `Dictionary<DamageSource, Stat>`  
**Required:** No  
**Description:** Maps each damage source (e.g., Skill, Trap, Environment) to a modifier stat.

**Usage:**
- Applied in the damage calculation pipeline if `ApplyDmgModifiersStep` is included in the used strategy

**Use cases:**
- Create specialized defenses (e.g., "Traps deal 50% less damage")
- For keeping track of damage origins for a combat log or analytics

**Stacking Behavior:**
- Stats stack additively with Generic and Type modifications

**Example:**

- Incoming Damage: 100 Physical damage from a Trap
- Generic Damage Modification: +30% (means +30% damage taken)
- Physical Damage Modification: -10 (means -10% damage taken)
- Trap Damage Modification: -50 (means -50% damage taken)
- **Total Damage Modification:** +30% -10% -50% = -30% → Final Damage: **70**

---

### Health Regeneration

#### Passive Health Regeneration Source
**Type:** `HealSource`  
**Required:** No  
**Description:** The heal source used for passive regeneration effects.

**Use cases:**
- Allows tracking and modifying passive regeneration separately from active healing
- Can be used for effects like "Increase Natural Regeneration by 50%"
- Tracking passive healing in analytics or combat logs

#### Passive Health Regeneration Stat (HP/10s)
**Type:** `Stat`  
**Required:** No  
**Description:** Determines the amount of health regenerated passively.

> [!WARNING]
> The stat value represents health regenerated **per 10 seconds**.

**Calculation:**
```
Health Per Tick = (Stat Value / 10) * Interval In Seconds
```

**Example:**
- Stat Value: 50 HP/10s
- Interval: 1 second
- Health Per Tick: (50 / 10) * 1 = **5 HP per second**

#### Passive Health Regeneration Interval
**Type:** `float`  
**Default:** 1.0 seconds  
**Required:** Yes (must be > 0)  
**Description:** The time (in seconds) between passive regeneration ticks.

**Configuration Examples:**

| Interval | Stat Value | Result |
|----------|------------|--------|
| 1.0s | 50 HP/10s | 5 HP every 1 second |
| 0.5s | 50 HP/10s | 2.5 HP every 0.5 seconds |
| 2.0s | 50 HP/10s | 10 HP every 2 seconds |

> [!WARNING]
> Smaller intervals increase CPU overhead. Recommended range: 0.5s - 2.0s

#### Manual Health Regeneration Stat
**Type:** `Stat`  
**Required:** No  
**Description:** Determines health regenerated when triggering manual regeneration via API.

**Use Cases:**
- **Turn-based systems:** Regenerate health at the end of each turn
- **Rest mechanics:** Trigger regeneration when resting at campfires
- **Time-skip systems:** Apply regeneration for elapsed time

**API Usage:**
```csharp
// Trigger manual regeneration
entityHealth.ManualHealthRegenerationTick();
```

---

### Lifesteal

#### Lifesteal Config
**Type:** `LifestealConfig`  
**Required:** No  
**Description:** Configuration for lifesteal mechanics (healing based on damage dealt).

**Typical Settings:**
- Lifesteal percentage stat
- Heal source for lifesteal effects
- Restrictions (e.g., only on critical hits, only physical damage)

> **See Also:** Lifesteal documentation

---

### Death

#### Default On Death Strategy
**Type:** `OnDeathStrategy`  
**Required:** Yes  
**Description:** The strategy executed when an entity dies (if the entity doesn't have its own strategy).

**Common Strategies Ideas:**
- **Destroy GameObject** - Removes the entity from the scene
- **Ragdoll** - Enables ragdoll physics
- **Respawn** - Respawns the entity after a delay
- **Loot Drop** - Spawns loot and destroys the entity

**Example:**
```
Death → Execute Strategy → Spawn Death VFX → Drop Loot → Destroy GameObject
```

#### Default On Resurrection Strategy
**Type:** `OnResurrectionStrategy`  
**Required:** No  
**Description:** The strategy executed when an entity is resurrected (if the entity doesn't have its own strategy).

**Common Strategies Ideas:**
- **Simple Resurrection** - Restores health and enables the entity
- **Resurrection VFX** - Plays visual effects during resurrection
- **Stat Penalties** - Applies temporary debuffs after resurrection

#### Default Resurrection Source
**Type:** `HealSource`  
**Required:** ✅ **Yes**  
**Description:** The heal source used when an entity is resurrected.

**Use cases:**
- Categorizes resurrection healing separately from normal healing
- Allows effects like "Increase Resurrection Healing by 50%"
- Used for analytics and gameplay feedback

---

## Troubleshooting

### ⚠️ "No Configuration found!" error

**Cause:** No configuration is assigned and no fallback exists.

**Solution:**
1. Check **Project Settings → Astra RPG Health**
2. Assign a configuration or create a new one
3. Alternatively, create a config named `Astra Rpg Health Config` in a `Resources` folder

### ⚠️ "Using Fallback" warning

**Cause:** No explicit configuration assigned in Project Settings.

**Solution:**
1. Open **Project Settings → Astra RPG Health**
2. Assign the fallback configuration explicitly
3. This warning is just informational and won't break functionality

### ⚠️ Configuration not updating in Play Mode

**Cause:** Configuration is cached on first access.

**Solution:**
```csharp
// Force reload
AstraRpgHealthConfigProvider.Reset();
```

### ⚠️ Missing Resources folder

**Cause:** The `Assets/Resources/` folder doesn't exist.

**Solution:** The package creates it automatically. If it's missing, create it manually:
1. Right-click `Assets`
2. Create → Folder → Name it `Resources`
3. Restart Unity to trigger the bootstrapper
