# Code Reference — Astra RPG Health

> Maps source files to their corresponding documentation sections.
> Use this to quickly know which files to read before writing or editing a given documentation section.

Base path for all source files: `C:\Users\emaci\Documents\AstraRpgPublishing_6_3\Packages\`

---

## Damage Pipeline

### `### Damage Reduction` and `#### Defense Penetration`

| File | What it contains |
|---|---|
| `com.electricdrill.astra-rpg-health\Runtime\Damage\DamageTypeSO.cs` | All DamageType fields: DefensiveStat, DamageReductionFn, DefensiveStatPiercedBy, DefenseReductionFn, PercentageDamageModificationStat, FlatDamageModificationStat, IgnoresBarrier, IgnoreGenericPercentageDamageModifiers, IgnoreGenericFlatDamageModifiers |
| `com.electricdrill.astra-rpg-health\Runtime\Damage\CalculationPipeline\ApplyDefenseStep.cs` | Pairing validation logic; missing-stat error logic; applies defensive stat → DamageReductionFn and DefensiveStatPiercedBy → DefenseReductionFn |
| `com.electricdrill.astra-rpg-health\Editor\Damage\DamageTypeEditor.cs` | Custom inspector; three inspector sections layout |

### Built-in Damage Reduction Functions

| File | What it contains |
|---|---|
| `com.electricdrill.astra-rpg-health\Runtime\Damage\DamageReductionFunctions\DamageReductionFnSO.cs` | Abstract base; `CalculateReducedDamage` method signature |
| `com.electricdrill.astra-rpg-health\Runtime\Damage\DamageReductionFunctions\FlatDamageReductionFnSO.cs` | Flat: `DefensiveStat × Factor` |
| `com.electricdrill.astra-rpg-health\Runtime\Damage\DamageReductionFunctions\PercentageDamageReductionFnSO.cs` | Percentage: `DefensiveStat`% |
| `com.electricdrill.astra-rpg-health\Runtime\Damage\DamageReductionFunctions\LogDamageReductionFnSO.cs` | Log formula: `Damage × (BV / (BV + Log(1 + Stat × SF)))`; BaseValue and ScaleFactor fields |

### Log Graph Visualizer

| File | What it contains |
|---|---|
| `com.electricdrill.astra-rpg-health\Editor\Window\LogDamageReductionGraphWindow.cs` | Custom window; two graphs (Scale Factor fixed / Base Value varying; Base Value fixed / Scale Factor varying); hover tooltip |
| `com.electricdrill.astra-rpg-health\Editor\Window\LogarithmicGraphDrawer.cs` | Graph rendering; color coding (red=lowest param value=most/least aggressive, blue=highest) |
| `com.electricdrill.astra-rpg-health\Editor\Damage\LogDamageReductionFnEditor.cs` | Inspector button "Open Graph Visualizer" |

### Built-in Defense Reduction Functions

| File | What it contains |
|---|---|
| `com.electricdrill.astra-rpg-health\Runtime\Damage\DefenseReductionFunctions\DefenseReductionFnSO.cs` | Abstract base; `CalculateReducedDefense` method signature |
| `com.electricdrill.astra-rpg-health\Runtime\Damage\DefenseReductionFunctions\FlatDefenseReductionFnSO.cs` | Flat defense reduction |
| `com.electricdrill.astra-rpg-health\Runtime\Damage\DefenseReductionFunctions\PercentageDefenseReductionFnSO.cs` | Percentage defense reduction |
| `com.electricdrill.astra-rpg-health\Runtime\Damage\DefenseReductionFunctions\LogDefenseReductionFnSO.cs` | Logarithmic defense reduction |

---

## Damage Modifiers

| File | What it contains |
|---|---|
| `com.electricdrill.astra-rpg-health\Runtime\Damage\CalculationPipeline\ApplyFlatDmgModifiersStep.cs` | Reads generic flat stat + source flat stat + type flat stat; sums additively |
| `com.electricdrill.astra-rpg-health\Runtime\Damage\CalculationPipeline\ApplyPercentageDmgModifiersStep.cs` | Reads generic % stat + source % stat + type % stat; immunity checks (AllDamageImmune, DamageSourceImmune, DamageTypeImmune) at ≤ −100 per category |
| `com.electricdrill.astra-rpg-health\Runtime\Config\AstraRpgHealthConfig.cs` | Config asset; GenericFlatDamageModificationStat and GenericPercentageDamageModificationStat fields |
| `com.electricdrill.astra-rpg-health\Runtime\Damage\DamageSourceSO.cs` | PercentageModifierStat and FlatModifierStat fields |

---

## True Damage Options (NOT YET DOCUMENTED)

| File | What it contains |
|---|---|
| `com.electricdrill.astra-rpg-health\Runtime\Damage\DamageTypeSO.cs` | `IgnoresBarrier` (bool), `IgnoreGenericPercentageDamageModifiers` (bool), `IgnoreGenericFlatDamageModifiers` (bool) |

Relevant pipeline steps to understand how these flags are consumed:
- Search for `IgnoresBarrier` usage in the pipeline (likely in a barrier application step)
- Search for `IgnoreGenericPercentageDamageModifiers` and `IgnoreGenericFlatDamageModifiers` in `ApplyPercentageDmgModifiersStep.cs` and `ApplyFlatDmgModifiersStep.cs`

---

## Damage Calculation Pipeline (NOT YET DOCUMENTED)

| File | What it contains |
|---|---|
| `com.electricdrill.astra-rpg-health\Runtime\Damage\CalculationPipeline\PreDamageContext.cs` | Fluent builder for damage input (damage amount, type, source, dealer, target, etc.) |
| `com.electricdrill.astra-rpg-health\Runtime\Damage\CalculationPipeline\DamageResolutionContext.cs` | Mutable context passed through each step; accumulates results |
| `com.electricdrill.astra-rpg-health\Runtime\Damage\CalculationPipeline\DamageStep.cs` | Abstract base for pipeline steps; `Execute(DamageResolutionContext)` |
| `com.electricdrill.astra-rpg-health\Runtime\Damage\CalculationPipeline\DamageCalculationStrategySO.cs` | SO listing the ordered steps; per-entity override mechanism |
| `com.electricdrill.astra-rpg-health\Runtime\Health\EntityHealth.cs` | `TakeDamage(PreDamageContext)` orchestrates the full pipeline |

---

## Health & Other Systems

| File | What it contains |
|---|---|
| `com.electricdrill.astra-rpg-health\Runtime\Health\EntityHealth.cs` | Main health component; TakeDamage, Heal, Death, Resurrection events |
| `com.electricdrill.astra-rpg-health\Runtime\Healing\` | Healing mechanics |
| `com.electricdrill.astra-rpg-health\Runtime\Lifesteal\` | Lifesteal mechanics |
| `com.electricdrill.astra-rpg-health\Runtime\Barrier\` | Barrier mechanics (related to IgnoresBarrier flag in DamageType) |
| `com.electricdrill.astra-rpg-health\Runtime\Resurrection\` | Resurrection mechanics |

---

## Documentation Images

Base path: `C:\Users\emaci\Documents\AstraRpgHealthDocs\DocFx\images\AstraRPG\workflows\`

### Confirmed existing images (damage/)

| File | Used in |
|---|---|
| `damage/damage-source.png` | `## Damage Sources` |
| `damage/damage-type/magic-damage-type-inspector.png` | `## Damage Types` intro |
| `damage/damage-type/damage-reduction-functions-flat.png` | `#### Flat Dmg Reduction` |
| `damage/damage-type/damage-reduction-functions-percentage.png` | `#### Percent Dmg Reduction` |
| `damage/damage-type/damage-reduction-functions-log.png` | `#### Log Dmg Reduction` |
| `damage/damage-type/log-dmg-red-graph-visualizer.png` | Log Graph Window section |
| `damage/damage-type/log-dmg-red-graph-visualizer-hover.png` | Log Graph Window tooltip |
| `damage/damage-type/defense-reduction-functions-flat.png` | `#### Defense Penetration` |
| `damage/damage-type/defense-reduction-functions-percentage.png` | `#### Defense Penetration` |
| `damage/damage-type/defense-reduction-functions-log.png` | `#### Defense Penetration` |

> Always verify with glob/view before adding new image references.
