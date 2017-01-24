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
- Un esponente 0 con mantissa a 0 rappresenta +0 o -0, a seconda del bit di segno.
- Un esponente di tutti 1 con mantissa 0 rappresenta l'infinito positivo o negativo, a seconda del bit di segno. Questo fatto lascia all'utente la
decisione finale di valutare come overflow come condizione di errore o come rappresentazione di infinito (ad esempio in un'applicazione dedicata al calcolo di limiti).
- Un esponente a 0 con una frazione non nulla rappresenta un numero denormalizzato. In questo caso, il bit a sinistra della virgola binaria
rappresenta un numero denormalizzato, e il vero esponente è -126 o -1022.
- Un esponente di tutti 1 con mantissa non nulla ha il valore simbolico di NaN, utile per segnalare condizioni di errore.

# Si descriva nel dettaglio la modalità di indirizzamento per spiazzamento.

L'indirizzamento per spiazzamento combina le capacità di quello diretto con quelle del registro indiretto. 
Il suo meccanismo di base è EA = A + (R), con:

  * A: contenuto di un campo indirizzo di un'istruzione
  * R: contenuto di un campo indirizzo di un'istruzione che riferisce a un registro
  * EA: indirizzo effetivo della locazione che contiene l'operando referenziato.
Tale tecnica prevede che l'istruzione abbia due campi indirizzo, di cui uno esplicito. Il valore A di un campo indirizzo è usato direttamente. L'altro campo indirizzo (oppure un riferimento implicito basato sull'OP) si riferisce ad un altro registro il cui valore, sommato ad A, dia l'indirizzo effettivo.
Esistono tre sottomodalità:
* indirizzamento relativo
* indirizzamento base-registro
* indicizzazione

1. Nell'indirizzamento relativo il registro implicitamente referenziato è il PC, quindi l'indirizzo dell'istruzione corrente viene sommato al campo indirizzo per produrre l'EA. Il campo indirizzo viene trattato come un numero in complemento a 2. Questa modalità di spiazzamento sfrutta la località dei riferimenti: se la maggior parte dei riferimenti alla memoria sono relativamenti vicini all'istruzione in esecuzione, allora l'impiego dell'indirizzamento relativo risparmia bit di indirizzo nell'istruzione.

2. Nell'indirizzamento registro-base il registro referenziato contiene un indirizzo di memoria, mentre il campo indirizzo contiene uno spiazzamento (solitamente rappresentato con un intero senza segno) rispetto a tale indirizzo. Il riferimento al registro può essere esplicito o implicito. Anche questa tecnica sfrutta la località degli accessi alla memoria. Si tratta di un metodo adatto all'implementazione della segmentazione.

3. Nell’indicizzazione il campo indirizzo rappresenta un indirizzo di memoria centrale e il registro referenziato contiene uno spiazzamento positivo da tale indirizzo. L'indicizzazione si rivela molto utile nelle operazioni operative. Supponendo di avere un elenco di numeri che parte dall'indirizzo A, e di dover scorrere tale elenco, sarà necessario incrementare A di (ad esempio) 1 ogni volta. Il valore A viene memorizzato nel campo dell'indirizzo dell'istruzione e al registro scelto, detto registro indice, viene inizialmente assegnato il valore 0. Dopo ciascuna operazione, il registro indice è incrementato. Alcune sistemi impiegano l'incremento all'interno dello stesso ciclo di clock dell'operazione; l'indicizzazione di tali sistemi è detta *autoindicizzazione*
di 1.

# Si metta a confronto criticamente il modo in cui un’architettura RISC utilizza l’ampio banco di registri a sua disposizione rispetto alla gestione di una cache

L’architettura RISC utilizza l’ampio banco di registri per conservare le variabili (prevalentemente scalari locali) che hanno un’alta probabilità di essere utilizzate con maggior frequenza. Sotto questo punto di vista il banco dei registri assomiglia molto alla memoria cache, sbbene sia molto più veloce. Ci sono però alcune sostanziali differenze:
Il BR contiene tutti gli scalari locali delle ultime N-1 procedure attivate e la cache contiene gli scalari locali usati di recente.
Le variabili globali sono assegnate dal compilatore per quanto riguarda il BR, mentre la cache contiene solo quelle usate di recente.
Mentre il BR contiene solo le variabili in uso, la cache importa un blocco di dati che potrebbe non essere utilizzato.
Il trasferimento dati tra registri e memoria è determinato in base alla profondità di annidamento delle procedure, mentre per la cache il salvataggio e il rimpiazzo è determinato in base all’algoritmo di sostituzione adottato.
Per accedere ad uno scalare locale in un BR viene utilizzato l’indirizzamento a registro, che è molto semplice e veloce.
Per quanto riguarda la cache invece, l’indirizzamento è molto più lento. La maggior parte delle cache infatti sono set-associative, il che comporta delle operazioni di confronto con il set e il tag per determinare la presenza o no di un dato in cache.
Anche nel caso della cache a indirizzamento diretto o fully-associative, questi modi sono comunque indirizzamenti a memoria che sono sempre più lenti di un indirizzamento a registro.

# Nel contesto di una pipeline, descrivere nel dettaglio la tecnica del data-forwarding: a cosa serve? Come funziona? Di che supporto hardware ha bisogno? 

Il data forwarding è una delle soluzioni utilizzate per superare le dipendenze dai dati che si possono verificare tra le istruzioni in una pipeline. 
Se viene individuata una dipendenza dai dati, il data forwarding, se il tipo di dipendenza lo permette, permette di trasferire i dati dall'output della ALU in ingresso alla ALU. Questo è possibile grazie ad appositi circuiti di ByPass e MUX, regolati da Unità di Controllo e potenzialmente anche da altre unità (dipende dall'architettura), che permettono alla ALU di caricare dati derivanti dalla memoria o dati che la ALU stessa ha mandato in output nel ciclo precedente. 
Nella realtà il dataforward può essere implementato in diversi modi che dipendono dall'architettura su cui lo si implementa.
Un esempio pratico di data-forwarding lo troviamo nell'architettura MIPS in cui vi è un apposito circuito di identificazione delle dipendenze e gestione del data-forwarding. Tale circuito ha tre componenti fondamentali:

- Forwarding Unit: unità che decide se attivare il forward attivando nell'opportuno modo i multiplexer della ALU 
- Hazard detection Unit: unità in grado di riconoscere le dipendenze e di generare stalli in caso di dipendenze non risolvibili 
- Control Unit: manda segnali di controllo che regolano il forward ed i dati che devono essere mandati (inoltre manda segnali di controllo che regolano l'esecuzione e l'utilizzo dell'hardware per la memorizzazione ed il write back) 
Nel caso in cui la dipendenza venga rilevata come risolvibile, allora nelle MIPS sarà possibile fare il forward di dati da fase EX a EX e da fase MEM a EX.

# Spiegare in che modo un compilatore possa aiutare l’utilizzo efficace dei registri da parte di un’architettura RISC.

L’obiettivo del compilatore è mantenere gli operandi necessari nei registri per la maggior parte del tempo e minimizzare il numero di accessi alla memoria. 
Il suo compito è eseguire un’approfondita analisi del programma sottopostogli e mappare ogni registro simbolico (o virtuale) a cui è stata assegnata una variabile, in un registro fisico del processore. 
Più registri simbolici possono esser mappati su uno stesso registro reale se il loro uso non si sovrappone temporalmente.
Se i registri reali non sono sufficienti per contenere tutte le variabili riferite in un certo intervallo di tempo, allora queste ultime vengono assegnate a locazioni di memoria.
L’essenza dell’ottimizzazione è quella di decidere quali variabili debbano essere assegnate ai registri in un certo punto del programma. Il mapping delle variabili è equivalente alla risoluzione del problema di colorazione di un grafo. In questo problema dato un grafo costituito da nodi connessi ad archi, si vuole assegnare un colore per ogni nodo in modo tale che:
Nodi adiacenti connessi da archi abbiamo colori diversi
Usare il minor numero possibile di colori
I nodi rappresentano i registri virtuali e i colori i registri reali.
Essendovi a disposizione n registri reali, il grafo dovrà essere colorato con al massimo n colori. Se vi sono nodi  che non potranno essere colorati, allora verranno copiati in memoria.

# Spiegare in dettaglio come un’architettura RISC possa trattare efficientemente la chiamata annidata di procedure

Dall'osservazione che le chiamate di procedura tipicamente coinvolgono pochi parametri e non presentano un grado di annidamento elevata, si è pensato di usare molti gruppi di registri, detti finestre di registri, per gestire le chiamate annidate. 
Una chiamata seleziona automaticamente un nuovo gruppo di registri e quando essa è conclusa ed effettua il ritorno, riseleziona il gruppo di registri riferito alla chiamata che l'aveva chiamata. 
Ogni gruppo di registri è suddiviso in tre sottogruppi: parametri passati alla procedura, registri che memorizzano il contenuto delle variabili locali della procedura e registri temporanei che gestiscono il ritorno della procedura. 
I registri temporanei di un gruppo si sovrappongono perfettamente con quelli che contengono i parametri del gruppo successivo, cioè del gruppo riferito ad una chiamata annidata. Tali registri sono fisicamente gli stessi e ciò permette il passaggio dei parametri senza trasferimento dei dati. 
La realizzazione fisica di finestre di registri sovrapposte avviene tramite buffer circolare. 
Nel buffer circolare, quando avviene una chiamata il puntatore alla finestra corrente (CWP) viene aggiornato per farlo puntare alla finestra attiva. Se si esaurisce la capacità del buffer, cioè tutte le finestre sono in uso a causa di chiamate annidate, la finestra che per prima è stata inserita nel buffer viene salvata in memoria principale e quindi sovrascritta dalla nuova. Quando una procedura termina, una finestra viene liberata e grazie ad un apposito puntatore (SWP) è possibile ripristinare l'ultima finestra salvata in memoria principale.

# Formato delle istruzioni

Il formato delle istruzioni descrive i campi dell'istruzione, la sua lunghezza ed numero di indirizzi. In qualsiasi formato è incluso il codice operativo, che discrimina quale operazione fare, ed zero, uno o più operandi in modo implicito o esplicito. 

Il formato delle istruzioni può essere a lunghezza: 
- fissa, tutte le istruzioni hanno la stessa lunghezza, ma posso avere più formati cambiando i campi. Questo formato è estremamente efficiente nell'uso della pipeline. 
- variabile, ogni istruzione ha una lunghezza che dipende dal numero di operandi e nel campo con l'opcode devo anche specificare il numero di operandi. Permettono una grande flessibilità, ma incrementano notevolmente la complessità. 
- ibrida, ho diversi formati con lunghezza fissa ma diversa 

La lunghezza è data da un compromesso tra repertorio di istruzioni potente e necessità di risparmiare spazio ed è condizionata da diversi fattori: dimensione memoria (deve poter essere completamente indirizzata), organizzazione della memoria, struttura del bus (in base a quanto è capiente posso discriminare il numero di accessi), complessità della CPU (CPU con più istruzioni necessita di più bit per l'opcode) e la velocità richiesta della CPU(se la CPU utilizza la pipeline). 
L'allocazione dei bit nei campi d'indirizzo dipende da: numero dei metodi d'indirizzamento, numero di operandi, numero di registri (più sono, più vengono utilizzati, minore sarà il numero di bit utilizzati per il più costoso indirizzamento a memoria), numero di banchi di registro (solo alcune architetture li posseggono), intervallo di indirizzi da rendere disponibile, granularità d'indirizzamento (byte o parola, a volte può essere utile il byte anche se più costoso nel numero di accessi). 

Prima di inoltrasi nella descrizione dei formati è utile precisare degli svantaggi e vantaggi generali: 
- Una grande varietà di istruzioni e di lunghezze differenti permettono l'utilizzo di un vasto numero di opcode e metodi di indirizzamento, permettendo una grande flessibilità. Questo tipo di istruzioni può essere realizzato utilizzando istruzioni a lunghezza variabile, ma il loro 
utilizzo complica notevolmente la CPU(rendendola anche più costosa), rende difficile il fetch e globalmente diminuisce la reattività sulle operazione più frequentemente utilizzate. 
Tale caratteristiche si avvicinano alla filosofia CISC. 
- Un numero fisso di formati ha come problemi la mancanza di ortogonalità(cioè operandi indipendenti dal codice operativo) tra opcode e metodo d'indirizzamento ed un numero limitato di operandi utilizzabili per operazione. In generale vi è una mancanza di flessibilità. La CPU è però più semplice da realizzare, generalmente più, permette il caricamento dell'istruzione in modo uniforme veloce ed una dimensione fissa delle istruzioni favorisce l'uso della pipeline. Tale caratteristiche si avvicinano alla filosofia RISC. 

I possibili formati di codifica di una istruzione sono: 
- PDP-8: architettura molto vecchia in cui era disponibile un solo registro, l'accumulatore. Nonostante le forti limitazioni vi è un indirizzamento abbastanza flessibile. Le istruzioni erano a lunghezza fissa, con molti formati, che permettevano un estensione degli opcode grazie all'utilizzo di microperazioni. 
Un formato ottimizzato al massimo per quel tempo. 
- PDP-10: innovativo rispetto al PDP-8: un unico formato a lunghezza fissa, le stesse tipologie di operazioni per diversi tipi di operandi, solo indirizzamento diretto ed introdotto il concetto di ortogonalità. 
Facilitava il lavoro dei programmatori e dei compilatori ma non utilizzava in modo efficiente lo spazio a disposizione. (scelta progettuale) 
- PDP-11: adotta una formato ibrido in cui ogni operando può utilizzare un qualsiasi tipo di indirizzamento. Ha un'ampia variabilità di formati e metodi di indirizzamento che la rendono costosa e complicata da realizzare, ma i programmi risultano essere efficienti e compatti. 
- VAX: un sistema estremamente variabile con lunghezza varabile e formati diversi, in quanto l'opcode può stare su un byte o su due. E' un sistema molto flessibile e potente che facilita il lavoro di programmatori e compilatori, ma il sistema è molto complesso. 
- PENTIUM: le istruzioni sono composte da vari pezzi i quali vengono assemblati in base alle necessità. Il risultato sono quindi istruzioni a lunghezza variabile e di vari formati che richiedono una complessa decodifica. Tale approccio è stato utilizzato per mantenere la retro compatibilità. 
- PowerPC: un sistema con istruzioni a lunghezza fissa e diversi formati, avendo una suddivisione non omogenea dei campi. Pensato per computer RISC.

# Si discuta nel dettaglio in cosa consista il formato variabile per le istruzioni. Se possibile, dare esempi di formati variabili. 

Un formato variabile per le istruzioni è un formato che permette una grande varietà di istruzioni ed una grande flessibilità. 
Tali istruzioni hanno lunghezze variabili, cioè ogni istruzione ha una lunghezza che dipende dal numero di operandi ed ha un campo opcode generalmente espanso, così da permettere la codifica di un vasto numero di operazioni e metodi di indirizzamento ed inoltre il campo nel campo opcode viene anche specificare il numero di operandi presenti nell'istruzione. Una istruzione con formato variabile può inoltre avere un numero variabile di campi di tipo diverso, come ad esempio nel PENTIUM. 
L'utilizzo di questo tipo di istruzioni complica notevolmente la CPU, la rende più costosa, rende difficile il fetch e globalmente diminuisce la reattività sulle operazione più frequentemente utilizzate. 
Tale caratteristiche si avvicinano alla filosofia CISC. 
Due esempi di formati variabili delle istruzioni li troviamo storicamente nel: 
 VAX: un sistema estremamente variabile con lunghezza varabile e formati diversi, in quanto l'opcode può stare su un byte o su due. E' un sistema molto flessibile e potente che facilita il lavoro di programmatori e compilatori, ma il sistema è molto complesso. 
 PENTIUM: le istruzioni sono composte da vari pezzi i quali vengono assemblati in base alle necessità. Il risultato sono quindi istruzioni a lunghezza variabile e di vari formati che richiedono una complessa decodifica. Tale approccio è stato utilizzato per mantenere la retro compatibilità. 

# Si descriva sinteticamente l'implementazione delle istruzioni attraverso la tecnica della microprogrammazione. Si dica se questa tecnica viene utilizzata per i processori CISC o RISC, motivare la risposta. 

La microprogrammazione è utilizzata per implementare l'Unità di Controllo della CPU la quale, grazie al microprogramma, implementa ogni istruzione tramite una sequenza di micro-operazioni eseguite direttamente dall'hardware e di generare nella giusta sequenza i segnali di controllo che provocano l'esecuzione di ogni operazione elementare. 
Il microprogramma (firmware) nell'unità di controllo ha una struttura ciclica in cui alterna l'esecuzione di una operazione speciale con l'esecuzione di un'operazione esterna il cui codice e dati da elaborare sono stati acquisiti dalla operazione speciale. Il microprogramma riunisce quindi tutte le micro- operazioni necessarie per effettuare l'operazione speciale del microprogramma e le micro- operazioni necessarie per effettuare ogni operazione esterna. 
In generale la parte operativa invia all'unità di controllo delle variabili di condizionamento. In base a queste variabili, l'Unità di controllo manda dei segnali di controllo (α β) che designano la micro-operazione da eseguire. 
La microprogrammazione è la soluzione tipica di architetture CISC per implementare l'Unità di Controllo. Questo perché l'adozione di tale tecnica permette una maggiore flessibilità nella progettazione, cioè rende facile modificare le sequenze di micro-operazioni che implementano le istruzioni eseguite dalla CPU, e premette la realizzazione di un vasto numero di istruzioni da parte della CPU.

# Discutere le motivazioni alla base dei processori multicore. 
 
I microprocessori hanno visto una crescita esponenziale delle prestazioni grazie al miglioramento dell'organizzazione ed all'incremento delle frequenza di clock. Il miglioramento dell'organizzazione del cip è stato fortemente focalizzato sull'incremento delle parallelismo tra istruzioni. Si è passati infatti dall'introduzione della pipeline, a CPU superscalari, in cui vi sono pipeline parallele, sino a CPU con multithreading simultaneo (SMT), in cui alle pipeline parallele sono associati banchi di registri replicati.Tali miglioramenti hanno però richiesto un aumento di complessità, dalla quale segue quindi una logica più complessa quindi difficile da realizzare, progettare e verificare, ed un aumento dell'area del chip per permettere di supportare il parallelismo. Inoltre all'aumentare della densità del cip, dovuta all'aumento delle complessità, ed all'aumentare della frequenza di clock è seguito un aumento esponenziale della potenza richiesta, quindi maggiore energia consumata e maggior calore prodotto. Con le CPU SMT si era giunti al limite della potenza erogabile ed ai limiti del parallelismo a livello di istruzioni, quindi per permettere un aumento delle capacità delle CPU si è scelto di passare adarchitetture multicore.Si ipotizza infatti che con le architetture multicore sia possibile un incremento prestazionale quasi lineare, anche se i vantaggi prestazionali dipendono dallo sfruttamento efficace delle risorse parallele da parte dei programmi (piccole quantità di codice seriale ha un impatto significativo sulle prestazioni).

# Si descrivano le possibili alternative di organizzazione per un processore multicore. 

L'organizzazione di un processore multicore dipende da: 
- il numero di core per chip 
- la tipologia di core, cioè sei i singoli core sono superscalari o sono multhithreading simultaneo (SMT). I core più vecchi avevano organizzazione superscalare, mentre le cpu multicore più recenti hanno organizzazione SMT. 
- numero di livelli di cache per chip (che possono essere L1,L2,L3) 
- quantità di cache condivisa, la cui caratteristica divide i multicore in 4 macrogruppi:

  * Cache L1 dedicata: ogni core ha la propria cache L1 dedicata, la quale è suddivisa tra cache dati e cache istruzioni. 
  * Cache L2 dedicata: ogni core ha la propria cache L1 ed L2 dedicata. 
  * Cache L2 condivisa: ogni core ha la propria cache L1 dedicata, ma vi è una cache L2 condivisa tra tutti i core. 
  * Cache L3 condivisa: ogni core ha la propria cache L1 ed L2 dedicata, ma vi è una cache L3 condivisa tra tutti i core. 

# Si elenchino, e si discutano, i fattori che condizionano e che sono condizionati dalla lunghezza del formato delle istruzioni.

# Nel contesto di una pipeline, si motivi e si spieghi in dettaglio la tecnica del buffer circolare per la gestione delle dipendenze da controllo.

# Si motivi la presenza nei processori RISC di un ampio banco di registri a uso generale. Spiegare in dettaglio il meccanismo di funzionamento di tale banco di registri.
