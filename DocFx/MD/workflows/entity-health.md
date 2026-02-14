# `EntityHealth` component
Con Astra RPG Health, il nuovo MonoBehaviour `EntityHealth` consente di aggiungere dei punti vita a un'entità. In questo modo, l'entità può subire danni e, di conseguenza, morire.

Ecco un esempio del componente `EntityHealth`:
![EntityHealth](../../images/AstraRPG/workflows/entity-health/entity-health-fresh-component.png)

La sezione degli eventi di default e' collassata poiché ci sono molti eventi, e risulta più pratico espanderla solo quando necessario.

Procediamo però con ordine e analizziamo ogni proprietà del componente `EntityHealth`.

## Health
- **Use Class Max HP**: booleano che indica se utilizzare i punti vita massimi definiti nella classe dell'entità come punti vita massimi base. Se è disabilitato, è possibile specificare manualmente i punti vita massimi base per l'entità.
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

- **Custom Damage Calculation Strategy**: campo di tipo `DamageCalculationStrategy`. Qualora l'entità dovesse utilizzare una strategia di calcolo del danno personalizzata, è possibile specificarla qui. Questa, se definita, ha precedenza su quella definita di default attraverso la configurazione. Un caso d'uso comune potrebbe essere, ad esempio, un boss che non può subire più del 10% della sua vita massima di danni alla volta. 
- **Override Damage Calculation Strategy**: campo di tipo `DamageCalculationStrategy`. Se definito, ha la precedenza su tutte le altre strategie di calcolo del danno. Pensato per essere assegnato a runtime per implementare particolari effeti o per testing/debug. Ad esempio, un'entità viene affetta da un debuff che trasforma tutti i danni fisici in critici garantiti. Questo debuff potrebbe pertanto essere implementato attraverso una strategia personalizzata assegnata all'entità in questo campo.
- **Is Immune**:

## Death
- **Health Can Be Negative**:
- **Override On Death Game Action**:
- **Override On Resurrection Game Action**:

## Events
Events are, by default, collapsed as there are many of them and they would expand excessively in the inspector. By opening them, we should see the following:
