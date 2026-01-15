---
title: Il File System
---

# 1. Indice

- [1. Indice](#1-indice)
- [2. Il File System](#2-il-file-system)
	- [2.1. Struttura Logica](#21-struttura-logica)
		- [2.1.1. File](#211-file)
		- [2.1.2. Directory](#212-directory)
		- [2.1.3. Gestione del File System](#213-gestione-del-file-system)
	- [2.2. Accesso al File System](#22-accesso-al-file-system)
		- [2.2.1. Rappresentazione delle Directory](#221-rappresentazione-delle-directory)
		- [2.2.2. Accesso ai File](#222-accesso-ai-file)
		- [2.2.3. Metodi di Accesso](#223-metodi-di-accesso)
			- [2.2.3.1. Accesso Sequenzale](#2231-accesso-sequenzale)
			- [2.2.3.2. Accesso diretto](#2232-accesso-diretto)
			- [2.2.3.3. Accesso a Indice](#2233-accesso-a-indice)
	- [2.3. Organizzazione Fisica](#23-organizzazione-fisica)
		- [2.3.1. Allocazione contigua](#231-allocazione-contigua)
		- [2.3.2. Allocazione a Lista Concatenata](#232-allocazione-a-lista-concatenata)
		- [2.3.3. Allocazione a Indice](#233-allocazione-a-indice)
		- [2.3.4. Allocazione in UNIX](#234-allocazione-in-unix)
			- [2.3.4.1. Strutture Dati del Kernel per l'accesso a File](#2341-strutture-dati-del-kernel-per-laccesso-a-file)
	- [2.4. Dispositivo Virtuale](#24-dispositivo-virtuale)

# 2. Il File System

Il **_file system_** è quella parte del sistema operativo che fornisce i meccanismi necessari per l'accesso e l'archiviazione delle informazioni nella memoria persistente.

<figure class="">
<img class="80" src="./images/File System/fs-scheme.png">
<figcaption>

La struttura del _file system_ può essere rappresentata da un insieme di componenti organizate in vari livelli, come nell'immagine a destra.
</figcaption>
</figure>


## 2.1. Struttura Logica

Il livello più alto rappresenta la **Struttura Logica**, che presenta all'utente una visione astratta delle informazioni presenti sul disco.
Questo livello prescinde dalle caratteristiche del dispositivo e dalle tecniche di allocazione e accesso alle informazioni adottate dal sistema.

Realizza i concetti astratti di:
- **_File_**: unità logica di memorizzazione
- **_Directory_**: insieme di _file_ e _directory_ (_cartella_)
- **_Partizione_**: insieme di _file_ associato ad un particolare dispositivo fisico o a una sua porzione

Le caratteristiche di questi concetti sono del tutto **indipendenti dalla natura e dal tipo di dispositivo utilizzato**.

I processi vedono quindi la memoria secondaria attraverso questa _struttura astratta_.

### 2.1.1. File

Un file è:
> Un insieme di informazioni rappresetnate mediante insieme di record logici (bit, byte, linee, record, ...)

Ogni file è caratterizzato da un insieme di **attributi**, ad esempio:
- **Tipo**: stabilisce l'appartenenza del file ad una classe (eseguibili, batch, testo, ...)
- **Indirizzo**: puntatore/i a memoria secondaria
- **Dimensione**: numero di record contenuti nel file
- **Data e ora**: vi sono sia di creazione che di ultima modifica

Nei sistemi multiutente troviamo anche:
- **Utente proprietario**: in `UNIX` sono l'`UID` (_User-ID_) e il `GID` (_Group-ID_)
- **Protezione**: diritti di accesso al file per gli utenti del sistema


### 2.1.2. Directory

Una _directory_:
> È un astrazione che cosente di raggruppare più file.
> Una _directory_ può contenere più file, così come altre _directory_.


Una _directory_ può quindi essere visto come un operatore di composizione di file e _directory_.

Nei sistemi operativi moderni la soluzione più comune è rappresentata da _file system_ **strutturati ad albero**, dove ogni _file_ appartiene in una sola _directory_, e ogni _directory_, tranne la `root`, appartiene ad un altra _directory_.

In alcuni sistemi, come in `UNIX` e `Linux`, viene permesso a più _directory_ di **_condividere lo stesso file_** attraverso il _linking_.
Questa modifica cambia la struttura logica del _file system_ rendendolo un **_grafo diretto aciclico_**.

<img class="" src="./images/File System/fs-dag.png">


### 2.1.3. Gestione del File System

Il _sistema operativo_ realizza anche i meccanismi per la gestione del _file system_ attraverso delle specifiche _system call_.

Le operazioni fondamentali per la gestione del file system sono:
- **Creazione e cancellazione di directory**: modificano la struttura logica del file system, aggiungendo/eliminando rami al grafo che rappresenta il dile system
- **Aggiunta/Cancellazione di file**
- **Listing**: permette di ispezionare il contenuto di uno o più _directory_
- **Attraversamento della directory**: permette la _navigazione_ attraverso la struttura logica del _file system_.

Queste funzionalità sono disponibili per gli utenti sotto forma di comandi di _shell_.

## 2.2. Accesso al File System

Il livello sottostante si chiama **Accesso** che definisce e realizza i meccanismi mediante i quali è possibile eseguire operazioni sul contenuto dei file, generalmente _lettura_ o _scrittura_.

In questo livello i _file_ sono visti come un insieme di **record logici**, ovvero di unità di trasferimenti tra un processo e il file.
Il _record logico_ è quindi caratterizzato da alcune proprietà come il _tipo_ e la _dimensione_, in particolare in `UNIX` un _record logico_ è **_1 Byte_**.

Queste operazioni elementari di accesso ai file sono messe a disposizione dei processi attraverso opportune _system calls_, subordinatamente al soddisfacimento delle **politiche di protezione**.
Queste stabiliscono _chi_ e _in che modo_ è abilitato ad accedere al file.

Il livello utilizza quindi delle _strutture dati_ che rappresnetano concretaente un file, chiamate **_descrittori di file_**. I _descrittori di file_ devono essere memorizzati in modo persistente mediante apposite strutture in memoria secondaria.

In `UNIX` questi descrittori sono gli `i-node`, conservati in `i-list` e indirizzati da `i-number`.

### 2.2.1. Rappresentazione delle Directory

Poiché ogni file appartiene ad una directory, ogni directory mantiene il collegamento con i descrittori dei file contenuti in esso.

Ad esempio, in _Windows_ si segue un approccio distribuito, dove la directory è una **struttura dati di tipo taellare che contiene i descritori dei file**, come nell'immagine di seguito.

<img class="60" src="./images/File System/windows-access-scheme.png">

In `UNIX` invece si segue un **approccio centralizzato**, dove la tabella che rappresenta la directory contiene i **riferimenti agli `i-node`** (`i-number`), memorizzati nella `i-list` che si trovano in una struttura separata.

<img class="60" src="./images/File System/UNIX-access-scheme.png">

### 2.2.2. Accesso ai File

Abbiamo detto che è compito di questo livello implementare e mettere a disposizione l'accesso _on-line_ ai file, che sia:
- **Lettura**: leggere il contenuto di _record logici_ dal file
- **Scrittura**: si divide in:
  - **Scrittura Pura**: modifica il contenuto dei _record_ all'interno di un file, mantenendo inalterato il numero di _record_ del file
  - **Append**: aggiunge ulteriori _record_ al file


Ogni operazione richiederebbe quindi la localizzazione di informazioni su disco, quindi:
- Indirizzi dei record logici a cui accedere
- Gli altri attributi del ifle
- I record logici

Introducendo un **_notevole overhead_**.

Per rendere efficienti queste operazioni i _file system_ mantengono in memoria una struttura che registra i file attualmente in uso, detta **tabella dei file aperti**.
Per ogni _file aperto_ sono conservate alcune informazioni, quali `puntatore_a_file`, `posizione_su_disco`, ...

Si fa inoltre **Memory Mapping** dei file aperti, ovvero questi file vengono temporaneamente copiati (nella loro interezza o solo alcune porzioni di esse) nella **RAM**, così da consentirne un _accesso più rapido_.

È quindi necessario introdurre due nuove operazioni:
- **Apertura**: permette di introdurre un nuovo elemento nela _tabella dei file aperti_ e eventuale _memory mapping_ del file
- **Chiusura**: salva il file in memoria secondara e elimina l'elemento corrispondente dalla tabella dei file aperti. Possono anche essere effettuate ulteriori operazioni che dipendono dai singoli sistemi operativi.

### 2.2.3. Metodi di Accesso

L'accesso a un file più avvenire secondo varie modalità:
- **Accesso Sequenziale**
- **Accesso Diretto**
- **Accesso a Indice**

Poiche ogni metodo di accesso deve essere **indipendente dal tipo di dispositivo utilizzato e dalla tecnica di allocazione dei blocchi in memoria**, presuppone implicitamente implicitamente un organizzazione interna del file, che viene visto come una **_sequenza di record logici numerati_** $\{R_1, ..., R_i, ..., R_N\}$.

#### 2.2.3.1. Accesso Sequenzale

Nell'accesso sequenzale per accedere ad un particolare record $R_i$ è necessario accedere prima agli $(i-1)$ _record_ che lo precedono nella sequenza.

Le primitive che permettono l'accesso seqeunziale seguono il modello generale:
```c
/**
* @brief permette di leggere il prossimo record
* @param f nome del file
* @param V buffer dove copiare l'elemento letto
*/
readnext(f, &V);

/**
* @brief permette di scrivere sul prossimo record
* @param f nome del file
* @param V buffer da dove recuperare l'elemento da scrivere
*/
writenext(f, &V);
```

Ogni operazione di accesso posizione il puntatore al file sull'elemento successivo a quello letto/scritto.

Questo tipo di accesso è **_quella che abbiamo nei sistemi_** `UNIX`.

#### 2.2.3.2. Accesso diretto

Nell'accesso diretto è possibile accedere direttamente ad un particolare record logico $R_i$ mediante il suo indice.

Le primitive che permettono l'accesso diretto seguono il modello generale:
```c
/**
* @brief permette di leggere il prossimo record
* @param f nome del file
* @param i indice del record da leggere
* @param V buffer dove copiare l'elemento letto
*/
readd(f, i, &V);

/**
* @brief permette di scrivere sul prossimo record
* @param f nome del file
* @param i indice del record sul quale scrivere
* @param V buffer da dove recuperare l'elemento da scrivere
*/
writed(f, i, &V);
```

#### 2.2.3.3. Accesso a Indice

Ad ogni file viene associata una struttura dati contenente l'indice delle informazioni contenute nel file.

Le operazioni di accesso seguono quindi il modello generale:
```c
/**
* @brief permette di leggere il prossimo record
* @param f nome del file
* @param key chiave che identifica l'elemento da leggere
* @param V buffer dove copiare l'elemento letto
*/
readk(f, key, &V);

/**
* @brief permette di scrivere sul prossimo record
* @param f nome del file
* @param key chiave che identifica l'elemento da scrivere
* @param V buffer da dove recuperare l'elemento da scrivere
*/
writek(f, key, &V);
```

QUesto tipo di accesso presenta due svantaggi:
- **Doppio accesso**: per accedere al contenuto di un record, devo prima accedere al record del _file indice_ dove è contenuta il riferimento al record logico
- **Difficile scalabilità**: per aggiungere un nuovo record è necessario aggiungerne due, raddoppiando la velocità di saturazione della memoria.


## 2.3. Organizzazione Fisica

Il terzo strato è il livello di **Organizzazione Fisica**, che ha come compito primario quello di allocare i record logici di ogni file nell'unità di memorizzazione secondaria.

In questo livello lo spazio disponibile per l'allocazione sul disco viene visto come un insieme di **_blocchi fisici_**.

Il **blocco fisico** è l'unità di allocazione e di trasferimento delle informazioni sul dispositivo. Ad ogni blocco è associata quindi una posizione particolare sulla superficie del disco. Un bloccco ha dimensione costante $D_b$, generalmente questa è **molto maggiore** della dimensione di un record logico $D_r$

In un singolo blocco possiamo contenere $N_b = \frac{D_b}{D_r}$ record logici.

Questo livello realizza quindi i **metodi di allocazione**, che stabiliscono il collegamento tra ogni file e l'insieme di blocchi fisici nel quale esso è allocato.

Non tutta la memoria è utilizzata per l'allocazione dei file, vedremo in seguito che una parte è riservata all'allocazione di alcune strutture dati per la descrizione della struttura logicaa del _file system_ e per il supporto della gestione degli accessi ai file da parte dei processi.

Dobbiamo quindi utilizzare delle tecniche di allocazione che stabiliscano uan corrispondenza tra i _record logici_ contenuti in ogni file e l'insieme dei blocchi nei quali sono effettivamente memorizzati.

### 2.3.1. Allocazione contigua

Ogni file è mappato su un insieme di blocchi **fisicamente contigui**.

<div class="grid2">
<div class="">

Per fare ciò è innanzitutto necessario calcolare il numero di blocchi necessari per salvare un file, e successivamente si procede a cercare una partizione di blocchi libero abbastanza grande da contenere il file.

Questo tipo di allocazione ha diversi vantaggi:
- **Ricerca di un blocco semplice**: un dato blocco si trova in $B + \Big\lfloor\frac{i}{N_b}\Big\rfloor$, dove $i$ è l'`IO pointer` di un dato record, e $B$ è l'indirizzo del primo blocco
- **Possibilità di accesso sequenzale e diretto** allo stesso costo

Tuttavia ha anche molti svantaggi:
- **Frammentazione Esterna**: man mano che si riempie il disco si rimangono zone contigue sempre più piccole, rendendo necessario il _compattamento_
- **Costo della ricerca dello spazio libero**: è necessario analizzare tutto il disco finché non si trova uno spazio sufficientemente grande per conservare il file
- **Crescita delle dimensioni del file**: qualora il file aumentasse di dimensione potrebbe essere necessario _rilocare l'intero file_ se non fossero disponibili ulteriori blocchi contigui liberi.

</div>
<div class="">
<img class="80" src="./images/File System/continuous-allocation.png">
</div>
</div>

### 2.3.2. Allocazione a Lista Concatenata

QUest atecnica memorizza ogni file in un insieme di blocchi non contigui organizzati in una lista concatenata

<div class="grid2">
<div class="">

L'organizzazione è ancora sequenziale, ma i blocchi successivi non devono necessariamente essere vicini.

Questa tecnica ha diversi vantaggi:
- **Non soffre di frammentazione esterna**
- **Minor costo di allocazione**
- **Accesso sequenzale a basso costo**: dato il primo blocco abbiamo il puntatore al successivo

Vi sono ancora diversi svantaggi:
- **Broken Links**: se un blocco vine danneggiato, non solo perdiamo le informazioni che contiene ma anche le informazioni sul successivo, rendendo irraggiungibile il resto del file
- **Overhead spaziale**: dover salvare i puntatori diminuisce lo spazio effettivo utilizzabile
- **Accesso diretto oneroso**: in questo caso, per accedere all'$i$-esimo blocco sono necessari $\Big\lfloor\frac{i}{N_b}\Big\rfloor$ accessi al disco
- **Costo della ricerca di un blocco**
</div>
<div class="">
<img class="80" src="./images/File System/linked-list-allocation.png">
</div>
</div>

Per arginare il problema dei _broken links_ è possibile realizzare una _double-linked-list_, andando a sacrificare un altra porzione del blocco per salvare il secondo puntatore.

<img class="" src="./images/File System/double-linked-list-allocation.png">


Alcuni sistemi operativi, ad esempio `Windows`, affronato il problema affiancando ad una allocazione basata su una _linked-list_ introducendo una struttura dati nella quale viene descritta la **mappa di allocazione di tutti i blocchi**, detta **_File Allocation Table_** `FAT` memorizzata in una posizione predefinita.

<div class="grid2">
<div class="">

Essa contiene un elemento per ogni blocco del dispositivo il cui valore indica:
- se il blocco è libero
- Se è occupato l'indice dell'elemento della tabella che rappresenta il blocco successivo nella lista

In questo modo, anche in perdita di concatenamento è possibile effettuare il recupero del puntatore perso accedendo alla `FAT`.

La `FAT` può anche essere copiata in memoria centrale o in una cache, così da velocizzare notevolmente l'accesso diretto.
</div>
<div class="">
<img class="80" src="./images/File System/FAT-allocation.png">
</div>
</div>

### 2.3.3. Allocazione a Indice

Questo tipo di allocazione si basa ancora sull'utilizzo di blocchi non contigui per l'allocazione di un file, ma in questo caso ad ogni file è associato un blocco **indice** in cui sono contenuti _tutti gli indirizzi dei blocchi su cui è allocato il file_.

<div class="grid2">
<div class="">

Questo tipo di allocazione ha gli stessi vantaggi dell'allocazione tramite _linked-list_ introducendo però anche:
- **Possibilità di accesso diretto**
- **Maggiore velocità di accesso**

Il grande svantaggio di questa tecnica è la **_non scalabilità_**, infatti se il file crescesse tanto da necessitare più blocchi di quelli indicizzabili da un unico blocco avremmo un problema.

In particolare dato un disco di capacità $C$, otteniamo quanti bit sono necessari per indicizzare tutti i blocchi di dimensione $D_b$ ($I = \log{(\frac{C}{D_b})}$ indici).
A questo punto dobbiamo capire quanti indici è possibile salvare in un singolo blocco, ovvero $\frac{D_b}{I}$ indici.
A questo punto si moltiplica ogni indice per la dimensione di un blocco:
$$
	D_M = \frac{D_b^2}{I} = \frac{D_b^2}{\log{(\frac{C}{D_b})}}
$$
</div>
<div class="">
<img class="80" src="./images/File System/index-allocation.png">
</div>
</div>


### 2.3.4. Allocazione in UNIX

Il metodo di allocazione utilizzato in `UNIX` è a **indicizzazione su più livelli di indirizzamento** basata su un _grafo aciclico diretto_.


<div class="grid2">
<div class="">

In `UNIX` **_tutto è un file_**.

I file si dividono in tre categorie:
- **Ordinari**: sono i veri e propri file
- **Directory**: sono dei file che rappresentano delle raccolte di altri _file_
- **Speciali**: rappresentano i dispositivi (ad esempio quelle in `/dev/`)

Ad ogni file possono essere associati **_uno o più nomi simbolici_** detti _link/alias_, che si **_riferiscono ad uno ed un solo descrittore_**, detto `i-node`.

I singoli `i-node` sono identificati univocamente dagli `i-number` e sono contenuti nella `i-list`.

</div>
<div class="">
<img class="" src="./images/File System/UNIX-allocation.png">
</div>
</div>

Il metodo di allocazione utilizzato in unix è quindi ad **_indice a più livelli_**.
Questa tecnica formatta il disco in blocchi fisici di dimensione costante e prefissata, dividendo la superficie del disco in quattro regioni.

<div class="grid2">
<div class="">

La prima regione, detta `BootBlock`, occupa un blocco fisico, allocato ad un **indirizzo prefissato**, e contiene il programma di inizializzazione del sistema da eseguire nella fase di _bootstrap_.

La regione `SuperBlock`, occupa anch'essa la dimensione di un blocco. e descrive l'allocazione del _file system_. In particolare i limiti delle quattro regioni, il puntatore alla lista dei blocchi liberi e il puntatore alla lista degli `i-node` liberi.

L'estensione dell'area `DataBlocks` è tipicamente molto maggiore delle altre, poiché rappresneta la zona effettivamente diposnibile per la memorizzazione dei file.
I blocchi liberi di questa zono sono organizzati in una lista collegata il cui indirizzo è ocntenuto nel `SuperBlock`.

Infine la `i-List` contiene il vettore di tutti gli `i-node`.
</div>
<div class="">
<img class="80" src="./images/File System/UNIX-physical-organization.png">
</div>
</div>

Un `i-node` quindi è formato da:
- **Tipo**: indica se il file è _ordinario_, _directory_ o _speciale_ (dispositivi)
- **Proprietario e Gruppo**: indicano chi è l'utente proprietario del file e qual è il gruppo di appartenenza del proprietario
- **Dimensione**: indica in numero di blocchi occupati dal file in memoria di massa
- **Dara**: indica la data dell'ultima modifica effettuata sul file
- **Link**: il numero di nomi che riferiscono il file (_hard-link_). Se si ha un solo nome, questo valore è `1`
- **Bit di Protezione**: è l'insieme dei `12bit` che esprime la politica di protezione da applicare sul file. contiene i 9 bit dei premessi, `SUID`, `GUID` e `STIcky bit`.
- **Vettore di Indirizzamento**: è costituito da un insieme di indirizzi che consente l'indirizzamento dei blocchi di dati sui quali è allocato il file (tipicamente sono `13 - 15`).

Nell'ipotesi che il vettore contenga **13 indirizzi** (così come nelle prime versioni del sistema operativo), e che un blocco abbia $D_b = 512 B$ con indirizzi su `32bit`, ovvero un blocco contiene $128$ indirizzi.
I primi 10 blocchi di dati sono accessibili direttamente dai primi 10 puntatori $(10 \cdot 512B = 5KB)$.
Altri 128 blocchi di dati sono accessibili con **indirizzazione singola** mediante puntatore `11` che punta ad un _blocco indice_ $(128 \cdot 512B = 64KB)$
Altri $128^2$ blocchi di dati sono accessibili con **indirizzazione doppia** mediante il puntatore `12` che punta ad un _blocco indice che punta a blocchi indici_. $(128 \cdot 128 \cdot 512B = 8MB)$
Il puntatore `13` tramite **indicazzione tripla** permette creazioen di file di dimnesione fino ad $1GB$.
Il questo modo la dimensione massima di un fiel è nell'ordine del **_Giga Byte_**.


Le _derectory_ sono rappresentati da un file, in cui contenuto ne descrive la struttura logica. In particolare ogni record coniene la coppia di informazioni `<nome_relativo, i-number>` associate al file, come nell'immagine di seguito.

<img class="" src="./images/File System/UNIX-directory-realization.png">

#### 2.3.4.1. Strutture Dati del Kernel per l'accesso a File

Ogni file è quindi organizzato come una sequenza di **byte** detta _stream_.

Il metodo di accesso adottato nel sistema `UNIX` è quello sequenzale, ad ogni file aperto è quindi associato un `I/O Pointer` che indica implicitamente il prossimo elemento a cui accedere. Ogni accesso in memoria provoca un avanzamento dell'`I/O pointer`.
Ogni file termina quindo con la sequenza `EOF` (_End-Of-File_).

Per comprendere al meglio i meccanismi di accesso analizziamo le caratteristiche delle strutture dati del _kernel_.

A livello globale il _kernel_ mantiene una **_Tabella dei File Aperti del Sistema_** (`TFAS`) allocata nella _user structure del processo_. Questa tavela contiene un elemento per **_ogni file aperto nel sistema_**, individuato da un indice intero detto **_file descriptor_** `fd`. Questo elemento non è altro che un puntatore all'`i-node` del file salvato nella _**Tabella dei File Attivi**_, copiata dalla memoria di massa.

<img class="" src="./images/File System/kernel-data-structure-for-file-access.png">


Più precisamento viene allocato un elemento nella `TFAS` per ogni **operaizone di apertura di file**. Ciò implica che se due processi distinti aprono lo stesso file, avremo **_due elementi distinti nella `TFAS`_**, ma comunque uno solo nella _tabella dei file attivi_
Se invece un processo genera dei figli (ad esempio con una `fork()`) questi **_condivideranno con il padre i vari file descriptor_** e di conseguenza anche gli `I/O Pointer`.

<img class="" src="./images/File System/father-son-pointer-sharing.png">


Di default sono aperti automaticamente i file speciali di _standard input_ `STD_IN`, _standard output_ `STD_OUT` e _standard error_ `STD_ERR`.

Per ulteriori informazioni su come manipolare i file da codice è possibile [consultare gli appunti di laboratorio](./laboratorio/Filesystem.md).

## 2.4. Dispositivo Virtuale
