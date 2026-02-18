# `EntityHealth` component
Con Astra RPG Health, il nuovo MonoBehaviour `EntityHealth` consente di aggiungere dei punti vita a un'entità. In questo modo, l'entità può subire danni e, di conseguenza, morire.

Ecco un esempio del componente `EntityHealth`:
![EntityHealth](../../images/AstraRPG/workflows/entity-health/entity-health-fresh-component.png)

La sezione degli eventi di default e' collassata poiché ci sono molti eventi, e risulta più pratico espanderla solo quando necessario.

Procediamo però con ordine e analizziamo ogni proprietà del componente `EntityHealth`.

## Health
- **Use Class Max HP**: Boolean che indica se utilizzare i punti vita massimi definiti nella classe dell'entità come punti vita massimi base. Se è disabilitato, è possibile specificare manualmente i punti vita massimi base per l'entità.
- **Base Max HP**: LongRef che rappresenta i punti vita massimi **base** dell'entità. Se "Use Class Max HP" è abilitato, questo valore viene marcato con `RO` di color teal (Read Only). Read Only, per i campi di tipo LongRef, rende non editabili i valori const, e suggersice di non modificare manualmente i valori contenuti dalla variabile LongVar associata al campo, qualora non si usi un valore const.
- **Total Max HP**: LongRef che rappresenta i punti vita massimi **totali** dell'entità, calcolati in base ai punti vita massimi base e a eventuali modificatori. Questo campo è sempre di sola lettura (RO) e viene aggiornato automaticamente quando i punti vita massimi base o i modificatori cambiano.
- **Current HP**: LongRef che rappresenta i punti vita attuali dell'entità. Campo editabile che viene automaticamente aggiornato quando l'entità subisce danni o viene curata. Questo campo viene anche aggiornato se si modifica, dall'inspector, `Base Max HP`.

> [!NOTE]
> Se i `Current HP` scendono a 0 e l'applicazione e' in play mode, modificare i `Base Max HP` non aggiornera' i `Current HP`. Questo e' un comporamento voluto, in quanto se un'entità e' morta, non ha senso che i suoi punti vita vengano modificati da un cambiamento dei punti vita massimi. Se invece l'entità e' viva, o l'applicazione non e' in esecuzione, modificare i `Base Max HP` aggiornera' i `Current HP` di conseguenza anche se i `Current HP` sono attualmente a 0.  
> Questo puo' capitare se, ad esempio, si passa da 10 HP base massimi a 25 cancellando la casella di testo con backspace: 10 (backspace) -> 1 (backspace) -> 0 (2 pressed) -> 2 (5 pressed) -> 25. Notate che cancellando tutti i numeri e lasciando la casella vuota, i `Base Max HP` diventano 0, e di conseguenza anche i `Current HP` diventano 0. A questo punto, l'entita' e' considerata morta.

- **Barrier**: LongRef che rappresenta eventuali punti barriera (o HP temporanei) dell'entità. La barrier assorbe i danni prima che essi intacchino i punti vita dell'entità.
- **Passive Health Regeneration**: Boolean che decreta se l'entità rigenera passivamente HP nel tempo o meno. La frequenza di rigenerazione, così come la statistica da considerare per la quantità di HP rigenerati passivamente, sono definiti nella configurazione [Astra RPG Health Config](./package-configuration.md/#health-regeneration).
- **Restore HP On Level Up**: Boolean che indica se l'entità viene curata al massimo quando sale di livello.

## Damage
Per una migliore comprensione delle prime due proprietà di questa sezione, consiglio di dare un occhiata alla documentazione delle [Damage Calculation Strategy](./damage.md/#damage-calculation-strategy). In parole povere, una Damage Calculation Strategy definisce come viene calcolato il danno che un'entità sta per subire (e.g., applicare prima la riduzione dei danni per la statistica difensiva, o prima l'assorbimento dei danni dalla barrier, quando applicare il moltiplicatore di critico, ecc.).  
Si ricorda inoltre che una strategia di default può essere assegnata attraverso la configurazione. Vedi [Default Damage Calculation Strategy](./package-configuration.md/#default-damage-calculation-strategy).  

- **Custom Damage Calculation Strategy**: Campo di tipo `DamageCalculationStrategy`. Qualora l'entità dovesse utilizzare una strategia di calcolo del danno personalizzata, è possibile specificarla qui. Questa, se definita, ha precedenza su quella definita di default attraverso la configurazione. Un caso d'uso comune potrebbe essere, ad esempio, un boss che non può subire più del 10% della sua vita massima di danni alla volta. 
- **Override Damage Calculation Strategy**: Campo di tipo `DamageCalculationStrategy`. Se definito, ha la precedenza su tutte le altre strategie di calcolo del danno. Pensato per essere assegnato a runtime per implementare particolari effeti o per testing/debug. Ad esempio, un'entità viene affetta da un debuff che trasforma tutti i danni fisici in critici garantiti. Questo debuff potrebbe pertanto essere implementato attraverso una strategia personalizzata assegnata all'entità in questo campo.
- **Is Immune**: Boolean che indica se l'entità è immune a tutti i danni o meno. Se abilitato, l'entità non subirà alcun danno, indipendentemente dalla strategia di calcolo del danno utilizzata.

### The most important method of the package: `TakeDamage`
Il metodo delle API che piu' userete con questo package e' sicuramente `TakeDamage`, la cui responsabilita' e' di applicare danni a l'entita', tenendo conto di modificatori', immunita', della strategia di calcolo del danno e altre meccaniche rilevanti. Questo metodo prende in input un `PreDamageContext` che contiene tutte le informazioni rilevanti sul danno che si intende infliggere. Per maggiori dettagli su questo contesto vi consiglio di leggere la documentazione [PreDamageContext and DamageResolutionContext](./damage.md/#predamagecontext-and-damageresolutioncontext).

Il modus operandi via codice consigliato per infliggere danni a un'entità è il seguente:
1. Costruire un istanza di `PreDamageContext` con tutte le informazioni rilevanti sul danno che si intende infliggere attraverso il suo fluent builder.
2. Chiamare `TakeDamage` passando il contesto appena costruito.

Supponiamo di star implementando una skill che infligge 50 danni da fuoco al bersaglio. Il codice per applicare il danno al bersaglio potrebbe essere il seguente:

```csharp
// Assuming that:
// - dmgType is a DamageType representing fire damage
// - dmgSource is a DamageSource representing the damage coming from a skill
// - target is the EntityCore that we want to damage
// - skillCaster is the EntityCore that casts the skill

// first we ensure that the target has an EntityHealth component
if (target.TryGetComponent(out EntityHealth targetHealth)) {
    // then we build the PreDamageContext with all the relevant information
    var preDamageContext = PreDamageContext.Builder
            .WithAmount(50)
            .WithType(dmgType)
            .WithSource(dmgSource)
            .WithTarget(target)
            .WithDealer(skillCaster)
            .Build();

    // finally, we call TakeDamage to apply the damage to the target
    targetHealth.TakeDamage(preDamageContext);
}
```

Ora, sappiamo che colare il valore del danno direttamente nel codice non è una buona pratica. Vediamo quindi come avvalerci di una `ScalingFormula` per calcolare dinamicamente l'ammontare di danno da infliggere. La creazione dello step builder diventerebbe la seguente:
```csharp
// Assuming that scalingFormula is a ScalingFormula that calculates the damage amount based on the skill caster's stats...

// ...we calculate the damage amount by evaluating the scaling formula
long damageAmount = scalingFormula.CalculateValue(skillCaster);
var preDamageContext = PreDamageContext.Builder
        .WithAmount(damageAmount)
        .WithType(dmgType)
        .WithSource(dmgSource)
        .WithTarget(target)
        .WithDealer(skillCaster)
        .Build();
```

## Healing
Un'entita' puo' venire curata in 4 maniere diverse:
- Cura diretta attraverso il metodo `Heal`.
- Rigenerazione della vita. Questa puo' essere passiva, come definito nella configurazione [Health Regeneration](./package-configuration.md/#health-regeneration), quindi automaticamente innescata dal framework ogni tot secondi, oppure puo' essere attivata manualmente attraverso il metodo `ManualHealthRegenerationTick`. Anche per quest'ultimo, si deve configurare la statistica da considerare per il calcolo della guarigione attraverso la configurazione.
- Attraverso il lifesteal. Vedi [Lifesteal](./package-configuration.md) per maggiori dettagli su questa meccanica.
- Tramite resurrezione con i due metodi `Resurrect`. Uno per resuscitare l'entita' con una percentuale di HP, e uno per resuscitarla con un ammontare fisso di punti vita. Applicabile solo se l'entità è morta.

La rigenerazione (passiva o manuale), il lifesteal e la resurrezione, usano tutti e tre il metodo `Heal` per curare effettivamente l'entità.

## Death
- **Health Can Be Negative**: Boolean che indica se i punti vita dell'entità possono scendere al di sotto di 0 prima di morire. Se disabilitato, i punti vita dell'entità non scenderanno mai al di sotto di 0, e l'entità morirà non appena i suoi punti vita raggiungono 0. Se abilitato, i punti vita dell'entità possono scendere al di sotto di 0, e l'entità morirà solo quando i suoi punti vita raggiungono un valore negativo specificato (Death Threshold).
- **Death Threshold**: LongRef che rappresenta la soglia di morte dell'entità. Visibile solo se "Health Can Be Negative" è abilitato. Se i punti vita dell'entità raggiungono questa soglia, l'entità muore.
- **Override On Death Game Action**: [Game Action](https://electricdrill.github.io/AstraRpgFrameworkDocs/MD/workflows.html#game-actions) che viene automaticamente eseguita quando l'entità muore. Se definita, questa ha la precedenza su quella definita [nella configurazione](./package-configuration.md/#default-on-death-game-action). Utile per implementare comportamenti particolari alla morte di un'entità (per esempio, entita' che esplodono alla morte infliggendo danni ad area).
- **Override On Resurrection Game Action**: [Game Action](https://electricdrill.github.io/AstraRpgFrameworkDocs/MD/workflows.html#game-actions) che viene automaticamente eseguita quando l'entità viene resuscitata. Se definita, questa ha la precedenza su quella definita [nella configurazione](./package-configuration.md/#default-on-resurrection-game-action). Utile per implementare comportamenti particolari alla resurrezione di un'entità.

## Damage vs RemoveHealth and Heal vs AddHealth
`EntityHealth` fornisce un paio di metodi pubblici per incrementare gli HP attuali e due per decrementarli:
- `Heal` e `AddHealth` per incrementare gli HP attuali. 
- `TakeDamage` e `RemoveHealth` per decrementare gli HP attuali.

E' importante chiarire perche' esistono due metodi diversi per incrementare e decrementare gli HP attuali, e quando utilizzare l'uno o l'altro.

`AddHealth` e `RemoveHealth` operano su un livello piu' basso di astrazione rispetto a `Heal` e `Damage`, in quanto modificano direttamente gli HP attuali dell'entità di un valore `long` specificato, senza passare attraverso la pipeline di calcolo del danno o senza tener conto di eventuali modificatori di guarigione o danno. Sono metodi molto prevedibili e diretti, e sono prevalentemente utilizzati internamente dal framework.  
Questi metodi possono essere utili in situazioni specifiche dove il guadagno di HP non e' dovuto a una guarigione, o la perdita di HP non e' dovuta a un danno, ma piuttosto a meccaniche particolari che richiedono una modifica diretta degli HP attuali. Ad esempio, supponiamo che in seguito ad una cutscene vogliamo forzare gli HP del giocatore a 1. In questo caso, potremmo utilizzare `RemoveHealth` per rimuovere tutti gli HP tranne 1, senza doverci preoccupare di eventuali modificatori di danno o della strategia di calcolo del danno che altererebbero i danni subiti in base alle statistiche ed equipaggiamento del giocatore, oltre a sollevare eventi di danno che potrebbero attivare logica di gioco che non vogliamo attivare in questo caso specifico.

`Heal` e `TakeDamage`, invece, sono metodi più complessi che tengono conto di vari fattori come la strategia di calcolo del danno, l'immunità ai danni, modificatori di guarigione, ecc. Questi metodi sono pensati per essere utilizzati principalmente dagli sviluppatori di gioco per applicare danni e guarigioni alle entità, in quanto garantiscono che tutte le meccaniche e le regole del sistema di salute vengano rispettate. Ad esempio, se si vuole infliggere danni a un'entità, è consigliabile utilizzare il metodo `TakeDamage`, in modo che il danno venga calcolato correttamente in base alla strategia di calcolo del danno configurata, e che eventuali immunità o modificatori vengano presi in considerazione.  
Senza troppe sorprese, `Heal` e `TakeDamage` utilizzano internamente `AddHealth` e `RemoveHealth` per modificare effettivamente gli HP attuali dell'entità dopo aver calcolato il guadagno o la perdita di HP netti da applicare.

Nel 99% dei casi, utilizzerete `Heal` e `TakeDamage` per applicare guarigioni e danni alle entità. `AddHealth` e `RemoveHealth` sono disponibili per coprire i casi d'uso piu' rari e particolari.

## Events
Events are, by default, collapsed as there are many of them and they would expand excessively in the inspector. By opening them, we should see something like this:
![EntityHealth Events](../../images/AstraRPG/workflows/entity-health/entity-health-events-section.png)

In the image, you see already assigned events, but when you first add the component, all events will be unassigned and empty.

Partiamo introducendo la differenza tra _Global Events_ e _Extra Events_.

### Global Events
I Global Events sono eventi fondamentali che trasmettono informazioni importanti a tutto il framework. Gli eventi assegnati a questi slot devono avere uno scope globale, cioè devono poter essere ascoltati da qualsiasi entità o sistema del gioco.  
Questi eventi vengono utilizzati dal framework per gestire funzionalità essenziali, come ad esempio il lifesteal e altre meccaniche che richiedono una comunicazione centralizzata tra le varie parti del sistema.

> [!WARNING]
> Riservare questi slot per eventi di natura globale è fondamentale per garantire il corretto funzionamento del framework.  
> Assegnare Game Event instances che sono specifiche per una o alcune entità, comporterebbe problemi di funzionamento a livello di framework.

### Extra Events
Gli Extra Events, invece, sono pensati per trasmettere informazioni a componenti specifiche o a una cerchia ristretta di entità. Un esempio pratico è la comunicazione tra l'EntityHealth del giocatore e l'HUD che visualizza i suoi HP: solo il giocatore possiede un HUD dedicato, quindi è utile assegnare un evento esclusivo per questa interazione. In questo modo, il GameEvent associato all'EntityHealth del giocatore viene ascoltato solo dall'HUD, garantendo una netta compartimentazione e maggiore efficienza. Così, l'HUD non riceve eventi da tutte le entità, ma solo quelli rilevanti per il giocatore.

### Events Breakdown
Qui segue una descrizione dettagliata di ogni evento. Ogni tipo di evento ha sia l'evento globale che una lista di eventi extra, ma è sufficiente descrivere ogni evento una singola volta.

#### Damage Related Events
To better understand the first two events of this section, I recommend taking a look at the [Damage](./damage.md) documentation to get a better understanding of damage in Astra RPG Health.

- **Pre Damage Info Event**: Evento sollevato prima che l'entita' subisca danni, e prima che la [pipeline del calcolo dei danni](./damage.md/#damage-calculation-pipeline) calcoli il net damage. Questo evento trasmette informazioni sul danno che l'entità sta per subire, come il tipo di danno, la fonte del danno (un'entità, `null` se non applicabile), la damage source (e.g., environmental, skill, trap, ecc.), il raw damage che si intende infliggere, e altre informazioni rilevanti. Per maggiori dettagli in merito al parametro di contesto e i suoi campi, si rimanda alla documentazione delle API [PreDamageContext](xref:ElectricDrill.AstraRpgHealth.Damage.PreDamageContext).  
Questo evento puo' risultare utile per implementare abilita' passive o, in generale, meccaniche avanzate che attivano effetti in risposta a specifiche condizioni legate al danno che un'entità sta per subire. Ad esempio, un debuff o status modifier che amplifica il danno critico subito dell 50% quando il damage type e' "Fire", o un'abilità passiva che nega tutte le istanze di danno che si stanno per subire qualora esse avessero raw damage inferiore a 50.
- **Damage Resolution Event**: Evento sollevato dopo che l'entità ha subito o ignorato dei danni. Similarmente al parametro di contesto dell'evento precedente, questo evento trasmette informazioni dettagliate sul danno appena subito o ignorato. Per maggiori dettagli in merito al parametro di contesto e i suoi campi, si rimanda alla documentazione delle API [DamageResolutionContext](xref:ElectricDrill.AstraRpgHealth.Damage.DamageResolutionContext).

In generale, la thumb rule per decidere se utilizzare il Pre Damage Info Event o il Damage Resolution Event è la seguente: se si vuole manipolare il danno che un'entità sta per subire, conviene sottoscriversi a Pre Damage Info Event, e modificare il contesto come desiderato; se invece si vuole reagire al danno che un'entità ha appena subito, e attivare effetti in risposta a esso, conviene sottoscriversi a Damage Resolution Event, e reagire in base alle informazioni trasmesse dal suo contesto.

#### Health Related Events
- **Gained Health Event**: Evento sollevato quando l'entità guadagna HP, sia attraverso la guarigione che altri meccanismi (e.g., Max HP modifiers that caused a gain of health).  
Fare riferimento alle API [EntityHealthChangedContext](xref:ElectricDrill.AstraRpgHealth.Events.EntityHealthChangedContext) per maggiori dettagli sul parametro di contesto.
- **Lost Health Event**: Evento sollevato quando l'entità perde HP, sia attraverso il subire danni che altri meccanismi (e.g., Max HP modifiers that caused a loss of health).  
Il parametro di contesto di questo evento e' lo stesso del precedente, quindi si rimanda alle API [EntityHealthChangedContext](xref:ElectricDrill.AstraRpgHealth.Events.EntityHealthChangedContext) per maggiori dettagli.
- **Max Health Changed Event**: Evento sollevato quando i punti vita massimi totali di un'entità cambiano. Questo evento viene sollevato sia quando i max hp massimi totali aumentano che quando diminuiscono.  
Vedi API [EntityMaxHealthChangedContext](xref:ElectricDrill.AstraRpgHealth.Events.EntityMaxHealthChangedContext) per maggiori dettagli sul parametro di contesto.
- **Entity Died Event**: Evento sollevato quando l'entità muore. Vedi API [EntityDiedContext](xref:ElectricDrill.AstraRpgHealth.Events.EntityDiedContext) per maggiori dettagli sul parametro di contesto.

#### Healing Related Events
- **Pre Heal Event**: Evento sollevato prima che l'entità venga curata. Questo evento trasmette informazioni sulla guarigione che l'entità sta per ricevere, come l'ammontare di HP che si intende curare, l'entita' che ha fornito la cura (`null` se non applicabile), la heal source (e.g., skill, item, potion, ecc.) e altre informazioni rilevanti. Per maggiori dettagli in merito al parametro di contesto e i suoi campi, si rimanda alla documentazione delle API [PreHealContext](xref:ElectricDrill.AstraRpgHealth.Heal.PreHealContext).  
L'ammontare dei punti vita che si intende curare, trasmesso da questo evento, e' il valore prima di applicare eventuali modificatori di guarigione. Questo evento puo' risultare utile per implementare abilita' passive o, in generale, meccaniche avanzate che attivano effetti in risposta a specifiche condizioni legate alla guarigione che un'entità sta per ricevere. Ad esempio, un buff che amplifica le guarigioni derivanti da pozioni (heal source) del 30%.
- **Entity Healed Event**: Evento sollevato quando l'entità e' stata curata. Questo evento trasmette informazioni sulla guarigione appena ricevuta, come l'ammontare di HP che l'entità ha effettivamente guadagnato dopo l'applicazione di eventuali modificatori di guarigione. Per maggiori sul parametro di contesto vedere le API [ReceivedHealContext](xref:ElectricDrill.AstraRpgHealth.Heal.ReceivedHealContext).

Anche qui, come per gli eventi del danno, la thumb rule per decidere se utilizzare il Pre Heal Event o l'Entity Healed Event è la seguente: se si vuole manipolare la guarigione che un'entità sta per ricevere, conviene sottoscriversi a Pre Heal Event, e modificare il contesto come desiderato; se invece si vuole reagire alla guarigione che un'entità ha appena ricevuto, e attivare effetti in risposta ad essa, conviene sottoscriversi a Entity Healed Event, e reagire in base alle informazioni trasmesse dal suo contesto.

#### Resurrection Related Events
- **Entity Resurrected Event**: Evento sollevato quando l'entità viene resuscitata. Vedi API [ResurrectionContext](xref:ElectricDrill.AstraRpgHealth.Resurrection.ResurrectionContext) per maggiori dettagli sul parametro di contesto.