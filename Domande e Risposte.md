# Domande e Risposte

## Indice

- [Domande e Risposte](#domande-e-risposte)
	- [Indice](#indice)
	- [Note per lo studio](#note-per-lo-studio)
- [LABORATORIO](#laboratorio)
	- [1. Introduzione ai comandi UNIX](#1-introduzione-ai-comandi-unix)
		- [Domanda 1.1 (answered)](#domanda-11-answered)
	- [2. Utenti e Gruppi](#2-utenti-e-gruppi)
		- [Domanda 2.1 (answered)](#domanda-21-answered)
	- [3. Processi](#3-processi)
		- [Domanda 3.1 (answered)](#domanda-31-answered)
	- [4. Thread](#4-thread)
	- [5. Filesystem](#5-filesystem)
- [TEORIA](#teoria)
	- [6. Concetti Introduttivi](#6-concetti-introduttivi)
		- [Domanda 6.1 (answered)](#domanda-61-answered)
	- [7. Classificazione delle Architetture](#7-classificazione-delle-architetture)
		- [Domanda 7.1 (answered)](#domanda-71-answered)
	- [8. Gestione Processi](#8-gestione-processi)
		- [Domanda 8.1 (answered)](#domanda-81-answered)
		- [Domanda 8.2 (answered)](#domanda-82-answered)
	- [9. Sincronizzazione dei Processi](#9-sincronizzazione-dei-processi)
		- [Domanda 9.1 (answered)](#domanda-91-answered)
	- [10. Gestione della Memoria](#10-gestione-della-memoria)
		- [Domanda 10.1 (answered)](#domanda-101-answered)
		- [Domanda 10.2 (answered)](#domanda-102-answered)
	- [11. Gestione delle Periferiche (I/O)](#11-gestione-delle-periferiche-io)
		- [Domanda 11.1 (answered)](#domanda-111-answered)
	- [12. Il File System](#12-il-file-system)
		- [Domanda 12.1 (answered)](#domanda-121-answered)
	- [13. Protezione e Sicurezza](#13-protezione-e-sicurezza)
		- [Domanda 13.1 (answered)](#domanda-131-answered)


## Note per lo studio
- Ogni risposta include riferimenti specifici ai file sorgente ai quali si riferisce
- Gli esempi pratici sono forniti per aiutare a consolidare i concetti teorici, ma potrebbero non essere perfettamente accurati
- Si consiglia di rispondere prima autonomamente e poi confrontare con la risposta fornita

---
<div class="stop"></div>

---

# LABORATORIO

## 1. Introduzione ai comandi UNIX

### Domanda 1.1 (answered)

**Domanda**: Spiega il funzionamento del comando `find` in UNIX, descrivendo la sintassi generale, la differenza tra test e azioni, e fornisci esempi pratici di utilizzo con opzioni come `-name`, `-type`, `-size` e `-exec`. Confronta inoltre `find` con `locate`, evidenziando vantaggi e svantaggi di entrambi gli approcci.

**Risposta:**

Il comando `find` è uno strumento fondamentale per la ricerca di file e cartelle all'interno del filesystem UNIX. A differenza di altre utility di ricerca, `find` offre una **sintassi complessa ma potente** che permette di effettuare ricerche basate su molteplici proprietà dei file.

> "Il comando `find` permette di trovare file e cartelle all'interno del sistema. Questo comando permette di effettuare la ricerca combinando dei test sulle proprietà dei file, che siano _filename_, _file type_, _owner_, _permessi_, _timestamp_,..."
>
> *Fonte: [Introduzione ai comandi UNIX](./laboratorio/Introduzione%20ai%20comandi%20UNIX#41-find)*

_**Sintassi Generale**_

La sintassi del comando è la seguente:

```bash
find [path1...] [espressione]
```

Il parametro `path` specifica i percorsi in cui effettuare la ricerca. È importante sottolineare che **la ricerca avviene soltanto nei percorsi specificati**. L'espressione descrive i criteri di ricerca e le eventuali azioni da eseguire sui file trovati.

_**Struttura delle Espressioni**_

Le espressioni di `find` sono composte da quattro tipi di elementi:

1. **Test**: valutano una proprietà del file e ritornano `true` o `false`
2. **Azioni**: operazioni da effettuare sui file trovati, ritornano `true` se hanno successo
3. **Opzioni Globali**: influenzano l'esecuzione di test o azioni, ritornano sempre `true`
4. **Opzioni Posizionali**: influenzano solo le azioni o i test che seguono, ritornano sempre `true`

> "Gli elementi di una espressioni sono collegati da **operatori**, ad esempio `-o` indica `OR` e `-a` indica `AND`. In caso non siano specificati operatori, l'utilizzo dell'operatore `AND` è **implicito per collegare due espressioni**."
>
> *Fonte: [Introduzione ai comandi UNIX](./laboratorio/Introduzione%20ai%20comandi%20UNIX#41-find)*

_**Test Principali**_

Dal file [Introduzione ai comandi UNIX](./laboratorio/Introduzione%20ai%20comandi%20UNIX#41-find):

- **`-name pattern`**: ricerca basata sul nome del file. Il pattern può includere metacaratteri e deve essere scritto tra apici per evitarne l'espansione
- **`-type [dfl]`**: ricerca basata sul tipo di file (`d` per directory, `f` per file regolari, `l` per symbolic link)
- **`-size [+-]n[ckMG]`**: ricerca basata sulla dimensione. Il prefisso `[+-]` indica se cercare file maggiori o minori della dimensione specificata, mentre `[ckMG]` rappresenta l'unità di misura (byte, kilobyte, megabyte, gigabyte)
- **`-user utente`**: cerca file appartenenti a un utente specifico (per UID o username)
- **`-group gruppo`**: cerca file appartenenti a un gruppo specifico (per GID o groupname)
- **`-perm [-/]mode`**: ricerca basata sui permessi del file

_**Azioni sui File Trovati**_

Le azioni più comuni sono:

- **`-delete`**: elimina i file trovati, ritorna `true` in caso di successo
- **`-exec command \;`**: esegue un comando sui file trovati. Gli argomenti dopo `command` sono considerati parte del comando fino al carattere `\;`. La stringa `{}` rappresenta il nome del file corrente

> "È importante sottolineare che questi comandi vanno inseriti **dopo i test**, altrimenti avranno effetto su tutti i file"
>
> *Fonte: [Introduzione ai comandi UNIX](./laboratorio/Introduzione%20ai%20comandi%20UNIX#41-find)*

_**Esempi Pratici**_

```bash
# Cerca tutti i file .txt nella directory corrente
find . -name "*.txt"

# Cerca tutte le directory nella home dell'utente
find ~ -type d

# Cerca file più grandi di 10MB con permessi di scrittura per il proprietario
find . -size +10M -perm -u=w

# Cerca file dell'utente pippo e crea una lista in list.txt
find . -size +10M -perm -u=w -user pippo -exec echo {} >> list.txt \;
```

_**Confronto: `find` vs `locate`**_

Il comando `locate` offre un approccio alternativo alla ricerca di file:

```bash
locate [options] file1 file2 ....
```

> "`locate` permette di ricercare un file specificato sfruttando un database aggiornato periodicamente dal sistema. È possibile forzare l'aggiornamento del database tramite comando `sudo updatedb`."
>
> *Fonte: [Introduzione ai comandi UNIX](./laboratorio/Introduzione%20ai%20comandi%20UNIX#42-locate)*

**Vantaggi di `locate`:**
- **Velocità**: molto più veloce di `find` poiché interroga un database pre-costruito
- **Semplicità**: sintassi più semplice e intuitiva per ricerche di base

**Svantaggi di `locate`:**
- **Aggiornamento**: dipende dall'aggiornamento periodico del database, quindi potrebbe non trovare file creati recentemente o mostrare file già eliminati
- **Funzionalità limitate**: non supporta test complessi e azioni come `find`
- **Non installato di default**: a differenza di `find`, `locate` non è sempre pre-installato sui sistemi UNIX

**Vantaggi di `find`:**
- **Risultati aggiornati**: effettua la ricerca in tempo reale sul filesystem
- **Flessibilità**: supporta test complessi, combinazioni logiche e azioni sui file trovati
- **Disponibilità**: installato di default su tutti i sistemi UNIX/Linux

**Svantaggi di `find`:**
- **Prestazioni**: più lento, specialmente su filesystem di grandi dimensioni
- **Complessità**: sintassi più complessa che richiede maggiore conoscenza

**Approfondimenti:**

La scelta tra `find` e `locate` dipende dalle esigenze specifiche: `locate` è ideale per ricerche rapide di file per nome, mentre `find` è la scelta migliore quando servono criteri di ricerca complessi, quando è necessaria la certezza di risultati aggiornati, o quando si devono eseguire azioni sui file trovati. L'importante ricordare che `find` è più appropriato per script automatizzati e operazioni critiche dove l'accuratezza è fondamentale.

---
<div class="stop"></div>

---

## 2. Utenti e Gruppi

### Domanda 2.1 (answered)

**Domanda**: Spiega il sistema di permessi di accesso al filesystem in UNIX, descrivendo le tre classi di utenti (Owner, Group Owner, Others) e i relativi permessi (r, w, x) per file e directory. Illustra inoltre il funzionamento dei permessi speciali SUID e SGID, spiegando come vengono rappresentati sia in formato simbolico che ottale, e come influenzano i privilegi di un processo attraverso gli identificatori EUID e EGID.

**Risposta:**

Il sistema di permessi di accesso al filesystem UNIX rappresenta uno dei meccanismi fondamentali per la protezione e la sicurezza dei dati. Permette di controllare in modo granulare chi può accedere a quali risorse e con quali modalità.

_**Struttura del Sistema dei Permessi**_

> "I file all'interno del filesystem sono presenti numerosi file sensibili, protetti dall'accesso degli utenti casuali. Il meccanismo dei permessi ne gestisce l'accesso."
>
> *Fonte: [Utenti e Gruppi](./laboratorio/Utenti%20e%20Gruppi#3-permessi-di-accesso-al-filesystem)*

Per ogni file o directory nel filesystem vengono definiti due attributi fondamentali:
- **Owner**: l'utente proprietario del file
- **Group Owner**: il gruppo proprietario del file

Questo porta alla definizione di **tre classi di utenti** per ciascun file:

1. **Owner**: il proprietario del file
2. **Group Owners**: gli utenti appartenenti al gruppo proprietario
3. **Others**: tutti gli altri utenti del sistema

_**Permessi per i File**_

Per i file regolari, a ciascuna classe vengono applicati tre tipi di permessi:

- **`r` (Read)**: permette di leggere il contenuto del file
- **`w` (Write)**: permette di modificare il contenuto del file. È importante notare che **non permette di cancellare il file** (questa operazione dipende dai permessi sulla directory che lo contiene)
- **`x` (eXecute)**: permette di eseguire il file come programma

> "Quando un utente prova ad utilizzare un file, vengono applicati i permessi specifici della categoria alla quale egli appartiene."
>
> *Fonte: [Utenti e Gruppi](./laboratorio/Utenti%20e%20Gruppi#3-permessi-di-accesso-al-filesystem)*

_**Permessi per le Directory**_

Per le directory, gli stessi permessi assumono significati diversi:

- **`r` (Read)**: permette di leggere il contenuto della cartella (elenco dei file). Se negato, non è possibile utilizzare il comando `ls` sulla directory
- **`w` (Write)**: permette di modificare il contenuto della cartella (aggiunta, rimozione e rinomina di file)
- **`x` (eXecute)**: permette di attraversare la cartella. Se negato, non è possibile utilizzare `cd` per entrare nella directory

_**Visualizzazione e Rappresentazione dei Permessi**_

Utilizzando il comando `ls -l`, i permessi vengono visualizzati nel seguente formato:

```
┌─── Tipo (d=directory, -=file, l=link)
│
│ ┌────────── Permessi Owner
│ │  ┌─────── Permessi Group Owner
│ │  │  ┌──── Permessi Others
│ │  │  │
│┌┴┐┌┴┐┌┴┐
drwxr-xr-x  2  owner_name  group_owner_name  512  2008-11-04 16:58  nome
```

**Rappresentazione Ottale:**

> "Le triple dei permessi sono codificate in cifre in base 8, ottenute sommando: 1 se è permessa l'esecuzione, 2 se è permessa la scrittura, 4 se è permessa la lettura"
>
> *Fonte: [Utenti e Gruppi](./laboratorio/Utenti%20e%20Gruppi#3-permessi-di-accesso-al-filesystem)*

Esempi:
- `777`: tutti i permessi a tutti (rwxrwxrwx)
- `750`: tutti i diritti all'owner, lettura ed esecuzione al group owner, niente agli others (rwxr-x---)
- `644`: lettura e scrittura all'owner, solo lettura a group owner e others (rw-r--r--)

_**Modifica dei Permessi**_

Il comando `chmod` permette di modificare i permessi, ed è utilizzabile dal `root` o dall'owner del file:

```bash
chmod +x file.txt           # aggiunge permessi di esecuzione a TUTTI
chmod u-x file.txt          # rimuove permessi di esecuzione all'OWNER
chmod g-x file.txt          # rimuove permessi di esecuzione al GROUP OWNER
chmod o=x file.txt          # assegna SOLO esecuzione agli OTHERS
chmod 755 file.txt          # imposta i permessi in formato ottale
chmod -R 755 directory/     # applica ricorsivamente
```

_**Permessi Speciali: SUID e SGID**_

Oltre ai permessi standard, esistono due permessi speciali particolarmente importanti:

> "Oltre a questi permessi, ne esistono altri due 'speciali' che vengono acquisiti durante l'esecuzione:
> - `SUID`: il processo acquisisce i privilegi dell'**owner** invece di quelli di chi lo esegue.
> - `SGID`: il processo acquisisce i privilegi del **group owner** invece di quelli del gruppo di chi lo esegue."
>
> *Fonte: [Utenti e Gruppi](./laboratorio/Utenti%20e%20Gruppi#3-permessi-di-accesso-al-filesystem)*

**Rappresentazione Simbolica:**

- **SUID**: invece del permesso di esecuzione dell'owner `x`, si utilizza `s` (o `S` se il permesso di esecuzione non è presente)
  - Esempio: `-rwsr-xr-x` (comando `/usr/bin/passwd`)
- **SGID**: analogo a SUID, ma si applica al campo dei permessi del group owner
  - Esempio: `-rwxr-sr-x`

**Rappresentazione Ottale:**

Per i permessi speciali si aggiunge una cifra antecedente alle tre cifre standard:
- Si somma `4` se è attivo SUID
- Si somma `2` se è attivo SGID

Esempi:
- `4755`: SUID attivo, rwsr-xr-x
- `2755`: SGID attivo, rwxr-sr-x
- `6754`: SUID e SGID attivi, rwsr-sr--

_**EUID e EGID: Privilegi di un Processo**_

I permessi speciali sono strettamente collegati al concetto di identificatori effettivi di un processo. Dal file [Processi](./laboratorio/Processi#61-funzioni-per-gli-identificatori), sappiamo che un processo ha diversi identificatori:

> "I privilegi di un processo dipendono da due parametri:
> - **Effective User ID** `EUID`
> - **Effective Group ID** `EGID`"
>
> *Fonte: [Utenti e Gruppi](./laboratorio/Utenti%20e%20Gruppi#31-privilegi-di-un-processo)*

**Funzionamento Normale:**

> "Quando un processo viene eseguito, normalmente `<EUID, EGID>` corrispondono rispettivamente all'`UID` dell'utente che ha eseguito il processo e al `GID` del gruppo principale di tale utente."
>
> *Fonte: [Utenti e Gruppi](./laboratorio/Utenti%20e%20Gruppi#31-privilegi-di-un-processo)*

**Con SUID/SGID:**

> "Per permettergli di eseguire con privilegi diversi, è possibile impostare i permessi `SUID` e `SGID`."
>
> *Fonte: [Utenti e Gruppi](./laboratorio/Utenti%20e%20Gruppi#31-privilegi-di-un-processo)*

Quando un file eseguibile ha il bit SUID attivo:
- L'`EUID` del processo diventa l'`UID` dell'owner del file, invece dell'UID dell'utente che lo esegue
- Questo permette al processo di accedere alle risorse come se fosse l'owner del file

Analogamente, con il SGID attivo:
- L'`EGID` diventa il `GID` del group owner del file

**Esempio Pratico: il comando `passwd`**

Il comando `passwd` è un esempio classico di utilizzo di SUID:

```bash
ls -l /usr/bin/passwd
# Output: -rwsr-xr-x 1 root root 68208 ... /usr/bin/passwd
```

> "Il comando `passwd` permette di cambiare file di sistema pur non avendo i privilegi di `su`."
>
> *Fonte: [Utenti e Gruppi](./laboratorio/Utenti%20e%20Gruppi#2-utenti-e-gruppi)*

Grazie al bit SUID, quando un utente normale esegue `passwd`, il processo acquisisce temporaneamente i privilegi di `root` (owner del file), permettendogli di modificare il file `/etc/shadow` che normalmente non sarebbe accessibile.

**Approfondimenti:**

Il sistema di permessi UNIX rappresenta un equilibrio tra sicurezza e funzionalità. I permessi SUID/SGID sono potenti ma devono essere usati con cautela, poiché rappresentano potenziali vettori di attacco se applicati a programmi non sicuri. Per questo motivo, solo `root` può modificare l'owner di un file tramite il comando `chown`, mentre `chgrp` può essere usato dal proprietario solo se appartiene al gruppo di destinazione.

---
<div class="stop"></div>

---

## 3. Processi

### Domanda 3.1 (answered)

**Domanda**: Spiega il meccanismo della system call `fork()` in UNIX, descrivendo come vengono creati i processi figli, cosa ereditano dal padre (codice, dati, file descriptors), e perché la funzione restituisce valori diversi al padre e al figlio. Illustra inoltre come il padre può attendere la terminazione dei figli mediante `wait()`, spiegando il concetto di stato zombie e come viene gestito il parametro `status` per determinare se un processo è terminato volontariamente o involontariamente.

**Risposta:**

La system call `fork()` rappresenta il meccanismo fondamentale per la creazione di nuovi processi nei sistemi UNIX. È uno degli elementi distintivi della programmazione di sistema UNIX e permette la realizzazione di applicazioni multiprogrammate.

_**Creazione di Processi con fork()**_

> "Ogni processo è in grado di creare dinamicamente processi tramite la chiamata di sistema `fork`. Il processo creato, detto **figlio**, ha uno spazio di dati separato, ma condivide con il processo che lo ha creato, detto **padre**, il codice."
>
> *Fonte: [Processi](./laboratorio/Processi#31-fork)*

La funzione è definita come segue:

```cpp
/**
* La funzione non richiede parametri
* Restituisce un risultato intero DIVERSO a padre e figlio
*/
pid_t fork(void)
```

_**Cosa Eredita il Figlio**_

Al momento della creazione, il processo figlio riceve dal padre:

1. **Codice condiviso**: padre e figlio eseguono lo stesso programma
2. **Copia delle aree dati globali**: lo spazio dati è separato, ma inizialmente identico (stack, heap, User Structure)
3. **File descriptors**: il figlio eredita una copia dei file descriptors aperti dal padre

> "Poiché il processo figlio condivide il codice con il padre, **_ne eredita una copia delle aree dati globali_** (_stack, heap, User Structure_, ...). Ciò significa che alla creazione il figlio ha il proprio `%RIP` che punta alla all'istruzione successiva alla `CALL fork`"
>
> *Fonte: [Processi](./laboratorio/Processi#31-fork)*

Per quanto riguarda i file, dal file [Filesystem](./laboratorio/Filesystem#22-accesso-ai-file):

> "Quando viene generato un processo figlio, questo erediterà una copia di tutte le strutture dati del padre, in particolare anche la `User Structure` e i relativi **file descriptor**. Ciò implica che un processo padre e i suoi processi figli descrittori che **_puntano allo stesso elemento della Tabella di File di Sistema_** e quindi **_condividono l'I/O pointer nell'accesso sequenziale al file_**."

Questo significa che padre e figlio possono utilizzare i file già aperti dal padre, condividendo la posizione di lettura/scrittura.

_**Valori di Ritorno Diversi**_

Una caratteristica peculiare di `fork()` è che restituisce valori diversi a padre e figlio:

> "La funzione è però progettata affinché **_restituisca un risultato intero diverso a padre e figlio_**:
> - **Padre**: restituisce il `PID` del figlio, o un valore negativo in caso di fallimento
> - **Figlio**: restituisce `0`"
>
> *Fonte: [Processi](./laboratorio/Processi#31-fork)*

Questo meccanismo permette di discriminare il comportamento di padre e figlio attraverso una semplice struttura condizionale:

```cpp
pid_t pid;
pid = fork();

if(pid < 0){
    // Errore nella fork
    perror("Errore fork");
    exit(-1);
}
else if(pid == 0){
    // Codice del FIGLIO
    printf("Sono il figlio, PID: %d\n", getpid());
}
else{
    // Codice del PADRE
    printf("Sono il padre, mio figlio ha PID: %d\n", pid);
}
```

_**Attesa della Terminazione: wait()**_

Il processo padre può sincronizzarsi con i figli attendendo la loro terminazione mediante la system call `wait()`:

```cpp
/**
* Il padre si mette in attesa della terminazione del figlio
* @param status puntatore a intero che contiene lo stato di terminazione del figlio
* @return il PID del figlio che è terminato, valore negativo se non ha processi figli
*/
pid_t wait(int* status)
```

> "La sospensione del padre accade solo **_se tutti i figli sono ancora in esecuzione_**. Nel caso in cui almeno un figlio è terminato, la funzione ritorna immediatamente le informazioni di terminazione."
>
> *Fonte: [Processi](./laboratorio/Processi#32-terminazione-processi---exit-e-wait)*

_**Stato Zombie**_

Il concetto di stato zombie è cruciale per comprendere il funzionamento di `wait()`:

> "Questo è possibile grazie all'esistenza dello stato `zombie`. I processi figli che terminano infatti entrano nello stato di `zombie`, proprio per permettere al padre di ottenere le informazioni sulla terminazione di quest'ultimo."
>
> *Fonte: [Processi](./laboratorio/Processi#32-terminazione-processi---exit-e-wait)*

Un processo zombie è un processo che ha completato l'esecuzione ma il cui descrittore (PCB) rimane nella tabella dei processi per permettere al padre di recuperare le informazioni sulla sua terminazione. Solo dopo che il padre ha chiamato `wait()`, il sistema può liberare completamente le risorse del processo figlio.

_**Gestione del Parametro status**_

Il parametro `status` passato a `wait()` contiene informazioni dettagliate sulla terminazione del figlio:

> "La variabile `status` contiene diverse informazioni su come il figlio è terminato, oltre allo stato di terminazione eventualmente fornito dal figlio stesso. Se il _byte meno significativo_ di `*status` fosse `0`, allora la terminazione è **stata volontaria** e il _byte più significativo_ contiene lo stato di terminazione."
>
> *Fonte: [Processi](./laboratorio/Processi#32-terminazione-processi---exit-e-wait)*

**Macro per analizzare status:**

Per gestire `status` in modo portabile, la libreria `<sys/wait.h>` fornisce delle macro:

- `WIFEXITED(status)`: restituisce `true` se il processo è terminato volontariamente (tramite `exit()` o ritorno da `main`)
- `WEXITSTATUS(status)`: restituisce lo stato di terminazione passato dal figlio tramite `exit()`

Esempio pratico:

```cpp
int status;
pid_t pid = fork();

if(pid == 0){
    // Figlio
    printf("Figlio in esecuzione\n");
    exit(42);  // Terminazione volontaria con codice 42
}
else{
    // Padre
    wait(&status);
    
    if(WIFEXITED(status)){
        printf("Figlio terminato volontariamente\n");
        printf("Codice di uscita: %d\n", WEXITSTATUS(status));
    }
    else{
        printf("Figlio terminato involontariamente\n");
    }
}
```

_**Terminazione Volontaria vs Involontaria**_

Un processo può terminare in due modi:

> "Un processo può terminare in due modi:
> - **Involontariamente**: accade in caso di azioni illegali (ad esempio _segmentation fault_ o divisioni per zero) oppure in caso di **interruzioni causate dalla ricezione di segnali**
> - **Volontariamente**: quando si esegue l'ultima istruzione o viene chiamata la system call `exit()`"
>
> *Fonte: [Processi](./laboratorio/Processi#32-terminazione-processi---exit-e-wait)*

La system call `exit()` è definita così:

```cpp
/**
* È una chiamata senza ritorno che permette di terminare volontariamente un processo.
* @param status permette di comunicare al padre lo stato di terminazione
*/
void exit(int status)
```

**Approfondimenti:**

Il meccanismo `fork()`-`wait()` è alla base della programmazione di sistema UNIX. È importante notare che i processi zombie non consumano risorse significative (solo l'entry nella tabella dei processi), ma un accumulo eccessivo può esaurire la tabella dei processi. Un padre che non chiama `wait()` sui propri figli crea "orfani zombie". Se il padre termina prima di chiamare `wait()`, i figli zombie vengono adottati dal processo `init` (PID=1) che si occupa di ripulirli. Questo meccanismo garantisce che nessun processo zombie rimanga indefinitamente nel sistema.

---
<div class="stop"></div>

---

## 4. Thread

---
<div class="stop"></div>

---

## 5. Filesystem

---
<div class="stop"></div>

---

# TEORIA

## 6. Concetti Introduttivi

### Domanda 6.1 (answered)

**Domanda**: Spiega il concetto di **multiprogrammazione** e **time-sharing** nei sistemi operativi moderni. Definisci cosa sono i **CPU-burst** e gli **IO-burst**, illustrando con un esempio come la multiprogrammazione migliora l'utilizzo della CPU rispetto all'esecuzione sequenziale. Confronta infine il paradigma time-sharing con la multiprogrammazione classica, spiegando il ruolo del **quanto di tempo** e del **cambio di contesto**, e descrivi quando e perché questi meccanismi comportano un _overhead_ nel sistema.

**Risposta:**

La multiprogrammazione e il time-sharing rappresentano due paradigmi fondamentali che hanno rivoluzionato l'utilizzo dei sistemi di elaborazione, permettendo di sfruttare al massimo le risorse hardware e di fornire un'esperienza interattiva agli utenti.

_**Nascita della Multiprogrammazione**_

L'evoluzione verso i sistemi multiprogrammati nasce da un'osservazione critica sull'efficienza dei mainframe:

> "Venne osservato che l'efficienza dei _mainframe_, che avevano costi nell'ordine dei milioni di euro, era tendenzialmente bassa. Questo accadeva perché le risorse, a causa di come venivano gestiti i programmi, venivano utilizzate in media meno della metà del loro potenziale."
>
> *Fonte: [Concetti Introduttivi](./Concetti%20Introduttivi#53-sistemi-multiprogrammati)*

Per risolvere questo problema venne introdotta la multiprogrammazione:

> "Per riuscire a ottenere un drastico miglioramento nell'efficienza di uso delle risorse della macchina fu realizzata la tecnica della _multiprogrammazione_. Questa tecnica permetteva a più programmi di venire caricati in memoria in parallelo, gestendoli in modo concorrente."
>
> *Fonte: [Concetti Introduttivi](./Concetti%20Introduttivi#53-sistemi-multiprogrammati)*

_**CPU-Burst e IO-Burst**_

Per comprendere come funziona la multiprogrammazione, è necessario introdurre due concetti fondamentali:

> "Nella multiprogrammazione andiamo quindi ad identificare due momenti durante l'esecuzione di un processo:
> - **_CPU-burst_**: intervalli di tempo nel quale un processo deve eseguire istruzioni e necessita di occupare la **CPU**
> - **_IO-burst_**: intervalli di tempo nel quale un processo deve eseguire un'operazione di **IO** e non deve utilizzare la **CPU**"
>
> *Fonte: [Concetti Introduttivi](./Concetti%20Introduttivi#53-sistemi-multiprogrammati)*

Questi due concetti sono cruciali perché permettono di capire che durante un _IO-burst_, la CPU rimarrebbe inutilizzata se non ci fosse un meccanismo per assegnarla ad un altro processo.

_**Confronto: Esecuzione Sequenziale vs Multiprogrammazione**_

Dal file [Concetti Introduttivi](./Concetti%20Introduttivi#53-sistemi-multiprogrammati) possiamo vedere un esempio concreto:

> "Nell'esecuzione _sequenziale_ i processi vengono eseguiti in ordine di arrivo, e vengono eseguiti dall'inizio alla fine. In questo modo notiamo che sono presenti diverse unità di tempo dove la **CPU** è in attesa di qualcosa. Possiamo calcolare l'efficienza di questo esempio, che è di $\frac{10}{27} \approx 37\%$"

In un sistema sequenziale, quando un processo entra in attesa per un'operazione di I/O, la CPU rimane inattiva sprecando cicli preziosi.

> "Nell'esecuzione _multi-tasking_ invece, quando il primo processo va in _IO-burst_, e si mette in attesa, la **CPU** inizia a lavorare prima sul secondo processo, e poi sul terzo quando anche il secondo va in attesa. Durante i momenti nei quali i vari programmi sono sospesi, questi vengono poi recuperati nell'ordine in cui ricevono i dati che attendono, e verranno messi in esecuzione quando colui che occupa l'esecuzione terminerà e/o andrà nuovamente in attesa. Possiamo quindi calcolare anche in questo caso l'efficienza, che stavolta è di $\frac{10}{12} \approx 83\%$"
>
> *Fonte: [Concetti Introduttivi](./Concetti%20Introduttivi#53-sistemi-multiprogrammati)*

Questo esempio dimostra come la multiprogrammazione permetta di **più che raddoppiare l'efficienza** del sistema, passando dal 37% all'83%.

_**Meccanismi della Multiprogrammazione**_

La nuova gestione richiede l'introduzione di algoritmi di scheduling:

> "Tuttavia, questa nuova gestione dei programmi creò la necessità dell'implementazione di algoritmi di _scheduling_, che permettevano alla **CPU** di eseguirli uno alla volta e di sostituirli quando venivano messi in attesa, così da ridurre il più possibile i "tempi morti"."
>
> *Fonte: [Concetti Introduttivi](./Concetti%20Introduttivi#53-sistemi-multiprogrammati)*

Un elemento chiave è l'introduzione della **preemption**:

> "I primi algoritmi di _scheduling_ deviarono dal principio di _first-come-first-serve_, e introdussero il concetto di _interruzione_. Non si lasciava più l'accesso alla **CPU** ad un programma per tutto il suo _time-to-live_, ma si riservava la possibilità sostituirlo anche durante la sua eseguzione qual'ora questo fosse andato in attesa, così da permettere ad un altro programma di sfruttare quei cicli che sarebbero altrimenti stati sprecati."
>
> *Fonte: [Concetti Introduttivi](./Concetti%20Introduttivi#53-sistemi-multiprogrammati)*

_**Time-Sharing: Un Passo Ulteriore**_

Il paradigma time-sharing rappresenta un'evoluzione della multiprogrammazione:

> "Sono sistemi che hanno come primo obiettivo quello di dividere il tempo d'uso della **CPU** tra i vari processi."
>
> *Fonte: [Concetti Introduttivi](./Concetti%20Introduttivi#54-sistemi-time-sharing)*

La differenza fondamentale è espressa chiaramente:

> "Mentre nei sistemi multiprogrammati quando la **CPU** viene assegnata ad un processo, gli altri non la possono utilizzare finché questo non termina la sua **cpu-burst**, nel paradigma _time-sharing_ la **CPU** è assegnata ad ogni processo per un **_quanto di tempo uguale e predeterminato per tutti_**."
>
> *Fonte: [Concetti Introduttivi](./Concetti%20Introduttivi#54-sistemi-time-sharing)*

_**Il Quanto di Tempo e la Preemption**_

Nel time-sharing, il quanto di tempo determina il comportamento del sistema:

> "La politica quindi diventa la seguente:
> - Se durante il quanto di tempo hai terminato la **cpu-burst** vai in attesa fino al prossimo **cpu-burst**
> - Se durante il quanto di tempo _**non**_ hai terminato, il tuo stato intermedio viene salvato e verrà ripristinato quando tornerà il tuo turno. Questa revoca viene chiamata _preemption_."
>
> *Fonte: [Concetti Introduttivi](./Concetti%20Introduttivi#54-sistemi-time-sharing)*

_**Il Cambio di Contesto**_

Il salvataggio e ripristino dello stato è un'operazione critica:

> "Il salvataggio e ripristino dello stato intermedio corrispondono a tutti gli effetti al **_cambio di contesto_** che avevamo visto nel corso di [**Calcolatori Elettronici**]."
>
> *Fonte: [Concetti Introduttivi](./Concetti%20Introduttivi#54-sistemi-time-sharing)*

_**L'Overhead del Sistema**_

Un aspetto cruciale da considerare è il costo della multiprogrammazione:

> "La multiprogrammazione non è però gratuita. Il costo di migliorare i tempi si chiama _**overhead**_, e consiste nel tempo aggiuntivo usato dall'_OS_ per eseguire il codice aggiuntivo introdotto dalle operazioni intermedie, come l'algoritmo di schedulazione, l'algoritmo di cambio di contesto, ..."
>
> *Fonte: [Concetti Introduttivi](./Concetti%20Introduttivi#55-overhead)*

L'overhead rappresenta tempo sottratto all'esecuzione dei programmi:

> "Questo tempo è a tutti gli effetti sottratto dall'esecuzione dei programmi applicativi. Per poter vedere un guadagno nei tempi di esecuzione è quindi necessario che l'_overhead_ sia contenuto. indicativamente attorno all' $1\%/2\%$."
>
> *Fonte: [Concetti Introduttivi](./Concetti%20Introduttivi#55-overhead)*

Le conseguenze di un overhead eccessivo possono essere gravi:

> "Se avessimo infatti _overhead_ del $70\%$ del tempo totale, potrebbe non essere conveniente avere un sistema multiprogrammato. Se fosse ancora più alto il sistema potrebbe persino andare in _crash_, in quanto impiegherebbe tutto il tempo a eseguire le istruzioni di _overhead_ e non avrebbe più tempo per eseguire i programmi applicativi."
>
> *Fonte: [Concetti Introduttivi](./Concetti%20Introduttivi#55-overhead)*

**Approfondimenti:**

La scelta del quanto di tempo nel time-sharing è un compromesso delicato. Un quanto troppo piccolo aumenta la frequenza dei cambi di contesto, incrementando l'overhead. Un quanto troppo grande riduce la reattività del sistema e lo fa avvicinare al comportamento di un sistema multiprogrammato classico. 

Un esempio notevole di time-sharing è l'algoritmo **Round-Robin** (`RR`), menzionato nel file [Concetti Introduttivi](./Concetti%20Introduttivi#54-sistemi-time-sharing), che rappresenta uno degli algoritmi di scheduling più utilizzati per i sistemi interattivi.

La memoria divenne rapidamente il bottleneck di questi sistemi, portando all'introduzione della gestione dinamica della memoria attraverso meccanismi di _swap_ e alla virtualizzazione della memoria, argomenti che approfondiremo nelle domande successive sulla gestione della memoria.

---
<div class="stop"></div>

---

## 7. Classificazione delle Architetture

### Domanda 7.1 (answered)

**Domanda**: Spiega la **Tassonomia di Flynn** per la classificazione delle architetture dei sistemi di elaborazione. Descrivi in dettaglio le quattro categorie principali (SISD, SIMD, MISD, MIMD), illustrando per ciascuna le caratteristiche distintive, esempi di applicazione e vantaggi/svantaggi. Confronta inoltre le architetture SIMD e MIMD in termini di costo hardware, memoria richiesta, flessibilità e modelli computazionali supportati, spiegando infine la differenza tra macchine MIMD a memoria distribuita (DM-MIMD) e a memoria condivisa (SM-MIMD).

**Risposta:**

La Tassonomia di Flynn rappresenta uno degli schemi di classificazione più utilizzati e riconosciuti per categorizzare le architetture dei sistemi di elaborazione, basandosi sulla capacità del sistema di gestire flussi multipli di istruzioni e dati.

_**La Tassonomia di Flynn**_

> "È un metodo di classificazione dei sistemi di elaborazione, che utilizza due punti di vista:
> - La capacità del sistema di avere **più flussi di istruzioni**
> - La capacità del sistema di avere **più flussi di dati**"
>
> *Fonte: [Classificazione delle Architetture](./Classificazione%20delle%20Architetture#2-tassonomia-di-flynn)*

Questa classificazione genera quattro categorie principali, rappresentabili in una matrice 2×2:

|                        | Single Instruction (SI) | Multiple Instruction (MI) |
| :--------------------: | :---------------------: | :-----------------------: |
|  **Single Data (SD)**  |          SISD           |           MISD            |
| **Multiple Data (MD)** |          SIMD           |           MIMD            |

_**1. Macchine SISD (Single Instruction Single Data)**_

> "Sono macchine a singolo stream, che rappresentano le tradizionali macchine sequenziali basate sul modello di **Von Neumann** usata da tutti i calcolatori convenzionali."
>
> *Fonte: [Classificazione delle Architetture](./Classificazione%20delle%20Architetture#21-macchine-sisd)*

Le macchine SISD rappresentano l'architettura più semplice: un singolo processore esegue una singola istruzione alla volta su un singolo dato. Sono caratterizzate da:
- Un'unica unità di elaborazione
- Una memoria per le istruzioni
- Una memoria per i dati
- Esecuzione strettamente sequenziale

**Esempi:** I computer personali tradizionali con un singolo core rappresentano macchine SISD.

_**2. Macchine SIMD (Single Instruction Multiple Data)**_

> "Si differenzia dalle macchine SISD per il numero di **_Data Processor_**, ciascuno dei quali possiede una propria **_Data Memory_**. Questo permette a più unità di elaborazione di eseguire contemporaneamente la stessa istruzione, lavorando su flussi di dati differenti."
>
> *Fonte: [Classificazione delle Architetture](./Classificazione%20delle%20Architetture#22-macchine-simd)*

Caratteristiche distintive delle SIMD:

> "La topologia di interconnessione tra i vari processori può essere sia _regolare_ che _creata ad hoc_. Questa architettura permette comunicazioni regolari efficienti e poco costose, che non creano conflitti."
>
> *Fonte: [Classificazione delle Architetture](./Classificazione%20delle%20Architetture#22-macchine-simd)*

Il modello computazionale è importante:

> "Il modello di computazione di queste macchine è **_Sincrono_**, ovvero gestito da un unica unità di controllo."
>
> *Fonte: [Classificazione delle Architetture](./Classificazione%20delle%20Architetture#22-macchine-simd)*

Questo permette due tipi di parallelismi:
- **Temporale**: fasi diverse di un'unica istruzione eseguite in parallelo in moduli connessi in cascata (_pipeline_)
- **Spaziale**: i medesimi passi eseguiti contemporaneamente su un array di processori sincronizzati

**Esempi di architetture SIMD:**
- Supercomputer vettoriali per elaborazione di grandi matrici
- Vector Processor con caratteristiche _pipeline_
- Array Processor
- Systolic Array

Un aspetto interessante è la portabilità:

> "I programmi che beneficiano dell'architettura `SIMD`, ad esempio per lavorare su grandi vettori, possono essere eseguiti, con opportune ma comunque piccole modifiche, da processori `SISD`. Infatti avendo un operazione tra vettori `c = a + b`, il compilatore può tradurla in `for (...) c[i] = a[i] + b[i]`."
>
> *Fonte: [Classificazione delle Architetture](./Classificazione%20delle%20Architetture#22-macchine-simd)*

_**3. Macchine MISD (Multiple Instruction Single Data)**_

> "Queste macchine hanno più flussi di istruzioni che lavorano contemporaneamente su un unico flusso di dati."
>
> *Fonte: [Classificazione delle Architetture](./Classificazione%20delle%20Architetture#23-macchine-misd)*

La categoria MISD è particolarmente interessante:

> "Molti considerano questa categoria ancora 'vuota', ovvero senza esempi reali. Altri invece categorizzano i processori basati su _pipeline_ proprio come macchine `MISD`."
>
> *Fonte: [Classificazione delle Architetture](./Classificazione%20delle%20Architetture#23-macchine-misd)*

Le pipeline dei processori moderni potrebbero essere considerate MISD, dove diverse unità funzionali (fetch, decode, execute) operano contemporaneamente su stadi diversi dello stesso stream di dati.

_**4. Macchine MIMD (Multiple Instruction Multiple Data)**_

> "In queste macchine abbiamo **tante unità di elaborazione** connesse a tante **unità di dati**. Abbiamo infatti più flussi di istruzioni in parallelo che elaborano insiemi di dati che possono essere _distinti_, _privati_ o _condivisi_."
>
> *Fonte: [Classificazione delle Architetture](./Classificazione%20delle%20Architetture#24-macchine-mimd)*

Le macchine MIMD si dividono in due sottocategorie fondamentali:

**A. Macchine a Memoria Distribuita (DM-MIMD):**

> "Ogni coppia `IP`-`DP` (con le relative memoria) costituisce in pratica **una macchina** `SISD`."
>
> *Fonte: [Classificazione delle Architetture](./Classificazione%20delle%20Architetture#241-macchine-a-memoria-distribuita-dm-mimd)*

Caratteristiche principali:

> "Una qualsiasi **_rete di calcolatori_** rappresenta una macchina `DM-MIMD`. Infatti queste reti di interconnessione regolari permettono ai nodi di scambiare informazioni secondo il paradigma _message passing_. Queste reti permettono _algoritmi ad elevata località_ e un _elevata scalabilità_."
>
> *Fonte: [Classificazione delle Architetture](./Classificazione%20delle%20Architetture#241-macchine-a-memoria-distribuita-dm-mimd)*

Due sottotipi importanti:
- **DM-MIMD MPP** (_Massively Parallel Processing_): migliaia di nodi con CPU standard, ciascuna con proprio OS e memoria, connessi da rete custom ad alta banda e bassa latenza
- **DM-MIMD COW** (_Cluster Of Workstations_): caratterizzato da _high-availability_ (migrazione del calcolo tra nodi) e _high-load-balancing_ (allocazione task sul nodo meno carico)

**B. Macchine a Memoria Condivisa (SM-MIMD):**

> "Sono macchine multiprocessore che permettono la condivisione della memoria tra processori attraverso delle **aree**. Affinché questa architettura funzioni lo _switch NxN_ deve essere molto efficiente."
>
> *Fonte: [Classificazione delle Architetture](./Classificazione%20delle%20Architetture#242-macchine-a-memoria-condivisa-sm-mimd)*

Una limitazione importante:

> "A differenza delle `MD-MIMD` questa architettura ha una **scalabilità limitata**."
>
> *Fonte: [Classificazione delle Architetture](./Classificazione%20delle%20Architetture#242-macchine-a-memoria-condivisa-sm-mimd)*

Questo vale quando il numero di processori N è "piccolo" (N < 100), ma l'accoppiamento fra i nodi può essere molto stretto con comunicazioni veloci.

_**Confronto SIMD vs MIMD**_

Dal file [Classificazione delle Architetture](./Classificazione%20delle%20Architetture#25-confronto-simd-e-mimd), una tabella comparativa evidenzia le differenze:

|     Aspetto      |                   SIMD                   |                         MIMD                         |
| :--------------: | :--------------------------------------: | :--------------------------------------------------: |
|   **Hardware**   |     Poco - Unica Unità di Controllo      |           Molto - Tante Unità di Controllo           |
|    **Costo**     | Costoso - Necessita processori specifici |     Poco costoso - Processori _general-purpose_      |
|   **Memoria**    |   Poca - Una sola copia del programma    |    Tanta - Ogni unità ha una copia del programma     |
| **Flessibilità** |                   Poca                   | Alta in termini di modelli computazionali supportati |

**Analisi dettagliata:**

1. **Hardware**: Le SIMD richiedono meno hardware complessivamente perché condividono un'unica unità di controllo, mentre le MIMD necessitano di tante unità di controllo quanti sono i processori.

2. **Costo**: Paradossalmente, le SIMD sono più costose perché richiedono processori specializzati per l'elaborazione vettoriale, mentre le MIMD possono utilizzare processori commerciali standard.

3. **Memoria**: Nelle SIMD il codice è condiviso (un'unica copia), mentre nelle MIMD ogni processore deve avere la propria copia del codice da eseguire.

4. **Flessibilità**: Le MIMD sono molto più flessibili perché ogni processore può eseguire programmi diversi, mentre le SIMD sono limitate ad applicazioni che possono essere espresse come operazioni vettoriali sincrone.

**Approfondimenti:**

La scelta tra SIMD e MIMD dipende fortemente dall'applicazione. Le SIMD eccellono in:
- Elaborazione di immagini e video
- Calcolo scientifico su matrici e vettori
- Simulazioni fisiche con griglia regolare

Le MIMD sono preferibili per:
- Applicazioni generali
- Sistemi operativi multiprogrammati
- Database distribuiti
- Applicazioni web scalabili

La maggior parte dei sistemi moderni è ibrida: i processori SISD tradizionali includono estensioni SIMD (SSE, AVX in Intel, NEON in ARM) per operazioni vettoriali, mentre i data center utilizzano architetture MIMD per la scalabilità. Le GPU moderne sono essenzialmente architetture SIMD massive, con migliaia di core che eseguono la stessa istruzione su dati diversi.

Un concetto importante è che le architetture MIMD permettono vera _asincronia_, mentre le SIMD richiedono sincronizzazione, rendendole più semplici da programmare a livello hardware ma potenzialmente meno efficienti quando i dati hanno comportamenti irregolari o imprevedibili.

---
<div class="stop"></div>

---

## 8. Gestione Processi

### Domanda 8.1 (answered)

**Domanda**: Confronta gli algoritmi di scheduling FCFS, SJF, SRTF e RR analizzandone il funzionamento, i vantaggi e gli svantaggi. Spiega in particolare la differenza tra algoritmi preemptive e non-preemptive, e come il quanto di tempo influenza le prestazioni del Round-Robin. Illustra inoltre il concetto di starvation e come può essere risolto nei sistemi Multi-Level Feedback Queue.

**Risposta:**

Lo scheduling della CPU rappresenta uno degli aspetti fondamentali nella gestione dei processi in un sistema operativo multiprogrammato. Gli algoritmi di scheduling determinano quale processo pronto debba essere assegnato alla CPU e per quanto tempo, influenzando direttamente le prestazioni del sistema.

_**Scheduling e Classificazione degli Algoritmi**_

Lo scheduling è definito come:

> "L'attività mediante la quale il sistema operativo effettua delle scelte tra i processi, riguardo al _caricamento in **RAM**_ e all'_assegnazione della **CPU**_."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#3-scheduling)*

Esistono tre livelli di scheduling, ma quello che ci interessa è lo **scheduling a breve termine**:

> "**A breve termine**: è lo _scheduling_ propriamente detto. È la politica con la quale il sistema operativo assegna la **CPU** ai processi pronti. Interviene quando il processo in esecuzione perde il controllo della **CPU**"
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#3-scheduling)*

Una distinzione fondamentale è tra algoritmi **preemptive** e **non-preemptive**:

> "Può essere di due tipi:
> - Non _preemptive scheduling_ (senza diritto di revoca): il sistema operativo non può revocare la **CPU**, ma deve essere lui a rilasciarla
> - _Preemptive scheduling_ (con diritto di revoca): il sistema operativo può forzare la revoca della **CPU** ad un processo in base a determiate variabili. (quanti di tempo, priorità, ...)"
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#3-scheduling)*

Questa distinzione è cruciale: negli algoritmi **non-preemptive**, un processo mantiene la CPU fino a quando non termina o si sospende volontariamente. Negli algoritmi **preemptive**, il sistema operativo può forzare il processo a rilasciare la CPU anche se non ha completato la sua esecuzione.

_**Algoritmo FCFS (First-Come-First-Served)**_

L'algoritmo FCFS è il più semplice tra gli algoritmi di scheduling:

> "Assegna la **CPU** al processo pronti in attesa da più tempo"
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#32-algoritmo-fcfs)*

**Funzionamento:**

> "Quando un processo entra nella coda dei processi `pronti` il suo descrittore viene collegato **all'ultimo elemento della coda**. Quando la **CPU** è libera viene assegnata al processo il cui descrittore si trova nella _testa della coda_. È a tutti gli effetti equivalente alla politica `FIFO`."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#32-algoritmo-fcfs)*

**Vantaggi:**
- **Semplicità di implementazione**: la gestione della coda ha complessità $O(1)$ sia per inserimenti che per estrazioni
- **Equità**: ogni processo viene servito nell'ordine di arrivo, senza discriminazioni
- **Nessun overhead aggiuntivo**: non richiede calcoli complessi

**Svantaggi:**

> "Questo algoritmo è altamente dipendente **dall'ordine nel quali i processi arrivano** per quanto riguarda i tempi medi di attesa e di _overhead_."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#32-algoritmo-fcfs)*

Inoltre:

> "Possiamo anche dedurre che questo algoritmo viene pesantemente deabilitato da processi con **CPU-Burst** grandi che arrivano prima di altri con **CPU-Burst** breve."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#32-algoritmo-fcfs)*

Questo fenomeno è noto come **convoy effect**: processi brevi devono attendere il completamento di processi lunghi, aumentando drammaticamente i tempi medi di attesa.

_**Algoritmo SJF (Shortest-Job-First)**_

L'algoritmo SJF rappresenta un miglioramento rispetto a FCFS:

> "È un algoritmo a priorità statica non preemptive. La priorità viene assegnata ad ogni processo su base inversa rispetto al suo **CPU-Burst**"
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#33-algoritmo-sjf)*

**Funzionamento:**
I processi vengono ordinati nella coda dei pronti in base alla durata prevista del loro CPU-Burst, con i processi più brevi che hanno priorità maggiore.

**Vantaggi:**
- **Ottimizza il tempo medio di attesa**: è dimostrabile che SJF minimizza il tempo medio di attesa rispetto agli altri algoritmi non-preemptive
- **Migliore utilizzo della CPU** per processi brevi

**Svantaggi:**

> "In questo algoritmo abbiamo più _overhead_. Infatti l'inserimento in coda pronti è adesso un inserimento ordinato, perciò ha una complessità $O(n)$."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#33-algoritmo-sjf)*

Inoltre:

> "Essendo **non preemptive**, ha la limitazione che nel caso di arrivo di processi con elevati **CPU-Burst** prima di altri con tempi più bassi, ritorniamo nel problema di `FCFS`"
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#33-algoritmo-sjf)*

Un altro problema fondamentale è la necessità di **conoscere a priori** il CPU-Burst, cosa non sempre possibile.

_**Algoritmo SRTF (Shortest-Remaining-Time-First)**_

SRTF è l'evoluzione preemptive di SJF:

> "È un miglioramento di `SJF`, che introduce la possibilità di essere **preemptive**. Inoltre, proprio per via della _preemption_, non si guarderà più la **CPU-Burst** iniziale del processo, ma quella che gli rimane da eseguire."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#34-algoritmo-srtf)*

**Funzionamento:**
Quando arriva un nuovo processo, il sistema confronta il tempo rimanente del processo in esecuzione con il CPU-Burst del nuovo processo. Se il nuovo processo ha un tempo di esecuzione inferiore, viene effettuato un cambio di contesto.

Dal file [Gestione Processi](./Gestione%20Processi#34-algoritmo-srtf):

> "Nel caso un processo con **CPU-Burst** elevata (100) che però sta per finire (rimanente 2), non ha senso sostituirlo con un altro appena arrivato che magari ha **CPU-Burst** (50), che è sì più breve di quella iniziale ma molto più alta di quella rimanente."

**Vantaggi:**
- **Ottimale per il tempo medio di attesa**: tra gli algoritmi preemptive, SRTF minimizza il tempo medio di attesa
- **Reattivo**: processi brevi ottengono rapidamente la CPU

**Svantaggi:**

> "È un sistema che funziona bene per sistemi statici, dove il numero di processi non varia e i loro **CPU-Burst** è noto"
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#34-algoritmo-srtf)*

Il problema più grave è la **starvation**:

> "Inoltre questo algoritmo, in caso di sistemi aperti con processi variabili, soffre di _starvation_. Infatti se arrivano continuamente processi con priorità più alta del primo, questo starà in attesa per un tempo che può diventare lunghissimo."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#34-algoritmo-srtf)*

**Soluzione alla starvation in SJF/SRTF:**

> "Per risolvere si utilizzano tecniche di _aging_, che monitorano i tempi di attesa e modificano opportunamente le priorità per rimediare alla _starvation_."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#34-algoritmo-srtf)*

_**Stima del CPU-Burst**_

Un problema comune a SJF e SRTF è stimare il CPU-Burst quando non è noto a priori:

> "Non sempre sappiamo a priori il **CPU-Burst** di un processo. Si utilizza quindi la media esponenziale per stimarlo, tenendo conto della storia dei valori misurati nei precedenti intervalli di esecuzione"
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#341-stima-della-cpu-burst)*

La formula utilizzata è: $s_{n+1} = a \cdot t_n + (1-a) \cdot s_n$ dove $t_n$ è la durata effettiva e $s_n$ la stima, con fattore $a$ tipicamente pari a $\frac{1}{2}$.

_**Algoritmo Round-Robin (RR)**_

Il Round-Robin è progettato specificamente per sistemi a partizione di tempo:

> "È un algoritmo progettato appositamente per i sistemi a _partizione di tempo_, rientrando negli algoritmi _preemptive_."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#35-algoritmo-rr)*

**Funzionamento:**

> "La coda dei proessi pronti è realizzata come una **coda circolare**, nella quale ogni processo ottiene la **CPU** per un **_quanto di tempo_** (tipicamente $10ms\sim100 ms$) al termine del quale **perde il controllo della CPU** e il suo descrittore viene inserito nella coda."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#35-algoritmo-rr)*

La coda viene gestita con modalità FIFO, garantendo equità tra i processi.

**Influenza del Quanto di Tempo:**

> "È un algoritmo particolarmente adatto per i sistemi interattivi, in quanto è in grado di assicurare tempi di risposta abbastaza brevi, determinati esclusivamente da due fattori:
> - **Il quanto di tempo**: il tempo di risposta è teoricamente migliore per piccoli valori del quanto di tempo. Tuttavia in questo caso non possiamo più _ignorare il cambio di contesto tra processi_. Infatti, cambi troppo frequenti comporterebbero un _overhead_ che potrebbe addirittura diventare più grande del quanto stesso."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#35-algoritmo-rr)*

Questo è un punto cruciale: 
- **Quanto troppo piccolo**: eccessivi cambi di contesto riducono l'efficienza
- **Quanto troppo grande**: il sistema si comporta come FCFS, perdendo reattività

Il secondo fattore è:

> "- **Il numero medio di processi pronti**: se fossero presenti tanti processi pronti, potremmo andare incontro a tempi di _turnaround_ molto elevati, anche per processi con **CPU-Burst** molto brevi, che dovrebbero comunque attendere diversi cicli prima di poter terminare"
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#35-algoritmo-rr)*

**Tempo di attesa massimo in RR:**

Conoscendo il numero medio di processi in coda $n_m$, possiamo calcolare: $A_m \le n_m \cdot Q$ dove $Q$ è il quanto di tempo.

**Vantaggi:**
- **Equità**: ogni processo riceve la stessa quantità di CPU
- **Nessuna starvation**: ogni processo è garantito di essere servito
- **Tempi di risposta prevedibili**: adatto per sistemi interattivi
- **Semplicità**: implementazione semplice con coda FIFO

**Svantaggi:**
- **Context switch overhead**: frequenti cambi di contesto
- **Non ottimale per il throughput**: processi lunghi e brevi sono trattati allo stesso modo
- **Difficoltà nella scelta del quanto**: richiede tuning per bilanciare reattività ed efficienza

_**Sistemi Multi-Level Feedback Queue**_

Per superare i limiti dei singoli algoritmi, i sistemi moderni utilizzano **code multiple con feedback**:

> "All'interno del nostro sistema non utilizzeremo solamente un algoritmo di scheduling, ma implementeremo un sistema che possiede **_più code_**, ognuna ordinata con un algoritmo diverso."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#36-sistemi-multi-level-queue)*

**Esempio di configurazione:**
1. Coda 0: `RR(10)` - priorità massima
2. Coda 1: `RR(50)` - priorità media  
3. Coda 2: `FCFS` - priorità minima

**Gestione della starvation:**

> "Come sempre, introducendo il concetto di priorità, introduciamo anche la generazione di problemi di _starvation_ nelle code con priorità più bassa, tuttavia abbiamo anche visto come è possibile implementare processi di _aging_ che permettono di aumentare la priorità di un processo, in questo caso cambandone eventualmente la coda."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#36-sistemi-multi-level-queue)*

**Funzionamento dinamico:**

Il sistema operativo inserisce i nuovi processi nella coda di livello più alto. Se un processo non termina entro il suo quanto di tempo:

> "Se il **CPU-Burst** del processo fosse maggiore, la _preemption_ lo sostituisce e lo inserisce nella **coda di livello 1**."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#36-sistemi-multi-level-queue)*

Questo meccanismo permette di:
- **Identificare automaticamente processi interattivi vs batch**: processi brevi (interattivi) terminano nelle code ad alta priorità, processi lunghi (batch) scendono nelle code a bassa priorità
- **Adattarsi dinamicamente**: il comportamento del processo determina il suo trattamento
- **Prevenire starvation**: attraverso aging e meccanismi di preemption controllati

**Preemption per code FCFS:**

> "Per ovviare a questo problema, rendiamo la coda `FCFS` **_preemptive per l'inserimento dei nuovi processi_**. Anche in questo caso il processo rimarrà nella coda di livello 2, ma faremo attenzione ad inserirlo **_in testa_**."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#36-sistemi-multi-level-queue)*

Questo garantisce che anche processi nelle code a bassa priorità possano progredire.

_**Confronto Complessivo**_

Dal file [Gestione Processi](./Gestione%20Processi#3-scheduling), possiamo classificare gli algoritmi:

| Algoritmo       | Tipo           | Complessità | Ottimalità        | Starvation  | Uso pratico                       |
| --------------- | -------------- | ----------- | ----------------- | ----------- | --------------------------------- |
| **FCFS**        | Non-preemptive | $O(1)$      | No                | No          | Sistemi batch semplici            |
| **SJF**         | Non-preemptive | $O(n)$      | Sì (tempo attesa) | Sì          | Teorico, batch con CPU-burst noti |
| **SRTF**        | Preemptive     | $O(n)$      | Sì (tempo attesa) | Sì          | Sistemi embedded statici          |
| **RR**          | Preemptive     | $O(1)$      | No                | No          | Sistemi time-sharing, interattivi |
| **Multi-Level** | Preemptive     | Variabile   | Bilanciato        | Risolvibile | Sistemi operativi moderni         |

**Approfondimenti:**

La scelta dell'algoritmo di scheduling dipende fortemente dal tipo di sistema:
- **Sistemi batch**: preferiscono throughput elevato (SJF/FCFS)
- **Sistemi interattivi**: richiedono tempi di risposta rapidi (RR, Multi-Level)
- **Sistemi real-time**: necessitano garanzie temporali (algoritmi specifici come Rate Monotonic o EDF)

I sistemi operativi moderni come Linux e Windows utilizzano varianti sofisticate di Multi-Level Feedback Queue, combinando priorità dinamiche, aging, e considerazioni sulla natura del processo (I/O-bound vs CPU-bound). Queste implementazioni bilanciano:
- **Fairness**: equità nell'allocazione della CPU
- **Throughput**: massimizzazione del lavoro completato
- **Response time**: minimizzazione del tempo di risposta per applicazioni interattive
- **Turnaround time**: minimizzazione del tempo totale di completamento
- **Starvation prevention**: garanzia che tutti i processi progrediscano

Il concetto di starvation è particolarmente importante: un processo che attende indefinitamente rappresenta non solo un problema di performance, ma anche di correttezza del sistema. Le tecniche di aging incrementano gradualmente la priorità dei processi in attesa, garantendo che prima o poi qualsiasi processo ottenga la CPU, indipendentemente dal suo CPU-burst o dalla sua priorità iniziale.

---

### Domanda 8.2 (answered)

**Domanda**: Spiega la schedulazione dei sistemi in tempo reale, illustrando le caratteristiche dei processi periodici e la definizione di sistema periodico. Analizza gli algoritmi Rate Monotonic (RM) e Earliest Deadline First (EDF), descrivendo come funzionano, le loro differenze (priorità statica vs dinamica), e il calcolo del fattore di utilizzo della CPU per determinare se un sistema è schedulabile. Fornisci esempi pratici di come vengono assegnate le priorità e gestite le deadline.

**Risposta:**

La schedulazione dei sistemi in tempo reale rappresenta un ambito specializzato della gestione dei processi, dove oltre all'efficienza nell'uso della CPU è necessario garantire il **rispetto di vincoli temporali stringenti**. Questi sistemi sono tipici di applicazioni embedded critiche dove il mancato rispetto di una deadline può avere conseguenze catastrofiche.

_**Caratteristiche dei Sistemi in Tempo Reale**_

Gli algoritmi di scheduling visti precedentemente funzionano per sistemi generici, ma non si applicano bene a sistemi embedded:

> "Gli algoritmi che abbiamo visto fin'ora, per quanto comunque funzionali, non si applicano bene a sistemi _embedded_ dove dobbiamo soddisfare anche altri requisiti. I sistemi _embedded_ infatti sono caratterizzati da un sistema operativo multiprogrammato che **elabora parametri in tempo reale**."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#37-schedulazione-di-sistemi-in-tempo-reale)*

L'architettura di questi sistemi è peculiare:

> "I sistemi in tempo reale possono essere rappresentati come sistemi con **CPU**, **RAM**, memoria flash e, soprattutto, due classi principali di periferiche: **_attuatori_** (_output_) e **_sensori_** (_input_)."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#37-schedulazione-di-sistemi-in-tempo-reale)*

_**Processi Periodici**_

La caratteristica distintiva di questi sistemi è la **periodicità**:

> "In questi sistemi i _sensori_ inviano _**periodicamente**_ al sistema operativo misurazioni di dati che è necessario elaborare per poter produrre output agli _attuatori_."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#37-schedulazione-di-sistemi-in-tempo-reale)*

La periodicità dipende dall'applicazione:

> "Il periodo con il quale campioniamo i dati **_dipende dall'oggetto che stiamo misurando_**. I periodi di campionamento possono variare dall'ordine dei microsecondi (sistemi di bilanciamento, braccia robotiche, ...) all'ordine dei secondi (misure di temperatura, pressione, ...)."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#37-schedulazione-di-sistemi-in-tempo-reale)*

Questa periodicità permette di modellare matematicamente il sistema:

> "Possiamo quindi sviluppare il nostro sistema tenendo conto del fatto che gli _input_, e i relativi processi di elaborazione dell'_input_, sono _**periodici**_."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#37-schedulazione-di-sistemi-in-tempo-reale)*

Per un processo $i$ con periodo $t_i$:

$$r_{j+1} = r_j + t_i = r_0 + (j+1)\cdot t_i$$

dove $r_j$ indica il $j$-esimo inserimento in coda pronti.

_**Definizione di Sistema Periodico**_

Un sistema periodico è caratterizzato da:

> "Quello che abbiamo descritto è quindi un sistema composto da $N$ processi periodici, ovvero di **_un sistema periodico_**."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#37-schedulazione-di-sistemi-in-tempo-reale)*

**Elementi fondamentali di un processo periodico $i$:**
- **Periodo** $t_i$: intervallo di tempo tra due attivazioni consecutive
- **CPU-Burst** $C_i$: tempo di elaborazione richiesto, definito come $C_i = \sum_{j=1}^{k_i}{C_i^{(j)}}$
- **Deadline** $d_i$: tempo massimo entro cui il processo deve completare l'elaborazione

**Vincolo temporale critico:**

> "È tuttavia necessario che il _turnaround_ del processo sia **_minore di_** $t_i$. In particolare vogliamo che termini entro una _deadline_ $d_i < t_{i+1}$. Questo avviene perché i risultati che il processo produrrà devono essere trasmessi agli _attuatori_ che dovranno quindi produrre cambiamenti opportunamente."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#37-schedulazione-di-sistemi-in-tempo-reale)*

**Periodo del sistema complessivo:**

Il sistema ha un periodo complessivo definito come:

$$T = \text{mcm}(t_i), \quad \forall i \in N$$

_**Sistemi Hard Real-Time vs Soft Real-Time**_

La classificazione dipende dalla criticità delle deadline:

> "In un sistema _hard real time_ lo _scheduler_ **_deve garantire che tutte le deadline vengano rispettate_**, altrimenti si genera un **errore fatale**. In sistemi _soft real time_ lo _scheduler_ **_può permettersi che qualche deadline possa non essere rispettata_** senza la generazioni di **errori fatali**, ma andando incontro a brevi errori temporanei."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#37-schedulazione-di-sistemi-in-tempo-reale)*

_**Algoritmo Rate Monotonic (RM)**_

Il Rate Monotonic è un algoritmo a **priorità statica**:

> "È un algoritmo a **priorità statica** $\quad p(i) \propto \frac{1}{t_i}$"
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#371-rate-monototic)*

**Principio di funzionamento:**

La priorità è assegnata in modo **inversamente proporzionale al periodo**: processi con periodo più breve hanno priorità più alta. Questa scelta è intuitiva: un processo che deve essere eseguito più frequentemente deve avere precedenza per non perdere la propria deadline.

**Esempio pratico:**

Dal file [Gestione Processi](./Gestione%20Processi#371-rate-monototic):

<div class="flexbox" markdown="1">

|       | Periodo $t_i$ | CPU-Burst $C_i$ | Priorità         |
| :---: | :-----------: | :-------------: | ---------------- |
| `Pa`  |       2       |        1        | Alta (periodo=2) |
| `Pb`  |       5       |        1        | Bassa (periodo=5) |

</div>

Il periodo del sistema è $T = \text{mcm}(2, 5) = 10$.

> "Possiamo subito calcolare che la priorità di `Pa` sarà **maggiore** di quella di `Pb`."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#371-rate-monototic)*

Nello schema di esecuzione, `Pa` viene sempre schedulato prima di `Pb` quando entrambi sono pronti. L'efficienza del sistema è del 70%, lasciando il 30% per gestione del sistema o altre routine.

**Varianti:**

> "Esiste di due tipi, sia _non preemptive_ che _preemptive_."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#371-rate-monototic)*

Nella versione preemptive, l'arrivo di un processo ad alta priorità può interrompere l'esecuzione di uno a bassa priorità.

**Ottimalità di RM:**

> "Questo algoritmo è **ottimo** nella classe degli algoritmi a priorità statica. In particolare, esiste la proprietà che: È possibile schedulare degli eventi a priorità statica **_se e solo se_** possiamo farlo tramite `RM`."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#371-rate-monototic)*

_**Fattore di Utilizzo della CPU**_

Per determinare se un sistema è schedulabile, si calcola il **fattore di utilizzo**:

> "La formula è abbastanza semplice, infatti: $$\sum_{i=0}^N{n_i \cdot C_i} \le T \quad \Rightarrow \quad \sum_{i=0}^N{\frac{T}{t_i} \cdot C_i} \le T \quad \Rightarrow \quad \boxed{\sum_{i=0}^N{C_i \over t_i} \le 1}$$"
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#372-valutazione-esistenza-dellalgoritmo)*

**Definizione formale:**

$$U := \sum_{i=0}^N{C_i \over t_i}$$

**Interpretazione:**
- Se $U \le 1$: il sistema è schedulabile (la CPU ha capacità sufficiente)
- Se $U > 1$: il sistema **non è schedulabile** (la CPU non ha capacità sufficiente)

**Esempio di calcolo:**

Per l'esempio precedente con `Pa` e `Pb`:

$$U = \frac{C_a}{t_a} + \frac{C_b}{t_b} = \frac{1}{2} + \frac{1}{5} = \frac{7}{10} = 0.7 \le 1$$

Il sistema è schedulabile con un margine del 30%.

**Considerazioni pratiche:**

> "Operativamente nella realtà la formula è leggermente diversa, dato che non possiamo far coincidere la deadline con il periodo, rendendo la formula reale qualcosa del genere: $$U \le 1 - \alpha$$"
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#372-valutazione-esistenza-dellalgoritmo)*

dove $\alpha$ rappresenta il margine di sicurezza per gestire overhead e ritardi.

_**Algoritmo Earliest Deadline First (EDF)**_

L'EDF rappresenta un approccio radicalmente diverso basato su **priorità dinamica**:

> "È un algoritmo _preemptive_ a **_priorità dinamica_**. Questa viene calcolata per ogni processo in base alla vicinanza alla sua deadline."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#373-earliest-deadline-first)*

**Principio di funzionamento:**

Ad ogni istante, lo scheduler seleziona il processo la cui deadline è più vicina. Le priorità cambiano dinamicamente durante l'esecuzione in base alle deadline rimanenti.

**Esempio pratico:**

Dal file [Gestione Processi](./Gestione%20Processi#373-earliest-deadline-first):

<div class="flexbox" markdown="1">

|       | Periodo $t_i$ | CPU-Burst $C_i$ |
| :---: | :-----------: | :-------------: |
| `Pa`  |       4       |        2        |
| `Pb`  |      10       |        5        |

</div>

**Analisi temporale:**

All'istante `t=0`:
- `Pa` ha deadline in `t=4`
- `Pb` ha deadline in `t=10`
- Quindi `p(Pa) > p(Pb)` → esegue `Pa`

All'istante `t=4`:
- `Pa` (nuova istanza) ha deadline in `t=8`
- `Pb` (in esecuzione) ha deadline in `t=10` ma ha già eseguito per 2 unità, quindi deadline rimanente = 6
- Quindi `p(Pa) > p(Pb)` → **preemption**, esegue `Pa`

All'istante `t=8`:
- `Pb` ha deadline in `t=10` (rimanente = 2)
- `Pa` ha deadline in `t=12` (rimanente = 4)
- Quindi `p(Pb) > p(Pa)` → esegue `Pb`

All'istante `t=16`:
- `Pa` ha deadline in `t=20` (rimanente = 4)
- `Pb` ha deadline in `t=20` (rimanente = 4)
- **Parità**: si mantiene il processo in esecuzione

> "All'istante `16` abbiamo che `Pa = 4 == Pb = 4`. La scelta che compiamo è quella di **_mantenere il processo in esecuzione_**."
>
> *Fonte: [Gestione Processi](./Gestione%20Processi#373-earliest-deadline-first)*

_**Confronto Rate Monotonic vs Earliest Deadline First**_

<div class="flexbox" markdown="1">

| Caratteristica            | Rate Monotonic (RM)                                   | Earliest Deadline First (EDF)                            |
| ------------------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Tipo di priorità**      | Statica (basata sul periodo)                          | Dinamica (basata sulla deadline)                         |
| **Assegnazione priorità** | $p(i) \propto \frac{1}{t_i}$                          | Processo con deadline più vicina                         |
| **Preemption**            | Opzionale (esistono versioni non-preemptive)          | Sempre preemptive                                        |
| **Complessità**           | Bassa (priorità calcolate una sola volta)             | Media (ricalcolo priorità ad ogni evento)                |
| **Overhead**              | Minimo                                                | Maggiore (gestione priorità dinamiche)                   |
| **Ottimalità**            | Ottimo tra algoritmi a priorità **statica**           | Ottimo tra **tutti** gli algoritmi                       |
| **Utilizzo CPU**          | Garantisce schedulabilità se $U \le 1$ (teorico)     | Garantisce schedulabilità se $U \le 1$                   |
| **Predicibilità**         | Alta (comportamento deterministico)                   | Media (dipende dal carico dinamico)                      |
| **Implementazione**       | Semplice                                              | Più complessa                                            |
| **Applicazioni tipiche**  | Sistemi embedded con carico prevedibile               | Sistemi con requisiti temporali variabili                |
| **Gestione overload**     | Processi a bassa priorità possono soffrire starvation | Distribuzione più equa del carico                        |
| **Teorema**               | Schedulabile se e solo se schedulabile con RM         | Se schedulabile, allora schedulabile con qualsiasi altro |

</div>

**Considerazioni aggiuntive:**

1. **RM** è preferito quando:
   - Il sistema ha carico statico e prevedibile
   - Si privilegia la semplicità implementativa
   - L'overhead deve essere minimizzato
   - I periodi dei task sono ben differenziati

2. **EDF** è preferito quando:
   - Il sistema ha carico variabile
   - Si vuole massimizzare l'utilizzo della CPU
   - Le deadline non coincidono esattamente con i periodi
   - Si può tollerare maggiore complessità

**Approfondimenti:**

Un aspetto fondamentale è che entrambi gli algoritmi richiedono la **conoscenza a priori** dei parametri dei processi (periodo, CPU-burst, deadline). Questo li rende adatti principalmente per sistemi embedded e applicazioni real-time dove il comportamento è deterministico.

La scelta tra RM e EDF in sistemi reali dipende anche da fattori pratici:
- **Certificazione**: sistemi critici (avionica, medicale) spesso preferiscono RM per la sua predicibilità
- **Jitter**: RM tende a produrre jitter inferiore (variabilità nei tempi di risposta)
- **Analizzabilità**: RM è più facile da analizzare formalmente per la verifica delle proprietà temporali

La condizione $U \le 1$ è **necessaria ma non sempre sufficiente** per RM. Esistono bound teorici più stringenti (come il bound di Liu e Layland: $U \le n(2^{1/n} - 1)$ per $n$ processi), ma nella pratica $U \le 1$ è un'ottima approssimazione, specialmente con $n$ grande.

Per EDF, la condizione $U \le 1$ è sia **necessaria che sufficiente**, rendendolo teoricamente superiore. Tuttavia, in caso di overload ($U > 1$), EDF può comportarsi peggio di RM, perché nessun processo è garantito di completare, mentre con RM almeno i processi ad alta priorità mantengono le loro deadline.

Un'applicazione pratica comune è nei **sistemi di controllo industriale**, dove sensori di temperatura, pressione, velocità devono essere letti a frequenze diverse ma tutte critiche. L'algoritmo RM permette di assegnare naturalmente priorità più alta ai sensori che richiedono campionamento più frequente, garantendo la stabilità del sistema di controllo.

---
<div class="stop"></div>

---

## 9. Sincronizzazione dei Processi

### Domanda 9.1 (answered)

**Domanda**: Spiega il problema dei Dining Philosophers, illustrando come una soluzione basata su semafori può portare a deadlock. Descrivi poi il concetto di Monitor come astrazione di alto livello per la sincronizzazione, spiegando la differenza tra "signal and wait" e "signal and continue", e mostra come implementare una soluzione al problema dei filosofi usando i Monitor che eviti il deadlock. Infine, spiega come implementare un Monitor utilizzando i semafori.

**Risposta:**

Il problema dei Dining Philosophers è uno dei classici problemi di sincronizzazione che illustra magistralmente le sfide della coordinazione tra processi concorrenti e i rischi di deadlock in sistemi con risorse condivise.

_**Il Problema dei Dining Philosophers**_

> "È un problema che considera **5 filosofi** `p_i` che passano tutta la loro vita a fare due cose: _Thinking_ e _Eating_"
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#33-dining-philosophers-problem)*

**Descrizione del problema:**

> "Questi filosofi si trovano attorno ad una tavola dove al centro si trova una **ciotola di riso infinita**. I filosofi stanno prevalentemente nello stato _thinking_, ma ogni tanto vogliono passare nello stato _eating_. Al tavolo però si trovano solamente **5 bacchette** (_chopsticks_) `c[i]` distribuite in modo da essercene una tra ogni coppia di filosofi."
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#33-dining-philosophers-problem)*

**Regole per mangiare:**

Quando un filosofo vuole mangiare deve seguire questa sequenza:
1. Prende la bacchetta a destra
2. Prende la bacchetta a sinistra
3. Mangia
4. Posa la bacchetta a sinistra
5. Posa la bacchetta a destra

_**Soluzione con Semafori e il Deadlock**_

Una prima soluzione intuitiva utilizza i semafori per proteggere l'accesso alle bacchette:

```cpp
sem chopstick[5] = 1;

proc philosopher{
	int i;

	do{
		// hungry
		wait(chopstick[i]);
		wait(chopstick[(i+1) % 5]);

		// eating

		signal(chopstick[(i+1) % 5]);
		signal(chopstick[i]);

		// thinking
	} while (true);
}
```

**Il problema del deadlock:**

> "Questo algoritmo possiede un principale punto critico [...] genera **deadlock**."
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#33-dining-philosophers-problem)*

**Come si verifica il deadlock:**

> "Immaginiamo che tutti i processi riescano a prendere possesso del primo `chopstick[i]` ma vengano interrotti 'in cascata' (`0` $\to \dots \to$`5`) prima di poter prendere il secondo. Quando i processi avranno recuperato la prima bacchetta e proveranno a recuperare la seconda, entreranno **_tutti nello stato `bloccato` in cascata_**, generando un **_deadlock_** (_blocco_)."
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#33-dining-philosophers-problem)*

**Visualizzazione grafica del deadlock:**

> "Per visualizzare graficamente l'accesso alle risorse possiamo utilizzare dei grafi"
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#33-dining-philosophers-problem)*

Nel grafo di allocazione risorse, il deadlock si manifesta come un **ciclo**: ogni filosofo possiede una bacchetta e attende quella del vicino, creando una catena circolare di attese.

> "Tutti _deadlock_ sono rappresentati da **_cicli_**. Non tutti i _cicli_ rappresentano _deadlock_: solo quando le risorse coinvolte sono presenti in **_singola istanza_**"
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#33-dining-philosophers-problem)*

_**Introduzione ai Monitor**_

Per risolvere elegantemente questo problema, vengono introdotti i **Monitor**:

> "È un astrazione di alto livello che fornisce un meccanismo semplice e efficace per la sincronizzazione dei processi."
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#34-monitor)*

**Struttura di un Monitor:**

> "Possiamo rappresentarlo come un `abstract data type`, con variabili interne accessibili solo all'interno delle varie procedure."
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#34-monitor)*

```cpp
monitor monitorName{
	// shared variable declarations
	procedure P1(/*...*/) {
		// ...
	}
	// ...
	procedure Pn(/*...*/) {
		// ...
	}

	initialization_code(/*...*/) {
		// ...
	}
}
```

**Proprietà fondamentale dei Monitor:**

> "Per definizione **_un solo processo alla volta può essere attivo dentro al monitor_**, ciò rende le procedure _mutualmente esclusive_."
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#34-monitor)*

Questo significa che se un processo sta eseguendo una procedura del monitor, gli altri processi che vogliono accedervi devono attendere in una coda di ingresso (`entryQueue`).

_**Variabili di Condizione (Condition Variables)**_

La mutua esclusione da sola non è sufficiente per schemi di sincronizzazione complessi:

> "Questo meccanismo non è però ancora sufficientemente potente da poter risolvere alcuni schemi di sincronizzazione. Per migliorarne l'efficacia si definiscono quindi all'interno del `monitor` delle **_variabili di condizionamento_**."
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#34-monitor)*

**Operazioni sulle condition variables:**

> "Ad ogni _condition variable_ (`condition x;`) all'interno del `monitor` il sistema assegna due operazioni:
> - `x.wait()`: sospende l'operazione in attesa di una `x.signal()`
> - `x.signal()`: riprende l'esecuzione di uno dei processi che ha chiamato la `x.wait()`. Se nessun processo aveva chiamato la `x.wait()` **non ha alcun effetto**."
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#34-monitor)*

_**Signal and Wait vs Signal and Continue**_

L'introduzione delle condition variables comporta una scelta progettuale importante:

> "L'introduzione di queste _condition variable_ comporta alcune scelte progettuali da fare. Ad esempio, se `P` invoca la `x.signal()`, mentre c'era `Q` in `x.wait()`, quale dei due processi dovrà riprendere prima?"
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#34-monitor)*

**Due approcci possibili:**

> "Abbiamo due opzioni possibili:
> - **Signal and wait**: facciamo **riprendere** `Q` dandogli la precedenza su `P` anche se questo era in esecuzione.
> - **Signal and continue**: facciamo **_proseguire_** `P`, mettendo in attesa della mutua esclusione `Q`"
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#34-monitor)*

**Considerazioni:**

> "Ambedue le opzioni hanno dei pro e dei contro, e sta all'implementatore scegliere quale delle due opzioni selezionare."
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#34-monitor)*

- **Signal and wait**: garantisce che il processo risvegliato trovi le condizioni che ha atteso ancora valide, ma può causare più cambi di contesto
- **Signal and continue**: riduce i cambi di contesto ma il processo risvegliato deve verificare nuovamente le condizioni

_**Soluzione al Dining Philosophers con Monitor**_

Ecco l'implementazione che evita il deadlock:

```cpp
monitor DiningPhilosophers{
	enum {THINKING, HUNGRY, EATING} state[5];
	condition self[5];

	void pickup(int i) {
		state[i] = HUNGRY;
		test(i);
		if (state[i] != EATING)
			self[i].wait();
	}

	void putdown(int i) {
		state[i] = THINKING;
		test((i-1) % 5);
		test((i+1) % 5);
	}

	void test(int i) {
		if (	(state[(i-1) % 5] != EATING) &&
			(state[i] == HUNGRY) &&
			(state[(i+1) % 5] != EATING)) {
				state[i] = EATING;
				self[i].signal();
			}
	}

	initialization_code() {
		for (int i = 0; i < 5; ++i) {
			state[i] = THINKING;
		}
	}
}
```

**Come ogni processo utilizza il monitor:**

```cpp
DiningPhilosophers.pickup(i);

// EAT

DiningPhilosophers.putdown(i);
```

**Come questa soluzione evita il deadlock:**

> "Questo metodo, che non si basa direttamente sul possesso delle bacchette, ma piuttosto sul fatto che:
> - Se i miei vicini sono `THINKING` io posso mangiare
> - Se almeno uno dei miei vicini è `EATING` passo allo stato `HUNGRY` e mi metto in attesa, in quanto prima o poi restituirà la bacchetta che mi serve
> - Se entrambi i miei vicini sono `HUNGRY` significa che io possono mangiare"
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#341-soluzione-al-dining-philosophers-problem)*

**L'intuizione chiave:**

> "L'ultimo di questi casi avviene proprio perché quando qualcuno è `HUNGRY` **_non prende nessuna bacchetta finché non le può prendere entrambe_**."
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#341-soluzione-al-dining-philosophers-problem)*

Questo elimina la possibilità di deadlock perché non si forma mai una catena circolare di attese. Un filosofo passa allo stato `EATING` solo quando **entrambe** le bacchette sono disponibili, evitando di trattenere risorse mentre ne attende altre.

**Nota sulla starvation:**

Questa soluzione potrebbe comunque provocare _starvation_ se i due vicini di un filosofo mangiano continuamente, impedendogli di acquisire le bacchette. Tuttavia, è un problema diverso dal deadlock e può essere risolto con politiche di fairness.

_**Implementazione dei Monitor con Semafori**_

In linguaggi come C++, che non forniscono nativamente i monitor, possiamo implementarli usando i semafori:

> "In `C++` non sono forniti nativamente i monitor, però possiamo implementarli tramite l'utilizzo dei semafori."
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#342-implementazione-dei-monitor-con-i-semafori)*

**Semafori necessari per l'implementazione "signal and wait":**

```cpp
/**
* Semaforo di mutua esclusione sul monitor
*/
sem mutex = sem_ini(1);
/**
* Semaforo sul quale si mettono in attesa i processi
* che si sono sospesi all'interno del monitor
*/
sem next = sem_ini(0);
/**
* Indica il numero di processi in attesa sul semaforo `next`
*/
int next_count = 0;
```

**Struttura delle procedure del monitor:**

> "Ogni procedura sarà quindi rimpiazzata con il seguente segmento di codice:"
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#342-implementazione-dei-monitor-con-i-semafori)*

```cpp
wait(mutex);

// corpo della procedura

if (next_count > 0)
	signal(next);
else
	signal(mutex);
```

**Implementazione delle condition variables (signal&wait):**

```cpp
class condition{
	sem x_sem = sem_ini(0);
	int x_count = 0;

	public void wait() {
		x_count++;
		if (next_count > 0)
			signal(next);
		else
			signal(mutex);

		wait(x_sem);
		x_count--;
	}

	public void signal() {
		if (x_count > 0) {
			next_count++;
			signal(x_sem);
			wait(next);
			next_count--;
		}
	}
}
```

**Spiegazione del meccanismo:**

1. **`mutex`**: protegge l'accesso al monitor, garantendo la mutua esclusione
2. **`next`**: contiene i processi che hanno fatto `signal()` e stanno cedendo il controllo (signal and wait)
3. **`next_count`**: traccia quanti processi sono in attesa su `next`
4. **`x_sem`**: per ogni condition variable, il semaforo su cui attendono i processi che hanno chiamato `x.wait()`
5. **`x_count`**: numero di processi in attesa sulla condition variable

**Il flusso di `wait()`:**
- Incrementa il contatore dei processi in attesa
- Rilascia il monitor (signal su `next` se ci sono processi, altrimenti su `mutex`)
- Si blocca sul semaforo della condition variable
- Quando viene risvegliato, decrementa il contatore

**Il flusso di `signal()`:**
- Se ci sono processi in attesa sulla condition variable:
  - Incrementa `next_count` (si prepara ad attendere)
  - Risveglia un processo dalla condition variable
  - Si blocca su `next` (cedendo il monitor al processo risvegliato)
  - Quando viene risvegliato, decrementa `next_count`

_**Gestione delle Priorità nelle Condition Variables**_

Un aspetto importante è la gestione dell'ordine di risveglio:

> "Se più processi fossero in coda sulla stessa condizione `x`, dobbiamo decidere quale risvegliare alla chiamata `x.signal()`. Un algoritmo `FCFS` non è adeguato, in quanto può portare i processi a _starvation_."
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#342-implementazione-dei-monitor-con-i-semafori)*

**Soluzione con conditional-wait:**

> "Quello che quindi possiamo fare è implementare un costrutto di **conditional-wait** nella forma `x.wait(c)` dove `c` indica la **_priorità_** (dove `0` è la priorità massima, come se fosse il tempo). A questo punto gestiamo la coda secondo un ordine di priorità decrescente."
>
> *Fonte: [Sincronizzazione dei processi](./Sincronizzazione%20dei%20processi#342-implementazione-dei-monitor-con-i-semafori)*

_**Monitor per una Singola Risorsa**_

Un esempio pratico di monitor è l'allocatore di risorse:

```cpp
monitor ResourceAllocator{
	bool busy;
	condition x;

	void acquire(int time) {
		while (busy) {
			x.wait(time);
		}
		busy = true;
	}

	void release() {
		busy = false;
		x.signal();
	}

	intialization_code() {
		busy = false;
	}
}
```

Questo semplice monitor gestisce l'allocazione di una singola risorsa, garantendo mutua esclusione e permettendo l'uso di priorità temporali.

**Approfondimenti:**

I monitor rappresentano uno dei contributi più significativi alla programmazione concorrente, introdotti da Per Brinch Hansen e C.A.R. Hoare negli anni '70. La loro eleganza sta nel nascondere la complessità della sincronizzazione dietro un'interfaccia ad alto livello, riducendo drasticamente la probabilità di errori rispetto alla gestione manuale dei semafori.

**Vantaggi dei Monitor rispetto ai Semafori:**
- **Incapsulamento**: la logica di sincronizzazione è racchiusa nel monitor
- **Sicurezza**: impossibile dimenticare una `signal()` o fare una `wait()` su un semaforo sbagliato
- **Leggibilità**: il codice risulta più chiaro e manutenibile
- **Verifica formale**: più semplice dimostrare la correttezza

**Svantaggi:**
- **Overhead**: l'implementazione tramite semafori introduce overhead aggiuntivo
- **Non universali**: non tutti i linguaggi li supportano nativamente
- **Limiti espressivi**: alcuni pattern di sincronizzazione sono più naturali con semafori

I monitor sono nativamente implementati in linguaggi come **Java** (attraverso `synchronized` e `wait()`/`notify()`) e **C#** (con `lock` e `Monitor.Wait()`/`Monitor.Pulse()`), dimostrando la loro rilevanza pratica nei sistemi moderni.

Il problema dei Dining Philosophers, oltre ad essere un classico esempio didattico, rappresenta situazioni reali come la gestione di risorse condivise in database distribuiti, dove transazioni multiple competono per lock su record diversi, e la corretta gestione è cruciale per evitare deadlock che bloccherebbe l'intero sistema.

---
<div class="stop"></div>

---

## 10. Gestione della Memoria

### Domanda 10.1 (answered)

**Domanda**: Spiega la differenza tra rilocazione statica e rilocazione dinamica nella gestione della memoria, descrivendo il ruolo della MMU (Memory Management Unit) nella rilocazione dinamica. Illustra come avviene la traduzione degli indirizzi virtuali in indirizzi fisici, quali controlli di protezione vengono effettuati, e perché la rilocazione dinamica è preferibile nei sistemi che supportano meccanismi di swapping.

**Risposta:**

La gestione della memoria richiede un meccanismo di traduzione tra gli indirizzi virtuali utilizzati dai processi e gli indirizzi fisici effettivi della memoria. Questa traduzione può avvenire in due modalità fondamentalmente diverse: rilocazione statica e rilocazione dinamica.

_**Rilocazione Statica**_

Nella rilocazione statica, la traduzione degli indirizzi virtuali in indirizzi fisici avviene **in fase di caricamento** del processo in memoria. Quando il linker genera il modulo di caricamento, utilizza indirizzi virtuali. Prima che il processo inizi l'esecuzione, il **caricatore** si occupa di tradurre questi indirizzi sommando il contenuto del program counter:

> "Quando compiliamo un programma, il _linker_ si occuperà quindi di tradurre gli indirizzi simbolici, inseriti dal programmatore, in indirizzi virtuali. [...] Nel caso di **_rilocazioni statiche_**, il **caricatore** genererà l'**_immagine del processo_** traducendo gli indirizzi virtuali sommandogli il contenuto del _program counter_"
>
> *Fonte: [Gestione della memoria](./Gestione%20della%20memoria#21-rilocazione-statica-e-dinamica---mmu)*

Una volta completata questa fase, l'immagine del processo contiene **direttamente indirizzi fisici**. Questo approccio presenta un limite critico per i meccanismi di swapping:

> "La rilocazione statica funziona in sistemi _embedded_ ma non supporta meccanismi di _swapping_. Infatti, queste porzioni contengono al loro interno **indirizzi fisici**, richiedendo quindi di essere reinserite **_nelle medesime porzione di memoria nelle quali si trovavano prima dello swap_**, cosa tendenzialemnte impossibile."
>
> *Fonte: [Gestione della memoria](./Gestione%20della%20memoria#21-rilocazione-statica-e-dinamica---mmu)*

_**Rilocazione Dinamica**_

Per superare i limiti della rilocazione statica, si adotta la **rilocazione dinamica**, dove la traduzione degli indirizzi viene ritardata fino alla **fase di esecuzione**:

> "Per evitare gli inconvenienti legati alla rilocazione statica è necessario ritardarla, attuandola solamente in **fase di esecuzione**. Questa nuova rilocazione si chiama **_rilocazione dinamica_**."
>
> *Fonte: [Gestione della memoria](./Gestione%20della%20memoria#21-rilocazione-statica-e-dinamica---mmu)*

In questo approccio, quando un processo viene allocato in memoria, il caricatore trasferisce nelle locazioni della partizione scelta **direttamente i contenuti del modulo di caricamento**, mantenendo quindi gli **indirizzi virtuali**. Poiché questi indirizzi sono indipendenti dalle particolari locazioni fisiche, il processo può essere successivamente spostato in altre aree di memoria senza problemi.

_**Il Ruolo della MMU (Memory Management Unit)**_

La traduzione degli indirizzi da virtuali a fisici in fase di esecuzione è delegata a un componente hardware chiamato **MMU (Memory Management Unit)**, che si interpone tra la CPU e la memoria:

> "Si delega quindi alla fase di esecuzione l'azione di **_traduzione degli indirizzi da virtuali a fisici_**, attraverso l'`MMU` che interfaccia la **CPU** alla memoria. In questa nuova architettura, la **CPU** **_utilizza solo ed esclusivamente indirizzi virtuali_**."
>
> *Fonte: [Gestione della memoria](./Gestione%20della%20memoria#21-rilocazione-statica-e-dinamica---mmu)*

La funzione di rilocazione, nel caso più semplice di spazio virtuale unico, è una funzione di somma che lega ogni indirizzo virtuale $x$ a un indirizzo fisico $y$:

$$y = f(x) \approx I + x$$

dove $I$ rappresenta l'indirizzo iniziale della partizione di memoria nella quale il processo è caricato.

_**Traduzione degli Indirizzi e Controlli di Protezione**_

La MMU non si limita a tradurre gli indirizzi, ma effettua anche **controlli di protezione** fondamentali. Prima di tradurre un indirizzo virtuale $x$, verifica che questo rientri nei limiti dello spazio virtuale del processo:

> "La `MMU`, oltre a tradurre gli indirizzi da virtuali a fisici, si occupa anche di effettuare dei **_controlli di protezione_**. Prima di tradurre un indirizzo virtuale, si effettua infatti in controllo che $x \in [0, 6140)$. Se questo non fosse vero allora la `MMU` **_genera un eccezione di protezione_**, tipicamente il _segmentation fault_, che il sistema operativo dovrà gestire opportunamente."
>
> *Fonte: [Gestione della memoria](./Gestione%20della%20memoria#21-rilocazione-statica-e-dinamica---mmu)*

Questo meccanismo di protezione impedisce a un processo di accedere ad aree di memoria al di fuori del proprio spazio virtuale, garantendo l'isolamento tra processi.

_**Vantaggi della Rilocazione Dinamica per lo Swapping**_

La rilocazione dinamica è preferibile nei sistemi che supportano meccanismi di swapping per diverse ragioni fondamentali:

1. **Flessibilità di rilocazione**: Poiché l'immagine del processo mantiene indirizzi virtuali, può essere scaricata sulla swap-area e successivamente ricaricata in **qualsiasi area libera** della memoria fisica, non necessariamente nella stessa posizione precedente.

2. **Supporto alla multiprogrammazione**: Permette di gestire efficacemente più processi in memoria, potendo spostare dinamicamente i processi per ottimizzare l'utilizzo dello spazio disponibile.

3. **Gestione trasparente**: Il processo non è consapevole della sua posizione fisica in memoria né degli eventuali spostamenti, rendendo lo swapping completamente trasparente all'applicazione.

4. **Efficienza della memoria**: Facilita tecniche come il compattamento della memoria per eliminare la frammentazione esterna, poiché i processi possono essere facilmente rilocati.

**Approfondimenti:**

La rilocazione dinamica costituisce la base per tecniche di gestione della memoria più avanzate come la segmentazione e la paginazione. Nel caso di spazio virtuale segmentato, la MMU deve gestire traduzioni più complesse mantenendo informazioni per ciascun segmento. Con la paginazione, la funzione di rilocazione diventa una vera e propria tabella di corrispondenza tra pagine virtuali e frame fisici, eliminando completamente il problema della frammentazione esterna pur introducendo un overhead nella gestione delle tabelle delle pagine.

---

### Domanda 10.2 (answered)

**Domanda**: Spiega la tecnica della paginazione a domanda, descrivendo come funziona il meccanismo di page fault e la gestione della swap-area. Illustra nel dettaglio gli algoritmi di rimpiazzamento delle pagine FIFO, LRU e Second-Chance (Clock Algorithm), confrontandone vantaggi, svantaggi e complessità implementativa. Spiega inoltre il fenomeno del thrashing e il ruolo dei bit di controllo U (uso) e M (modifica) negli algoritmi di rimpiazzamento.

**Risposta:**

La paginazione a domanda rappresenta una delle tecniche più importanti e utilizzate nella gestione della memoria virtuale, permettendo di eseguire processi la cui memoria virtuale eccede la capacità della memoria fisica disponibile.

_**Tecnica della Paginazione a Domanda**_

La paginazione a domanda si basa sul concetto fondamentale che un processo non necessita sempre di tutte le sue pagine contemporaneamente in memoria:

> "È una tecnica di _rilocazione dinamica_, con _allocazione non contigua_ in uno _spazio virtuale unico_ a _caricamento a domanda_. Basandosi sui concetti della _paginazione_, sfrutta il fatto che un processo **non necessita sempre di tutte le sue pagine**."
>
> *Fonte: [Gestione della memoria](./Gestione%20della%20memoria#36-paginazione-a-domanda)*

La tecnica è analoga alla segmentazione a domanda e rappresenta **la soluzione più utilizzata anche nei sistemi paginati**. Durante l'esecuzione di un processo è possibile mantenere in memoria **soltanto una parte del suo spazio virtuale**, ovvero solo quella necessaria al punto corrente dell'esecuzione.

_**Meccanismo di Page Fault**_

Il meccanismo hardware che supporta la paginazione a domanda introduce nuovi bit di controllo nel descrittore di ogni pagina:

> "Il meccanismo hardware è molto simile a quello della _segmentazione a domanda_, e aggiunge ai campi di lettura `R` e scrittura `W`:
> - `P`: _bit di presenza_, caratterizza la presenza della pagina in memoria
> - `U`: _bit di uso_, utilizzato dagli algoritmi di rimpiazzamento
> - `W`: _bit di scrittura_, utilizzato dagli algoritmi di rimpiazzamento"
>
> *Fonte: [Gestione della memoria](./Gestione%20della%20memoria#36-paginazione-a-domanda)*

Quando un processo viene creato, tutti i descrittori delle sue pagine hanno `P == 0`. In questo stato, il campo che normalmente contiene l'indice del frame fisico contiene invece **l'indirizzo della pagina sul disco** nella swap-area.

Il processo di gestione del page fault avviene nel seguente modo:

> "Quando il processo farà l'accesso ad un indirizzo `x`, che appartiene alla pagina `pg`, si verificherà innanzitutto che `pg` sia tra le pagine del processo, e successivamente, il valore di `P` del descrittore associato. Se `P == 0` si genera `page fault`, fermando l'esecuzione del processo."
>
> *Fonte: [Gestione della memoria](./Gestione%20della%20memoria#36-paginazione-a-domanda)*

La routine di gestione del page fault esegue i seguenti passi:

1. **Verifica disponibilità frame**: controlla se esistono frame liberi in memoria tramite la variabile `pf`
2. **Eventuale rimpiazzamento**: se non ci sono frame liberi, utilizza un algoritmo di rimpiazzamento
3. **Caricamento dalla swap-area**: consulta l'indirizzo contenuto nel descrittore e copia il contenuto dal disco alla RAM
4. **Blocco del processo**: durante il trasferimento, il processo viene tipicamente portato nello stato `bloccato`
5. **Aggiornamento tabella**: una volta completato il caricamento, aggiorna il descrittore settando `P = 1` e inserendo l'indirizzo del frame
6. **Ripresa esecuzione**: il processo torna prima in `pronto` poi in `esecuzione`, con il PC decrementato di uno per rieseguire l'istruzione che ha causato il page fault

Questo meccanismo è completamente **trasparente al processo**.

_**Gestione della Swap-Area**_

La swap-area è una porzione dedicata della memoria secondaria (disco) utilizzata per contenere le pagine non presentemente in RAM:

> "Nel caso di sistemi che utilizzano la _paginazione a domanda_ anche la _swap-area_ è divisa in **settori fisici di dimensioni uguali a quelle delle pagine**."
>
> *Fonte: [Gestione della memoria](./Gestione%20della%20memoria#36-paginazione-a-domanda)*

Questa organizzazione permette una corrispondenza diretta tra pagine virtuali e settori della swap-area, semplificando notevolmente la gestione rispetto alle tecniche di segmentazione.

_**Algoritmi di Rimpiazzamento delle Pagine**_

Quando non ci sono frame liberi disponibili, è necessario selezionare una pagina da rimuovere dalla memoria. Esistono diversi algoritmi per questa scelta:

**Algoritmo Ottimo (Teorico)**

L'algoritmo ottimo rimpiazza le pagine che non verranno più riferite o che verranno utilizzate più lontano nel futuro:

> "**Ottimo**: rimpiazza le pagine che non verranno più riferite o che verranno utilizzate più nel futuro rispetto alle altre. (Teorico e irrealizzabile)"
>
> *Fonte: [Gestione della memoria](./Gestione%20della%20memoria#36-paginazione-a-domanda)*

Questo algoritmo è **irrealizzabile** in pratica poiché richiederebbe la conoscenza anticipata del futuro comportamento del processo, ma serve come riferimento teorico per valutare l'efficienza degli altri algoritmi.

**Algoritmo FIFO (First In First Out)**

L'algoritmo FIFO implementa una politica semplice:

> "**FIFO**: rimpiazza la pagina **_locale_** (aka del processo in esecuzione) che è _da più tempo in memoria_. Il rimpiazzamento è locale per evitare di propagare in futuro l'errore anche ad altri processi."
>
> *Fonte: [Gestione della memoria](./Gestione%20della%20memoria#36-paginazione-a-domanda)*

**Vantaggi:**
- Semplicità implementativa
- Facile da comprendere e debuggare
- Overhead minimo di gestione

**Svantaggi:**
- Non considera la frequenza d'uso delle pagine
- Può rimpiazzare pagine frequentemente utilizzate solo perché sono state caricate per prime
- Non ottimale in termini di riduzione dei page fault
- Il rimpiazzamento è locale al processo per evitare di propagare errori

**Algoritmo LRU (Least Recently Used)**

L'algoritmo LRU rappresenta un miglioramento significativo rispetto al FIFO:

> "**LRU**: rimpiazza la pagina non utilizzata da più tempo. È più efficiente rispetto al **FIFO**, ma anche più difficile da realizzare. Infatti è necessario modificare opportunamente la tabella delle pagine per introdurre un campo dove salvare un _timestamp_, ed è necessario consultarli tutti per recuperare il più vecchio e aggiornare i nuovi. Tra gli algoritmi implementabili è quello che riduce al minimo il **_trashing_**, ovvero diminuisce al minimo il `page-fault`."
>
> *Fonte: [Gestione della memoria](./Gestione%20della%20memoria#36-paginazione-a-domanda)*

**Vantaggi:**
- Più efficiente del FIFO nella riduzione dei page fault
- Si basa sul principio di località temporale
- Riduce al minimo il thrashing tra gli algoritmi implementabili
- Buone prestazioni nella maggior parte dei casi pratici

**Svantaggi:**
- Complessità implementativa elevata
- Richiede timestamp per ogni pagina
- Necessita di scansionare tutte le pagine per trovare la vittima
- Overhead significativo per mantenere aggiornati i timestamp ad ogni accesso
- Costi hardware o software per l'implementazione

**Algoritmo Second-Chance (Clock Algorithm)**

Per superare le limitazioni implementative dell'LRU, UNIX utilizza un'approssimazione chiamata Second-Chance o Clock Algorithm:

> "L'algoritmo utilizzato da `UNIX` è un'approssimazoine dell'algoritmo `LRU`, noto come **_second-chance_** o **_clock algorithm_**."
>
> *Fonte: [Gestione della memoria](./Gestione%20della%20memoria#36-paginazione-a-domanda)*

Il funzionamento dell'algoritmo è il seguente:

> "La tabella delle pagine fisiche viene gestita come un array circolare. Viene mantenuta una variabile `vittima`. che consiste in un puntatore contenente l'indice della pagina fisica successiva a quella che è stata rimpiazzata per ultima. Al prossimo `page-fault` si inizia a verificare la pagina il cui indice è contenuto nella variabile `vittima`."
>
> *Fonte: [Gestione della memoria](./Gestione%20della%20memoria#36-paginazione-a-domanda)*

La logica di selezione esamina il bit `U`:

> "In particolare si esamina il bit `U` della pagina associata:
> - `U == 0`: la pagina viene scelta per il rimpiazzamento
> - `U == 1`: si azzera il bit e si **incrementa la variabile** `vittima`, andando ad esaminare la pagina successiva, riverificando il bit `U`."
>
> *Fonte: [Gestione della memoria](./Gestione%20della%20memoria#36-paginazione-a-domanda)*

L'incremento è circolare: `vittima = (vittima + 1) % M`, garantendo che si troverà **certamente** una pagina con `U == 0`. Nel caso peggiore, si selezionerà la pagina iniziale dopo aver azzerato tutti i bit `U` durante la scansione completa.

**Vantaggi:**
- Complessità implementativa ragionevole
- Buon compromesso tra efficienza e semplicità
- Richiede solo un bit `U` per pagina invece di timestamp
- Approssima bene il comportamento LRU
- Garantisce terminazione

**Svantaggi:**
- Non ottimale come il vero LRU
- Nel caso peggiore deve scansionare tutte le pagine
- Può dare una "seconda possibilità" a pagine poco utilizzate

_**Ottimizzazione con il Bit di Modifica (M)**_

Per ridurre il numero di trasferimenti tra memoria e disco, l'algoritmo può considerare anche il bit `M`:

> "Per ridurre il numero di trasferimenti tra memoria e disco, viene spesso preso in considerazione anche il valore di `M`. Le pagine sono quindi classificate prima rispetto al valore di `U`, e successivamente si privilegiano le pagine con `M == 0`"
>
> *Fonte: [Gestione della memoria](./Gestione%20della%20memoria#36-paginazione-a-domanda)*

Questo perché una pagina con `M == 0` (non modificata) non richiede di essere riscritta sulla swap-area, risparmiando un'operazione di I/O costosa.

_**Il Ruolo dei Bit di Controllo U e M**_

I bit di controllo sono fondamentali per l'efficienza degli algoritmi di rimpiazzamento:

> "Al fine di facilitare l'implementazione di algoritmi di rimpiazzamento efficienti, sono spesso disponibili ulteriori _bit di controllo_ nel descrittore di un segmento. Ad esempio abbiamo i bit `U` (_uso_) e `M` (_modifica_), che possono essere letti e azzerati via software, e che vengono settati automaticamente via _hardware_ ogni volta che viene fatto un riferimento/modifica al segmento."
>
> *Fonte: [Gestione della memoria](./Gestione%20della%20memoria#34-segmentazione-a-domanda)*

**Bit U (Uso):**
- Settato automaticamente dall'hardware quando la pagina viene referenziata
- Può essere letto e azzerato via software
- Utilizzato per valutare la frequenza di utilizzo di ogni pagina
- Fondamentale per gli algoritmi Second-Chance e LRU

**Bit M (Modifica/Dirty Bit):**
- Settato automaticamente quando la pagina viene modificata
- Permette di capire se una pagina è stata scritta
- Evita riscritture non necessarie sulla swap-area (se `M == 0` la pagina su disco è già aggiornata)
- Ottimizza le prestazioni riducendo le operazioni di I/O

_**Il Fenomeno del Thrashing**_

Il thrashing è un fenomeno critico che si verifica quando il sistema passa più tempo a gestire page fault che ad eseguire realmente i processi:

> "Tra gli algoritmi implementabili è quello che riduce al minimo il **_trashing_**, ovvero diminuisce al minimo il `page-fault`."
>
> *Fonte: [Gestione della memoria](./Gestione%20della%20memoria#36-paginazione-a-domanda)*

Il thrashing si manifesta quando:
- Troppi processi sono caricati in memoria contemporaneamente
- Ciascun processo non ha sufficienti pagine del proprio working-set in memoria
- Si generano page fault frequenti
- Il sistema passa la maggior parte del tempo a spostare pagine tra RAM e disco invece di eseguire codice utile

Per mitigare il thrashing, i sistemi operativi implementano strategie come:
- Controllo del grado di multiprogrammazione
- Algoritmi di rimpiazzamento efficienti (come LRU)
- Gestione del working-set
- Meccanismi di swapping dei processi (come descritto nella Domanda 10.1)

**Approfondimenti:**

La paginazione a domanda rappresenta il fondamento della memoria virtuale moderna. UNIX implementa meccanismi sofisticati per gestire la memoria, come il `pagedaemon` (pid=2) che monitora periodicamente il numero di frame liberi e interviene quando scende sotto la soglia `lotsfree`. Se la situazione peggiora ulteriormente e il numero di frame liberi scende sotto `minfree`, interviene lo `swapper` per trasferire interi processi dalla memoria centrale alla swap-area. Questa gestione a più livelli (lotsfree > desfree > minfree) permette di mantenere il sistema reattivo anche sotto carico elevato, evitando situazioni di thrashing critico.

---
<div class="stop"></div>

---

## 11. Gestione delle Periferiche (I/O)

### Domanda 11.1 (answered)

**Domanda**: Spiega il concetto di "processo esterno" nella gestione delle periferiche, descrivendo come avviene la sincronizzazione tra processi applicativi e dispositivi di I/O. Illustra il ruolo dei semafori nella gestione delle interruzioni, analizza il problema dell'inefficienza del trasferimento byte per byte confrontandolo con soluzioni più efficienti, e descrivi la struttura del descrittore di un dispositivo e le sue funzioni principali.

**Risposta:**

La gestione delle periferiche di I/O rappresenta uno degli aspetti più critici di un sistema operativo, richiedendo meccanismi sofisticati di sincronizzazione tra l'attività della CPU e quella dei dispositivi esterni. Il concetto di "processo esterno" è fondamentale per comprendere questa interazione.

_**Il Concetto di Processo Esterno**_

Un dispositivo di I/O, insieme al suo controllore, esegue permanentemente una sequenza di azioni che può essere equiparata all'esecuzione di un processo. Tuttavia, a differenza dei processi tradizionali, questa sequenza non corrisponde all'esecuzione di un programma software, ma piuttosto a **una sequenza di azioni cablate eseguite in parallelo al flusso della CPU**:

> "Ogni dispositivo esegue **_permanentemente una sequenza di azioni equiparabile con l'esecuzione di un processo_**. La differenza però è che il processo non corrisponde all'eseuzione di un programma, ma piuttosto ad una **sequenza di azioni cablate eseguite in parallelo al flusso della CPU**. Per distiguenre questo tipo di 'processi' li identificheremo con il termine di **_processi esterni_**."
>
> *Fonte: [Gestione delle Periferiche (I/O)](./Gestione%20delle%20Periferiche%20(IO)#22-processi-esterni)*

Il comportamento tipico di un processo esterno può essere schematizzato come segue:

```c
while (true) {
	// Attendo l'invio di un comando
	while (Start == 0);

	// Eseguo il comando
	executeComand();

	// Registro l'evento
	setComandOutputAndFlag();
}
```

Normalmente un dispositivo si trova in stand-by, aspettando che il bit di start del registro di controllo venga settato. Una volta attivato, esegue il comando e alla fine registra l'evento ponendo a uno il relativo bit di flag.

_**Sincronizzazione tramite Semafori**_

La gestione dei dispositivi di I/O diventa quindi un problema di **sincronizzazione tra processi applicativi e processi esterni**. Per evitare l'attesa attiva (busy waiting), che sarebbe estremamente inefficiente, si utilizza un meccanismo basato su semafori:

> "Per forzare la sospensione del processo in attesa su un flag è possibile **associare al dispositivo un semaforo inizializzato a zero** e sostituire il ciclo di attesa attiva una `wait` su tale semaforo."
>
> *Fonte: [Gestione delle Periferiche (I/O)](./Gestione%20delle%20Periferiche%20(IO)#22-processi-esterni)*

Il comportamento del processo applicativo diventa:

```cpp
semaforo dato_disponibile = 0;

for (int i = 0; i < n; ++i) {
	// preparo il comando
	executeComand();
	wait(dato_disponibile);
	// verifico l'esito
}
```

Poiché il processo si sospende sulla `wait`, non può più controllare direttamente la disponibilità del dato. Sarà quindi il **controllore del dispositivo** a segnalare l'evento risvegliando il processo applicativo attraverso un'interruzione e la relativa funzione di gestione (`inth`) che eseguirà una `signal` sul semaforo.

_**Inefficienza del Trasferimento Byte per Byte**_

Il primo approccio alla gestione delle interruzioni presenta un problema di efficienza significativo. Nel trasferimento byte per byte, ogni singolo byte trasferito richiede:

1. Schedulazione del processo applicativo
2. Caricamento del dato
3. Attivazione del dispositivo
4. Sospensione del processo
5. Context switch verso un altro processo
6. Interruzione e gestione
7. Risveglio del processo per il byte successivo

Come illustrato dal diagramma temporale:

> "L'esempio appena fatto questo è altamente inefficiente, per trasferire $n$ byte abbiamo infatti un processo che per $n$ volte viene schedulato solo per poi tornare in attesa."
>
> *Fonte: [Gestione delle Periferiche (I/O)](./Gestione%20delle%20Periferiche%20(IO)#22-processi-esterni)*

Questo comporta un numero elevatissimo di context switch, con un overhead inaccettabile soprattutto per trasferimenti di grandi quantità di dati.

_**Soluzione Efficiente: Delega alla Routine di Interruzione**_

Una soluzione molto più efficiente consiste nel delegare alla routine di interruzione (`inth`) l'operazione di gestire i singoli byte:

> "Come suggerisce l'immagine [...] possiamo delegare ad `inth` l'operazione di caricare il singolo byte affinché `PE` lo possa leggere. In questo caso `PI` non carica il singolo dato, ma piuttosto la locazione di memoria dove si trovano tutti i dati. `inth` si occuperà di accedervi e fornire i singoli byte a `PE`. Questo ci permette di **almeno dimezzare il numero di cambi di contesto** nello stesso arco temporale."
>
> *Fonte: [Gestione delle Periferiche (I/O)](./Gestione%20delle%20Periferiche%20(IO)#22-processi-esterni)*

In questo approccio, il processo applicativo fornisce al dispositivo:
- L'indirizzo del buffer contenente i dati
- Il numero totale di byte da trasferire

La routine `inth` gestisce autonomamente i trasferimenti intermedi, risvegliando il processo applicativo solo al completamento dell'intera operazione o in caso di errore.

_**Struttura del Descrittore di un Dispositivo**_

Per implementare questa comunicazione efficiente, è necessaria una struttura dati condivisa tra il processo applicativo e la routine di interruzione: il **descrittore del dispositivo**. Insieme alle funzioni di accesso, costituisce il **device driver**:

> "L'unione tra il _descrittore del dispositivo_ e le _funzioni di accesso_ costituisce il **_device driver_**."
>
> *Fonte: [Gestione delle Periferiche (I/O)](./Gestione%20delle%20Periferiche%20(IO)#221-descrittore-e-driver-di-un-dispositivo)*

Il descrittore del dispositivo ha due scopi principali:
1. Nascondere al proprio interno tutte le informazioni associate al dispositivo
2. Consentire la comunicazione di informazioni tra processo applicativo e dispositivo (in entrambe le direzioni)

I campi tipicamente presenti in un descrittore sono:

> "I _descrittori_ possono avere parametri diversi, ma in generale i seguenti dati sono sempre presenti:
> - **Indirizzo registro di controllo**
> - **Indirizzo registro di stato**
> - **Indirizzo registro dati**
> - **Stato di sincronizzazione**: `dato_disponibile`
> - **Contatore dati da trasferire**: `contatore`
> - **Indirizzo del buffer**: `puntatore`
> - **Risultato del trasferimento**: `esito`"
>
> *Fonte: [Gestione delle Periferiche (I/O)](./Gestione%20delle%20Periferiche%20(IO)#221-descrittore-e-driver-di-un-dispositivo)*

Le **funzioni del descrittore** includono:

1. **Memorizzazione indirizzi hardware**: mantiene gli indirizzi dei registri del controllore (controllo, stato, dati)

2. **Sincronizzazione**: il semaforo `dato_disponibile` permette al processo di sospendersi e alla routine di interruzione di risvegliarlo

3. **Gestione del trasferimento**: i campi `contatore` e `puntatore` permettono alla routine `inth` di gestire autonomamente i trasferimenti multipli

4. **Comunicazione dell'esito**: il campo `esito` permette di comunicare al processo applicativo il risultato dell'operazione (successo o codice d'errore)

_**Implementazione della Funzione Read**_

Un esempio di funzione del driver che utilizza il descrittore:

```c
int read(int disp, char* pbuf, int cont) {
	descrittore[disp].contatore = cont;
	descrittore[disp].puntatore = pbuf;
	
	<attivo il dispositivo settando `bitStart = 1`>
	
	descrittore[disp].dato_disponibile.wait();
	
	if (descrittore[disp].esito == ERROR_CODE) {
		return -1;
	}
	else {
		return (cont - descrittore[disp].contatore);
	}
}
```

La routine di interruzione gestisce i trasferimenti intermedi decrementando `contatore` e incrementando `puntatore`, riattivando il dispositivo per il byte successivo finché `contatore != 0`. Solo alla fine risveglia il processo applicativo.

**Approfondimenti:**

Questo meccanismo di sincronizzazione tra processi applicativi e processi esterni rappresenta un esempio fondamentale di comunicazione tra componenti hardware e software. La struttura a livelli del sottosistema I/O, con l'interfaccia device-independent che nasconde i dettagli specifici dei dispositivi, permette di scrivere applicazioni portabili. In UNIX, questa astrazione è realizzata trattando i dispositivi come file, rendendo le operazioni di I/O uniformi indipendentemente dal tipo di periferica utilizzata.

---
<div class="stop"></div>

---

## 12. Il File System

### Domanda 12.1 (answered)

**Domanda**: Confronta i metodi di allocazione fisica dei file su disco: allocazione contigua, allocazione a lista concatenata e allocazione a indice. Per ciascun metodo, descrivi il funzionamento, i vantaggi e gli svantaggi in termini di frammentazione, costo di ricerca dei blocchi e supporto all'accesso diretto. Spiega poi in dettaglio il metodo di allocazione utilizzato in UNIX, illustrando la struttura degli i-node, l'organizzazione del disco in regioni (BootBlock, SuperBlock, i-List, DataBlocks), e il meccanismo di indirizzamento multiplo che permette di gestire file di dimensioni variabili.

**Risposta:**

L'organizzazione fisica dei file su disco è uno degli aspetti più critici nella progettazione di un file system. I diversi metodi di allocazione rappresentano differenti compromessi tra efficienza, semplicità e funzionalità. L'analisi di queste tecniche ci permette di comprendere le scelte implementative dei sistemi moderni come UNIX.

_**Contesto: Il Livello di Organizzazione Fisica**_

Prima di analizzare i metodi specifici, è importante comprendere il ruolo di questo livello:

> "Il terzo strato è il livello di **Organizzazione Fisica**, che ha come compito primario quello di allocare i record logici di ogni file nell'unità di memorizzazione secondaria."
>
> *Fonte: [Il file system](./Il%20file%20system#23-organizzazione-fisica)*

In questo livello, lo spazio disponibile viene visto come:

> "In questo livello lo spazio disponibile per l'allocazione sul disco viene visto come un insieme di **_blocchi fisici_**."
>
> *Fonte: [Il file system](./Il%20file%20system#23-organizzazione-fisica)*

Il **blocco fisico** rappresenta l'unità fondamentale:

> "Il **blocco fisico** è l'unità di allocazione e di trasferimento delle informazioni sul dispositivo, e gli è associata una posizione particolare sulla superficie del disco e una dimensione costante $D_b$, generalmente **molto maggiore** della dimensione di un record logico $D_r$."
>
> *Fonte: [Il file system](./Il%20file%20system#23-organizzazione-fisica)*

In un singolo blocco possiamo contenere $N_b = \frac{D_b}{D_r}$ record logici.

_**1. Allocazione Contigua**_

L'allocazione contigua rappresenta il metodo più semplice concettualmente:

> "Questa tecnica mappa ogni file su un insieme di **blocchi fisicamente contigui**."
>
> *Fonte: [Il file system](./Il%20file%20system#231-allocazione-contigua)*

**Funzionamento:**

> "Per fare ciò è innanzitutto necessario calcolare il numero di blocchi necessari per salvare un file, e successivamente si procede a cercare una partizione di blocchi liberi abbastanza grande da contenere il file."
>
> *Fonte: [Il file system](./Il%20file%20system#231-allocazione-contigua)*

**Vantaggi:**

L'allocazione contigua offre vantaggi significativi in termini di prestazioni:

> "Questo tipo di allocazione ha diversi vantaggi, tra i quali:
> - **Semplice ricerca di un blocco**: noto l'indirizzo del primo blocco del file $(B)$ e l'`IO pointer` di un dato record $(i)$, un dato blocco si trova in posizione: $B + \Big\lfloor\frac{i}{N_b}\Big\rfloor$
> - **Possibilità di accesso sequenziale e diretto allo stesso costo**"
>
> *Fonte: [Il file system](./Il%20file%20system#231-allocazione-contigua)*

La formula $B + \lfloor\frac{i}{N_b}\rfloor$ permette di calcolare direttamente la posizione di qualsiasi blocco, rendendo l'accesso diretto molto efficiente. Inoltre, l'accesso sequenziale beneficia della contiguità dei blocchi, riducendo i movimenti della testina del disco.

**Svantaggi:**

Tuttavia, gli svantaggi sono significativi e limitanti:

> "Tuttavia ha anche molti svantaggi:
> - **Frammentazione Esterna**: man mano che si riempie il disco si rimangono zone contigue sempre più piccole, rendendo necessario il _compattamento_
> - **Costo della ricerca dello spazio libero**: è necessario analizzare tutto il disco finché non si trova uno spazio sufficientemente grande per conservare il file
> - **Crescita delle dimensioni del file**: qualora il file aumentasse di dimensione potrebbe essere necessario _rilocare l'intero file_ se non fossero disponibili ulteriori blocchi contigui liberi."
>
> *Fonte: [Il file system](./Il%20file%20system#231-allocazione-contigua)*

La frammentazione esterna è particolarmente problematica: dopo ripetute operazioni di creazione ed eliminazione di file, il disco si frammenta in piccole zone contigue, rendendo difficile allocare file di grandi dimensioni anche se lo spazio totale libero sarebbe sufficiente. Il compattamento richiede di spostare tutti i file per raccogliere lo spazio libero, operazione estremamente costosa.

_**2. Allocazione a Lista Concatenata**_

Per superare i limiti dell'allocazione contigua, si adotta l'allocazione a lista concatenata:

> "Questa tecnica memorizza ogni file in un insieme di **blocchi non contigui** organizzati in una **lista concatenata**."
>
> *Fonte: [Il file system](./Il%20file%20system#232-allocazione-a-lista-concatenata)*

**Funzionamento:**

> "L'organizzazione è ancora di tipo sequenziale, ma stavolta blocchi successivi non devono necessariamente essere vicini."
>
> *Fonte: [Il file system](./Il%20file%20system#232-allocazione-a-lista-concatenata)*

Ogni blocco contiene, oltre ai dati, un puntatore al blocco successivo nella catena. Il descrittore del file mantiene solo il puntatore al primo blocco.

**Vantaggi:**

> "Cio ci fornisce diversi vantaggi:
> - **Non soffre di frammentazione esterna**
> - **Minor costo di allocazione**: non è più necessario scorrere l'intero disco per allocare un file
> - **Accesso sequenziale a basso costo**: dato il primo blocco abbiamo il puntatore al successivo"
>
> *Fonte: [Il file system](./Il%20file%20system#232-allocazione-a-lista-concatenata)*

L'eliminazione della frammentazione esterna è un vantaggio cruciale: qualsiasi blocco libero può essere utilizzato, indipendentemente dalla sua posizione.

**Svantaggi:**

> "Vi sono ancora diversi svantaggi:
> - **Broken Links**: se un blocco viene danneggiato, non solo perdiamo le informazioni che contiene ma anche le informazioni sul successivo, rendendo irraggiungibile il resto del file
> - **Overhead spaziale**: dover salvare i puntatori diminuisce lo spazio effettivo utilizzabile
> - **Accesso diretto oneroso**: in questo caso, per accedere all'$i$-esimo blocco sono necessari $\Big\lfloor\frac{i}{N_b}\Big\rfloor$ accessi al disco
> - **Costo della ricerca di un blocco**"
>
> *Fonte: [Il file system](./Il%20file%20system#232-allocazione-a-lista-concatenata)*

Il problema dei broken links è particolarmente grave. Una soluzione parziale consiste nell'utilizzare liste doppiamente concatenate:

> "Per arginare il problema dei _broken links_ è possibile realizzare una _double-linked-list_, andando a sacrificare un altra porzione del blocco per salvare un secondo puntatore che punta al blocco precedente."
>
> *Fonte: [Il file system](./Il%20file%20system#232-allocazione-a-lista-concatenata)*

Questo aumenta la tolleranza ad errore da zero a uno, ma raddoppia l'overhead spaziale per i puntatori.

**Ottimizzazione: File Allocation Table (FAT):**

Alcuni sistemi operativi, come Windows, introducono una struttura dati aggiuntiva:

> "Alcuni sistemi operativi, ad esempio `Windows`, affronato il problema affiancando ad una allocazione basata su una _linked-list_ una struttura dati nella quale viene descritta la **mappa di allocazione di tutti i blocchi**, detta **_File Allocation Table_** `FAT`."
>
> *Fonte: [Il file system](./Il%20file%20system#232-allocazione-a-lista-concatenata)*

La FAT risolve elegantemente diversi problemi:

> "Essa contiene tanti elementi quanti sono i blocchi del dispositivo, ed in ogniuno è presente un valore che indica:
> - Se il blocco è libero
> - L'indice dell'elemento della tabella che rappresenta il blocco successivo nella lista"
>
> *Fonte: [Il file system](./Il%20file%20system#232-allocazione-a-lista-concatenata)*

Vantaggi della FAT:

> "In questo modo, anche in perdita di concatenamento è possibile effettuare il recupero del puntatore perso accedendo alla `FAT`. La `FAT` può anche essere copiata in memoria centrale o in una cache, così da velocizzare notevolmente l'accesso diretto."
>
> *Fonte: [Il file system](./Il%20file%20system#232-allocazione-a-lista-concatenata)*

_**3. Allocazione a Indice**_

L'allocazione a indice rappresenta un approccio radicalmente diverso:

> "Questo tipo di allocazione si basa ancora sull'utilizzo di blocchi non contigui per l'allocazione di un file, ma in questo caso ad ogni file è associato un blocco **indice** in cui sono contenuti _tutti gli indirizzi dei blocchi su cui è allocato il file_."
>
> *Fonte: [Il file system](./Il%20file%20system#233-allocazione-a-indice)*

**Funzionamento:**

Il descrittore del file punta a un blocco indice che contiene un array di puntatori, ciascuno dei quali punta a un blocco dati del file. Questo permette di accedere direttamente a qualsiasi blocco conoscendo il suo indice.

**Vantaggi:**

> "Questo tipo di allocazione ha gli stessi vantaggi dell'allocazione tramite _linked-list_ introducendo però anche:
> - **Possibilità di accesso diretto**
> - **Maggiore velocità di accesso**"
>
> *Fonte: [Il file system](./Il%20file%20system#233-allocazione-a-indice)*

L'accesso diretto è efficiente perché non richiede di attraversare una lista: è sufficiente leggere il blocco indice e poi accedere direttamente al blocco desiderato.

**Svantaggi - Il Problema della Scalabilità:**

> "Il grande svantaggio di questa tecnica è la **_non scalabilità_**, infatti se il file crescesse tanto da necessitare più blocchi di quelli indicizzabili da un unico blocco avremmo un problema."
>
> *Fonte: [Il file system](./Il%20file%20system#233-allocazione-a-indice)*

Il calcolo della dimensione massima è matematicamente determinabile:

> "In particolare dato un disco di capacità $C$, otteniamo quanti bit sono necessari per indicizzare tutti i blocchi di dimensione $D_b$:
> $$\quad I = \log{(\frac{C}{D_b})} \text{ indici}$$"
>
> *Fonte: [Il file system](./Il%20file%20system#233-allocazione-a-indice)*

Conseguentemente:

> "Ciò significa che in un singolo blocco è possibile salvare al massimo $\frac{D_b}{I}$ indici. Per ottenere la dimensione massima di un file è quindi sufficiente moltiplicare per ogni indice salvato la dimensione di un blocco:
> $$\quad D_M = \frac{D_b^2}{I} = \frac{D_b^2}{\log{(\frac{C}{D_b})}}$$"
>
> *Fonte: [Il file system](./Il%20file%20system#233-allocazione-a-indice)*

Questa limitazione rende il metodo inadatto per file di grandi dimensioni senza ulteriori estensioni.

_**4. Allocazione in UNIX: Indicizzazione Multilivello**_

UNIX adotta una soluzione elegante che combina i vantaggi dell'allocazione a indice superandone i limiti:

> "Il metodo di allocazione utilizzato in `UNIX` è a **indicizzazione su più livelli di indirizzamento** basata su un _grafo aciclico diretto_."
>
> *Fonte: [Il file system](./Il%20file%20system#234-allocazione-in-unix)*

**Principio Fondamentale di UNIX:**

> "La regola sulla quale ci dobbiamo basare è che:
> > <strong><em>In <code>UNIX</code> tutto è un file</em></strong>."
>
> *Fonte: [Il file system](./Il%20file%20system#234-allocazione-in-unix)*

I file si dividono in tre categorie:

> "I file si dividono in tre categorie:
> - **Ordinari**: sono i veri e propri file
> - **Directory**: sono dei file che rappresentano delle raccolte di altri _file_
> - **Speciali**: rappresentano i dispositivi (ad esempio quelle in `/dev/`)"
>
> *Fonte: [Il file system](./Il%20file%20system#234-allocazione-in-unix)*

**Struttura con i-node:**

> "Ad ogni file possono essere associati **_uno o più nomi simbolici_** detti _link/alias_. Ogni _link_ si **_riferirà però ad uno ed un solo descrittore_**, detto `i-node`. I singoli `i-node` sono identificati univocamente dagli `i-number` e sono contenuti nella `i-list`."
>
> *Fonte: [Il file system](./Il%20file%20system#234-allocazione-in-unix)*

Questo permette la condivisione di file senza duplicazione: più nomi possono puntare allo stesso i-node.

_**Organizzazione del Disco in UNIX**_

Il disco in UNIX è diviso in quattro regioni distinte:

> "Questa tecnica formatta il disco in blocchi fisici di dimensione costante e prefissata, dividendo la superficie del disco in quattro regioni."
>
> *Fonte: [Il file system](./Il%20file%20system#234-allocazione-in-unix)*

**1. BootBlock:**

> "La prima regione, detta `BootBlock`, occupa un blocco fisico, allocato ad un **indirizzo prefissato**, e contiene il programma di inizializzazione del sistema da eseguire nella fase di _bootstrap_."
>
> *Fonte: [Il file system](./Il%20file%20system#234-allocazione-in-unix)*

**2. SuperBlock:**

> "La seconda regione è il `SuperBlock`. Occupa anch'essa la dimensione di un blocco e descrive l'allocazione del _file system_. In particolare qui sono contenuti:
> - I limiti delle quattro regioni
> - Il puntatore alla lista dei blocchi liberi
> - Il puntatore alla lista degli `i-node` liberi."
>
> *Fonte: [Il file system](./Il%20file%20system#234-allocazione-in-unix)*

Il SuperBlock è fondamentale per la gestione del file system: contiene tutte le metainformazioni necessarie per allocare nuovi file e i-node.

**3. i-List:**

> "La terza regione è la  `i-List` che contiene il vettore di tutti gli `i-node`."
>
> *Fonte: [Il file system](./Il%20file%20system#234-allocazione-in-unix)*

**4. DataBlocks:**

> "L'ultima area è il `DataBlocks`. Questa area è tipicamente molto più grande delle altre, poiché rappresenta la zona effettivamente diposnibile per la memorizzazione dei file. I blocchi liberi di questa zono sono organizzati in una lista collegata il cui primo indirizzo è contenuto nel `SuperBlock`."
>
> *Fonte: [Il file system](./Il%20file%20system#234-allocazione-in-unix)*

_**Struttura degli i-node**_

Gli i-node contengono tutte le informazioni necessarie per gestire un file:

> "Gli `i-node` sono quindi formati da:
> - **Tipo**: indica se il file è _ordinario_, _directory_ o _speciale_ (dispositivi)
> - **Proprietario e Gruppo**: indicano chi è l'utente proprietario del file e qual è il gruppo di appartenenza del proprietario
> - **Dimensione**: indica in numero di blocchi occupati dal file in memoria di massa
> - - **Data**: indica la data dell'ultima modifica effettuata sul file
> - **Link**: il numero di nomi che riferiscono il file (_hard-link_). Se si ha un solo nome, questo valore è `1`
> - **Bit di Protezione**: è l'insieme dei `12bit` che esprime la politica di protezione da applicare sul file. contiene i 9 bit dei premessi, `SUID`, `GUID` e `STIcky bit`.
> - **Vettore di Indirizzamento**: è costituito da un insieme di indirizzi che consente l'indirizzamento dei blocchi di dati sui quali è allocato il file (tipicamente sono `13 - 15`)."
>
> *Fonte: [Il file system](./Il%20file%20system#234-allocazione-in-unix)*

_**Meccanismo di Indirizzamento Multiplo**_

Il vettore di indirizzamento è il cuore del sistema di allocazione UNIX. Analizziamo il funzionamento con un esempio concreto:

> "Per capire meglio come vengono gestiti gli indirizzi nel vettore di indirizzamento supponiamo di avere:
> - **13 indirizzi** nel vettore (così come nelle prime versioni del sistema operativo)
> - Blocchi di dimensione $D_b = 512 B$ con indirizzi su `32bit`. Ciò significa che un blocco contiene $128$ indirizzi."
>
> *Fonte: [Il file system](./Il%20file%20system#234-allocazione-in-unix)*

**Il Problema dell'Indirizzamento Diretto:**

> "Se tutti gli indirizzi indirizzassero direttamente ai blocchi di dati avremmo che un file può essere al massimo $13 \cdot 512B = 6,5KB$, poco per gli standard moderni."
>
> *Fonte: [Il file system](./Il%20file%20system#234-allocazione-in-unix)*

**La Soluzione: Indirizzamento a Più Livelli:**

> "Quello che si fa quindi è sfruttare l'**indirizzamento multiplo**:
> - I primi `10` indirizzi nel vettore accedono direttamente ai _blocchi di dati_, per una dimensione totale di $10 \cdot 512B = 5KB$
> - L'`11`-esimo indirizzo nel vettore punta a un _blocco indice_ sfruttando l'**indirizzamento singolo**. Ciò significa che solo questo permette l'accesso a $128 \cdot 512B = 64KB$ di memoria
> - Il `12`-esimo indirizzo nel vettore punta invece a un _blocco indice che punta a blocchi indici_ sfruttando l'**indirizzamento doppio**. In questo caso questo indirizzo permette l'accesso a $128 \cdot 128 \cdot 512B = 8MB$ di memoria
> - Il `13`-esimo indirizzo nel vettore utilizza l'**indirizzamento triplo** indicizzando $128^3 \cdot 512B = 1GB$ di memoria."
>
> *Fonte: [Il file system](./Il%20file%20system#234-allocazione-in-unix)*

**Efficienza del Meccanismo:**

Questa struttura è estremamente efficiente perché:

1. **File piccoli** (< 5KB): accesso diretto con un solo accesso al disco
2. **File medi** (< 64KB): massimo due accessi (i-node + blocco indice singolo)
3. **File grandi** (< 8MB): massimo tre accessi (i-node + indice doppio + indice singolo)
4. **File molto grandi** (< 1GB): massimo quattro accessi (i-node + indice triplo + doppio + singolo)

**Dimensione Massima:**

> "Avendo solamente 13 indirizzi nel vettore la dimensione massima di un file è nell'ordine del **_Giga Byte_** $(5KB + 64KB + 8MB + 1GB)$."
>
> *Fonte: [Il file system](./Il%20file%20system#234-allocazione-in-unix)*

_**Rappresentazione delle Directory**_

Le directory in UNIX sono implementate come file speciali:

> "Le _directory_ sono rappresentati da un file, in cui contenuto ne descrive la struttura logica. In particolare ogni record coniene la coppia di informazioni `<nome_relativo, i-number>` associate al file"
>
> *Fonte: [Il file system](./Il%20file%20system#234-allocazione-in-unix)*

Questa semplicità permette di utilizzare gli stessi meccanismi di allocazione e accesso dei file ordinari anche per le directory.

_**Confronto Complessivo dei Metodi**_

|                      | **Contigua**       | **Lista Concatenata** | **Indice Semplice** | **UNIX (Indice Multiplo)** |
| -------------------- | ------------------ | --------------------- | ------------------- | -------------------------- |
| **Frammentazione**   | Esterna            | Nessuna               | Nessuna             | Nessuna                    |
| **Accesso Diretto**  | Efficiente (O(1))  | Inefficiente (O(n))   | Efficiente (O(1))   | Efficiente (O(livelli))    |
| **Accesso Sequenz.** | Efficiente         | Efficiente            | Efficiente          | Efficiente                 |
| **Overhead Spazio**  | Nessuno            | Puntatori in blocchi  | Blocco indice       | Blocchi indice multipli    |
| **Scalabilità**      | Limitata           | Buona                 | Limitata            | Eccellente                 |
| **Crescita File**    | Difficile          | Facile                | Limitata            | Facile                     |
| **Affidabilità**     | Buona              | Problematica          | Buona               | Molto buona                |
| **Complessità**      | Bassa              | Media                 | Media               | Alta                       |

**Approfondimenti:**

Il metodo di allocazione UNIX rappresenta un capolavoro di ingegneria del software, bilanciando efficienza per file piccoli (la maggioranza) con capacità di gestire file grandi. La maggior parte dei file nei sistemi UNIX è inferiore a 5KB, quindi beneficia dell'indirizzamento diretto senza overhead. Solo i file più grandi pagano il costo degli accessi indiretti multipli.

La separazione tra nome del file (nelle directory) e descrittore (i-node) permette elegantemente la realizzazione degli hard link: più nomi per lo stesso file condividono lo stesso i-node. Il campo "Link" nell'i-node conta quanti nomi puntano al file, e solo quando questo contatore raggiunge zero il file viene effettivamente eliminato.

Il meccanismo degli i-node è stato così efficace che è stato adottato, con variazioni, da molti file system moderni (ext2, ext3, ext4 in Linux). La struttura continua a evolversi: i file system moderni utilizzano vettori di indirizzamento più grandi (15 indirizzi invece di 13) e blocchi di dimensioni maggiori, permettendo di gestire file nell'ordine dei Terabyte mantenendo la stessa struttura concettuale.

---
<div class="stop"></div>

---

## 13. Protezione e Sicurezza

### Domanda 13.1 (answered)

**Domanda**: Spiega il concetto di matrice degli accessi come modello per rappresentare lo stato di protezione di un sistema. Descrivi le due modalità di realizzazione della matrice: Access Control List (ACL) e Capability List (CL), confrontandone vantaggi e svantaggi. Illustra come UNIX implementa il meccanismo di protezione combinando ACL statiche (bit di protezione negli i-node) con Capability List dinamiche (tabella dei file aperti di processo). Infine, spiega il modello di sicurezza multilivello Bell-La Padula, descrivendo le due proprietà fondamentali (semplice sicurezza e proprietà *) e come questo modello previene attacchi di tipo Trojan Horse attraverso l'esempio fornito con soggetti e oggetti a diversi livelli di classificazione.

**Risposta:**

La protezione e la sicurezza nei sistemi operativi richiedono modelli formali per rappresentare e gestire i diritti di accesso alle risorse. La matrice degli accessi costituisce il fondamento teorico su cui si basano i meccanismi di protezione moderni, mentre i modelli multilivello come Bell-La Padula affrontano scenari di sicurezza più complessi.

_**Il Concetto di Matrice degli Accessi**_

La matrice degli accessi rappresenta uno strumento fondamentale per modellare la protezione:

> "Un sistema di protezione può essere rappresentato utilizzando il modello della **_matrice degli accessi_**."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#241-matrice-degli-accessi)*

**Struttura e Funzionamento:**

> "Questo modello mantiene le informazioni che specificano il tipo di accesso che i soggetti hanno per gli oggetti all'interno di una struttura tabellare. Ciò consente di rappresentare lo **stato di protezione** garantendo il rispetto dei vincoli."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#241-matrice-degli-accessi)*

La matrice è una tabella bidimensionale dove:
- Le **righe** rappresentano i domini/soggetti ($D_i$)
- Le **colonne** rappresentano gli oggetti ($O_i$)
- Ogni cella $(i,j)$ contiene l'insieme dei diritti di accesso che il dominio $D_i$ ha sull'oggetto $O_j$

**Esempio di matrice:**

Dal file [Protezione e sicurezza](./Protezione%20e%20sicurezza#241-matrice-degli-accessi):

<div class="flexbox" markdown="1">

|       |       $O_1$       |      $O_2$      |   $O_3$   |        $D_1$        |      $D_2$       |    $D_3$    |
| :---: | :---------------: | :-------------: | :-------: | :-----------------: | :--------------: | :---------: |
| $D_1$ |      `read*`      |     `read`      | `execute` |                     |   `terminate`    |  `receive`  |
| $D_2$ |                   | `owner` `write` |           | `control` `receive` |                  | `terminate` |
| $D_3$ | `write` `execute` |                 |  `read`   |       `send`        | `send` `receive` |             |

</div>

**Dinamicità del Sistema:**

> "Il numero di oggetti e dei soggetti all'interno della matrice è _dinamicamente modificabile_, permettendo ai processi di cambiare dominio anche durante l'esecuzione."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#241-matrice-degli-accessi)*

Questo permette di modificare lo stato di protezione in modo controllato attraverso **transizioni di stato**.

**Meccanismo di Verifica:**

> "Il _meccanismo_ associato a questa struttura ha il compito di **verificare se le richieste di accesso di un processo che opera in un certo dominio sono _consentite oppure no_**."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#241-matrice-degli-accessi)*

Quando un processo nel dominio $D_i$ richiede di eseguire un'operazione $M$ su un oggetto $O_j$, il meccanismo controlla che $M$ sia presente nella cella $(i,j)$ della matrice.

**Politica DAC e Copy Flag:**

> "La politica del _Discretional Access Control_ (`DAC`) permette agli utenti di decidere il contenuto degli elementi della matrice. In particolare, alla creazione di un nuovo oggetto $O$, questo sarà aggiunto alle colonne della tabella, e **l'utente `owner`** stabilirà come settare le intersezioni con le righe della tabella."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#241-matrice-degli-accessi)*

La **propagazione dei diritti** è regolata dal copy flag:

> "Un soggetto $S_i$ può propagare un diritto di accesso $\alpha$ per un oggetto $X$ ad un altro soggetto $S_j$ **se e solo se** $S_i$ ha accesso a $X$ e $\alpha$ ha il **_copy flag_**."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#241-matrice-degli-accessi)*

Il copy flag (`*`) determina se un diritto può essere copiato da un dominio ad un altro.

_**Realizzazione della Matrice: ACL vs CL**_

La matrice può essere memorizzata in due modi fondamentali:

> "Possiamo vedere la matrice per accessi come una serie di **_righe_** (**Capability List**) o di **_colonne_** (**Access Control List**)."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#2411-realizzazione-della-matrice)*

**Access Control List (ACL) - Memorizzazione per Colonne:**

> "La memorizzazione per colonne prende il nome di **_Access Control List_** (`ACL`), ed è quella utilizzata nei sistemi `UNIX`. In questo modo ad ogni oggetto è associata una lista che contiene tutti i soggetti che possono accedevi e i relativi diritti di accesso."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#2411-realizzazione-della-matrice)*

**Vantaggi dell'ACL:**
- Numero di liste proporzionale al numero di oggetti (fisso e prevedibile)
- Facile modificare i diritti su un oggetto per tutti i soggetti
- Semplice revocare l'accesso a una risorsa
- Memorizzazione naturale nel descrittore dell'oggetto

**Svantaggi dell'ACL:**
- Ogni accesso richiede di trovare l'ACL dell'oggetto e scansionarla
- Inefficiente per determinare tutti i diritti di un soggetto
- Overhead temporale nella verifica dei permessi

**Capability List (CL) - Memorizzazione per Righe:**

> "La memorizzazione per righe si dice invece **_Capability List_** (`CL`), e associa ad ogni soggetto una lista che contiene gli oggetti da lui accessibili i relativi diritti di accesso."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#2411-realizzazione-della-matrice)*

**Vantaggi della CL:**

> "A differenza delle `ACL`, questa tecnica permette di avere **un unica struttura dati** sulla quale effettuare le ricerche da parte di un processo. I meccanismi di protezione che si basano su questa struttura sono quindi molto più efficaci, in quanto non è più necessario scorrere l'`ACL` dopo averla trovata per effettuare un _check_ sui diritti del processo."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#2411-realizzazione-della-matrice)*

- Verifica efficiente: accesso diretto ai diritti del processo
- Non richiede ricerca nell'ACL ad ogni operazione
- Ideale per gestione runtime

**Svantaggi della CL:**

> "Nasce però un altro problema, ovvero quello di dove salvare le `CL`. I soggetti, a differenza degli oggetti, _cambiano dinamicamente e frequentemente_, rendendo più complesso il salvataggio e la gestione della `CL` all'interno del sistema, andando a compensare il guadagno che ottenevamo sulla verifica a _runtime_."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#2411-realizzazione-della-matrice)*

> "Ipotizziamo infatti che volessimo cambiare i diritti di accesso ad un sottoinsieme dei soggetti per un dato oggetto $O$. Mentre nell'`ACL` ci basta accedere all'oggetto stesso e agire in maniera sequenziale, nel caso della `CL` sarà necessario **_aggiornare tutte le singole liste sul sistema_**, introducendo un enorme _overhead_."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#2411-realizzazione-della-matrice)*

_**Implementazione in UNIX: Approccio Ibrido**_

UNIX adotta una soluzione elegante che combina i vantaggi di entrambi gli approcci:

**ACL Statiche tramite Bit di Protezione:**

> "In `UNIX`, oltre ad avere i **_bit di protezione_**, si mette in atto un altro meccanismo."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#2412-meccanismi-unix)*

Negli i-node, UNIX memorizza:

> "Poiché in `UNIX` gli oggetti sono _file_, le singole `ACL` sono conservate all'interno del **descrittore del file**, sottoforma dei **_9 bit di protezione_**:
> - I primi `3 bit` rappresentano i diritti di un singolo soggetto, l'`owner`.
> - I secondi `3 bit` rappresentano i diritti dei soggetti che sono nel **gruppo del proprietario**.
> - Gli ultimi `3 bit` stabiliscono i diritti **per tutti gli altri soggetti**."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#2411-realizzazione-della-matrice)*

Un elemento della ACL assume la forma: `UID, GID: <diritti>`.

**Il Processo di Verifica alla Open:**

> "Quando un processo `PID` apre un _file_ tramite una `open`, specificando un azione, il sistema operativo verifica:
> 1. Che il file esista
> 2. Che i diritti combacino"
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#2412-meccanismi-unix)*

La verifica procede per step:

> "La verifica di queste condizioni accade per step:
> 1. La `open` controlla se `UID` del processo corrisponde con quello dell'`owner` dell'oggetto. Se il soggetto è _owner_ controlla i **diritti di owner** per verificare se l'operazione è consentita o meno
> 2. Altrimenti, controlla se il `GID` corrisponde con quello dell'`group` dell'oggetto. Se corrispondono allora effettuerà il controllo di protezione sui **diritti del group owner**
> 3. Altrimenti effettua il controllo sui **diritti degli others**"
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#2412-meccanismi-unix)*

**CL Dinamiche tramite File Descriptor:**

> "Solamente quando si ha esito positivo su questi controlli, verrà inserito un nuovo _file descriptor_ `fd` nella **_Tabella dei File Aperti di Processo_**, che punterà alla **tabella dei file aperti di sistema**. La _tabella dei file aperti di processo_ agisce a tutti gli effetti come una `CL` dinamica del processo."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#2412-meccanismi-unix)*

**Il Meglio di Entrambi i Mondi:**

> "A livello statico quindi `UNIX` salva la matrice per `ACL`, ma quando un processo esegue si costruisce dinamicamente la propria `CL`, per avere i vantaggi di entrambe le soluzioni."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#2412-meccanismi-unix)*

Questo approccio ibrido combina:
- **ACL statiche**: gestione semplice dei permessi sui file, modifiche centralizzate
- **CL dinamiche**: accesso efficiente durante l'esecuzione, nessuna ricerca ripetuta

_**Modello di Sicurezza Multilivello: Bell-La Padula**_

Per ambienti che richiedono sicurezza rigorosa, vengono introdotti modelli multilivello:

> "La maggior parte dei sistemi operativi permette ai singoli utenti di determinare chi possa leggere e scrivere i loro file e i loro oggetti, secondo il **_Controllo Discrezionale degli Accessi_**, o `DAC`. In alcuni ambienti è però richiesta una _sicurezza più rigorosa e controllata_ (ospedali, aziende, ambienti militari, ...). In questo caso sono stabilite delle **regole** su chi può vedere cosa, modificabili solo da chi **_possiede permessi speciali_**. Questa tecnica ha il nome di **_Controllo degli Accessi Obbligatorio_**, o `MAC`."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#242-sicurezza-multilivello)*

**Il Modello Bell-La Padula:**

> "Un ulteriore criterio di sicurezza può essere quello di avere diversi **_livelli di protezione_**, ognuno con criteri di accesso diversi. Questo modello si chiama **_Modello Bell-La Padula_**, progettato per gestire la sicurezza in ambiente militare."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#242-sicurezza-multilivello)*

**I Quattro Livelli di Classificazione:**

> "Il **_Modello Bell-La Padula_** stabilisce 4 livelli di sicurezza:
> 1. Non classificato (pubblico)
> 2. Confidenziale
> 3. Segreto
> 4. Top-Secret (privato)"
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#242-sicurezza-multilivello)*

**Applicazione ai Soggetti e agli Oggetti:**

> "I livelli di sicurezza si applicano sia agli oggetti $O$ che **_ai soggetti_** $S$."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#242-sicurezza-multilivello)*

Il modello definisce una funzione $SC(\cdot)$ che rappresenta il livello di sicurezza di un'entità (Security Classification).

_**Le Due Proprietà Fondamentali**_

**1. Proprietà di Semplice Sicurezza (Simple Security Property):**

> "Le regole che sfruttano la funzione:
> - **_Proprietà di semplice sicurezza_**: un soggetto in esecuzione a livello di sicurezza $k$, può **leggere** solo _oggetti al suo livello o a livelli inferiori_, secondo la regola che $SC(S) \ge SC(O)$."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#242-sicurezza-multilivello)*

Questa regola previene che informazioni riservate vengano lette da soggetti non autorizzati: un soggetto può "guardare in basso" ma non "in alto" nella gerarchia.

**2. Proprietà * (Star Property):**

> "- **_Proprietà *_**: un soggetto in esecuzione al livello di sicurezza $k$, può **scrivere** solo _oggetti al suo livello o a livelli superiori_, secondo la regola che $SC(S) \le SC(O)$."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#242-sicurezza-multilivello)*

Questa regola cruciale previene che informazioni riservate vengano copiate in oggetti di livello inferiore: un soggetto può "scrivere in alto" ma non "in basso".

_**Prevenzione di Attacchi Trojan Horse**_

Il modello Bell-La Padula è stato specificamente progettato per prevenire attacchi come i Trojan Horse. Analizziamo l'esempio fornito:

> "Per chiarire come opera il modello prendiamo la seguente tabella:"
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#242-sicurezza-multilivello)*

<div class="flexbox" markdown="1">

|       |    $O_1$ (file riservato)    | $O_2$ (trojan-horse) |   $O_3$ (file di appoggio)   |
| :---: | :--------------------------: | :------------------: | :--------------------------: |
| $S_1$ | `owner`<br>`read`<br>`write` |      `execute`       |           `write`            |
| $S_2$ |                              | `owner`<br>`execute` | `owner`<br>`read`<br>`write` |

</div>

**Scenario dell'Attacco:**

> "Ipotizziamo che $O_2$ abbia accesso alle seguenti istruzioni:
> - Aprire in lettura $O_1$
> - Aprire in scrittura $O_3$"
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#242-sicurezza-multilivello)*

**Comportamento senza Bell-La Padula (solo DAC):**

> "Se $O_2$ venisse eseguito da $S_2$ queso non potrebbe accedere in nessun modo ad $O_1$, in quanto $S_2$ non possiede alcun diritto sull'oggetto."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#242-sicurezza-multilivello)*

Ma:

> "Se $O_2$ venisse invece eseguito da $S_2$, questo opererà con i diritti di $S_2$ potendo:
> - Accedere in lettura a $O_1$
> - Accedere in scrittura a $O_3$
>
> Facendo così $S_2$ è in grado di **leggere indirettamente il contenuto di** $O_1$, attraverso la copia in $O_3$, effettuata da $O_2$ per conto di $S_1$."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#242-sicurezza-multilivello)*

Questo è precisamente un attacco Trojan Horse: $S_2$ inganna $S_1$ a eseguire un programma ($O_2$) che copia informazioni riservate in un file accessibile a $S_2$.

**Protezione con Bell-La Padula:**

> "Sfruttando invece la classificazione _Bell-La Padula_ possiamo classificare come **Pubblico** $(P)$ e **Privato** $(R)$ i soggetti e gli oggetti:
> - $SC(S_1) = SC(O_1) = R$
> - $SC(S_2) = SC(O_2) = SC(O_3) = P$"
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#242-sicurezza-multilivello)*

Ora, quando $O_2$ viene eseguito da $S_1$:

> "Se adesso $O_2$ venisse eseguito da $S_1$:
> - Potrebbe accede in lettura per la **_Proprietà di semplice sicurezza_** &emsp; $SC(S_1) = R = SC(O_1)$
> - Non potrebbe accedere in scrittura per la **_Proprietà *_** &emsp;$SC(S_1) = R \ge SC(O_3) = P$"
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#242-sicurezza-multilivello)*

**L'Attacco è Bloccato:**

La Proprietà * impedisce al processo (che opera con clearance $R$) di scrivere su $O_3$ (che ha classificazione $P$), poiché $SC(S_1) = R \not\le SC(O_3) = P$. Il Trojan Horse può leggere il file riservato, ma **non può copiarne il contenuto** in un file di livello inferiore!

_**Limiti e Considerazioni del Modello**_

**Obiettivo del Modello:**

> "È importante sottolineare che il modello _Bell-La Padula_ è stato concepito per __**mantenere segreti i dati_** e non **_per garantirne l'integrità_**."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#242-sicurezza-multilivello)*

**Il Modello BiBa per l'Integrità:**

> "Esiste infatti un altro modello, detto **_BiBa_**, che si preoccupa di garantire l'integrità dei dati. Questo modello di fatto inverte le regole del modello _Bell-La Padula_:
> - La lettura è permessa solo su file di livello equo o superiore $SC(S) \le SC(O)$.
> - La scrittura solo su file di livello equo o inferiore $SC(S) \ge SC(O)$."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#242-sicurezza-multilivello)*

**Incompatibilità tra i Modelli:**

> "Ovviemente i due modelli, _BiBa_ e _Bell-La Padula_, **_non sono compatibili_** e non possono essere realizzati contemporaneamente."
>
> *Fonte: [Protezione e sicurezza](./Protezione%20e%20sicurezza#242-sicurezza-multilivello)*

Questa incompatibilità deriva dal fatto che le regole sono esattamente opposte. Un sistema deve scegliere se privilegiare la **confidenzialità** (Bell-La Padula) o l'**integrità** (BiBa).

_**Tabella Comparativa dei Metodi di Protezione**_

| Aspetto                  | ACL                              | CL                                | UNIX (Ibrido)                    |
| ------------------------ | -------------------------------- | --------------------------------- | -------------------------------- |
| **Memorizzazione**       | Per oggetto (colonne)            | Per soggetto (righe)              | ACL statiche + CL dinamiche      |
| **Efficienza accesso**   | Richiede ricerca                 | Accesso diretto                   | Accesso diretto (runtime)        |
| **Modifica permessi**    | Facile per singolo oggetto       | Richiede aggiornare tutte le CL   | Modifica centralizzata su i-node |
| **Scalabilità**          | Numero di oggetti                | Numero di soggetti                | Ben bilanciata                   |
| **Revoca**               | Immediata                        | Complessa                         | Effettiva alla prossima open     |
| **Gestione runtime**     | Inefficiente                     | Efficiente                        | Efficiente (file descriptor)     |
| **Persistenza**          | Naturale (nel descrittore)       | Problematica                      | ACL persistenti, CL temporanee   |
| **Uso pratico**          | Windows (NTFS), componente UNIX  | Raramente pura                    | Sistemi UNIX/Linux               |

**Approfondimenti:**

La matrice degli accessi e il modello Bell-La Padula rappresentano contributi fondamentali alla teoria della sicurezza informatica, sviluppati negli anni '70 ma ancora rilevanti oggi. Il modello Bell-La Padula è stato utilizzato in sistemi militari classificati e ha influenzato lo sviluppo di SELinux (Security-Enhanced Linux), che implementa il Mandatory Access Control sopra il tradizionale DAC di UNIX.

Un aspetto interessante è che il concetto di "ruolo" nei moderni sistemi RBAC (Role-Based Access Control) estende ulteriormente questi modelli, permettendo una gestione più flessibile delle autorizzazioni separando i diritti dagli utenti specifici e assegnandoli invece a ruoli funzionali.

La scelta tra ACL e CL ha implicazioni pratiche significative: le ACL sono più adatte per sistemi dove i permessi cambiano raramente ma vengono verificati frequentemente, mentre le CL sono preferibili quando i soggetti cambiano dinamicamente ma i permessi rimangono stabili. L'approccio ibrido di UNIX rappresenta una soluzione pragmatica che ottimizza entrambi gli scenari: verifiche frequenti durante l'esecuzione (CL) e gestione centralizzata dei permessi (ACL).

