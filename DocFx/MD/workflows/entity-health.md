# `EntityHealth` component
Con Astra RPG Health, il nuovo MonoBehaviour `EntityHealth` consente di aggiungere dei punti vita a un'entità. In questo modo, l'entità può subire danni e, di conseguenza, morire.

Ecco un esempio del componente `EntityHealth`:
![EntityHealth](../../images/AstraRPG/workflows/entity-health/entity-health-fresh-component.png)

La sezione degli eventi di default e' collassata poiché ci sono molti eventi, e risulta più pratico espanderla solo quando necessario.

Procediamo però con ordine e analizziamo ogni proprietà del componente `EntityHealth`:
- **Use Class Max HP**: booleano che indica se utilizzare i punti vita massimi definiti nella classe dell'entità come punti vita massimi base. Se è disabilitato, è possibile specificare manualmente i punti vita massimi base per l'entità.
- **Base Max HP**: LongRef che rappresenta i punti vita massimi **base** dell'entità. Se "Use Class Max HP" è abilitato, questo valore viene marcato con `RO` di color teal (Read Only). Read Only, per i campi di tipo LongRef, rende non editabili i valori const, e suggersice di non modificare manualmente i valori contenuti dalla variabile LongVar associata al campo, qualora non si usi un valore const.
- **Total Max HP**: LongRef che rappresenta i punti vita massimi **totali** dell'entità, calcolati in base ai punti vita massimi base e a eventuali modificatori. Questo campo è sempre di sola lettura (RO) e viene aggiornato automaticamente quando i punti vita massimi base o i modificatori cambiano.
- **Current HP**: LongRef che rappresenta i punti vita attuali dell'entità. Campo editabile che viene automaticamente aggiornato quando l'entità subisce danni o viene curata.