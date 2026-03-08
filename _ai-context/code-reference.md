# Code Reference — Astra RPG Health

> Maps source files to their corresponding documentation sections.
> Use this to quickly know which files to read before writing or editing a given documentation section.

Base path for all source files: `C:\Users\emaci\Documents\AstraRpgPublishing_6_3\Packages\com.electricdrill.astra-rpg-health\`

---

## Package Configuration

| File | What it contains |
|---|---|
| `Runtime\Config\IAstraRpgHealthConfig.cs` | Interface; all configurable fields for every subsystem |
| `Runtime\Config\AstraRpgHealthConfigSO.cs` | Concrete SO; 6 inspector sections (Health, Damage, HealthRegeneration, Lifesteal, Death, Experience) |
| `Runtime\Config\AstraRpgHealthGlobalSettingsSO.cs` | Pointer to active config; must be in `Resources/` |
| `Runtime\Config\AstraRpgHealthConfigProvider.cs` | Static singleton; 3-step fallback load strategy |
| `Runtime\Config\AstraRpgHealthBootstrap.cs` | Runtime warm-load on startup |
| `Editor\Config\AstraRpgHealthConfigEditor.cs` | Inspector for config SO; 6 organized foldable sections |
| `Editor\Config\AstraRpgHealthSettingsProviderEditor.cs` | Project Settings panel; status indicators for active config |
| `Editor\Config\AstraRpgHealthEditorBootstrapper.cs` | [InitializeOnLoad]; auto-creates GlobalSettings and assigns fallback |

---

## Damage Sources (## Damage Sources in damage.md)

| File | What it contains |
|---|---|
| `Runtime\Damage\DamageSourceSO.cs` | PercentageModificationStat, FlatModificationStat |
| `Editor\Damage\DamageSourceEditor.cs` | Inspector with tooltips for both modifier stats |

---

## Damage Types — Damage Reduction (### Damage Reduction in damage.md)

| File | What it contains |
|---|---|
| `Runtime\Damage\DamageTypeSO.cs` | All DamageType fields including DefensiveStat, DamageReductionFn, DefensiveStatPiercedBy, DefenseReductionFn, modifier stats, true damage flags |
| `Runtime\Damage\CalculationPipeline\ApplyDefenseStep.cs` | Pairing validation; missing-stat errors; applies DefensiveStat to DamageReductionFn; applies piercing via DefensiveStatPiercedBy to DefenseReductionFn |
| `Editor\Damage\DamageTypeEditor.cs` | Custom inspector; 4 sections; consistency warnings; "Make a Damage Reduction Simulation" button |

### Built-in Damage Reduction Functions

| File | Formula |
|---|---|
| `Runtime\DamageReductionFunctions\DamageReductionFnSO.cs` | Abstract base; CalculateReducedDamage(amount, defensiveStatValue) |
| `Runtime\DamageReductionFunctions\FlatDamageReductionFnSO.cs` | Damage - (DefensiveStat x Factor); _factor field |
| `Runtime\DamageReductionFunctions\PercentageDamageReductionFnSO.cs` | Damage - (Damage x DefensiveStat / 100) |
| `Runtime\DamageReductionFunctions\LogDamageReductionFnSO.cs` | Damage x (BaseValue / (BaseValue + Log(1 + DefensiveStat x ScaleFactor))); _baseValue (default 1), _scaleFactor (default 0.1) |
| `Editor\DamageReductionFunctions\FlatDamageReductionFnEditor.cs` | Formula help box |
| `Editor\DamageReductionFunctions\PercentageDamageReductionFnEditor.cs` | Formula help box |
| `Editor\DamageReductionFunctions\LogDamageReductionFnEditor.cs` | Formula help box + "Open Graph Visualizer" button to LogDamageReductionGraphWindow |

### Log Damage Reduction Graph Window

| File | What it contains |
|---|---|
| `Editor\Window\LogDamageReductionGraphWindow.cs` | Two graphs (Scale Factor fixed / Base Value varying; Base Value fixed / Scale Factor varying); 5 curves each; EditorPrefs persistence; hover tooltip |
| `Editor\Window\LogarithmicGraphDrawer.cs` | Graph rendering; color coding (red = lowest param = most aggressive, blue = highest) |

### Damage Reduction Simulation Window (NOT YET DOCUMENTED)

| File | What it contains |
|---|---|
| `Editor\Window\DmgReductionGraphWindow.cs` | Opened by "Make a Damage Reduction Simulation" in DamageTypeEditor; shows clamped (green) + unclamped (yellow) curves for a specific DamageTypeSO; supports piercing stat, growth formula for piercing, configurable damage range; hover shows defensive stat, reduced defense, net damage, reduction % |

---

## Damage Types — Defense Penetration (#### Defense Penetration in damage.md)

Same source files as Damage Reduction above (all fields on DamageTypeSO, handled in ApplyDefenseStep).

### Built-in Defense Reduction Functions

| File | Formula |
|---|---|
| `Runtime\DefenseReductionFunctions\DefenseReductionFnSO.cs` | Abstract base; CalculateReducedDefense(piercingStat, defensiveStat, stat, applyClamp) |
| `Runtime\DefenseReductionFunctions\FlatDefenseReductionFnSO.cs` | Defense - (PiercingStat x Factor) |
| `Runtime\DefenseReductionFunctions\PercentageDefenseReductionFnSO.cs` | Defense - (Defense x PiercingStat / 100) |
| `Runtime\DefenseReductionFunctions\LogDefenseReductionFnSO.cs` | Defense x (BaseValue / (BaseValue + Log(1 + PiercingStat x ScaleFactor))); _baseValue (default 100), _scaleFactor (default 1.0) |
| `Editor\DefenseReductionFunctions\FlatDefenseReductionFnEditor.cs` | Formula help box |
| `Editor\DefenseReductionFunctions\PercentageDefenseReductionFnEditor.cs` | Formula help box |
| `Editor\DefenseReductionFunctions\LogDefenseReductionFnEditor.cs` | Formula help box + "Open Graph Visualizer" button to LogDefenseReductionGraphWindow |

### Log Defense Reduction Graph Window (NOT YET DOCUMENTED)

| File | What it contains |
|---|---|
| `Editor\Window\LogDefenseReductionGraphWindow.cs` | Same layout as LogDamageReductionGraphWindow but for defense (piercing) reduction; two graphs varying Base Value / Scale Factor; 5 curves each; EditorPrefs persistence |

---

## Damage Types — True Damage Options (### True Damage Options in damage.md — NOT YET DOCUMENTED)

| File | What it contains |
|---|---|
| `Runtime\Damage\DamageTypeSO.cs` | IgnoresBarrier (bool), IgnoreGenericPercentageDamageModifiers (bool), IgnoreGenericFlatDamageModifiers (bool) |
| `Runtime\Damage\CalculationPipeline\ApplyBarrierStep.cs` | Skips entirely when DamageType.IgnoresBarrier == true |
| `Runtime\Damage\CalculationPipeline\ApplyPercentageDmgModifiersStep.cs` | Skips generic % stat when IgnoreGenericPercentageDamageModifiers == true |
| `Runtime\Damage\CalculationPipeline\ApplyFlatDmgModifiersStep.cs` | Skips generic flat stat when IgnoreGenericFlatDamageModifiers == true |

---

## Damage Modifiers (## Damage Modifiers in damage.md)

| File | What it contains |
|---|---|
| `Runtime\Damage\CalculationPipeline\ApplyFlatDmgModifiersStep.cs` | Reads generic flat stat + source flat stat + type flat stat; sums additively |
| `Runtime\Damage\CalculationPipeline\ApplyPercentageDmgModifiersStep.cs` | Reads generic % stat + source % stat + type % stat; immunity checks at <= -100 per category |
| `Runtime\Config\AstraRpgHealthConfigSO.cs` | GenericFlatDamageModificationStat, GenericPercentageDamageModificationStat |
| `Runtime\Damage\DamageSourceSO.cs` | Source-specific PercentageModificationStat, FlatModificationStat |
| `Runtime\Damage\DamageTypeSO.cs` | Type-specific PercentageDamageModificationStat, FlatDamageModificationStat |

---

## Damage Calculation Pipeline (## Damage Calculation Pipeline in damage.md — NOT YET DOCUMENTED)

| File | What it contains |
|---|---|
| `Runtime\Damage\PreDamageContext.cs` | Fluent builder; fields: Amount, Type, DamageSource, Target, Source (dealer), IsCritical, CriticalMultiplier, Ignore; builder entry: PreDamageContext.Builder |
| `Runtime\Damage\CalculationPipeline\DamageInfo.cs` | Mutable pipeline state; holds DamageAmountContext, type, source, target, Reasons (flags), TerminationStepType, IsPrevented |
| `Runtime\Damage\DamageAmountContext.cs` | Tracks InitialAmount, Current (clamped >= 0), Records (list of before/after per step); RecordStep(), GetStepAmount() |
| `Runtime\Damage\DamageResolutionContext.cs` | Result; Outcome (Applied/Prevented), Reasons, TerminationStepType, FinalDamageInfo, PreDamageContext; factory: Applied(), Prevented() |
| `Runtime\Damage\DamageOutcome.cs` | Enum: Applied, Prevented |
| `Runtime\Damage\DamagePreventionReason.cs` | Flags enum with all prevention reasons; IsTerminal(), ToDetailedString() |
| `Runtime\Damage\CalculationPipeline\DamageStep.cs` | Abstract; DisplayName property; ProcessStep(DamageInfo) abstract; Process(DamageInfo) concrete |
| `Runtime\Damage\CalculationPipeline\DamageCalculationStrategySO.cs` | steps list; CalculateDamage(DamageInfo) executes pipeline sequentially |
| `Runtime\Damage\CalculationPipeline\ApplyBarrierStep.cs` | Barrier consumption; skips if IgnoresBarrier; adds BarrierAbsorbed if fully absorbed |
| `Runtime\Damage\CalculationPipeline\ApplyCriticalMultiplierStep.cs` | Applies CriticalMultiplier; skips if multiplier = 1 or not critical |
| `Editor\Damage\CalculationPipeline\ConfigurableDamageStrategyEditor.cs` | Reorderable list; type-picker for adding DamageStep types; numbered display of step DisplayName |
| `Runtime\EntityHealth.cs` | TakeDamage(PreDamageContext) orchestrates full pipeline; per-entity strategy override |

---

## Entity Health Component (entity-health.md — not yet reviewed)

| File | What it contains |
|---|---|
| `Runtime\EntityHealth.cs` | MaxHp, Hp, Barrier; TakeDamage, Heal, Resurrect; SetupMaxHp, AddMaxHpFlatModifier, AddMaxHpPercentageModifier, AddMaxHpScaling; ManualHealthRegenerationTick; IsImmune; OverrideOnDeathGameAction; passive regen; extra event channel subscriptions |
| `Runtime\Damage\IDamageable.cs` | Interface contract |
| `Runtime\Heal\IHealable.cs` | Interface contract |
| `Runtime\Resurrection\IResurrectable.cs` | Interface contract |
| `Runtime\Utils\EntityCoreHealthExtensions.cs` | GetEntityHealth(this EntityCore) cached extension method |
| `Runtime\Exceptions\DeadEntityException.cs` | Thrown on illegal ops on dead entities |
| `Editor\EntityHealthEditor.cs` | Complex inspector; 4 foldable sections (Health, Damage, Death, Events); extra event channel add/remove buttons |

---

## Healing (healing.md — not yet reviewed)

| File | What it contains |
|---|---|
| `Runtime\Heal\IHealable.cs` | Heal(PreHealContext) returns ReceivedHealContext |
| `Runtime\Heal\HealSourceSO.cs` | FlatHealModificationStat, PercentageHealModificationStat |
| `Runtime\Heal\HealAmountContext.cs` | RawAmount, AfterModifierAmount, NetAmount |
| `Runtime\Heal\PreHealContext.cs` | Fluent builder; Amount, HealSource, Source (healer), Target, IsCritical, CriticalMultiplier |
| `Runtime\Heal\ReceivedHealContext.cs` | HealAmount (HealAmountContext), HealSource, Source, Target, IsCritical |

---

## Lifesteal (lifesteal.md — not yet reviewed)

| File | What it contains |
|---|---|
| `Runtime\Heal\LifestealConfigSO.cs` | Maps DamageTypeSO to (Stat, HealSourceSO, LifestealAmountSelector); LifestealBasisMode enum (Initial/Step/Final); StepValuePoint enum (Pre/Post); SetMapping, TryGetMapping, GetOrCreateMapping |
| `Editor\Heal\LifestealStatConfigDrawer.cs` | Property drawer; conditional step picker (type-selection menu for DamageStep types); LifestealBasisMode dropdown |

---

## Resurrection (resurrection.md — not yet reviewed)

| File | What it contains |
|---|---|
| `Runtime\Resurrection\IResurrectable.cs` | Resurrect(long, HealSourceSO) and Resurrect(Percentage, HealSourceSO) |
| `Runtime\Resurrection\ResurrectionContext.cs` | Target, PreviousValue, NewValue, AbsAmount, HealSource |
| `Runtime\GameActions\Actions\ResurrectGameActionSO.cs` | Generic base; _usePercentageHp, _percentageHpToRecover, _flatHpToRecover, _healSource |
| `Runtime\GameActions\Actions\Component\ResurrectComponentGameActionSO.cs` | Concrete SO for Component context |

---

## Experience Collection (experience-collection.md — not yet reviewed)

| File | What it contains |
|---|---|
| `Runtime\Experience\ExpCollectionStrategySO.cs` | Abstract; TryCollectExp(collector, victim, dmgContext); template method pattern |
| `Runtime\Experience\ExpCollector.cs` | MonoBehaviour; RequireComponent(EntityDiedGameEventListener); ActiveStrategy property |
| `Runtime\Experience\Strategies\DirectKillExpStrategySO.cs` | Grants exp to the killing blow dealer; _expMultiplier |
| `Runtime\Experience\Strategies\DamageSourceKillExpStrategySO.cs` | Grants exp when kill matches configured DamageSource; _damageSource, _expMultiplier |
| `Runtime\Experience\Strategies\FirstMatchExpStrategySO.cs` | Composite: tries sub-strategies in order |
| `Editor\Experience\ExpCollectorEditor.cs` | Inspector with strategy status boxes |

---

## Health Scaling Component (health-scaling-component.md — not yet reviewed)

| File | What it contains |
|---|---|
| `Runtime\Scaling\ScalingComponents\HealthScalingComponentSO.cs` | HealthScalingMetrics enum (Hp, MaxHp, MissingHp); HealthScalingMetricValue struct; CalculateValue(EntityCore) |
| `Editor\Components\HealthScalingComponentEditor.cs` | Add/remove list for scaling metric entries; metric dropdown + portion input |

---

## Events System

All game events and listeners are auto-generated. Key event payloads:

| Context class | Raised when |
|---|---|
| EntityHealthChangedContext | HP gained or lost |
| EntityMaxHealthChangedContext | Max HP changed |
| EntityDiedContext | Entity died (includes DamageResolutionContext) |
| ResurrectionContext | Entity resurrected |
| ReceivedHealContext | Entity healed |
| PreHealContext | Before heal applied |
| PreDamageContext | Before damage applied |
| DamageResolutionContext | After damage pipeline completed |

---

## Documentation Images

Base path: `C:\Users\emaci\Documents\AstraRpgHealthDocs\DocFx\images\AstraRPG\workflows\`

### Confirmed existing images (damage/)

| File | Used in |
|---|---|
| `damage/damage-source.png` | ## Damage Sources |
| `damage/damage-type/magic-damage-type-inspector.png` | ## Damage Types intro |
| `damage/damage-type/damage-reduction-functions-flat.png` | #### Flat Dmg Reduction |
| `damage/damage-type/damage-reduction-functions-percentage.png` | #### Percent Dmg Reduction |
| `damage/damage-type/damage-reduction-functions-log.png` | #### Log Dmg Reduction |
| `damage/damage-type/log-dmg-red-graph-visualizer.png` | Log Graph Window section |
| `damage/damage-type/log-dmg-red-graph-visualizer-hover.png` | Log Graph Window hover tooltip |
| `damage/damage-type/defense-reduction-functions-flat.png` | #### Defense Penetration |
| `damage/damage-type/defense-reduction-functions-percentage.png` | #### Defense Penetration |
| `damage/damage-type/defense-reduction-functions-log.png` | #### Defense Penetration |

> Always verify with glob/view before using an existing image filename. For new images, use the IMAGE MISSING comment pattern (see style-guide.md).
