# Healing

An entity can be healed in 4 different ways:
- Direct healing through the `Heal` method.
- Health regeneration. This can be passive, as defined in the [Health Regeneration](package-configuration.md#health-regeneration) configuration, so automatically triggered by the framework every so often, or it can be manually activated via the `ManualHealthRegenerationTick` method. Also for the latter, you must configure the statistic to consider for the calculation of healing through the configuration.
- Through lifesteal, that is healing the entity based on the damage dealt. However, lifesteal functioning is detailed in [Lifesteal](lifesteal.md) as it deserves a dedicated section.
- Via resurrection with the two `Resurrect` methods. One to resurrect the entity with a percentage of HP, and one to resurrect it with a fixed amount of health points. Applicable only if the entity is dead. Also resurrection is detailed in a dedicated section, [Resurrection](resurrection.md).

> [!NOTE]
> Regeneration (both passive and manual), lifesteal, and resurrection all use, behind the scenes, the `Heal` method to effectively heal the entity.

We will first introduce some concepts that are common to all the four healing methods, and then we will analyze direct healing and health regeneration. As aforesaid, lifesteal and resurrection are analyzed in their respective sections, [Lifesteal](lifesteal.md) and [Resurrection](resurrection.md).

## Heal Source


## Heal Modifiers


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

Per fare un esempio, se vogliamo che un'entità rigeneri 5HP ogni 0.5 secondi, dobbiamo configurare il parametro "Passive Health Regeneration Interval" a 0.5 e l'entita' deve avere la statistica associata a "Passive Health Regeneration Stat (HP/10s)" con un valore di 100 (5 HP ogni 0.5 secondi corrispondono a 10HP al secondo, che corrispondono a **100 HP ogni 10 secondi**).

#### Performance Considerations
Prima di fare alcuna considerazione sulle performance, ci tengo a ricordare che l'ottimizzazione prematura è la radice di tutti i mali. Prima di intraprendere azioni per ottimizzare le performance, andrebbe identificato un reale problema di performance attraverso il profiling.
Detto questo, dal momento che di default la rigenerazione passiva innalza gli Entity Healed Events (global ed extra), è importante considerare l'impatto sulle performance qualora si abbiano un gran numero di entità che rigenerano vita contemporaneamente con un intervallo di rigenerazione molto breve. Se ci dovessero essere anche svariati listener sottoscritti al global Entity Healed Event, si otterrebbero svariate chiamate di funzione ad ogni tick di rigenerazione X ogni entita'.
Se questo dovesse diventare un problema per le performance del vostro gioco, avete a disposizione diverse leve per ottimizzare le performance:
- **Aumentare l'intervallo di rigenerazione**: aumentando l'intervallo di rigenerazione, si riduce il numero di tick di rigenerazione per unità di tempo, e quindi il numero di Entity Healed Events innalzati.
- **Disabilitare l'innalzamento di Entity Healed Events per la rigenerazione passiva**: se non avete bisogno di reagire alla rigenerazione passiva attraverso i listener degli Entity Healed Events, potete disabilitare l'innalzamento di questi eventi per la rigenerazione passiva attraverso la configurazione. In questo modo, anche se avrete un gran numero di entità che rigenerano vita contemporaneamente con un intervallo di rigenerazione molto breve, non avrete un impatto significativo sulle performance dovuto all'innalzamento di eventi e alle chiamate di funzione associate.
- **Utilizzare gli Extra Entity Healed Event**: se avete bisogno di reagiere solo alla rigenerazione passiva di una o alcune specifiche entita', potete configurare queste entità per innalzare un Extra Entity Healed Event specifico per la rigenerazione passiva, e sottoscrivere i vostri listener a questo evento invece che al global Entity Healed Event. In questo modo, le uniche chiamate di funzione che avrete ad ogni tick, saranno quelle dei listener sottoscritti all'Extra Entity Healed Event specifico delle entita' configurate per innalzare questo evento, riducendo cosi' l'impatto sulle performance.

Ci tengo a ribadire che, a meno che non abbiate identificato un reale problema di performance attraverso il profiling, non è necessario intraprendere nessuna di queste azioni per ottimizzare le performance. Probabilmente non avrete alcun problema di performance anche con un migliaio di entita' e decine di listener sottoscritti al global Entity Healed Event.

### Manual Health Regeneration
La rigenerazione manuale è invece triggerata manualmente attraverso il metodo [`ManualHealthRegenerationTick()`](xref:ElectricDrill.AstraRpgHealth.EntityHealth.ManualHealthRegenerationTick) delle API.

Questo metodo, quando chiamato, fa scattare un tick di rigenerazione per l'entità. Attenzione che la statistica considerata per il calcolo della vita NON coincide con quella configurata per la rigenerazione passiva, ma è una statistica a parte, sempre configurata attraverso la configurazione del package, per la precisione: [Manual Health Regeneration Stat](package-configuration.md#manual-health-regeneration-stat).
Il valore di questa statistica determina la quantità di HP rigenerati ad ogni tick di rigenerazione manuale. Ovviamente, i modificatori di guarigione, sia generici che specifici per la heal source configurata per la rigenerazione, influenzeranno anche la rigenerazione manuale, proprio come per la rigenerazione passiva.

### Passive vs Manual Health Regeneration: Which One Should I Use?
Se avete dubbi su quale tipo di rigenerazione utilizzare, ecco alcune linee guida che potrebbero aiutarvi nella scelta:
- **Real Time**: se il vostro gioco è in tempo reale, la rigenerazione passiva è generalmente più adatta, in quanto si integra meglio con il flusso di gioco continuo.
- **Turn-Based**: se il vostro gioco è a turni, la rigenerazione manuale è generalmente più adatta, in quanto potete farla scattare alla fine di ogni turno, garantendo un controllo più preciso sul flusso di gioco.
- **Regeneration in the base**: se il vostro gioco prevede una meccanica di rigenerazione che avviene solo quando il personaggio è in un luogo specifico (es. una base), potete utilizzare la rigenerazione passiva, ma attivarla solo quando il personaggio si trova in quel luogo specifico, e disattivarla quando esce da quel luogo. Se invece vi interessa averla attiva anche quando il personaggio è fuori dalla base, ma volete comunque che sia più forte quando è in base, potete conferire un Heal Modifier specifico per la Heal Source della rigenerazione quando il personaggio è in base, in modo da aumentare la quantità di HP rigenerati quando il personaggio si trova in base. All'uscita dalla base, potete rimuovere questo Heal Modifier, in modo da ridurre la quantità di HP rigenerati quando il personaggio è fuori dalla base.
- **Regeneration out of combat**: se il vostro gioco prevede una meccanica di rigenerazione che avviene solo quando il personaggio è fuori dal combattimento, potete utilizzare la rigenerazione passiva, ma attivarla solo quando il personaggio è fuori dal combattimento, e disattivarla quando entra in combattimento. Se invece vi interessa averla attiva anche durante il combattimento, ma volete comunque che sia più forte quando il personaggio è fuori dal combattimento, potete, come per la rigenerazione in base, conferire un Heal Modifier specifico per la Heal Source della rigenerazione quando il personaggio è fuori dal combattimento.
- **Regeneration as a buff**: se l'entita' riceve un buff che fornisce rigenerazione HP secondo un timing particolare, potete utilizzare la rigenerazione manuale, in modo da far scattare i tick di rigenerazione in corrispondenza del timing particolare previsto dal buff.

Chiaramente queste sono solo delle linee guida e non delle regole rigide. Per ogni problema ci sono sempre svariate soluzioni possibili, e la scelta della soluzione migliore dipende sempre dal contesto specifico del vostro gioco e dalle meccaniche che volete implementare. Quindi, se ritenete che una soluzione diversa da quelle suggerite sia più adatta al vostro gioco, sentitevi liberi di utilizzarla.