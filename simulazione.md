# simulazione-architettura-degli-elaboratori
Svolgimento simulazione di Architettura degli Elaboratori @unipd

# Si spieghi in dettaglio la rappresentazione dei numeri reali secondo lo standard IEEE 754.

Lo IEEE 754 è lo standard internazionale per rappresentare i numeri a virgola mobile, ed è impiegato in tutti i moderni processori e coprocessori aritmetici.
Tale standard definisce un formato a 32 bit (detto *float*) e un formato a 64 bit (*double*), la cui dimensione del campo esponente (in formato polarizzato)
è rispettivamente di 8 e 11 bit. La base è 2 ed è implicita, il primo bit è relativo al segno del numero, e i restanti bit sono dedicati alla mantissa. 
Alcune combinazioni di bit servono per rappresentare dei particolari valori. I valori estremi dell'esponente, composti da tutti 0 e tutti 1
(2^8-1 in formato singolo, 2^11-1 in formato doppio), definiscono difatti dei valori speciali. 
Per esponenti nell'intervallo 1..2^8-1 in float e 1..2^11-2 in double vengono rappresentati i numeri normalizzati non nulli in virgola mobile.
L'esponente è polarizzato e varia da -2^7+2 a 2^7-1 in float e -2^10+2 e 2^10-1 in double. Un numero normalizzato richiede un bit a 1 a sinistra 
della virgola binaria. Tale bit è implicito, costituendo di fatto un significando effettivo a 24 o 53 bit, detto mantissa o frazione.
- Un esponente 0 con mantissa a 0 rappresente +0 o -0, a seconda del bit di segno.
- Un esponente di tutti 1 con mantissa 0 rappresenta l'infinito positivo o negativo, a seconda del bit di segno. Questo fatto lascia all'utente la
decisione finale di valutare come overflow come condizione di errore o come rappresentazione di infinito (ad esempio in un'applicazione dedicata al calcolo di limiti).
- Un esponente a 0 con una frazione non nulla rappresenta un numero denormalizzato. In questo caso, il bit a sinistra della virgola binaria
rappresenta un numero denormalizzato, e il vero esponente è -126 o -1022.
- Un esponente di tutti 1 con mantissa non nulla ha il valore simbolico di NaN, utile per segnalare condizioni di errore.

- - -

# Si descriva nel dettaglio la modalità di indirizzamento per spiazzamento.

L'indirizzamento per spiazzamento combina le capacità di quello diretto con quelle del registro indiretto. 
Il suo meccanismo di base è EA = A + (R), con:

* A: contenuto di un campo indirizzo di un'istruzione
* R: contenuto di un campo indirizzo di un'istruzione che riferisce a un registro
* EA: indirizzo effetivo della locazione che contiene l'operando referenziato.
Tale tecnica prevede che l'istruzione abbia due campi indirizzo, di cui uno esplicito. Il valore A di un campo indirizzo è usato direttamente. L'altro campo indirizzo (oppure un riferimento implicito basato sull'OP) si riferisce ad un altro registro il cui valore, sommato ad A, dia l'indirizzo effettivo.
Esistono tre sottomodalità:

1. indirizzamento relativo
2. indirizzamento base-registro
3. indicizzazione

1. Nell'indirizzamento relativo il registro implicitamente referenziato è il PC, quindi l'indirizzo dell'istruzione corrente viene sommato al campo indirizzo per produrre l'EA. Il campo indirizzo viene trattato come un numero in complemento a 2. Questa modalità di spiazzamento sfrutta la località dei riferimenti: se la maggior parte dei riferimenti alla memoria sono relativamenti vicini all'istruzione in esecuzione, allora l'impiego dell'indirizzamento relativo risparmia bit di indirizzo nell'istruzione.

2. Nell'indirizzamento registro-base il registro referenziato contiene un indirizzo di memoria, mentre il campo indirizzo contiene uno spiazzamento (solitamente rappresentato con un intero senza segno) rispetto a tale indirizzo. Il riferimento al registro può essere esplicito o implicito. Anche questa tecnica sfrutta la località degli accessi alla memoria. Si tratta di un metodo adatto all'implementazione della segmentazione.

3. Nell’indicizzazione il campo indirizzo rappresenta un indirizzo di memoria centrale e il registro referenziato contiene uno spiazzamento positivo da tale indirizzo. L'indicizzazione si rivela molto utile nelle operazioni operative. Supponendo di avere un elenco di numeri che parte dall'indirizzo A, e di dover scorrere tale elenco, sarà necessario incrementare A di (ad esempio) 1 ogni volta. Il valore A viene memorizzato nel campo dell'indirizzo dell'istruzione e al registro scelto, detto registro indice, viene inizialmente assegnato il valore 0. Dopo ciascuna operazione, il registro indice è incrementato. Alcune sistemi impiegano l'incremento all'interno dello stesso ciclo di clock dell'operazione; l'indicizzazione di tali sistemi è detta *autoindicizzazione*
di 1.
