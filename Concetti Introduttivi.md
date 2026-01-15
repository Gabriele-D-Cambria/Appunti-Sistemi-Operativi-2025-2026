---
title: Concetti Introduttivi
---

# 1. Indice

- [1. Indice](#1-indice)
- [2. Introduzione](#2-introduzione)
- [3. Struttura di un Sistema Operativo](#3-struttura-di-un-sistema-operativo)
- [4. Definizioni di OS](#4-definizioni-di-os)
- [5. Tipi di Sistemi Operativi](#5-tipi-di-sistemi-operativi)
	- [5.1. Sistemi Batch monoprogrammati](#51-sistemi-batch-monoprogrammati)
	- [5.2. Sistemi di Spooling](#52-sistemi-di-spooling)
	- [5.3. Sistemi Multiprogrammati](#53-sistemi-multiprogrammati)
	- [5.4. Sistemi time-sharing](#54-sistemi-time-sharing)
	- [5.5. Overhead](#55-overhead)
	- [5.6. Sistemi in tempo reale](#56-sistemi-in-tempo-reale)
- [6. Architettura di un sistema di Elaborazione](#6-architettura-di-un-sistema-di-elaborazione)
	- [6.1. **CPU**](#61-cpu)
	- [6.2. Cache](#62-cache)
	- [6.3. Memoria](#63-memoria)
	- [6.4. IO](#64-io)
	- [6.5. Interruzioni](#65-interruzioni)
	- [6.6. Protezione](#66-protezione)
	- [6.7. DMA](#67-dma)
- [7. Struttura dei Sistemi Operativi](#7-struttura-dei-sistemi-operativi)

# 2. Introduzione

Iniziamo sottolineando la differenza tra meccanismo e politica:
- Un meccanismo riassume una serie di passaggi e operazioni per compiere un determinato compito.
- Una politica invece si preoccupa di effettuare la scelta di come compiere un determinato scopo.

Una volta adoperata la scelta si utilizza un meccanismo per compiere lo scopo.

Detto ciò introduciamo cos'è un Sistema Operativo.
> È un programma che agisce da intermediario tra un utente di un computer e l'hardware dello stesso

Ha come obiettivi quelli di:
- Eseguire programmi utente e semplificare la risoluzione dei problemi degli utenti
- Rendere il sistema di elaborazione semplice da utilizzare
- Utilizzare l'hardware del computer in modo efficiente (vredremo dieversemetriche per valutarle)


I programmi utente creeranno dei **processi** a partire dai dati dell'utente.

Secondo il modello di _Von Neumann_ i programmi, insieme ai dati, devono essere caricati nella memoria principale **RAM**. Solo in questo modo la **CPU** può raccogliere le istruzioni che poi eseguirà.

Oltre alla **CPU** e alla **RAM**, componenti fondamentali per l'esecuzione di qualsiasi programma, possono anche essere necessarie delle _periferiche di I/O_ per poter far comunicare i programmi con l'utente.
Possono inoltre essere anche necessare delle _risorse logiche_, alle quali fanno capo tutte quelle variabili e/o strutture dati condivise necessarie per il funzionamento del sistema. Anche i file rientrano nelle risorse logiche.

Alla base di ogni sistema operativo saranno quindi fondamentali due funzioni:
- Meccanismi per la gestione del processore
- Meccanismi per la gestione della memoria

Per le _risorse logiche_ abbiamo altri moduli dedicati, che non costituiscono una componente fondamentale di un sistema operativo.

Il _Sistema Operativo_ è fondamentale poiché si occupa di arbitrare l'allocazione delle risorse dei vari processi e della loro _schedulazione_, oltre a definire le modalità e i requisiti di accesso tra le varie componenti _hardware_ di un dato sistema, che possono variare da calcolatore a calcolatore.

# 3. Struttura di un Sistema Operativo

Concettualmente un sistema operativo può essere rappresentato con quetso schema a strati:

<img class="50" src="./images/Concetti Introduttivi/Struttura di un sistema operativo.png">

Il livello _hardware_ è quello che corrisponde ai livelli fisici del sistema che comprende oltre a **CPU** e **RAM** anche le unità periferiche e la memoria persistente.
Quest'ulima, per quanto non strettamente necessaria (si vedano i primi calcolatori con memoria persistente esterna tramite _floppy-disk_), è trattataoggi come tale.

Il livello del **sistema operativo** comprende un insieme dicomponenti software che hanno il compito di gestire le risorse fisiche della macchina offrendo ai programmi applicativi un0interfaccia _standard_ più semplice da utilizzare. QUesta è composta da una serie di funzioni che possono essere invocate dai programmi applciativi per intervenire sulle componenti hardware del sistema in maniera controllata ed efficiente.
Sempre a questo livello si trovano i _driver_ per le periferiche.
Grazie a questo livello possiamo utilizzare le funzioni del livello _hardware_ **senza conoscere la loro implementazione**. Viceversa possiamo cambiare l'implementazione di queste funzioni senza necessità di dover modificare l'interfaccia, che continuerà a funzionare.

Il livello dei _programmi applicativi_ corrisponde invece all'insieme delle applicazioni utilizzate direttamente dagli utenti del sistema. I programmi utente infatti **noninteragiscono mai direttamente con l'_hardware_**, ma sempre tramite l'interfaccia dell'_OS_.

Grazie all'_OS_ permettiamo lo sviluppo e la portabilità dei programmi applicativi. In questo modo chi programma non ha la necessità di conoscere nel dettagliotutte le componenti di un determinato sistema, ma gli è sufficente conoscere l'interfaccia di _OS_.

L'_OS_ realizza politiche di gestione delle risorse del sistema di elaborazione. Ad esempio gli _scheduling_ permettono di gestire le assegnazioni dei processi alla **CPU**, ma si occupa anche di come/dove caricare le pagine di memoria e di molti altre politiche.

Inoltre si occupa di fornire meccanismi di protezione, garantire la scurezza del sistema e la tolleranza ai guasti.
Non vedremo gli ultimi due scopi, ma analizzeremo invece l'implementazione dei **meccanismi di protezione**.

In fondo un _OS_ funziona in maniera simile ad una **API** (_Application Programming Interface_), ovvero è come se generasse una macchina astratta più semplice, efficiente e sicura.

# 4. Definizioni di OS

Se volessimo vedere l'_OS_ da altri punt idi vista possiamo dire che si tratta di un **allocatore di risorse**. Deve quindi avere il diritto di poter accedere a <u>tutte</u> le risorse, ed essere in grado di decidere tra richieste conflittutali per garantire un uso efficiente ed equo delle risorse.

È vero anche che l'_OS_ è un **programma di controllo**, nfatti controlla l'esecuzione dei programmi per prevenire errori e un uso improprio del Sistema di Elaborazione.


È importante sottolineare che:
> L'unico programma in esecuzione in ogni momento sul computer è il **_kernel_**.

Tutto il resto non sono altro che programmi applicativi o programmi di sistema, forniti proprio dal sistema operativo.


# 5. Tipi di Sistemi Operativi

## 5.1. Sistemi Batch monoprogrammati

Tipici degli anni '60, si chiamano così perché i vari programmi venivano consegnati agli operatori su schede forate, chiamate _batch_, e venivano inserite una ad una all'interno del calcolatore.

Il calcolatore in ogni momento era presente un solo programma, che veniva eseguito dall'inizio alla fine senza interruzioni.

Anche questo calcolatore, estremamente semplice, necessitava di alcune _routine_ per poter funzionare correttamente. Ad esempio:
- _routine_ di lettura schede
- _routine_ di scrittura

Quando vennero introdotte le memorie di massa diventò necessario anche l'implementazione di _routine_ di lettura/scrittura dei _job_ (programmi) da essa.


Nasce quindi l'esigenza di creare un `JCL` (_Job Control Language_). Erano delle `$direttive` che venivano interpretate da un monitor. Queste `$direttive` corrispondevano a delle _routine_ che potevano essere chiamate all'occorenza.
Possiamo identificare in loro gli antenati della moderna _shell_.

In questi sistemi erano quindi presenti solamente il _monitor_ e il _BIOS_.

## 5.2. Sistemi di Spooling

Questi sistemi permetteva la lettura dei programmi da dischi (_floppy-disk_) in modo tale che durante le loro esecizoioni la **CPU** leggesse dati e producesse risultati operando solo con il disco.
Le scritture verso la stampande avvenivano operando in `DMA`.


_Spooling_ è un acronimo di _Simultaneous Peripheral Operation on-line_, che sta proprio a indicare che tutto avviene su flussi di comunicazione indimendenit e concorrenti.

<img class="40" src="./images/Concetti Introduttivi/scheme spooling system.png">


## 5.3. Sistemi Multiprogrammati

<div class="grid2">
<div class="">

Venne osservato che l'efficienza dei _mainframe_, che avevano costi nell'ordine dei milioni di euro, era tendenzialmente bassa.
Questo accadeva perché le risorse, a causa di come venivano gestiti i programmi, venivano utilizzate in media meno della metà del loro potenziale.
Inoltre vi era un altro problema, legato al numero costantemente crescente di programmatori che nel tempo si erano formati, che rendeva quindi il modello di _first-come-first-serve_ ingestibile.

Per riuscire a ottenere un drastico miglioramento nell'efficienza di uso delle risorse della macchina fu realizzata la tecnica della _multiprogrammazione_.
Questa tecnica permetteva a più programmi di venire caricati in memoria in parallelo, gestendoli in modo concorrente.

Tuttavia, questa nuova gestione dei programmi creò la necessità dell'implementazione di algoritmi di _scheduling_, che permettevano alla **CPU** di eseguirli uno alla volta di sostituirli quando venivano messi in attesa, così da ridurre il più possibile i "tempi morti".

I primi algoritmi di _scheduling_ deviarono dal principio di _first-come-first-serve_, e introdussero il concetto di _interruzione_, ovvero non si lasciava più l'accesso alla **CPU** ad un programma per tutto il suo _time-to-live_, ma si riservava la possibilità sostituirlo anche durant ela sua eseguzione qual'ora fosse andato in attesa, così da permettere ad un altro programma di sfruttare quei cicli che sarebbero altrimenti stati sprecati.

Si notò subito che la memoria diventava il _bottleneck_ di questo nuovo sistema, e si iniziò a introdurre una gestione **dinamica della memoria**.

Venne quindi introdotta la _preemption_ anche nella gestione spaziale della memoria, attraverso _swap_ da e verso il disco.
Si rese quindi necessaria la _virtualizzazione della memoria_, così da poter riuscire a gestire al meglio i vari accessi in meoria dei programmi che potevano essere swappati e reinseriti in punti diversi di **RAM**, generando quindi errori e corruzione dei dati.

</div>
<div class="">
<img class="75" src="./images/Concetti Introduttivi/configurazione memoria.png">
</div>
<div class="">

Nella multiprogrammazione andiamo quindi ad indentificare due momenti durante l'esecuzione di un processo:
- **_CPU-burst_**: intervalli di tempo nel quale un processo deve eseguire istruzioni e necessita di occupare la **CPU**
- **_IO-burst_**: intervalli di tempo nel quale un processo deve eseguire un'operazione di **IO** e non deve utilizzare la **CPU**

Come possiamo vedere dall'esempio a destra, nell'esecuzione _sequenziale_ i processi vengono eseguiti in ordine di arrivo, e vengono eseguiti dall'inizio alla fine.
In questo modo notiamo che sono presenti diverse unità di tempo dove la **CPU** è in attesa di qualcosa. Possiamo calcolare l'efficienza di questo esempio, che è di  $\frac{10}{27} \approx 37\%$

Nell'esecuzione _multi-tasking_ invece, quando il primo processo va in _IO-burst_, e si mette in attesa, la **CPU** inizia a lavorare prima sul secondo processo, e poi sul terzo quando anche il secondo va in attesa.
Durante i momenti nei quali i vari programmi sono sospesi in attesa di ricevere dati, questi vengono poi recuperati nell'ordine in cui questi dati sono arrivati, e verranno messi in esecuzione quando colui che occupa l'esecuzione terminerà e/o andrà in attesa.
Possiamo quindi calcolare anche in questo caso l'efficienza, che stavolta è di $\frac{10}{12} \approx 83\%$
</div>
<div class="">
<img class="75" src="./images/Concetti Introduttivi/confronto esecuzioni.png">


</div>
</div>


## 5.4. Sistemi time-sharing

Sono sistemi che hanno come primo obiettivo quello di dividere tra i vari processi il tempo d'uso della **CPU**.

Mentre nei sistemi multiprogrammati qundo la **CPU** viene assegnata ad un processo, gli altri non la possono utilizzare finché questo non termina la sua **cpu-burst**.

Nel _time-sharing_ la **CPU** è assegnata ad ogni processo per un **_quanto di tempo uguale e prederminato per tutti_**.

La politica quindi diventa la seguente:
- Se durante il quanto di tempo hai terminato la **cpu-burst** vai in attesa fino al prossimo **cpu-burst**
- Se durante il quanto di tempo _**non**_ hai terminato, il tuo stato intermedio viene salvato e verrà ripristinato quando tornerà il tuo turno. Questa revoca viene chiamata _preemption_.

Il salvataggio e ripristino dello stato intermedio corrisponde a tutti gli effetti al **_cambio di contesto_** che avevamo visto nel corso di **Calcolatori Elettronicoi**.

Un esempio noto di _time-sharing_ è il _round-robin_.

## 5.5. Overhead

La multiprogrammazione non è gratuita.
Il costo di migliorare i tempi si chiama _**overhead**_, e consiste nel tempo necessario al sistema operativo ad eseguire codice aggiuntivo per eseguire le operazioni intermedie come l'algoritmo di schedulazione, il cambio di contesto, ...

Questo tempo è a tutti gli effetti sottratto dall'esecuzione dei programmi applicativi. Per poter vedere un guadagno nei tempi di esecuzione è necessario che l'_overhead_ sia contenuto (1%/2%)

Se avessimo infatti _overhead_ del 70% del tempototale, potrebbe non essere conveniente avere un sistema multiprogrammato.
Se fosse ancora più alto il sistema potrebbe andare in _crash_, in quanto impiegherebbe tutto il tempo a eseguire le istruzioni di _overhead_ e non avrebbe più tmepo per eseguire i programmi applciativi.

## 5.6. Sistemi in tempo reale

<div class="grid2">
<div class="">

Sono uno degli ultimi step dell'evoluzione dei sistemi Operativi. Questa tipologia si interfaccia in tempo reale con l'**ambiente operativo** attraverso _sensori_ per recepire informazioni dall'esterno, e _attuatori_ per poter comunicarvi.

Un esempio sono gli _smartphone_, che tramite il _touchpad_ introducono il concetto di grafica, finestra, ...
Un altro esempio di sistemi in tempo reale sono quei sistemi _special purpose_, che dipendono strettamente dall'evoluzione di un sitema fisico nel tempo reale, come braccia robotiche, robot pulitori, macchine a guida autonoma, ...
Per questi sistemi _embedded_ infatti è fondamentale riuscire a generare risposte agli input in tempi brevi per non avere errori.

Vedremo più avanti che i sistemi in tempo reale si dividono in:
- **hard-real-time**: la violazione di una _deadline_ provoca effetti catastrofici sul sistema. Un esempio può essere un'auto a guidaautonoma che non si rende conto di dover frenare o fare una curva
- **soft-real-time**: la violazione di una _deadline_ non provoca errori distruttivi. Un esempio sono i servizi di _streaming_, dove un ralentamento provoca solo errori temporanei che non hanno effetti catastrofici.
</div>
<div class="">
<img class="60" src="./images/Concetti Introduttivi/sistema in tempo reale.png">
</div>
</div>

# 6. Architettura di un sistema di Elaborazione

In questa parte rivedremo buona parte delle nozioni di architettura del calcolatore affrontati a **Reti Logiche** e **Calcolatori Elettronici**.

A livello _hardware_ nei calcolatori si segue il modello di **Von Neumann**, che consiste in un _bus_ sul quale sono collegate tutte le componenti del calcolatore: **CPU**, **RAM**, e tutte le varie periferiche (video, disco, tastiera, porte seriali, ...).

La **CPU** è l'elemento architetturalmente più complesso all'interno del calcolatore (esclusa la **GPU** ndr.), che ha il compito di _**eseguire le istruzioni**_ che le arrivano in un formato detto `CISC` (_Complex Istruction Set Computing_). In realtà oggi la maggior parte dei processori converte le istruzioni `CISC` in istruzioni `RISC` (_Reduced ISC_) che semplifica e velocizza l'esecuzione delle istruzioni.
Infatti nei processori `CISC` si notò la regola dell'80-20: nell'80% dei casi utilizzava sempre lo stesso 20% delle istruzioni. Attraverso le `RISC` si ottimizza quel 20% di istruzioni, e si eseguono le altre come se fossero _routine_ composte da quelle semplificate.
In questo modo le istruzioni più comuni utilizzano 1 ciclo di clock, mentre le altre, molto più rare, utilizzano più cicli.

Le istruzioni che le **CPU** esegue non sono salvate nei suoi _registri_, bensì si trovano nella memoria centrale, ovvero la **RAM**. I _registri_ della **CPU** hanno due ruoli:
- Registri di appoggio per dati intermedi
- Amministrano l'evoluzione del programma (`%rip`, `%rbp`, `%rsp`) e ne monitorano lo stato(`%rflag`)

Per ottimizzare i tempi di lettura **Von Neumann** propose semplicemente di avere dei _bus_ molto efficaci che non avessere un effetto _bottleneck_ rispetto alla **RAM** e alla **CPU**, successivamente venne introdotto il concetto di _cache_.

La **RAM** (_Random Access Memory_) per il programmatore è un _array_, dove l'indice di ogni cella viene chiamato _indirizzo_. Il prefisso _Random_ indica proprio l'accesso casuale alla memoria, ovvero libero e diretto. Sulla **RAM** possiamo effettuare due operazioni:
- **Lettura**: non varia lo stato della memoria
- **Scrittura**: modifica lo stato della memoria aggiornandolo escliusivamente nelle locazioni desiderate

Il _bus_ ha unruolo fondamentale all'interno del calcolatore, poiché permette la comunicazione tra le varie componenti.
È composta da centinaia di fili, ognuno con un ruolo diverso (indirizzi, dati, controlli, ...).

Esistono due politiche di comunicazione, in particolare vediamo al politica **master-slave**..

In questa politica solamente il _master_ (nel nostro caso la **CPU**) può iniziare la comunicazione.
In un sistema dove si ha un solo _master_ si rimuove il problema della competizione di accesso al _bus_. In questo modo, tutti i dispositivi rimangono in "attesa" sul bus, ovvero sono in ascolto come ricevitori di segnale. 
Dovremo quindi dedicare dei fili del bus per riuscire a identificare chi è il destinatario delle informazioni comunicate (**RAM**, disco, video, ...) e in cosa consiste la comunicazione (`r`/`w`).

Il protocollo di comunicazione è semplice:
- Tutti i dispositivi stanno in ascolto sul bus attendendo modifiche al suo stato
- Quando il _master_ vuole iniziare la comunicazione inizializza intanto quei bit che definiscono:
  - Il dipositivo _slave_ con il quale si vuole comunicare
  - L'operazione di comunicazione desiderata
- Tutti gli slave stanno in ascolto e solamente quello selezionato risponde alla **CPU** quando è pronto
- Dopo il segnale di _ack_ la **CPU** inizia a comunciare i vari dati (indirizzi, valori, ...)


## 6.1. **CPU**

La **CPU** a grandi linee esegue un ciclo di fasi:
- **Fetch**/**Prelievo**: la **CPU** recupera la prossima istruzione da eseguire. Per fare ciò si affida al contenuto di un registro interno (_IP_ (_Istruction Pointer_) o _PC_ (_Program Counter_)) sul quale effettua dei calcoli per ottenere l'indirizzo dove si trova la prossima istruzione da eseguire. 
  Successivamente effettua una copia dalla memoria per recuperare l'istruzione all'interno di un _Istruciton Register_.
  Ultima azione, importante per il proseguimento del ciclo, si incrementa il valore all'interno di _IP_ affinché la prossima lettura sia dell'istruzione successiva
- **Decodifica**: si decodificano i bit che compongono l'`%ir` che decodificano le istruzioni assembler in segnali che specificano quale operazione della `ALU` utilizzare con quali dati.
  Rispetto al _fetch_ questa operazione è molto più veloce
- **Esecuzione**: l'operazione viene eseguita dalla `ALU` e i risultati vengono propagati nei registri e/o in memoria

All'interno della **CPU** ci sono diversi registri di appoggio per le comunicazioni con il bus, in aprticolare:
- `MAR` (_Memory Address Register_): registro di appoggio dove salvare l'indirizzo della cella di memoria desiderata
- `MDR` (_Memory Data Register_): registro di appoggio dove salvare il valore della cella di memoria desiderata
- `PSW` (_Program Status Word_): registro che conserva informazioni relative al privilegio del processo attualmente in esecuzione (`USER`/`SUPERUSER`)


## 6.2. Cache

È un ottimizzazione introdotta per minimizzare le letture versola memoria, che è molto più lenta rispetto ai cicli di _clock_ ddella **CPU**.
La cache si basa su due principi:
- **Principio di Località Temorale**
- **Principio di Località Spaziale**

Grazie a questi principi riusciamo ad aumentare il numero di istruzioni eseguite nell'unità di tempo.

Esistono diverse tecnologie di cache, dovuto al fatto che la _cache_ (più veloce della **RAM**) è anche molto più piccola, quindi si possono generare collisioni nei contenuti.

Per ulteriori informazioni consultare [gli appunti di Calcolatori dedicati](https://gabriele-d-cambria.github.io/Appunti-Calcolatori-Elettronici-2024-2025/Memoria%20e%20Periferiche#3-memoria-cache)

L'unica aggiunta che facciamo è distinguere due tipi di cache:
- **I-Cache**: cache relativa alle istruzioni. Non necessita la propagazione in memoria al cambio di contesto, è sufficiente invalidarla.
- **D-Cache**: cache relativa ai dati delle istruzioni. Necessita la propagazione in memoria dei dati al cambio di contesto

Esistono cache fino a tre livelli, in particolare la **cache di I° Livello** è implementata direttamente all'interno del circuito della **CPU**. La lettura di questi dati è quindi effettuabile in una piccola frazione di _clock_

## 6.3. Memoria

Si può tracciare una gerarchia tra i vari tipi di memoria:

<figure class="">
<img class="100" src="./images/Concetti Introduttivi/memory-gerarchy.png">
<figcaption>

Possiamo associare questa gerarchia a due parametri: capacità totale della memoria, velicità di accesso in termini di costo per bit
</figcaption>
</figure>

Per il programmatore:
- Il disco: esistono diversi modi per gestire l'accesso del software
- **RAM**: vista come se fosse un vettore

## 6.4. IO

Possiamo riassumere un calcolatore in un semplice schema a blocchi:

<img class="" src="./images/Concetti Introduttivi/calculator-scheme-io.png">


Da questo punto di vista possiamo riassumere tutte le periferiche di IO come l'insieme di due componenti:
- **Interfaccie**: dette anche **controllori**, si preoccupa digestire la comunicazione ad uno specifico trasduttore esterno
- **Trasduttore Esterno**: mezzo di comunicazione con l'esterno (_touchscreen_, )

In questo modo possiamo scriverre i programmi ignorando ccompletmente il funzionamento del tarsduttore esterno, ma concentrandoci esclusivamente sulla comunicazione con le interfaccie.
Infatti tratteremo la comunicazione tra interfaccia e trasduttore come se fosse un **processo esterno**. Dal processore sarà possibile accedere tendenzialmente a tre registri:
- Registro di controllo: specifica il tipo di comunicazione
- Registro di stato: utilizzato per comunicare un'aggiornamento dello stato del registro dei dati
- Registro dei dati: utilizzato per comunicare i dati

## 6.5. Interruzioni

Le interruzioni permettono di interromprere il flusso di un programma per eseguire altri processi.

Per maggiori informazioni sul loro funzionamento [consultare il seguente link](https://gabriele-d-cambria.github.io/Appunti-Calcolatori-Elettronici-2024-2025/Protezione)

## 6.6. Protezione

Per i meccanismi di protezione [consultare il seguente link](https://gabriele-d-cambria.github.io/Appunti-Calcolatori-Elettronici-2024-2025/Interruzioni)

## 6.7. DMA

Per i meccanismi di `DMA` [consultare il seguente link](https://gabriele-d-cambria.github.io/Appunti-Calcolatori-Elettronici-2024-2025/DMA)

# 7. Struttura dei Sistemi Operativi

Prendiamo in considerazione i sistemi operativi basati su `Unix`. Questo sistema, detto **modulare**, prevede una struttura a strati divisa in diversi livelli, ognuno separato da una particolare interfaccia che specifica la funzionalità offerta dal modulo e come utilizzarla.
È inoltre presente un corpo contenente l'implementazione del modulo, _non visibile al suo esterno_.

Nei sistemi `Unix` in particolare fanno parte del sistema sia le tipiche componenti di un _OS_ invocate tramite le chiamate di sistema, eseguite in uno stato privilegiato e identificate globalmente con il termine di _kernel_, sia l'insieme dei programmi di utilità del sistema (_shell_, compilatori, caricatori, linker, librerie, ...) eseguiti in uno stato non privilegiato come normali programmi.

<img class="" src="./images/Concetti Introduttivi/unix-structure.png">


A livello _kernel_ troviamo il **process control system**, quindi troviamo la struttura necessaria alla schedulazione e alla virtualizzazione dei processi. Essa è a sua volta suddivisa in:
- `IPC`: _Inter Process Communication_, permette la comunicazione tra processi diversi
- `Scheduler`: il suo compito è ripartire l'uso del processore tra i vari programmi in memoria, in modo che la **CPU** possa sviluppare un insieme di processi contemporaneamente
- `Memory Manager`: consente l'evoluzione concorrente di un insieme di processi mantenendo in memoria più programmi, ognuno con le sue esigenze

Nelle prime versioni `UNIX` al medesimo livello troviamo anche il **file subsystem** e i **device drivers**.

La necessità di proteggere le componenti del sistema operativo ha portat alla necessità di introdurre il **doppio stato di esecuzione**, e di garantire che soltanto le componenti del sistema operativo possano girare in stato privilegiato. Per risolvere le problematiche dovute a questa doppio stato è stat proposta una soluzione nota come **struttura a _microkernel_**.
In questa struttura, per ogni risorsa vengono definite due componenti del sistema operativo:
- I meccanismi che il sistema deve fornire
- Le specifiche strategia di gestione realizzate con i precedenti meccanismi

Ad esempio, nel caso del processore:
- Il _cambio di contesto_ è il meccanismo che dobbiam ofornire
- Lo _scheduler_ implementa la strategia vera e propria per il cambio

In questi sistemi, l'insieme dei meccanismi costituisce il _microkernel_ del sistema, unico componente a girare in stato privilegiato.
Le strategia invece fanno parte di programmi di sistema che girano come normali processi applicativi.
Questa struttura presuppone che, quando un processo applicativo (_client_) deve richiedere l'uso di una risorsa deve prima **interagire col corrispondente processo server** mediante un meccanismo di comunicazione tra processi (`IPC`).
Questi sistemi implementano maggiore portanilità e modificabilità, ma si registrano perdite di efficienza legate al fatto che ogni chiamate di sistema si traduce in comunicazioni tra processi.

Per questo motivo venne introdotta una nuova struttura, detta **struttura _client-server_**.
Tipica dei sistemi distribuiti in rete, immaginiamo di avere tanti nodi connessi ad una rete comune. Su alcuni nodi è implementato una parte del _kernel_, mentre altri fanno girare normali processi appliativi
In questo modo non modifichiamo l'interfaccia al sistema operativo indipendentemente che esso sia allocato su uno o più dischi diversi. Si parla in questo caso di Sistemi Trasparenti.


