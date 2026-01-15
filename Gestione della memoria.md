---
title: Gestione della memoria
---

# 1. Indice

- [1. Indice](#1-indice)
- [2. Gestione della memoria](#2-gestione-della-memoria)
	- [2.1. Rilocazione statica e dinamica - MMU](#21-rilocazione-statica-e-dinamica---mmu)
	- [2.2. Spazio Virtuale Segmentato](#22-spazio-virtuale-segmentato)
	- [2.3. Paginazione](#23-paginazione)
- [3. Tecniche di gestione della memoria](#3-tecniche-di-gestione-della-memoria)
	- [3.1. Partizioni Fisse e Variabili](#31-partizioni-fisse-e-variabili)
	- [3.2. Partizioni multiple](#32-partizioni-multiple)
	- [3.3. Segmentazione](#33-segmentazione)
	- [3.4. Segmentazione a domanda](#34-segmentazione-a-domanda)
	- [3.5. Paginazione](#35-paginazione)
	- [3.6. Paginazione a domanda](#36-paginazione-a-domanda)
	- [3.7. Memoria segmentata e paginata](#37-memoria-segmentata-e-paginata)
- [4. Gesitone della memoria](#4-gesitone-della-memoria)

# 2. Gestione della memoria

Quando parliamo di memoria parliamo di una risorsa _hardware_ indispesabile per l'esecuzione di qualsiasi processo.

Abbiamo parlato di tecniche di virtualizzazione delle risorse attraverso una **CPU-virtuale**.

Questa astrazione funziona solo se associamo ad un processore virtuale anche una **_memoria virtuale_**, anche più grande dell'effettivo spazio fisico disponibile.

Avremo quindi necessità di avere delle strutture dati specializzate (descrittori) che ci permettano di gestire opportunamente questa nuova astrazione.

Poiché ogni processo adesso ha una memoria virtuale ed un suo processore virtuale, potrebbe accadere che si hanno talmente tanti processi che non si ha sufficiente spazio fisico per contenere tutte queste aree virtuali.
Dovremo quindi pensare ad come sviluppare un meccanismo di _swap_ che ci permette di spostare porzioni di memoria virtuale dalla **RAM** alla _memoria persistente nel disco_, in particolare alla **_swap-aread_**.

La gestione della memoria avviene è supportata a livello _hardware_ dalla `MMU` (_Memory Managment Unit_).

Un secondo aspetto riguarda l'organizzazione logita della memoria virtuale. La memoria virtuale non è altro che una **struttura dati destinata a rappresentare le esigenze di memoria di un processo**.
Queste esigenze possono essere rappresentate in termini di un _unico insieme di locazioni_, con indirizzi contigui, oppure mediante più insiemi di locazioni, detti **segmenti**.
Un singolo _segmento_ è un insieme di locazioni contigue, indipendente dagli altri.

Il terzo aspetto da considerare è l'**organizzazione fisica** della porzione di memoria utilizzata per allocare un processo.
L'astrazione della memoria virtuale ci permette infatti sia di riservare la memoria virtuale di un processo in un area di memoria fisica costituita da lcoazioni contigue, ma ci da anche la facoltà di **_mappare ogni segmente in zone di memoria non necessariamente contigue_**.

Per una prima introduzione sull'argomento paginazione e memoria virtuale è possibile [consultare i seguenti appunti](https://gabriele-d-cambria.github.io/Appunti-Calcolatori-Elettronici-2024-2025/Paginazione#3-paginazione).

## 2.1. Rilocazione statica e dinamica - MMU

Ipotizzando che tutta la memoria fisica sia a disposizione di un solo processo, è necessario tener conto che l'area di memoria fisica assegnata ad un processo non potrà coincidere con quella virutale, che consideremo di `6KB` con indirizzi `[0, 6140)`.

In generale, l'area assegnata ad un processo dovrà avere una dimensione in byte sufficiente a contenere l'intera memoria virutale del processo, ma localizzata nella memoria fisica a partire da un indirizzo $I$.

Sarà quindi opportuno, come descritto anche negli appunti di approfondimento sopra,distinguere:
- **_Indirizzi Virtuali_**: utilizzati da un processo per indirizzare la sua memoria
- **_Indirizzi Fisici_**: indirizzi di memoria corrispondenti alle locazioni fisiche nelle quali il codice e i dati del processo sono caricati

I due spazi di indirizzi non sono ovviamente indipendenti, ma sono legati da una funzione, detta f**unzione di rilocazione** (una somma), che dipende dala porzione di memoria fisica utilizzata utilizzata per caricare l'immagine del processo, che fa corrispondere ad ogni indirizzo virtuale $x$ l'inndirizzo fisico $y$:
$$
	y = f(x) \approx I + x
$$

Quando compiliamo un programma, il _linker_ si occupa di tradurre gli indirizzi simbolici in indirizzi virtuali.

Nel caso di **_rilocazioni statiche_**, il **caricatore** genererà l'**_immagine del processo_** traducendo gli indirizzi virtuali sommandogli il contenuto del _program counter_

<img class="" src="./images/Gestione Memoria/static-relocation.png">

La rilocaizone statica funziona in sistemi _embedded_ ma non funziona con i meccanismi di _swapping_. Infatti, richiederebbe di essere reinserita **_nella medesima porzione di memroia nella quale era prima_**, cosa tendenzialemnte impossibile.

Per evitare questi inconvenienti legati alla rilocazione statica è neessarie ritardare la rilocazione stessa, attuandola in **fase di esecuzione**. Questa nuova rilocazione si chiama _rilocazione dinamica_.

In questo caso, quando un processo viene allocato in memoria, il caricatore trasferisce nelle locazioni della partzione scelta, **_direttamente i contenuti del modulo di caricamento_**, ovvero vengono mantenuti **_gli indirizzi virtuali_**.

<img class="" src="./images/Gestione Memoria/dynamic-relocation.png">

Poiché gli indirizzi contenuti sono indipendenti dalle particolari locazioni fisiche contenenti le istruzioni e i dati, nei registri `PC` e `SP` vengono caricati rispettivamente l'indirizzo virtuale della prima istruzione da eseguire e l'indirizzo virtuale della base dello _stack_.

Si delega quindi alla fase di esecuzione l'azione di **_traduzione degli indirizzi da virtuali a fisici_**, attraverso l'`MMU` che interfaccia la **CPU** alla memoria.

In questa nuova architettura, la **CPU** **_utilizza solo ed esclusivamente indirizzi virtuali_**.

<img class="" src="./images/Gestione Memoria/MMU-simple-scheme.png">

<div class="grid2">
<div class="">

La `MMU`, oltre a tradurre gli indirizzi da virtuali a fisici, si occupa anche di effettuare dei **_controlli di protezione_**.

Prima di tradurre un indirizzo virtuale, si effettua infatti in controllo che $x \in [0, 6140)$.

Se questo non fosse vero allora la `MMU` **_genera un eccezione di protezione_**, tipicamente il _segmentation fault_, che il sistema operativo dovrà gestire opportunamente.

</div>
<div class="">
<img class="" src="./images/Gestione Memoria/MMU-simple-example.png">
</div>
</div>


## 2.2. Spazio Virtuale Segmentato

Durante la virtualizzazione della memoria possiamo **_segmentare lo spazio virtuale_**. Questa azione ha diversi vantaggi:
- Facilita il caricamento delle partizioni più piccole
- Permette di condividere dei segmenti per più processi diversi, senza dover conservare più copie delle porzioni
- Migliora le politiche di protezione sui singoli segmenti

La segmentazione della memoria provoca anche un aumento della complessità della traduzione degli indirizzi.

Se infatti ci basiamo sull'esempio fatto prima, nella `MMU` dovremo avere $2\cdot n\cdot m$ registri, dove $n$ indica il nuemro di segmenti di un singolo processo, e $m$ è il numero di processi.

L'utilizzo di memoria segmentata potrebbe generare una memoria fisica "a buchi", ovvero dove tra zone di memoria fisica di un processo e un altro si trovano delle aree vuote.

Una tecnica per rimuovere il problema della **frammentazione esterna** è quella di **_compattamento_**.

<div class="grid2">
<div class="">

Questa tecnica permette di rilocare i segmenti frammentati effettuando degli _swap-out_ dei processi frammentati e successivamente dei _swap-in_ per ricaricarli in modo di non avere più i "buchi" tra processi.

</div>
<div class="">
<img class="75" src="./images/Gestione Memoria/compacting.png">

</div>
</div>

## 2.3. Paginazione

È una tecnica che permette di prendere uno spazio virtuale e suddividerlo in _pagine logiche_, caricando nella memoria fisica, divisa anch'essa in _frame_, le singole pagine in maniera non contigua.

Quetso permette di **_rimuovere il problema delal frammentazione esterna_**.

Tuttavia questa nuova tecnica ci costringe a **_cambiare la funzione di rilocazione_**, rendendola una **_tabella di corrispondenza_** tra _pagina logica_ e _frame fisico_, che dovrà essere inizializzata e mantenuta consistente dalla `MMU`, generando un _overhead_.


# 3. Tecniche di gestione della memoria

Nel seguente schema possiamo vedere le **tecniche di gestione della memoria**:

<img class="75" src="./images/Gestione Memoria/memory-handling-tecniques.png">

Vediamo adesso nel dettaglio le "foglie dell'albero"

## 3.1. Partizioni Fisse e Variabili

È una tecnica di _rilocazione statica_, con _allocazione contigua_ in uno _spazio virtuale unico_ a _caricamento unico_.

Una partizione è definita come un'insieme di byte in memoria.

<div class="grid2">
<div class="">

Il sistema a partizione fisse consiste nel suddividere la memoria in $(n+1)$ partizioni di **indirizzo iniziale e dimensioni fisse**, definite in fase di installazione dell'_OS_.à

La prima delle partizioni è destinata a contenere la **parte residente del sistema stesso**, mentre le successive $n$ sono riservate per ospitarvi le _immagini di altrettanti processi_ una volta rilocati i rispettivi spazi virtuali al loro interno.

Sulla destra possiamo vedere un esempio di memoria divisa in $6$ partizioni, dove la prima, con indirizzo virtuale $0$ e dimensioni $D_0$, è quella riservata al sistema operativo.

Le altre $5$ sono utilizzate per allocare i processi, e hanno dimensioni $D_1,\;D_2,\;D_3,\;D_4$ e $D_5$.

Quando viene creato un nuovo processo questo viene normalmente inserito **_nella più piccola partizione che lo può contenere tra quelle disponibili_**. È infatti estremamente raro che la dimensione di un processo coincida con quella di una partizione. Ne consegue che una considerevole porzione di memoria non viene utilizzata, fenomeno chiamato _**frammentazione interna**_.

Chiamando $N_i$ la dimensione necessaria dello spazio virtuale di un processo $P_i$, nella figura sulla destra la _frammentazione interna_ corrisponde a:
$$
(D_1 - N_1) + (D_2 - N_2) + (D_3 - N_3) + (D_4 - N_4) + (D_5 - N_5)
$$

</div>
<div class="">
<img class="" src="./images/Gestione Memoria/fixed-partition.png">
</div>
</div>

Pur adottando il cirterio della rilocazione statica degli indirizzi, questa tecnica si presta bene ad _allocare dinamicamente la memoria ai processi_.

Per ogni partizione, il sistema operativo mantiene **una lista ordinata di descrittori**, corrispondenti ai processi destinati a quella partizione.
Il primo elemento della lista è **fisicamente allocato in memoria** mentre gli altri sono sono residenti nella _swap-area_.
L'_OS_ si occuperà, secondo una politica di _round-robin_, di revocare la memoria al primo processo di ogni partizione, inserendo il descrittore in fondo ala rispettiva coda, e allocare il processo successivo, come nell'immagine sotto:

<img class="100" src="./images/Gestione Memoria/fixed-partition-lists.png">


Per semplificare sia la fase di richiesta di una partizione libera che quella di rilascio si possono utilizzare due schemi.

Il primo è lo schema _best-fit_:
> Vengono ordinate le partizioni nella lista per **dimensione crescente**.
> In fase di richiesta di una partizione di almeno $N$ byte la lista viene scandita e la prima partizione che viene trovata in grado di soddisfare la richiesta è **_sicuramente la più piccola tra tutte quelle di dimensioni superiori a $N$_**

Questo schema soffre di due inconvenienti:
1. Una volta effettuata l'allocazione, la parte della partizione non utilizzata è sicuramente quella di dimensioni più piccole, comportando _frammentazione_ della memoria
2. In fase di rilascio, vedremo che con le partizioni variabili è necessario verificare se la partizione resa disponibile è adiacente ad uno o due partizioni libere per eventualmente compattarle insieme.

Un secondo schema spesso più usato è noto come _first-fit_
> Si mantiene una lista ordinata **per indirizzi crescenti** delle partizioni

Questo schema risulta particolarmente efficiente in fase di rilascio, poiché le partizioni adiacenti nella lista sono anche quelle adiacenti inferiormente e/o superiormente, rendendo semplice il controllo sulle partizioni adiacenti.

Poiché lo schema di allocazione in partizioni fisse è molto inefficiente per quanto riguarda l'uso della memoria, è stato proposto un **diverso shcema** per suddividere la memoria in partizioni, detto **_partizioni variabili_**.

<div class="grid2">
<div class="">

Questo schema consiste nel _definire dinamicamente_ le caratteristiche delle singole partizioni, in modo che queste **corrispondano esattamente alle esigenze di memoria**.

In questo modo **_eliminiao la frammentazione interna_**, introducendo però la **_frammentazione esterna_**.

Nell'istante `e` dell'immagine sulla destra il processo $P_1$ viene rimosso dalla memoria generando una nuova partizione $D_5$.
Nell'istante `f` viene allocato $P_4$ con dimensioni più piccole di $P_1$, andando a generare una nuova partizione $D_6 = D_5 - N_4$.
Nell'istante `g` si libera $P_2$ generando $D_7 = D_6 + N_2$.

Se immaginiamo che in un istante `h` arrivi un nuovo processo $P_5$ con dimensione $N_5 \in \bigl[\max\{D_4, D_7\},\; D_4 + D_7\bigr)$, accade che **abbiamo complessivamente sufficiente memoria**, ma questa **_non è contigua_**, impossibilitandoci ad inserirla nel momento nel quae si genera.

Dovremo attendere che un altro dei processi che si trova attualemnte in memoria termini o venga sostituito, dato che essendo **a rilocazione statica** non possiamo effetttuare rilocazione.

</div>
<div class="">
<img class="80" src="./images/Gestione Memoria/variable-partition.png">
</div>
</div>

Per implementare questa soluzione si riserva nella memoria del sistema una locazione, indichiamola come `memoria_libera`, che contiene l'indirizzo di una partizione libera.

Questa struttura dati è una _lista_ che contiene la _dimensione in byte_ dell'attuale locazione e l'indirizzo di un'altra partizione libera.

<img class="" src="./images/Gestione Memoria/variable-partition-lists.png">

## 3.2. Partizioni multiple

È una tecnica di _rilocazione statica_, con _allocazione contigua_ in uno _spazio virtuale segmentato_ a _caricamento unico_.

Permette a due o più processi di **condividere il codice**, così da eseguire quindi lo stesso programa, anche se su dati diversi.

Lo schema delle _partizioni multiple_ coincide con il precedente per quanto riguarda l'allocazione di una generica partizione, ma la creazione di un processo implica adesso non solo la ricerca di **una grande partizione**, ma di tre più piccole, anche se non contigue.

La maggiore complessità richiesta al _linker_ è ampliamente controbilanciata dal fatto che il segmento di codice può facilmente allocato in locazioni contigue al segmento dati.

## 3.3. Segmentazione

È una tecnica di _rilocazione dinamica_, con _allocazione contigua_ in uno _spazio virtuale segmentato_ a _caricamento unico_.

Questa tecnica prende lo spazio virtuale e lo divide in un **numero variabile di segmenti**.

Un indirizzo virtuale avrà adesso due dimensioni `x = <segment, offset>`, il primo identifica il segmento, mentre il secondo identifica l'_offset_ all'interno del segmento.

Per tradurre un indirizzo virtuale in un indirizzo fisico dobbiamo quindi ridefinire la _funzione di rilocazione_.

Nei primi esempi di _segmentazione_, nella `MMU` erano contenuti **tre segmenti**:
- `Fetch` $\to$ `sg = 0`: si riferisce alla fase di _fetch_. In questo caso l'accesso avverrà sicuramente nel segmento che comprende il codice del programma
- `Excecute` $\to$ `sg = 1`: si riferisce alla fase di _esecuzione_. In questo caso l'accesso avverrà sicuramente nel segmento che comprende i dati del programma
- `POP/PUSH` $\to$ `sg = 2`: si riferisce alle operazioni che manipolano lo _stack_ del programma.

<img class="" src="./images/Gestione Memoria/segments-og-translation-scheme.png">



Questa tecnica è stata poi migliorata, **togliendo un limite al numero di segmenti**.
È possibile ottenere questo risultato attraverso una **tabella dei segmenti**, allocata nella memoria fisica del sistema, che sarà consultata dalla `MMU`.

Il descrittore di un processo dovrà quindi contenere anche un campo relativo alla sua memoria virtuale, mantenendo aggiornate due informazioni:
- Numero di segmenti che compongono la memoria virtuale
- L'indirizzo in memoria della tabella dei segmenti

Questi valori, nel momento in cui il processo viene _schedulato_ e passa in esecuzione vengono caricati in due nuovi registri:
- `STLR` (_Segment Table Limit Register_): contiene il numero di segmenti del processi
- `STBR` (_Segment Table Base Register_): contiene l'indirizzo fisico della tabella dei segmenti

<div class="grid2">
<div class="">

Nell'immagine sulla desta possiamo vedere come viene tradotto un indirizzo virtuale `x` generato dalla **CPU**.

Per prima cosa `x` viene diviso in `<sg, of>`.

Per prima cosa si confronta `sg` con il contenuto di `STLR`. Se `sg >= STLR` allora viene generata un'eccezione `trap` per indicare il tentativo di accedere ad un segmento inesistente.

In caso contrario `sg` viene utilizzato come indice per accedere alla tabella dei segmenti, per trovare l'elemento contenente i valori relarivi all'indirizzo iniziale `I` e la dimensione `D` della partizione di memoria nella quale il segmento è stato caricati.

A questo punto si confronta `of`  con la dimensione `D` del segmento. Se `of >= D` allora viene nuovamente generata una `trao` per indicare il tentativo di accedere ad informazioni fuori dal segmento.
Altrimenti, ricaviamo l'indirizzo fisico come `y = I + of`.

</div>
<div class="">
<img class="80" src="./images/Gestione Memoria/segments-translation-scheme.png">
</div>
</div>

Il ricorso alla tabella dei segmenti, allocata in _memoria principale_, genera una notevole **perdita di efficienza della CPU**.
Infatti per ogni indirizzo generato è necessario **_raddoppiare gli accessi alla memoria_**: il primo per accedere alla tabella, il seocndo per recuperare le informazioni.

AL fine di ridurre questa efficienza, nella `MMU` sono mantenuti alcuni registri associativi (tipicamente da 4 a 8), che fanno da _cache_ per gli utlimi segmenti ai quali è stato fatto riferimento.
Questa memoria associativa è chiamata `TLB` (_Translation Look-aside Buffer_).

Qual'ora il valore del segmento `sg` fosse già presente in uno dei registri, non avremmo più bisogno di accedere alla memoria, effettuando un solo accesso alla memoria.

Questa maggiore complicazione del meccanismo di traduzione, trova la sua giustificazione nei vantaggi che che la segmentazione offre.

PEr quanto riguarda la **protezione**, la segmentazione consente di effettuare **_tre diversi controlli nel momento in cui viene fatto 'accesso ad una locazione di memoria_**.

I primi due sono quelli già visti durante la traduzione, il terzo viene invece effettuato associando ad ogni segmento degli **_specifici diritti di accesso_** in modo che ogni riferimento a quel segmento sia consistente a tali diritti.

Questo si ottiene aggiungendo alle singole entrate della tabella, ovvero ai _descrittori di segmento_, dei campi di **controllo**.

Per esempio, due campi di controllo sono il campo `R` (_read_) che consente l'accesso il lettura del segmento, e il campo `W` (_write_), che consente l'accesso il scrittura sul segmento.
Qual'ora un istruzione provasse a scrivere su un segmento che non ha il campo `W` settato, si genererà un **eccezione di protezione**.

Con la tecnica della segmentazione un processo può essere in due diverse condizioni:
- **Allocato in memoria**: tutti i segmenti che compongono il suo spazio virtuale sono residenti in altrettante partizioni
- **Non allocato in memoria**: nessuno dei suoi segmenti è contenuto nella _swap-area_

Per questo motivo è necessario aumentare il numero degli stati in cui un processo più trovarsi:

<figure class="">
<img class="75" src="./images/Gestione Memoria/swapped-states-scheme.png">
<figcaption>

Le transizioni tra gli stati `pronto` o `bloccato` e i corrispondenti stati `swapped` corrispondo all'esecuzione delle primitive `swap_in` e `swap_out`.
</figcaption>
</figure>

La scelta del processo pronto a cui assegnare la CPU e quella del processo pronto da caricare in memoria **_corrispondono ai due livelli di schedulazione_** che avevamo indicato come _short-termi scheduling_ e _medium-term scheduling_.

## 3.4. Segmentazione a domanda

È una tecnica di _rilocazione dinamica_, con _allocazione contigua_ in uno _spazio virtuale segmentato_ a _caricamento a domanda_.

Con questa tecnica è necessario complicare la funzione di rilocazione in modo tale che, durante l'esecuzione di un processo sia possibile mantenere in memoria **_soltanto una parte del suo spazio virtuale_**, ovvero solo quelli necessari a quel punto della sua esecuzione.

Per questo motivo è necesasrio abilitare la funzione di rilocazione a restituire:
- Il corrispondente indirizzo fisico `y` se il segmento `sg` è presente in memoria
- Generare un interruzione (`segment-fault`) nel caso in cui il segmento non fosse presente

La funzione di gestione di questa interruzione avrà quindi il compito di **cercare in memoria fisica una partizione libera nella quale caricare il segmento richiesto e riattivare l'esecuzione del processo stesso** per poter rieseguire l'istruzione che ha generato l'eccezione.

Se nell'atto di caricamento di un segmento **_non ci fosse spazio in memoria fisica_** è possibile scaricare sulla _swap-area_ uno o più segmenti, sia dello stesso processo sia di altri processi, attraverso lo schema del **rimpiazzamento**.

Al fine di facilitare l'implementazione di algoritmi di rimpiazzamento efficienti, sono spesso disponibili ulteriori _bit di controllo_ nel descrittore di un segmento. Ad esempio abbiamo i bit `U` (_uso_) e `M` (_modifica_), che possono essere letti e azzerati via software, e che vengono settati automaticamente via _hardware_ ogni volta che viene fatto un riferimento/modifica al segmento.

Il bit `M` è utilizzato per capire se un segmento è stato o meno modificato, così da poter evitare eventuali riscritture sulla _swap-area_.
Il bit `U` invece è un bit utilizzato per valutare la freqeunza di utilizzo di ogni segmento, così da rendere più efficienti gli algoritmi di scelta dei segmenti da rimpiazzare.

## 3.5. Paginazione

È una tecnica di _rilocazione dinamica_, con _allocazione non contigua_ in uno _spazio virtuale unico_ a _caricamento unico_.

Per eliminare alla radice il problema della framentazione, che richiede un tempo di **CPU** non inifferente, sarebbe necessario poter allocare in memoria informazioni i cui indirizzi virtuali sono contigui.

Quello che facciamo è **_dividere lo spazio virtuale in blocchi di indirizzi virtuali_** di dimensioni fisse, detti **_pagine_**.

ANche lo _spazio fisico_ viene **_diviso in blocchi di indirizzi fisici delle stesse dimnesioni delle pagine_**, detti **_frame_**.

Ogni _pagina_ verrà allocata in **un frame libero**.

Questo può comportare che **_pagine consecutive nello spazio virtuale non saranno necessariamente allocati in frame consecutivi_**.

Per tradurre un indirizzo virtuale nel corrispondente indirizzo fisico è quindi necessario registrare in una tabella, detta _tabella delle pagine_, la corrispondenza **pagina-frame**.

<div class="grid2">
<div class="">

Nell'esempio a destra possiamo vedere una suddivisione in pagine di `1K` su una memoria virtuale di `4K`, ovvero di quattro pagine.

Per tradurre un indirizzo viruale `x` in un indirizzo fisico `y` è **necessario sapere a in quale frame è stata salvata**.

Il quoziente della divisione fornisce l'indice dalla pagina virtuale a cui `x` appartiene, mentre il resto fornisce l'_offset_ nell'ambito della pagina.

Tendenzialmente la dimensione della pagina `d` è una potenza del due, in questo modo se $d = 2^x$ gli `x` bit meno significativi saranno l'`offset`, mentre i bit più significativi rappresenteranno il numero di _pagina_ `pf` individuata.

<img class="" src="./images/Gestione Memoria/pagination-offest-example.png">

</div>
<div class="">
<img class="80" src="./images/Gestione Memoria/pagination-example.png">
</div>
<div class="">

Ogni processo allocato in memoria possiede una propria tabella delle pagine.

La dimensione di questa tabella dipende dal numero di pagine che compongono lo spazio virtuale del processo, che dipende a sua volta dalla dimensione dell'intero spazio e dalla dimensione delle pagine. In generale le dimensioni delle bagine oscillano tra `512 Byte` e `4 KByte`, dimensioni tali per i quali abbiamo ancora il vantaggio di dividere in pagine, e non introduciamo il problema della frammentazione interna all'interno delle pagine.

Supponendo che una pagina contenga `1024` locazioni, e lo spazio virtuale di un programma di medie dimensioni sia composto da `256` pagine, la tabella delle pagine dovrà poter contenere almeno 256 elementi.

Queste tabelle non sono allocate nei rigistri della `MMU`, ma nella memora fisica. È tuttavia necessario che la `MMU` abbia un particolare registro di macchina che contiene l'indirizzo della tabella delle pagine del procesos in esecizione.
Possiamo chiamarlo, come nell'immagine, `RPTP` (_Registro Puntatore alla Tabella delle Pagine_).

Anche in questo caso si implementa il `TLB` per risparmiare tempo.
</div>
<div class="">
<img class="" src="./images/Gestione Memoria/pagination-translation-example.png">
</div>
</div>

Nella paginazione possono essere effettuati gli stessi controlli visti nel caso della segmentazione, anche se alcuni in questo caso sono molto meno significativi.

QUando un processo prova ad accedere ad un indirizzo `x` (che appartiene ad una pagina `pg`) il primo controllo è quello di verificare che la pagina `pg` sia una delle pagine del processo, verificando che il suo valore sia minore o uguale al contenuto di un particolare registro di macchina `PTLR` (_Page Table Lenght Register_).

Se `pg` supera il valore allora si genera un eccezione di protezione.

Anche in questo caso possiamo assegnare ad ogni pagina dei campi di controllo in lettura e scrittura.

## 3.6. Paginazione a domanda

È una tecnica di _rilocazione dinamica_, con _allocazione non contigua_ in uno _spazio virtuale unico_ a _caricamento a domanda_.

Permette di implementare la paginazione sfruttando il fatto che un processo **non necessita sempre di tutte le sue pagine**.

La tecnica, analoga a quella della _segmentazione a domanda_, è quella **_più utilizzata anche nei sistemi paginati_**.

Il meccanismo hardware è molto simile a quello della _segmentazione a domanda_, e aggiunge ai campi di lettura `R` e scrittura `W`:
- `P`: _bit di presenza_, caratterizza la presente in memoria o meno della pagina
- `U`: _bit di uso_, utilizzato dagli algoritmi di rimppiazzamento
- `W`: _bit di scrittura_, utilizzato dagli algoritmi di rimppiazzamento

Nel caso di sistemi che utilizzano la _paginazione a domanda_ anche la _swap-area_ è divisa in **settori fisici di dimensioni uguali a quelle delle pagine**.

Quando un processo viene creato tutti i descrittori associati hanno `P == 0`, e il campo dove si trova l'indice della pagina fisica contiene in realtà **l'indirizzo della pagina sul disco**.

Quando il processo farà l'accesso ad un indirizzo `x`, che appartiene alla pagina `pg`, si verificherà innanzitutto che `pg` sia tra le pagine del processo, e successivamente, il valore di `P` del descrittore associato.

<div class="grid2">
<div class="">

Se `P == 0` si genera `page fault`, fermando l'esecuzione del processo.

Il sistema operativo eseguirà quindi una _routine_ per recuperare la pagina.
<small>([figura 1](#fig1))</small>

Il primo passo è quello di verificare che **ci siano frame liberi nella memoria dove salvare la pagina**, attraverso la variabile `pf`
Nel caso non vi fossero _frame_ liberi, si utilizza un algoritmo di rimpiazzamneto delle pagine. Questo algoritmo è molto più semplice di quello della _segmentazione a domanda_, in quanto **_tutte le pagine hanno le stesse dimensioni_**.
<small>([figura 2](#fig2))</small>


Se invece ci fosse sufficiente spazio, il sistema operativo consulta la _swap-area_ nell'indirizzo contenuto nel _campo indirizzo_ del descrittore, copiandone il contenuto in memoria.

Una volta caricata la pagina virtuale richiesta, si aggiorna di conseguenza la tabella delle pagine di processo, azzerando settando `P = 1` e aggiornando il contenuto del _campo indirizzo_ della pagina con l'indirizzo del _frame_ . Inoltre si aggiorna cierentemente `pf` diminuendo il suo valore di uno.

L'istruzione che ha generato il `page-fault` può essere rieseguita e il processo continuare la sua esecuzione in maniera completamente trasparente.

</div>
<div class="">
<figure id="fig1" class="100">
<img class="75" src="./images/Gestione Memoria/demand-paging-page-fault-handling.png">
<figcaption>
Figura 1
</figcaption>
</figure>
<figure id="fig2" class="100">
<img class="75" src="./images/Gestione Memoria/demand-paging-page-swapping.png">
<figcaption>
FIgura 2
</figcaption>
</figure>
</div>
</div>

L'algoritmo di rimpiazzamento consulta diverse informazioni per poter decidere su quale pagina effettuare la _swap-out_.
Definiamo innanzitutto come **_working-set_** l'insieme delle località dei riferimenti utilizzati. Inizialmente questo set sarà vuoto, e successivamente, nel corso dell'esecuzione, auemnta lentamente (per via dei principi di località), andando prima o poi a saturarsi.

Esistono diversi tipi di algoritmo di rimpiazzamento:
- **Ottimo**: rimpiazza le pagine che non verranno più riferite o che verranno utilizzate più nel futuro rispetto alle altre. (Teorico, irrealizzabile)
- **FIFO**: rimpiazza la pagina **_locale_** (aka del processo in esecuzione) che è _da più tempo in memoria_. Il rimpiazzamento è locale per evitare di propagare l'errore anche ad altri processi.
- **LRU**: rimpiazza la pagina non utilizzata da più tempo. È più efficiente rispetto al **FIFO**, ma anche più difficile da realizzare. Infatti è necessario modificare opportunamente la tabella delle pagine per introdurre un campo dove salvare un _timestamp_ , ed è necessario consultarli tutti per recuperare il più vecchio e aggiornare i nuovi. Tra gli arlgoritmi implementabili è quello che riduce al minimo il **_trashing_**, ovvero diminuisce al minimo il `page-fault`.

L'algoritmo utilizzato da `UNIX` è un'approssimazoine dell'algoritmo `LRU`, noto come **_second-chance_** o **_clock algorithm_**-

<div class="grid2">
<div class="">

La tabella delle pagine fisiche viene gestita come un array circolare.
Viene mantenuta una variabile `vittima`. che consiste in un puntatore contenente l'indice della pagina fisica successiva a quella che è stata rimpiazzata per utlima.

Al prossimo `page-fault` si inizia a verificare la pagina il cui indice è contenuto nella variabile `vittima`.

In particolare si esamina il bit `U` della pagina associata:
- `U == 0`: la pagina viene scelta per il rimpiazzamento
- `U == 1`: si azzera il bit e si **incrementa la variabile** `vittima`, andando ad esaminare la pagina successiva, riverificando il bit `U`.

Poiché l'incremento della variabile è modulato al numero delle pagine fisiche `M` (`vittima = (vittima + 1) % M`) abbiamo la garanzia che **_troveremo certamente un pagina con_** `U == 0`.
Infatti, il _Worst-Case_ è quello in cui andiama a selezionare la pagina con la quale avevamo iniziato la scansione, dopo aver iterato sull'intera tabella

Per ridurre il nuemro di trasferimenti tra memoria e disco, viene spesso preso in considerazione anche il valore di `M`.
Le pagine sono quindi classificate prima rispetto al valore di `U`, e successivamente si da privilegiano le pagine con `M == 0`

</div>
<div class="">
<img class="80" src="./images/Gestione Memoria/second-chance-algorithm-example.png">
</div>
</div>

## 3.7. Memoria segmentata e paginata

È una tecnica di _rilocazione dinamica_, con _allocazione non contigua_ in uno _spazio virtuale segmentato_ a _caricamento a domanda_.

Questa tecnica, utilizzata nei sistemi `UNIX`, offre i vantaggi **sia della segmentazione che dalla paginazione**.
In particolare, nei sistemi `UNIX` è possibile abilitare/disabilitare l'opzione della segmentazione.

<div class="grid2">
<div class="">

La **CPU** genera un indirizzo virtuale `x` composto dalle due componenti `<sg, sc>` (<_segmento_, _scostamento_>).

Dopo le solite verifiche di protezione, `sg` viene utilizzato per accedere al descrittore del segmento nella tabella dei segmenti.
Il descrittor econtiene ancora due campi: `limite` e `base`, dove adesso però `base` rappresenta **_l'indirizzo di memoria di una partizione utilizzata come tabella delle pagine del segmento_**.

Lo _scostamento_ è a sua volta composto da `sc = <pg, of>`. L'indice `pg` viene utilizzato come indice nella tabella delle pagine (sempre previ controlli di protezione), per ottenere l'indirizzo del _frame_ `pf` e andare a costruire l'indirizzo fisico `y = <pf, of>`

</div>
<div class="">
<img class="75" src="./images/Gestione Memoria/paged-segmentation-scheme.png">
</div>
</div>

La tabella delle pagine del segmento **_non è necessariamente in memoria_**, quindi l'interruzione dei `segment-fault` viene gestita proprio per caricare tale tabella.
Così come potrebbe verificarsi `P == 0` nella tabella delle pagine, generando un normale `page-fault`.

# 4. Gesitone della memoria

Per informazioni aggiuntive riguardanti come le tabelle di indirizzi sono salvate in memoria, è sufficiente consultare il seguente [capitolo degli appunti di Calcolatori Elettronici](https://gabriele-d-cambria.github.io/Appunti-Calcolatori-Elettronici-2024-2025/Paginazione#32-trie-mmu).

`UNIX` disaccoppia la gestione del `page fault` con la sostituzione delle pagine, facendo in modo che **_ci sia sempre almeno un numero minumo di frame liberi_**, detto `lotsfree`.
Se vi sono almeno `lotsfree` _frame_ liberi, allora **si evita la sostituzione**.
Èer farlo il _kernel_ mantiene una descrizione dell ostato di allocazione della memoria all'interno della tabella delle pagine fisiche (_core map_) nella quale ogni elemento rappresenta la pagina fisica e contiene le informazioni relative ad essa (libera/allocata, informaizoni nella pagina logica, a che processo appartiene, ...)

Tuttavia l'evoluzione del sistema non è controllabile, ed è quindi possibile che il numero di frame liberi sia minore di `lotsfree`.

Per ovviare a questo problema, i sistemi `UNIX` hanno un `pagedaemon` (`pid=2`), che esegue periodicamente (ogni `250ms`) e sostituisce le pagine **solo se**:
```cpp
	num_frame_liberi < lotsfree
```

Qualora la frequenza di paginazione sia troppo elevata, ed il numero di pagine libbere rimanfa comunque inferiore a `lotsfree`, interviene un altro _demon_, chiamato `swapper`, che provvede al trasferimento di uno o più processi della memoria centrale a quella secondaria, al verificarsi di due condizioni:
```cpp
	num_frame_liberi < minfree && num_medio_frame_liberi < desfree
```
La variabile `minfree` rappresenta **il numero minimo di frame liberi per evitare lo swapping dei processi**.
La variabile `desfree` invece è il **numero minimo di frame desiderabili**, e agisce da _filtro-passa-basso_. Limita infatti il numero di _swap_ in caso di **momenti di picco di richieste**, ma solamente quando `page-fault` avviene per _lunghi periodi di tempo_.

La relazione tra le costanti è la seguente:
<div class="p"><code>lotsfree > desfree > minfree</code></div>