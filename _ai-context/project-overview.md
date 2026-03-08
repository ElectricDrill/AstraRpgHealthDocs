# Project Overview — Astra RPG Health

> **For AI assistants**: read all four files in this folder at the start of each session before touching any documentation or source code.

## What This Is

**Astra RPG Health** is a Unity package built on top of **Astra RPG Framework**. It adds health, damage, healing, death/resurrection, lifesteal, and barrier mechanics to entities managed by the Framework's stat/attribute system.

The user is writing English documentation for this package using DocFx. All conversations happen in Italian; documentation is always written in English.

---

## Folder Paths

| Asset | Path |
|---|---|
| Health package source | `C:\Users\emaci\Documents\AstraRpgPublishing_6_3\Packages\com.electricdrill.astra-rpg-health\` |
| Framework source | `C:\Users\emaci\Documents\AstraRpgPublishing_6_3\Packages\com.electricdrill.astra-rpg-framework\` |
| Documentation markdown | `C:\Users\emaci\Documents\AstraRpgHealthDocs\DocFx\MD\` |
| Documentation images | `C:\Users\emaci\Documents\AstraRpgHealthDocs\DocFx\images\AstraRPG\` |
| AI context (this folder) | `C:\Users\emaci\Documents\AstraRpgHealthDocs\_ai-context\` |

---

## Astra RPG Framework — Key Concepts

The Framework provides the foundation. Key systems relevant to the Health package:

- **EntityCore**: base component all entities share
- **Stats (`StatSO`)**: numeric values (e.g., Armor, Magic Resistance, Armor Penetration). Used as defensive stats, piercing stats, modifier stats.
- **StatSet**: a collection of stats; entities have StatSets, allowing centralized stat management.
- **Attributes**: derived values (e.g., HP max); attributes drive `EntityHealth`.
- **Scaling / GrowthFormula**: how stats grow with level.
- **Percentage**: framework utility type for percentage values.

---

## Astra RPG Health — Key Classes

### Runtime

| Class | Path (relative to Runtime/) | Purpose |
|---|---|---|
| `EntityHealth` | `Health/EntityHealth.cs` | Main health component; orchestrates the damage pipeline |
| `DamageTypeSO` | `Damage/DamageTypeSO.cs` | ScriptableObject defining a damage type |
| `DamageSourceSO` | `Damage/DamageSourceSO.cs` | ScriptableObject defining a damage source |
| `PreDamageContext` | `Damage/CalculationPipeline/PreDamageContext.cs` | Fluent builder for damage input data |
| `DamageResolutionContext` | `Damage/CalculationPipeline/DamageResolutionContext.cs` | Mutable context passed through each damage step |
| `DamageStep` | `Damage/CalculationPipeline/DamageStep.cs` | Abstract base for pipeline steps |
| `DamageCalculationStrategySO` | `Damage/CalculationPipeline/DamageCalculationStrategySO.cs` | SO defining the ordered list of steps |
| `ApplyDefenseStep` | `Damage/CalculationPipeline/ApplyDefenseStep.cs` | Applies defensive stat + damage reduction; also defense penetration |
| `ApplyFlatDmgModifiersStep` | `Damage/CalculationPipeline/ApplyFlatDmgModifiersStep.cs` | Applies flat damage modifiers (all three categories) |
| `ApplyPercentageDmgModifiersStep` | `Damage/CalculationPipeline/ApplyPercentageDmgModifiersStep.cs` | Applies percentage damage modifiers + immunity checks |
| `DamageReductionFnSO` | `Damage/DamageReductionFunctions/DamageReductionFnSO.cs` | Abstract base for damage reduction functions |
| `FlatDamageReductionFnSO` | `Damage/DamageReductionFunctions/FlatDamageReductionFnSO.cs` | Flat reduction: `DefensiveStat × Factor` |
| `PercentageDamageReductionFnSO` | `Damage/DamageReductionFunctions/PercentageDamageReductionFnSO.cs` | Percentage reduction: `DefensiveStat`% |
| `LogDamageReductionFnSO` | `Damage/DamageReductionFunctions/LogDamageReductionFnSO.cs` | Log reduction: `Damage × (BV / (BV + Log(1 + Stat × SF)))` |
| `DefenseReductionFnSO` | `Damage/DefenseReductionFunctions/DefenseReductionFnSO.cs` | Abstract base for defense (penetration) reduction functions |

### Editor

| Class | Path (relative to Editor/) | Purpose |
|---|---|---|
| `DamageTypeEditor.cs` | `Damage/DamageTypeEditor.cs` | Custom inspector for DamageTypeSO; shows three sections |
| `LogDamageReductionGraphWindow.cs` | `Window/LogDamageReductionGraphWindow.cs` | Custom window: two graphs for log formula visualization |
| `LogDamageReductionFnEditor.cs` | `Damage/LogDamageReductionFnEditor.cs` | Inspector button "Open Graph Visualizer" |
| `LogarithmicGraphDrawer.cs` | `Window/LogarithmicGraphDrawer.cs` | Graph rendering; color coding (red=low, blue=high) |

---

## DamageTypeSO Inspector Sections

Defined in `DamageTypeEditor.cs`:

1. **Damage Reduction**: `DefensiveStat`, `DamageReductionFn`, `DefensiveStatPiercedBy`, `DefenseReductionFn`
2. **Damage Modifiers**: `PercentageDamageModificationStat`, `FlatDamageModificationStat`
3. **True Damage Options**: `IgnoresBarrier`, `IgnoreGenericPercentageDamageModifiers`, `IgnoreGenericFlatDamageModifiers`

---

## Key Business Rules (from source)

- **Pairing rule (Damage Reduction)**: `DefensiveStat` and `DamageReductionFn` must both be set or both null. Mismatch → `Debug.LogWarning` + step skipped. Source: `ApplyDefenseStep.cs`.
- **Pairing rule (Defense Penetration)**: `DefensiveStatPiercedBy` and `DefenseReductionFn` must both be set or both null. Same behavior. Source: `ApplyDefenseStep.cs`.
- **Missing stat (Damage Reduction)**: if target entity lacks the `DefensiveStat` → error logged. Source: `ApplyDefenseStep.cs`.
- **Missing stat (Defense Penetration)**: if dealer lacks `DefensiveStatPiercedBy` → error logged. Source: `ApplyDefenseStep.cs`.
- **Modifier stacking**: all three modifier categories (generic, source, type) are summed additively. Source: `ApplyFlatDmgModifiersStep.cs`, `ApplyPercentageDmgModifiersStep.cs`.
- **Percentage immunity thresholds**: generic % ≤ −100 → `AllDamageImmune`; source % ≤ −100 → `DamageSourceImmune`; type % ≤ −100 → `DamageTypeImmune`. Checked per category before summing.

---

## AstraRpgHealthConfig

Global config asset. Relevant fields:
- **Generic Flat Damage Modification Stat**: applies to all damage, flat.
- **Generic Percentage Damage Modification Stat**: applies to all damage, percentage.

Documented in `package-configuration.md`.
