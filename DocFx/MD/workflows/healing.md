# Healing

An entity can be healed in 4 different ways:
- Direct healing through the `Heal` method.
- Health regeneration. This can be passive, as defined in the [Health Regeneration](package-configuration.md#health-regeneration) configuration, so automatically triggered by the framework every so often, or it can be manually activated via the `ManualHealthRegenerationTick` method. Also for the latter, you must configure the statistic to consider for the calculation of healing through the configuration.
- Through lifesteal. See [Lifesteal](package-configuration.md) for more details on this mechanic.
- Via resurrection with the two `Resurrect` methods. One to resurrect the entity with a percentage of HP, and one to resurrect it with a fixed amount of health points. Applicable only if the entity is dead.

Both regeneration (both passive and manual), lifesteal, and resurrection use, behind the scenes, the `Heal` method to effectively heal the entity.

# Healing an Entity

Since the `Heal` method will be widely used, I show here an example of its use via API.
Suppose we want to heal an entity for 20% of its total max HP. The code could be the following:

```csharp
// Assuming that:
// - healSource is a HealSource representing the healing coming from a skill
// - skillCaster is the EntityCore that casts the healing skill
// - target is the EntityCore that we want to heal

if (target.TryGetComponent(out EntityHealth targetHealth)) {
    // in case the target has a EntityHealth component...
    long healAmount = target.GetMaxHpPortion(0.2d);

    target.Heal(PreHealContext.Builder
        .WithAmount(healAmount)
        .WithSource(healSource)
        .WithHealer(skillCaster)
        .WithTarget(target.EntityCore)
        .Build());
}
```

It is not a good practice to hardcode the healing amount directly in code. Hardcoded values make tuning and balancing difficult, prevent reuse across different skills or items, and force code changes and recompilation for values that designers or content creators should be able to tweak. Prefer one of these approaches instead:

- **Serialize the amount** as an inspector-editable field so designers can tweak values without touching code.
- **Use a `ScalingFormula`** (or equivalent) to compute the heal amount dynamically from the caster's or target's stats, allowing the effect to scale with progression and remain data-driven.

Example using a `ScalingFormula` to calculate the heal amount at runtime:

```csharp
// Assuming that scalingFormula is a ScalingFormula that calculates the heal amount
// based on the skill caster's stats...
long healAmount = scalingFormula.CalculateValue(skillCaster);

target.Heal(PreHealContext.Builder
    .WithAmount(healAmount)
    .WithSource(healSource)
    .WithHealer(skillCaster)
    .WithTarget(target.EntityCore)
    .Build());
```

Recommended pattern when healing programmatically:
1. Construct a `PreHealContext` with all relevant information using the fluent builder.
2. Call `Heal` passing the constructed context.

The `PreHealContext` fluent builder is helpful because it guides you through required inputs first — the IDE will typically present required builder steps one at a time. When the builder presents multiple fields together, those fields are optional and can be omitted.