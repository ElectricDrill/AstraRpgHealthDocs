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

## Death
- **Health Can Be Negative**: Boolean che indica se i punti vita dell'entità possono scendere al di sotto di 0 prima di morire. Se disabilitato, i punti vita dell'entità non scenderanno mai al di sotto di 0, e l'entità morirà non appena i suoi punti vita raggiungono 0. Se abilitato, i punti vita dell'entità possono scendere al di sotto di 0, e l'entità morirà solo quando i suoi punti vita raggiungono un valore negativo specificato (Death Threshold).
- **Death Threshold**: LongRef che rappresenta la soglia di morte dell'entità. Visibile solo se "Health Can Be Negative" è abilitato. Se i punti vita dell'entità raggiungono questa soglia, l'entità muore.
- **Override On Death Game Action**: [Game Action](https://electricdrill.github.io/AstraRpgFrameworkDocs/MD/workflows.html#game-actions) che viene automaticamente eseguita quando l'entità muore. Se definita, questa ha la precedenza su quella definita [nella configurazione](./package-configuration.md/#default-on-death-game-action). Utile per implementare comportamenti particolari alla morte di un'entità (per esempio, entita' che esplodono alla morte infliggendo danni ad area).
- **Override On Resurrection Game Action**: [Game Action](https://electricdrill.github.io/AstraRpgFrameworkDocs/MD/workflows.html#game-actions) che viene automaticamente eseguita quando l'entità viene resuscitata. Se definita, questa ha la precedenza su quella definita [nella configurazione](./package-configuration.md/#default-on-resurrection-game-action). Utile per implementare comportamenti particolari alla resurrezione di un'entità.

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