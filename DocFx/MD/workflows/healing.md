# Healing

An entity can be healed in 4 different ways:
- Direct healing through the `Heal` method.
- Health regeneration. This can be passive, as defined in the [Health Regeneration](package-configuration.md#health-regeneration) configuration, so automatically triggered by the framework every so often, or it can be manually activated via the `ManualHealthRegenerationTick` method. Also for the latter, you must configure the statistic to consider for the calculation of healing through the configuration.
- Through lifesteal, that is healing the entity based on the damage dealt. However, lifesteal functioning is detailed in [Lifesteal](lifesteal.md) as it deserves a dedicated section.
- Via resurrection with the two `Resurrect` methods. One to resurrect the entity with a percentage of HP, and one to resurrect it with a fixed amount of health points. Applicable only if the entity is dead. Also resurrection is detailed in a dedicated section, [Resurrection](resurrection.md).

> [!NOTE]
> Regeneration (both passive and manual), lifesteal, and resurrection all use, behind the scenes, the `Heal` method to effectively heal the entity.

## Direct Healing
The `Heal` method is the most straightforward way to heal an entity. It takes a `PreHealContext` as input, which contains all relevant information about the healing you intend to apply and operates as follows:
1. It checks if the entity is dead. If the entity is dead, will rais an exception, as you cannot heal a dead entity. If you want to resurrect a dead entity, check the [Resurrection](resurrection.md) section.
2. It checks if the healing to be applied is marked as a critical hit and, if so it applies the critical multiplier to the healing amount.
3. It retrieves the generic heal modifiers for the entity.
4. It retrieves the heal modifiers specific to the heal source.
5. It sums the retrieved modifiers and applies them to the healing amount.
6. It increases the entity's current HP by the final healing amount.
7. It raises the _Entity Healed Event_ with a `HealResolutionContext` containing all relevant information about the healing that was just applied.
8. It returns the `HealResolutionContext` created in the previous step so that the caller can use it for further processing if needed.

The `Heal` method, differently from the `TakeDamage` method, cannot be configured to reorder the steps of the healing pipeline. This choice was made as healing is generally more straightforward than damage, and there are usually no mechanics that require reordering the steps of the healing pipeline. Therefore, I evaluated simplicity and ease of use more important than flexibility for this method.

### `Heal` Usage Example
Suppose we want to heal an entity for 20% of its total max HP. The code could be the following:

```csharp
// Assuming that:
// - healSource is a HealSource representing the healing coming from a skill
// - skillCaster is the EntityCore that casts the healing skill
// - target is the EntityCore that we want to heal

if (target.TryGetComponent(out EntityHealth targetHealth)) {
    // in case the target has a EntityHealth component...
    long healAmount = target.GetMaxHpPortion(0.2d);

    PreHealContext preHealContext = PreHealContext.Builder
        .WithAmount(healAmount)
        .WithSource(healSource)
        .WithHealer(skillCaster)
        .WithTarget(target.EntityCore)
        .Build();

    targetHealth.Heal(preHealContext);
}
```

It is not a good practice to hardcode the healing amount directly in code (20% of max hp in the example). Hardcoded values make tuning and balancing difficult, prevent reuse across different skills or items, and force code changes and recompilation for values that designers or content creators should be able to tweak. Prefer one of these approaches instead:

- **Serialize the amount** as an inspector-editable field so designers can tweak values without touching code.
- **Use a `ScalingFormula`** (or equivalent) to compute the heal amount dynamically from the caster's or target's stats, allowing the effect to scale with progression and remain data-driven.

Example using a `ScalingFormula` to calculate the heal amount at runtime:

```csharp
// Assuming that scalingFormula is a ScalingFormula that calculates the heal amount
// based on the skill caster's stats...
long healAmount = scalingFormula.CalculateValue(skillCaster);

PreHealContext preHealContext = PreHealContext.Builder
    .WithAmount(healAmount)
    .WithSource(healSource)
    .WithHealer(skillCaster)
    .WithTarget(target.EntityCore)
    .Build();

targetHealth.Heal(preHealContext);
```

Recommended pattern when healing programmatically:
1. Construct a `PreHealContext` with all relevant information using the fluent builder.
2. Call `Heal` passing the constructed context.

The `PreHealContext` fluent builder is helpful because it guides you through required inputs first — the IDE will typically present required builder steps one at a time. When the builder presents multiple fields together, those fields are optional and can be omitted.

## Health Regeneration
La rigenerazione della vita è un meccanismo che permette a un'entità di rigenerare la propria vita passivamente nel tempo. Nei giochi in generale, questa meccanica e' implementata attraverso tick discreti di rigenerazione. I tick, in base al gioco, possono essere scanditi dal tempo che passa, o da certi eventi, come il finire di un turno in un gioco a turni.  
Questo package supporta sia la rigenerazione passiva che quella manuale.

### Passive Health Regeneration
La rigenerazione passiva, per funzionare, necessita di essere configurata attraverso la sezione [Health Regeneration](package-configuration.md#health-regeneration) della configurazione del package. Li' vanno configurati i seguenti parametri:
- **Health Regeneration Source**: la `HealSource` da associare alla rigenerazione (passiva e manuale). Il package userà questa `HealSource` per creare i contesti di guarigione da passare al metodo `Heal` quando è il momento di rigenerare la vita.
- **Passive Health Regeneration Stat (HP/10s)**: la quantità di HP che l'entità rigenera ogni 10 secondi. Il package si occuperà di scalare questa quantità in base al tempo che passa tra un tick e l'altro, in modo da garantire una rigenerazione fluida e coerente.
- **Passive Health Regeneration Interval**: l'intervallo di tempo, in secondi, tra un tick di rigenerazione e l'altro.

Inoltre, e' importante ricordarsi di attivare nell'`EntityHealth` dell'entita' la rigenerazione passiva.

### Performance Considerations