# `EntityHealth` component
Con Astra RPG Health, il nuovo MonoBehaviour `EntityHealth` consente di aggiungere dei punti vita a un'entità. In questo modo, l'entità può subire danni e, di conseguenza, morire.

Ecco un esempio del componente `EntityHealth`:
![EntityHealth](../../images/AstraRPG/workflows/entity-health/entity-health-fresh-component.png)

La sezione degli eventi di default e' collassata poiché ci sono molti eventi, e risulta più pratico espanderla solo quando necessario.

Procediamo però con ordine e analizziamo ogni proprietà del componente `EntityHealth`:
- **Use Class Max HP**: booleano che indica se utilizzare i punti vita massimi definiti nella classe dell'entità come punti vita massimi base. Se è disabilitato, è possibile specificare manualmente i punti vita massimi base per l'entità.
- **Base Max HP**: LongRef che rappresenta i punti vita massimi **base** dell'entità. Se "Use Class Max HP" è abilitato, questo valore viene marcato con `RO` di color teal (Read Only). Read Only, per i campi di tipo LongRef, rende non editabili i valori const, e suggersice di non modificare manualmente i valori contenuti dalla variabile LongVar associata al campo, qualora non si usi un valore const.
- **Total Max HP**: LongRef che rappresenta i punti vita massimi **totali** dell'entità, calcolati in base ai punti vita massimi base e a eventuali modificatori. Questo campo è sempre di sola lettura (RO) e viene aggiornato automaticamente quando i punti vita massimi base o i modificatori cambiano.
- **Current HP**: LongRef che rappresenta i punti vita attuali dell'entità. Campo editabile che viene automaticamente aggiornato quando l'entità subisce danni o viene curata. Questo campo viene anche aggiornato se si modifica, dall'inspector, `Base Max HP`.

> [!NOTE]
> Se i `Current HP` scendono a 0 e l'applicazione e' in play mode, modificare i `Base Max HP` non aggiornera' i `Current HP`. Questo e' un comporamento voluto, in quanto se un'entità e' morta, non ha senso che i suoi punti vita vengano modificati da un cambiamento dei punti vita massimi. Se invece l'entità e' viva, o l'applicazione non e' in esecuzione, modificare i `Base Max HP` aggiornera' i `Current HP` di conseguenza anche se i `Current HP` sono attualmente a 0.  
> Questo puo' capitare se, ad esempio, si passa da 10 HP base massimi a 25 cancellando la casella di testo con backspace: 10 (backspace) -> 1 (backspace) -> 0 (2 pressed) -> 2 (5 pressed) -> 25. Notate che cancellando tutti i numeri e lasciando la casella vuota, i `Base Max HP` diventano 0, e di conseguenza anche i `Current HP` diventano 0. A questo punto, l'entita' e' considerata morta.

- **Barrier**: LongRef che rappresenta eventuali punti barriera (o HP temporanei) dell'entità. 