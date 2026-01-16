---
title: Gestione Processi
---

# 1. Indice

- [1. Indice](#1-indice)
- [2. Processi](#2-processi)
	- [2.1. Stato di un processo](#21-stato-di-un-processo)
	- [2.2. Spazio Virtuale di Memoria](#22-spazio-virtuale-di-memoria)
	- [2.3. Descrittori di Processo e Code processi](#23-descrittori-di-processo-e-code-processi)
	- [2.4. Cambi di Contesti](#24-cambi-di-contesti)
	- [2.5. Creazione e Terminazioni di Processi](#25-creazione-e-terminazioni-di-processi)
	- [2.6. Processi concorrenti](#26-processi-concorrenti)
- [3. Scheduling](#3-scheduling)
	- [3.1. Valutazione algoritmi di Scheduling](#31-valutazione-algoritmi-di-scheduling)
	- [3.2. Algoritmo `FCFS`](#32-algoritmo-fcfs)
	- [3.3. Algoritmo `SJF`](#33-algoritmo-sjf)
	- [3.4. Algoritmo `SRTF`](#34-algoritmo-srtf)
		- [3.4.1. Stima della CPU-Burst](#341-stima-della-cpu-burst)
	- [3.5. Algoritmo `RR`](#35-algoritmo-rr)
	- [3.6. Sistemi Multi Level Queue](#36-sistemi-multi-level-queue)
	- [3.7. Schedulazione di sistemi in tempo reale](#37-schedulazione-di-sistemi-in-tempo-reale)
		- [3.7.1. Rate Monototic](#371-rate-monototic)
		- [3.7.2. Valutazione Esistenza dell'algoritmo](#372-valutazione-esistenza-dellalgoritmo)
		- [3.7.3. Earliest Deadline First](#373-earliest-deadline-first)
- [4. Thread](#4-thread)
- [5. Processi in Unix](#5-processi-in-unix)

# 2. Processi

Il termine _processo_ viene utilizzato in modo informale per indicare un programa in esecuzione.
Una definizione più formale è la seguente:
> Un **processo** rappresenta la _sequenza di eventi osservabili_ generati dall'elaboratore durante l'esecuzione di un programma. Identifica quindi _l'unità di esecuzione_ all'interno di un sistema operativo multiprogrammato.

Più processi possono essere associati allo stesso programma, in quel caso prendono il nome di _istanze_. Ciascuna istanza rappresenta quindi l'esecuzione dello stesso codice con dati in ingresso diversi.

Infatti al variare dei valori di ingresso uno stesso processo può avere storie diverse (svilupparsi in _if-else-branch_ diversi, iterare più o meno volte uno stesso ciclo, ...).

## 2.1. Stato di un processo

Un sistema operativo multiprogrammato in particolare **consente l'esecuzione concorrente di più processi**.

La **CPU**, in un sistema multiprogrammato, passa da un processo all'altro ad elevatissima velocità, tale per cui l'utente ha l'impressione che i vari processi vengano eseguiti _parallelamente_.
In realtà la **CPU** non esegue nulla in parallelo, in quanto può eseguire sempre e solo una istruzione alla volta. Chiamiamo però questo fenomeno: **_parallelismo virtuale_**.
Il _parallelismo fisico_ invece è possibile solo in presenza di **più CPU**.

Dato il continuo scambio di processi la **CPU** deve essere in grado di salvare lo stato di ciasuno di esso nel momento in qui viene interrotto. Così facendo, quando riprenderemo con la sua esecuzione, saremo in grado di farlo senza generare errori.

Nei sistemi _monoprogrammati_ gli stati possibili nei quali un processo può essere nel suo _lifetime_ sono due:
- `attivo`: quando il processo ha il possesso della **CPU**
- `bloccato`: quando il processo non ha il possesso della **CPU**, perché è in attesa del verificarsi di un evento.

In questi sistemi un processo cambia i due stati tramite:
- **Sospensione**: il processo si mette in _CPU-idle_ e mette a lavoro una periferica
- **Riattivazione**: la periferica, terminato il lavoro, invia un _interruzione esterna_ alla **CPU** riattivando il processo

Attenzione, il fatto che il processo è bloccato non significa che le sue risorse non possano cambiare. Infatti, se il processo è stato bloccato a seguito di una richiesta di recupero dati da parte di un dispositivo esterno nella memoria, il `DMA` sovrascrive la memoria del processo mentre questo è bloccato.

Inoltre, il fatto che la **CPU** sia nello stato _idle_, non significa che è spenta. È infatti proprio lei a rendersi conto dell'arrivo dell'interruzione esterna segnalata dall'`APIC`.
Dovrà quindi esistere un modo per effettuare controlli sull'arrivo delle interruzioni esterne.

Quindi i sistemi monoprogrammati hanno senso solo se:
> L'unico processo presente **_deve includere anche il sistema operativo e le sue strutture dati_**

In sistemi _multiprogrammati_, se il numero di **CPU** è minore del numero di processi, esiste un ulteriore sotto-stato:
- `attivo`: diviso adesso in due sotto-stati:
  - `in esecuzione`: il processo ha assegnata la **CPU**
  - `pronto`: il processo è in attesa di andare in esecuzione
- `bloccato`: il processo è in attesa di qualche evento

La transizione `esecuzione` $\to$ `pronto` si chiama _revoca della **CPU**_, oppure _reapage_, mentre la transizione inversa si dice _assegnazione_.

In questa modalità solo i processi in `esecuzione` possono _sospendersi_ nello stato **`bloccato`**. Inoltre, la **CPU** in questo caso **_non è necessariamente idle_**, ma potrebbe essere _busy_, sia con processi utente sia con processi sitema (come processi di manutenzione).
Oltre a ciò, un `processo` bloccato che viene _riattivato_ va a finire nello stato `pronto`.

In questi sistemi si introduce adesso il problema di scegliere però quale processo `pronto` dovrà andare in esecuzione.
Il meccanismo che effettua questa scelta si chiama _scheduler_.
Lo _scheduler_ va in esecuzione quando:
- Il processo in `esecuzione` si sospende
- Al processo in `esecuzione` viene revocata la **CPU**
- Il processo in `esecuzione` termina

A seconda del criterio di _scheduling_, lo _scheduler_ potrebbe avviarsi anche quando un nuovo processo viene `creato` (_scheduling_ per priorità).

Questo modello viene chiamato **_Modello a cinque stati_**:

<img class="" src="./images/Gestione Processi/five-state-model.png">


Per riuscire a mantenere in maniera consistente e corretta le varie informazioni del processo mentre esso transita nei vari stati è necessario avere una struttura dati specifica, chiamata **descrittore di processo**. L'allocazione del **descrittore di processo**, e della memoria a lui risevata sanciscono la creazione del processo.

In questo modo il singolo processo non si rende conto che la sua esecuzione è stata interrotta, ma crede di aver avuto il possesso della **CPU**, chiamata proprio per questo **CPU virtuale**, per tutto il suo _lifespan_.
Le **CPU virtuali** sono dotate di _program counter_ e _registri_, che contengono le informazioni relative ad ogni processo alle quali sono associate,.

Lo processo è quindi rappresentato da:
- **Codice**: puntatore a un file che contiene il codice del processo
- **Dati**: Dipendono dal programma. È quindi necessario che il codice eseguibile sia definito in modo tale da dare un idea chiara delle variabili e delle strutture dati necessarie al programa. Verosibilmente, anche in questo caso avremo un puntatore alla memoria principale, in quanto memorizzare tutti i dati in una struttura sarebbe estremamente difficile
- **Program Counter**, **Registri** e **Stack**: per poter descrivere un processo sospeso/bloccato è necessario avrer memorizzato lo stato di quando era in _esecuzione_. Questi strumenti ci permettono di farlo.

Un processo è quindi identificato anche dallo _stato_ che specifica cosa esso sta facendo in questo momento. Il fatto che possa essere bloccato o messo in attesa giustifica la necessità di memorizzare i campi del processo in un **descrittore**.

Ad un processo possono anche essere associate delle risorse:
- **Memoria**: il programma potrebbe chiedere di descrivere delle strutture dati in memoria _heap_ attraverso delle `new`. È quindi importante mantenere anche un puntatore a questa memoria
- **File**: il processo potrebbe accedere anche ad un elenco di file per operazioni di lettura/scrittura, dobbiamo mantenere traccia anche di questi
- **Dispositivi I/O**: dobbiamo sapere anche a quali dispositivi di I/O il processo ta utilizzando.

## 2.2. Spazio Virtuale di Memoria

<div class="grid2">
<div class="">

Lo _spazio virtuale di memoria_  di un processo è generalmente rappresentato come sulla destra.

Dobbiamo sicuramente trovare una sezione `text` nel quale è contenuto il **codice del processo**.
Nella sezione `data` si trovano invece le strutture dati definite dal processo.

Successivamente abbiamo due spazi complementari:
- `Heap`: dall'alto verso il basso
- `Stack`: dal basso verso l'alto

Questi hanno dimensione variabile nel corso del _lifespan_ del processo.

</div>
<div class="">
<img class="20" src="./images/Gestione Processi/virtual-memory-scheme.png">
</div>
</div>

## 2.3. Descrittori di Processo e Code processi

Abbiamo già detto che ad ogni processo è associato un **descrittore di processo** (`PCB`, _Process Control Block_), che a livello software si implementa come una struttura dati.
I descrittori sono a loro volta organizzati in una tabella, chiamata **tabella dei processi**.

All'interno di un descrittore di processo sono salvati i dati:
- **Nome del processo**: nei sistemi `Unix` si utilizza un numero naturale, detto _Process ID_ (`pid`). Il numero di `bit` sul quale si codifica il `pid` determina il numero massimo di processi. Inoltre, vanno gestiti i `pid` dei processi termianti (vedremo che lo farà direttamente un meccanismo all'interno del _kernel_)
- **Stato del processo**
- **Modalità di servizio dei processi**: ha un ruolo diverso a seconda del tipo di sistema operativo:
  - Nei sistemi _priority_ contiene l'importanza relativa del processo nei confronti degli altri
  - Nei sistemi a _suffivisione del tempo_ contiene invece il quanto di tempo che la **CPU** può dedicare allo stesso
  - Nei sistemi _real-time_ contiene il tempo massimo entro il quale la richiesta deve essere soddisfatto
- **Informazioni sulla gestione della memoria**: vedremo che qui saranno contenuti dei puntatori che permettono di mantenere le informazioni sugli indirizzi di memoria allocati
- **Contesto del processo**: contiene le informaizoni relative ai registri della **CPU** utilizzati durante il cambio di contesto
- **Utilizzo delle risposte**: in `Unix` ci sono dei puntatori a tabelle logiche del sistema (dispositivi I/O assegnati, file aperti, tempo di uso della **CPU**, ...)
- **Identificazione del processo successivo**: L'ultimo dato è necessario per poter correttamente implementare le code di processi.


All'interno del sistema sono presenti tante code di processi, si dividono generalmente in due macrocategorie:
- `coda pronti`: contiene i descrittori dei processi in attesa di andare in esecuzione.Vedremo come in alcuni casi potremmo anche averne più di una
- `coda processi bloccati`:contiene i descrittori dei processi che attendono l'arrivo di una _interruzione esterna_ per poter tornare nello stato `pronti`

## 2.4. Cambi di Contesti

L'utilizzo della **CPU** viene commutato da un processo all'altro. Ogni volta che si effetta questa commutazione hanno luogo una serie di azioni
1. **Salvataggio stato**: si salva il contesto del processo in `esecuzione` nel suo descrittore
2. Inserimento del descrittore nella coda adeguata (`bloccati`, `pronti`, ...)
3. **Short term scheduling**: Lo _scheduler_ seleziona un altro processo dalla coda dei processi `pronti` e si carica il suo `pid` nel registro che identifica il _processo in esecuzione_.
4. **Ripristino dello stato**: detto anche _dispatch_, carica il contesto del nuovo processo dal suo descrittore ai registri del processore.

Vedremo più a fondo che non sono i processi ad effettuare le azioni, ma i **Soggetti**.

I **Soggetti** sono composti da:
- _**Processo**_
- _**User ID**_
- _**Group ID**_

Inoltre, analizzeremo come gli ultimi due permettono di implementare i sistemi di protezione in sistemi multiutente.

I processi si dividono in:
- **Processi Pesanti**: è necessario descrivere sia lo stato di esecuzione che lo stato di tutto lo spazio di memoria.
- **Processi Leggeri** o **_Thread_**: è necessario descrivere solo lo stato di esecuzione.

La più grande differenza è il fatto che i **processi pesanti** hanno _spazi di memoria non condivisi_, mentre i **_thread_** lavorano su spazi di memoria condivisi.
Se un processo pesante ha dei _thread_ interni, questi condividono lo spazio di memoria tra di loro e con il processo pesante che li contiene.

## 2.5. Creazione e Terminazioni di Processi

Un processo, detto _padre_, può richiedere tramite apposite _syscall_ la creazione di un nuovo processo, chiamato _figlio_, generando una gerarchia di processi.
Le politiche di scelta di quale programma il processo _figlio_ eseguirà (così come i dati e le risorse condivise) possono variare da sistema a sistemi.

Ogni processo è quindi **figlio di un altro processo**, e può essere a sua volta _padre di altri processi_.

L'_OS_ mantiene nel descrittore le informazioni relative alle relazioni paternali. 

Alla terminazione di un processo ci possono essre più politiche di segnalazione ai processi antenati.
Nei sistemi `UNIX` quando un processo figlio termina mentre il padre non è ancora terminato non entra nello stato `terminato`, ma in uno stato `zombie`.
Grazie a questo stato intermedio, il processo padre può rilevare la terminazione dei propri processi figli. Quando la terminazione di un processo figlio viene rilevata, questo effettivamente termina.

Inoltre, sempre in `UNIX` la terminazione di un processo padre comporta **la terminazione di tutti i processi figli**.

## 2.6. Processi concorrenti

Due processi si dicono **concorrenti** se le loro esecuzioni si _sovrappongono nel tempo_, o, più in generale:
> Se la prima operazione di un processo avviene prima che termini l'ultima operazione dell'altro, generando fenomeni di _interleaving_ (1 processore) o _overlapping_ (più processori).

Possiamo definire anche i processi **indipendenti**:
> Se il risultato prodotto dall'esecuzione di uno **non è influenzato** da quella di dell'altro, e viceversa.

Non consideriamo nella relazione di _indipendenza_ l'influenza temporale, che è innegabile e non trascurabile, ma ci limitiamo a considerare solamente l'influenza logica, chiamando la relazione **proprietà della riproducibilità**.

Definiamo quindi i processi **interagenti**:
> Se il risultato prodotto dall'esecuzione di uno **è influenzato** da quella dell'altro, e viceversa

In questo caso, la presenza di un altro processo può andare a modificare il risultato prodotto dal processo.

I processi possono quindi interagire:
- **Competizione**: se due processi vogliono utilizzare risorse comuni che non possono essere utilizzate ccontemporaneamente, generando problemi di **_mutua esclusione_**.
  Questi problemi non sono risolti internamente al programma (dato che non sa nemmeno che esistano altri processi), ma vengono gestiti dal sistema operativo
- **Cooperazione**: se due processsi vogliono eseguire un'attività comune mediante scambio di informazioni

Il ruolo del _kernel_ è quello di realizzare l'astrazione di **CPU virtuale** fornendo la possibilità di:
- Avere funzioni di risposta alle interruzioni
- Permettere il cambio di contesto tra i processi

# 3. Scheduling

È l'attività mediante la quale il sistema operativo effettua delle scelte tra i processi, riguardo al _caricamento in **RAM**_ e all'_assegnazione della **CPU**_.

Vi possono essere tre diverse attività di _scheduling_:
- **A breve termine**: è lo _scheduling_ propriamente detto. È la politica con la quale il sistema operativo assegna la **CPU** ai processi pronti.
  Interviene quando il processo in esecuzione perde il controllo della **CPU**, e può essere di due tipi:
	- Non _preemptive scheduling_ (senza diritto di revoca): il sistema operativo non può revocare la **CPU**, ma deve essere lui a rilasciarla
	- _Preemptive scheduling_ (con diritto di revoca): il sistema operativo può forzare la revoca della **CPU** ad un processo in base a determiate variabili. (quanti di tempo, priorità, ...)
- **A medio termine**: Si riferisce principalmente allo **_swapping_**, che permette di utilizzare la memrooia secondaria per trasferire temporaneamente i processi qualora la memoria principale non sia sufficiente per soddisfare le richieste di tutti i processi nel sistema
- **A lungo termine**: si occupa di scegliere quali programmi caricare in memoria centrale dalla memoria secondaria. Controlla quindi il _grado di multiprogrammazione_, ovvero quanti processi possono essere attivi contemporaneamente.

Lo scheduling di breve termine è invocato molto spesso (nell'ordine dei millisecondi), perciò deve essere **molto veloce**. Quello di lungo termine invece viene invocato nell'ordine di secondi se non minuti, perciò può essere meno efficiente.

Per comprendere meglio le differenze tra gli algoritmi di _scheduling_, descriviamo i processi come:
- **Legati all'I/O**: impiegano la maggior parte del tempo effettuando comunicazioni `I/O` piuttosto che occupare la **CPU**
- **Legati alla CPU**: impiegano la maggior parte del tempo effettuando computazioni sulla **CPU** piuttosto che comunicare attraverso le periferiche `I/O`

<div class="grid2">
<div class="">

Gli algoritmi di scheduling che vedremo sono:
- **First-Come-First-Served** [`FCFS`](#32-algoritmo-fcfs)

- **Shortest-Job-First** [`SJF`](#33-algoritmo-sjf)

- **Shortest-Remaining-Time-First** [`SRTF`](#34-algoritmo-srtf)

- **Round-Robin** [`RR`](#35-algoritmo-rr)

Questi possono essere classificati classificarli come sulla destra

Vedremo inoltre altre tecniche di schedulazione:
- **Schedulazione su base prioritaria**

- **Schedulazione "a code multiple"**

- **Schedulazione di sistemi in tempo reale**
  - _Rate Monotonic_ `RM`
  - _Earliest Deadline First_ `EDF`

</div>
<div class="">
<img class="50" src="./images/Gestione Processi/scheduling-category.png">
</div>
</div>

## 3.1. Valutazione algoritmi di Scheduling

Definendo come $\Delta_{B_i}$ i **CPU-Burst** e come $\Delta_{a_i}$ i **IO-Burst**, possiamo valutare gli algoritmi secondo diversi parametri:

<div class="flexbox" markdown="1">

| Parametro                                            | Formula                                         | Valore Desiderato                          |
| ---------------------------------------------------- | ----------------------------------------------- | ------------------------------------------ |
| **Utilizzo della CPU**                               | $E = \frac{\sum{\Delta_{B_i}}}{T}$              | Tendente a $1$                             |
| **Tempo medio di completamento** (_turnaround time_) | $T_m = \sum{\Delta_{B_i}} + \sum{\Delta_{a_i}}$ | Il più piccolo possibile                   |
| **Produttività** (_throughput rate_)                 | $\frac{1}{T_M}$                                 | Il più grande possibile                    |
| **Tempo di Risposta**                                | $T_m = \sum{\Delta_{B_i}} + \sum{\Delta_{a_i}}$ | Minimizzato (coincide con il _turnaround_) |
| **Tempo di Attesa**                                  | $A_m = \sum_i{t_{a_i}}$                         | Minimizzato                                |
| **Rispetto dei Vincoli Temporali**                   | -                                               | Soddisfatto                                |

</div>

## 3.2. Algoritmo `FCFS`

È il più semplice tra gli algoritmi di scheduling della **CPU**.
> Assegna la **CPU** al processo pronti in attesa da più tempo

Quando un processo entra nella coda dei processi `pronti` il suo descrittore viene collegato **all'ultimo elemento della coda**. Quando la **CPU** è libera viene assegnata al processo il cui descrittore si trova nella _testa della coda_.
È a tutti gli effetti equivalente alla politica `FIFO`.

La gestione della coda processi, mantenendo riferimenti alla cima e al fondo, ha quindi complessità:
- $O(1)$ per gli inserimenti
- $O(1)$ per le estrazioni

Uno schema per capire come funziona può essere il seguente:

<div class="grid2">
<div class="">

Ipotizziamo i seguenti processi in arrivo:

<div class="flexbox" markdown="1">
<table><thead>
<tr>
	<th align="center">Processo</th>
	<th align="center">Tempo di Arrivo</th>
	<th align="center">CPU-Burst</th>
</tr>
</thead>
<tbody>
<tr>
	<td align="center"><code>P0</code></td>
	<td align="center">0</td>
	<td align="center">10</td>
</tr>
<tr>
	<td align="center"><code>P1</code></td>
	<td align="center">2</td>
	<td align="center">100</td>
</tr>
<tr>
	<td align="center"><code>P2</code></td>
	<td align="center">4</td>
	<td align="center">24</td>
</tr>
<tr>
	<td align="center"><code>P3</code></td>
	<td align="center">6</td>
	<td align="center">16</td>
</tr>
</tbody></table>
</div>

</div>
<div class="">
<figure class="100">
<img class="100" src="./images/Gestione Processi/scheduling-example/FCFS-example.png">
<figcaption>
Grafico non in scala
</figcaption>
</figure>

</div>
</div>


Con questo algoritmo, in questo caso, abbiamo che:

<div class="flexbox" markdown="1">

| Parametro |                        Valore                         |
| :-------: | :---------------------------------------------------: |
|    $E$    |        $\to 1$ (trascuro i cambi di contesto)         |
|   $T_m$   |  $\frac{10 + (110-2) + (134-4) + (150 - 6)}{4} = 98$  |
|   $A_m$   | $\frac{0 + (10 - 2) + (110 - 4) + (150-8)}{4} = 60,5$ |

</div>


Questo algoritmo è altamente dipendente **dall'ordine nel quali i processi arrivano** per quanto riguarda i tempi medi di attesa e di _overhead_.
Se infatti invertissimo l'entrata del processo `P1` e del processo `P3` otterremmo che $T_m = 56$ e $A_m = 18,5$.
Possiamo anche dedurre che questo algoritmo viene pesantemente deabilitato da processi con **CPU-Burst** grandi che arrivano prima di altri con **CPU-Burst** breve.


## 3.3. Algoritmo `SJF`

È un algoritmo a priorità statica non preemptive. La priorità viene assegnata ad ogni processo su base inversa rispetto al suo **CPU-Burst**, che per ora supponiamo noto.
<div class="grid2">
<div class="">

Con lo stesso esempio di prima

<div class="flexbox" markdown="1">
<table><thead>
<tr>
	<th align="center">Processo</th>
	<th align="center">Tempo di Arrivo</th>
	<th align="center">CPU-Burst</th>
	<th align="center">Priorità</th>
</tr>
</thead>
<tbody>
<tr>
	<td align="center"><code>P0</code></td>
	<td align="center">0</td>
	<td align="center">10</td>
	<td align="center">0</td>
</tr>
<tr>
	<td align="center"><code>P1</code></td>
	<td align="center">2</td>
	<td align="center">100</td>
	<td align="center">3</td>
</tr>
<tr>
	<td align="center"><code>P2</code></td>
	<td align="center">4</td>
	<td align="center">24</td>
	<td align="center">2</td>
</tr>
<tr>
	<td align="center"><code>P3</code></td>
	<td align="center">6</td>
	<td align="center">16</td>
	<td align="center">1</td>
</tr>
</tbody></table>
</div>

</div>
<div class="">
<figure class="100">
<img class="100" src="./images/Gestione Processi/scheduling-example/SJF-example.png">
<figcaption>
Grafico non in scala
</figcaption>
</figure>
</div>
</div>

Questo algoritmo, indipendentemente dall'ordine di arrivo, ordina i processi nel modo che abbiamo visto ottimizzava `FCFS`.
Tuttavia in questo algoritmo abbiamo più _overhead_.

Infatti l'inserimento in coda pronti è adesso un inserimento ordinato, perciò ha una complessità $O(n)$. Nel totale quindi sarà: $O(FCFS) + O(n) = O(1) + O(n) = O(n)$

Inoltre, essendo **non preemptive**, ha la limitazione che nel caso di arrivo di processi con elevati **CPU-Burst** prima di altri con tempi più bassi, ritorniamo nel problema di `FCFS`

## 3.4. Algoritmo `SRTF`

È un miglioramento di `SJF`, che introduce la possibilità di essere **preemptive**.
Inoltre, proprio per via della _preemption_, non si guarderà più la **CPU-Burst** iniziale del processo, ma quella che gli rimane da eseguire. 

Infatti, nel caso un processo con **CPU-Burst** elevata (100) che però sta per finire (rimanente 2), non ha senso sostituirlo con un altro appena arrivato che magari ha **CPU-Burst** (50), che è sì più breve di quella iniziale ma molto più alta di quella rimanente.

<div class="grid2">
<div>
<div class="flexbox" markdown="1">
<table><thead>
<tr>
	<th align="center">Processo</th>
	<th align="center">Tempo di Arrivo</th>
	<th align="center">CPU-Burst</th>
</tr>
</thead>
<tbody>
<tr>
	<td align="center"><code>P0</code></td>
	<td align="center">6</td>
	<td align="center">10</td>
</tr>
<tr>
	<td align="center"><code>P1</code></td>
	<td align="center">0</td>
	<td align="center">100</td>
</tr>
<tr>
	<td align="center"><code>P2</code></td>
	<td align="center">4</td>
	<td align="center">24</td>
</tr>
<tr>
	<td align="center"><code>P3</code></td>
	<td align="center">2</td>
	<td align="center">16</td>
</tr>
</tbody></table>
</div>

</div>
<div class="">
<figure class="100">
<img class="100" src="./images/Gestione Processi/scheduling-example/SRTF-example.png">
<figcaption>
Grafico non in scala
</figcaption>
</figure>
</div>
</div>

È un sistema che funziona bene per sistemi statici, dove il numero di processi non varia e i loro **CPU-Burst** è noto, ad esempio sistemi _embedded_ che hanno _overhead_ bassissimo.
Inoltre questo algoritmo, in caso di sistemi aperti con processi variabili, soffre di _starvation_. Infatti se arrivano continuamente processi con priorità più alta del primo, questo starà in attesa per un tempo che può diventare lunghissimo.

Per risolvere si utilizzano tecniche di _aging_, che monitorano i tempi di attesa e modificano opportunamente le priorità per rimediare alla _starvation_.

### 3.4.1. Stima della CPU-Burst

Non sempre sappiamo a priori il **CPU-Burst** di un processo.
Si utilizza quindi la media esponenziale per stimarlo, tenendo conto della storia dei valori misurati nei precedenti intervalli di esecuzione:

$$
\begin{align*}
t_n &: \text{durata del CPU-Burst} \\
s_n &: \text{la sua stima} \\
a &: \text{fattore con } 0 \le a \le 1 \\
s_{n+1} &: at_n + (1-a)s_n \\
\end{align*}
$$

Al variare di $a$ abbiamo che:
- $a = 0$: tutte le stime sono uguali a quella iniziale $\to s_{n+1} = 0\cdot t_n + (1-0)s_n = s_n$
- $a = 1$: non si tiene conto della storia precedente $\to s_{n+1} = 1 \cdot t_n + (1- 1)s_n = t_n$

Tipicamente avremo che $a = \frac{1}{2} \to s_{n+1} = \frac{t_n + s_n}{2}$, performando di fatto una media artimetica tra l'effettiva durata del **CPU-Burst** e la sua stima.


## 3.5. Algoritmo `RR`

È un algoritmo progettato appositamente per i sistemi a _partizione di tempo_, rientrando negli algoritmi _preemptive_.

La coda dei proessi pronti è realizzata come una **coda circolare**, nella quale ogni processo ottiene la **CPU** per un **_quanto di tempo_** (tipicamente $10ms\sim100 ms$) al termine del quale **perde il controllo della CPU** e il suo descrittore viene inserito nella coda.

La coda viene gestita in questo caso con modalità `FIFO` (_First-In-Fisrt-Out_).

<div class="grid2">
<div class="">

Prendendo questo esempio:
<div class="flexbox" markdown="1">
<table><thead>
<tr>
	<th align="center">Processo</th>
	<th align="center">Ordine della coda</th>
	<th align="center">CPU-Burst</th>
</tr>
</thead>
<tbody>
<tr>
	<td align="center"><code>P0</code></td>
	<td align="center">1°</td>
	<td align="center">100</td>
</tr>
<tr>
	<td align="center"><code>P1</code></td>
	<td align="center">2°</td>
	<td align="center">150</td>
</tr>
<tr>
	<td align="center"><code>P2</code></td>
	<td align="center">3°</td>
	<td align="center">10</td>
</tr>
<tr>
	<td align="center"><code>P3</code></td>
	<td align="center">4°</td>
	<td align="center">10</td>
</tr>
</tbody></table>
</div>

Con il quanto di tempo $Q = 20$ abbiamo la situazione mostrata sulla destra
</div>
<div class="">
<figure class="100">
<img class="100" src="./images/Gestione Processi/scheduling-example/RR-example.png">
<figcaption>
Grafico non in scala
</figcaption>
</figure>
</div>
</div>

È un algoritmo particolarmente adatto per i sistemi interattivi, in quanto è in grado di assicurare tempi di risposta abbastaza brevi, determinati esclusivamente da due fattori:
- **Il quanto di tempo**: il tempo di risposta è teoricamente migliore per piccoli valori del quanto di tempo. Tuttavia in questo caso non possiamo più _ignorare il cambio di contesto tra processi_. Infatti, cambi troppo frequenti comporterebbero un _overhead_ che potrebbe addirittura diventare più grande del quanto stesso.
- **Il numero medio di processi pronti**: se fossero presenti tanti processi pronti, potremmo andare incontro a tempi di _turnaround_ molto elevati, anche per processi con **CPU-Burst** molto brevi, che dovrebbero comunque attendere diversi cicli prima di poter terminare

Conoscendo il numero medio di processi in coda $n_m$, possiamo però calcolare il tempo di attesa massimo:
$$
A_m \le n_m \cdot Q
$$

## 3.6. Sistemi Multi Level Queue

All'interno del nostro sistema non utilizzeremo solamente un algoritmo di scheduling, ma implementeremo un sistema che possiede **_più code_**, ognuna ordinata con un algoritmo diverso.

Ipotizziamo di avere `3 code` ordinate con:
1. `RR(10)`
2. `RR(50)`
3. `FCFS`

Questo sistema introduce nuovi problemi:
- In quale coda inseriamo i nuovi processi?
- In quale coda inseriamo i processi al momento del'uscita dall'esecuzione
- Da quale delle code estraiamo il nuovo processo?

Per risolvere questi problemi assegnamo a ciascuna coda **una priorità**. 

La logica che seguiremo sarà quindi la seguente:
1. Lo _scheduler_ controlla la coda con priorità più alta non ancora controllata
   1. Se è **non vuota** ne estrae un processo secondo l'algoritmo associato
   2. Se è vuota procede con la coda successiva

Come sempre, introducendo il concetto di priorità, introduciamo anche la generazione di problemi di _starvation_ nelle code con priorità più bassa, tuttavia abbiamo anche visto come è possibile implementare processi di _aging_ che permettono di aumentare la priorità di un processo, in questo caso cambandone eventualmente la coda.

Nell'assegnare le priorità ai processi viene spesso fatta una distinzione tra due gruppi di processi: **interattivi** (_foreground_) e **batch** (_background_). 

Questi tipi di processi hanno tempi di risposta diversi, necessitando esigenze di priorità diverse: 
- I processi _interattivi_ sono caratterizzati da **CPU-Burst** brevi, supportando bene algoritmi `RR` anche con intervalli piccoli, poiché tendono a terminare prima della fine della loro finestra sulla **CPU**. 
- I processi _batch_ sono caratterizzati da **CPU-Burst** più grandi e prediligono algoritmi che effettuano meno scambi per unità di tempo.

Sarà uno dei compiti dello _schedulatore_ quello di assegnare ad ogni processo la coda che lo rende più efficiente.

All'interno dei sistemi però non è noto a priori se un nuovo processo è _batch_ o _interattivo_, perciò dobbiamo cercare di gestire dinamicamente queste code, implementando di fatto una **_Multi-Level Feedback Queue_**.

Il sistema operativo, alla creazione di un nuovo processo, lo inserisce nella **coda di livello zero**, ovvero in quella a priorità più alta.
Un processo inserito in questa coda attenderà in media $\overline{N_0}\cdot 10$ unità di tempo prima di andare in esecuzione.
<small>($\overline{N_i}$ indica il numero di processi per la coda $i$)</small>

Se il **CPU-Burst** del processo fosse minore dell'intervallo di `10ms` il processo può:
- Terminare: alché semplicemente si procede ad estrarne un altro
- Si sospende: il processo viene rimosso dalle code, per essere poi reinserito quando sarà di nuovo pronto

Se invece il **CPU-Burst** fosse maggiore, la _preemption_ lo sostituisce e lo inserisce nella **coda di livello 1**.
Lo _scheduler_ allora procede ad estrarre un nuovo processo **_sempre dalla coda 0_**.
Nel momento in cui questa fosse vuota (o per lo meno ci sia solamente il processo `DUMMY_0`), allora lo schedulatore procederà a estrarre dalla coda di livello 1.

Una stima del tempo medio di attesa di un processo nella coda 1 è $\overline{N_0} \cdot 10 + \overline{N_1} \cdot 50$. Ipotizzando che la coda di livello 0 sia vuota, attende "solamente" un tempo medio di $\overline{N_1} \cdot 50$.

Anche in questo caso il processo potrebbe andare in terminazione/sospensione, comportandosi analogamente a quando abbiamo fatto l'analisi per il livello 0.

Se il **CPU-Burst rimanente** fosse maggiore anche di `50ms` verrà nuovamente sostituito dalla _preemption_, venendo inserito stavolta nella coda di livello 2.
Qui il processo attenderà non solo che le altre due code debbano essere vuote, ma che anche i processi inseriti precedentemente nella coda terminino. Questo tempo, per quanto non calcolabile, è certamente finito.

Nello studio fino a questo punto abbiamo però ignorato il fatto che il nostro sistema sia _dinamico_. Ciò comporta che con il passare del tempo si hanno costantemente nuovi processi in entrata nella coda di livello 0. Ciò comporta una _starvation_ per le altre code a priorità minore, che si vedono sempre passare davanti i nuovi processi.

Nel caso del livello 1, l'algoritmo `RR` è di sua natura _preemptive_. Perciò alla creazione di un nuovo processo abbiamo già gli strumenti per effettuare _preemption_ sul processo in esecuzione per reinserirlo nella sua coda, stando però attenti a **mantenerlo allo stesso livello**. Non dobbiamo infatti né penalizzare né premiare un processo perché ne è stato casualmente creato un altro durante la sua esecuzione.

Se invece stiamo eseguendo un processo `P` proveniente dalla cosa `FCFS`, che ricordiamo **non implementa preemption**, maggiore è il suo **CPU-Burst**, maggiore sarà il numero di nuovi processi che vengono inseriti nella coda `RR(10)`, potendo arrivare persino a saturarla.

Per ovviare a questo problema, rendiamo la coda `FCFS` **_preemptive per l'inserimento dei nuovi processi_**. Anche in questo caso il processo rimarrà nella coda di livello 2, ma faremo attenzione ad inserirlo **_in testa_**. In questo modo, quando il sistema andrà a ripescare dalla coda a livello 2, il primo processo estratto sarà proprio quello al quale avevamo precedentemente revocato la **CPU**, senza penalizzarlo.

## 3.7. Schedulazione di sistemi in tempo reale

Gli algoritmi che abbiamo visto fin'ora, per quanto comunque funzionali, non si applicano bene a sistemi _embedded_ dove dobbiamo soddisfare anche altri requisiti.
I sistemi _embedded_ infatti sono caratterizzati da un sistema operativo multiprogrammato che **elabora parametri in tempo reale**.

I sistemi in tempo reale possono essere rappresentati come sistemi con **CPU**, **RAM**, memoria flash e, soprattutto, due classi principali di periferiche: **_attuatori_** (_output_) e **_sensori_** (_input_).

In questi sistemi i _sensori_ inviano _**periodicamente**_ al sistema operativo misurazioni di dati che è necessario elaborare per poter produrre output agli _attuatori_.

Il periodo con il quale campioniamo i dati **_dipende dall'oggetto che stiamo misurando_**. I periodi di campionamento possono variare dall'ordine dei microsecondi (sistemi di bilanciamento, braccia robotiche, ...) all'ordine dei secondi (misure di temperatura, pressione, ...).

Proprio questa periodicità negli _input_ è la prima caratteristica dei sistemi _embedded_, poiché di conseguenza anche **i processi verranno generati periodicamente**.

Possiamo quindi sviluppare il nostro sistema tenendo conto del fatto che gli _input_, e i relativi processi di elaborazione dell'_input_, sono _**periodici**_.

Dato un processo $i$ con periodo $t_i$, possiamo calcolare il prossimo inserimento in coda `pronti` in termini di quello precedente $r_j$:
$$
	r_{j+1} = r_j + t_i = r_0 + (j+1)\cdot t_i
$$

Possiamo inoltre defire il **CPU-Burst** come la somma di $k_i$ **CPU-Burst** diversi
$$
	C_i = \sum_{j=1}^{k_i}{C_i^{(j)}}
$$

È tuttavia necessario che il _turnaround_ del processo sia **_minore di_** $t_i$. In particolare vogliamo che termini entro una _deadline_ $d_i < t_{i+1}$. Questo avviene perché i risultati che il processo produrrà devono essere trasmessi agli _attuatori_ che dovranno quindi produrre cambiamenti opportunamente. Questo processo richiede del tempo, che identifichiamo proprio con l'intervallo tra $d_i$ e $t_{i+1}$.
Tuttavia, _**esclusivamente per gli studi teorici**_ possiamo considerare $d_i = t_{i+1}$.

Quello che abbiamo descritto è quindi un sistema composto da $N$ processi periodici, ovvero di **_un sistema periodico_**.

Possiamo quindi definire $T$ come il **_periodo del sistema nel suo complesso_**:
$$
	T = mcm(t_i), \quad \forall i \in N
$$

In un sistema _hard real time_ lo _scheduler_ **_deve garantire che tutte le deadline vengano rispettate_**, altrimenti si genera un **errore fatale**.
In sistemi _soft real time_ lo _scheduler_ **_può permettersi che qualche deadline possa non essere rispettata_** senza la generazioni di **errori fatali**, ma andando incontro a brevi errori temporanei.

Avendo questa conoscenza aggiuntiva rispetto ai sistemi aperti, andiamo ad analizzare alcuni algoritmi di schedulazione.

### 3.7.1. Rate Monototic

È un algoritmo a **priorità statica** $\quad p(i) \propto \frac{1}{t_i}$

Esiste di due tipi, sia _non preemptive_ che _preemptive_.

<div class="grid2">
<div class="">

Vediamo il caso _non preemptive_

<div class="flexbox" markdown="1">

|       | Periodo | CPU-Burst massimo |
| :---: | :-----: | :---------------: |
| `Pa`  |    2    |         1         |
| `Pb`  |    5    |         1         |

</div>

Possiamo subito calcolare che la priorità di `Pa` sarà **maggiore** di quella di `Pb`.
Inoltre abbiamo che $T = mcm(t_i) = 10$

Sulla destra vediamo un esempio dell'evoluzione di questo sistema, ipotizzando che entrambi i processi siano già inseriti in coda pronti all'istante `0`.

Notiamo come l'efficienza di questo sistema teorico sia al $70\%$.
Il $30\%$ di tempo _idle_ è tuttavia comodo per poter eseguire routine di sistema, gestire routine asincrone o fornire spazio di manovra per eventuali modifiche. Possiamo quindi ancora non considerarlo come un problema.

È banale notare che anche se avessimo dato la priorità proporzionalmente al periodo, questo esempio _teorico_ sarebbe comunque schedulabile, mentre quello _reale_ no. Infatti non sarebbe cambiato niente se non l'ordine di esecuzione nelle prime due unità. Il processo `Pb` sarebbe andato in esecuzione prima di `Pa`, che sarebbe entrato solamente solamente dopo la prima unità.
Questo avrebbe comportato che la terminazione della prima occorrenza di `Pa` sarebbe avvenuta in corsa con l'arrivo della seconda occorrenza, che nei sistemi reali **è un problema**.
</div>
<div class="">
<figure class="100">
<img class="100" src="./images/Gestione Processi/scheduling-example/RM-example.png">
<figcaption>

Le sezioni barrate sono le sezioni di **CPU-idle**.
</figcaption>
</figure>
</div>
</div>

Questo algoritmo è **ottimo** nella classe degli algoritmi a priorità statica.
In particolare, esiste òa proprietà che:
> È possibile schedulare degli eventi a priorità statica **_se e solo se_** possiamo farlo tramite `RM`.

### 3.7.2. Valutazione Esistenza dell'algoritmo

Fin'ora abbiamo quindi visto che abbiamo sistemi periodici di periodo $T$, composti da $N$ processi $i$ ognuno con periodo $t_i$ e **CPU-Burst** $C_i$.
Come facciamo però a valutare che è possibie schedulare tutti i processi senza generare overflow?

La formula è abbastanza semplice, infatti:
$$
\begin{CD}
	{
		\sum_{i=0}^N{\underbrace{n_i}_{\text{frequenza}} \cdot C_i} \le T
	}
	@>>>
	{
		\sum_{i=0}^N{\frac{T}{t_i} \cdot C_i} \le T
	}
	@>>>
	\boxed{
		\sum_{i=0}^N{C_i \over t_i} \le 1
	}
\end{CD}
$$

Definiamo come **_Fattore di utilizzo della CPU_**:
$$
	\quad U := \sum_{i=0}^N{C_i \over t_i}
$$

Prendendo l'esempio precedente, otteniamo che il fattore di utilizzo:
$$
	\quad U = \frac{1}{2} + \frac{1}{5} = \frac{7}{10} \le 1
$$

Operativamente nella realtà la formula è leggermente diversa, dato che non possiamo far coincidere la deadline con il periodo, rendendo la formula r9eale qualcosa del genere:
$$
	\quad U \le 1 - \alpha
$$

### 3.7.3. Earliest Deadline First

È un algoritmo _preemptive_ a **_priorità dinamica_**. Questa viene calcolata per ogni processo in base alla vicinanza alla sua deadline.

<div class="grid2">
<div class="">

Vediamo un esempio

<div class="flexbox" markdown="1">

|       | Periodo | CPU-Burst massimo |
| :---: | :-----: | :---------------: |
| `Pa`  |    4    |         2         |
| `Pb`  |   10    |         5         |

</div>

All'istante `0`, `p(Pa) > p(Pb)` in quanto `Pa` ha la deadline prima di `Pb`.

Anche all'istante `4`, sostituiamo il processo in esecuzione poiché `Pa = 4` mentre `Pb = 6`.
All'istante `8` invece abbiamo che `Pb = 2 < Pa = 4`, e quindi manteniamo `Pb` in esecuzione.

All'istante `16` abbiamo che `Pa = 4 == Pb = 4`. La scelta che compiamo è quella di **_mantenere il processo in esecuzione_**.

</div>
<div class="">
<img class="100" src="./images/Gestione Processi/scheduling-example/EDF-example.png">
</div>
</div>


# 4. Thread

Fin'ora abbiamo visto che un processo è sia un elemento che possiede risorse, sia un elemento al quale viene assegnata la **CPU**.


Ogni processo ha una spazio di indirizzamento distinto da quello degli altri processi. Questo dipende dalla tecnica di gestione della memoria adottata (pagine, segmenti, ...).

In un sistema di processi concorrenti le operazioni di scambio possono generare _overhead_ onerosi, comportando salvataggio e ripristino dello spazio di indirizzamento.
Questo accade anche alla creazione e alla terminazione di un processo.

La separazione degli spazi di indirizzamento be favorsice sì l'utilizzo nei casi di interazioni basate sullo scambio di messaggi, ma rende complesso l'utilizzo di frequenti interazioni basate su strutture dati condivise.

Proprio per ottenere una soluzione a questi problemi è stata introdotta la separazione dei processi in due elementi:
- **_Thread_** (_Processo Leggero_): l'elemento al quale viene assegnata la **CPU**
- **_Task_** (_Processo Pesante_): l'elemento che possiede le risorse

Un _thread_ rappresenta un **flusso di esecuzione** all'interno di un _task_. All'interno di un _task_ è possibile definire più _thread_, attraverso tecniche di _multithreading_, ciascuno dei quali **condivide le risorse del processo**, **_risiedendo nello stesso spazio di indirizzamento e accedendo agli stessi dati_**.

Non possedendo risorse indipendenti (se non lo _stack_), i _thread_ possono essere creati e distrutti più facilmente rispetto ai processi, così come ne è più semplice ed efficace il cambio di contesto.

A livello utente esistono delle librerie che permettono di strutturare un programma attraverso i _thread_ in esecuzione parallela, permettendo i passaggi tra _thread_ senza richiedere il supporto del sistema operativo.


Nei sistemi `Unix` originali i programmi erano caratterizzati dal possedere un solo _thread_.

Nei sistemi `Windows` e `Linux` moderni, il _kernel_ gestisce **_direttamente i thread_**, utilizzando al massimo le potenzialità di un sistema multiprocessore.


# 5. Processi in Unix

Il sistema `UNIX` suddivide le informaiozni tipicamente contenute nel `PCB` di un processo in due strutture dati distinte:
- **Process Structure**: contiene informazioni indispensabili per la gestione di un processo, anche se _swapped_. (non soggetta a _swap-out_)
- **User Structure**: contiene informazioni necessarie al sistema per la gestione del processo solamente quando esso è residente in memoria. (soggetta a _swap-out_)

<img class="" src="./images/Gestione Processi/UNIX-des-proc.png">

Il codice dei processi `UNIX` si dice **rientrante**, ovvero può essere condiviso da più processi. Per permettere ciò, il _kernel_ gestisce una struttura dati globale detta **_text table_** nella quale ogni elemento rappresenta il codice di un programma correttamente eseguito da uno o più processi.
In particolare, ogni elemento della _text table_ è detto **_text structure_** e contiene un puntatore all'area di memoria in cui è allocato il codice.
Se il processo fosse _swapped_ esso è un riferimento alla **memoria secondaria**.


<figure class="">
<img class="100" src="./images/Gestione Processi/UNIX-process-image.png">
<figcaption>

Una classificazione delle componenti in base alla **_visibilità_** (`user`/`kernel`) e alla **_possibilità di swapping_** (`resident`/`swappable`).
</figcaption>
</figure>