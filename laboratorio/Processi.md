---
title: Processi
---

# 1. Indice

- [1. Indice](#1-indice)
- [2. Processi in UNIX](#2-processi-in-unix)
- [3. System Call per i Processi](#3-system-call-per-i-processi)
	- [3.1. Fork](#31-fork)
	- [3.2. Terminazione Processi - `exit` e `wait`](#32-terminazione-processi---exit-e-wait)
	- [3.3. Sostituzione di codice - `exec..()`](#33-sostituzione-di-codice---exec)
- [4. Interazione tra Processi](#4-interazione-tra-processi)
	- [4.1. 4.1 Sincronizzazione - `signal`](#41-41-sincronizzazione---signal)
		- [4.1.1. System calls per i segnali](#411-system-calls-per-i-segnali)
			- [4.1.1.1. Signal](#4111-signal)
			- [4.1.1.2. Kill](#4112-kill)
			- [4.1.1.3. Pause](#4113-pause)
			- [4.1.1.4. Sleep](#4114-sleep)
			- [4.1.1.5. Alarm](#4115-alarm)
	- [4.2. Comunicazione - `pipe`](#42-comunicazione---pipe)
- [5. Gestione dei processi da terminale](#5-gestione-dei-processi-da-terminale)
	- [5.1. Invio segnali - `kill`](#51-invio-segnali---kill)
	- [5.2. Visualizzazione Processi - `ps`](#52-visualizzazione-processi---ps)
- [6. Gerarchia dei processi](#6-gerarchia-dei-processi)
	- [6.1. Funzioni per gli Identificatori](#61-funzioni-per-gli-identificatori)
	- [6.2. Gruppi di processi](#62-gruppi-di-processi)
- [7. Priorità dei processi](#7-priorità-dei-processi)
- [8. Gestione dei processi da terminale](#8-gestione-dei-processi-da-terminale)
	- [8.1. Monitor di sistema - `top`](#81-monitor-di-sistema---top)


# 2. Processi in UNIX

`UNIX` è una famiglia di sistemi operativi multiprogrammati basati su processi.

Il processo `UNIX` mantiene spazi di indirizzamento separati:
- **Privato**: la comunicazione avviene tramite scambio di messaggi
- **Condiviso**: più processi possono eseguire lo stesso codice

La politica di assegnamento della **CPU** ai processi adottata da UNIX è basata sulla divisione di tempo, facendo attraversare loro vari stati:
- `init`
- `printo`
- `running`
- `swapped`
- `sleep`/`bloccato`
- `zombie`
- `terminato`

Il descrittore di un processo `PCB` (_Process Control Block_), è suddiviso in due strutture dati distinte:
- **Process Structure**: contiene informazioni indispensabili che sono sempre salvate in memoria
- **User Structure**: sono informazioni utili solo quando il processo è residente in memoria, è vengono rimossi in caso di _swap-out_

# 3. System Call per i Processi

Essistono diversi comandi di sistema che permettono la manipolazione dei processi:
- `fork`: crea un processo
- `exit`: termina un processo
- `wait`: sospende un processo in attesa della terminazione dei figli
- `exec..`: sostituzione di codice e dati

I processi sono identificati da quello che si chiama `PID` (_Process ID_). Esistono alcune funzioni di sistema che permettono di ottenere informazioni su di esso:
```cpp
// Restituisce il PID del processo
pid_t getpid();

// Restituisce il PID del processo padre
pid_t getppid();
```

## 3.1. Fork

Ogni processo è in grado di creare dinamicamente processi tramite la chiamata di sistema `fork`.

Il processo creato, detto **figlio**, ha uno spazio di dati separato, ma condivide con il processo che lo ha creato, detto **padre**, il codice.
I processi figli possono a loro volta avere nuovi processi figli.

La funzione è così definita:
```cpp
/**
* La funzione non richiede parametri
* Restituisce un risultato intero DIVERSO a padre e figlio
*/
pid_t fork(void)
```

Poiché il processo figlio condivide il codice con il padre, **_ne eredita una copia delle aree dati globali_** (_stack, heap, User Structure_, ...).
Ciò significa che alla creazione il figlio ha il proprio `%RIP` che punta alla al'istruzione successiva alla `CALL fork`, e, di conseguenza, ha nel registro `%RAX` il valore di ritorno della funzione.

La funzione è però progettata affinché **_restituisca un risultato intero diversso a padre e figlio_**:
- **Padre**: restituisce il `PID` del figlio, o un valore negativo in caso di fallimento
- **Figlio**: restituisce `0`

In questo modo, nel codice del padre, è possibile inserire dopo la chiamata alla fork una `if-statement` che permette di discriminare il comportamento del padre e del figlio:
```cpp
pid_t pid;
pid = fork();

printf("%d\n", pid);
```
> _Padre_ : `1509`
> _Figlio_: `0`

## 3.2. Terminazione Processi - `exit` e `wait`

Un processo può terminare in due modi:
- **Involontariamente**: accade in caso di azioni illegali (ad esempio _segmentation fault_ o divisioni per zero) oppure in caso di **interruzioni causate dalla ricezione di segnali** (ad esempio l'utente che inserice la combinazione `Ctrl+C`)
- **Volontariamente**:  quando si esegue l'ultima istruzione o viene chiamata la system call `exit()`

```cpp
/**
* È una chiamata senza ritorno che permette di terminare
* volontariamente un processo.
* @param status permette di comunicare al padre lo stato di terminazione
*/
void exit(int status)
```

Il processo padre può ottenere lo stato di terminazione del figlio mediante la system call `wait()`
```cpp
/**
* Il padre si mette in attesa della terminazione del figlio
* @param status puntatore a intero che contiene lo stato di terminazione del figlio
* @return il PID del figlio che è terminato, valore negativo se non ha processi figli
*/
pid_t wait(int* status)
```

La sospensione del padre accade solo **_se tutti i figli sono ancora in esecuzione_**. Nel caso in cui almeno un figlio è terminato, la funzione ritorna immediatamente le informazioni di terminazione. Questo è possibile grazie all'esistenza dello stato `zombie`.
I processi figli che terminano infatti entrano nello stato di `zombie`, proprio per permettere al padre di ottenere le informazioni sulla terminazione di quest'ultimo.

La variabile `status` contiene diverse informazioni su come il figlio e terminato, oltre allo stato di terminazione eventualmente fornito dal figlio stesso. Se il _byte meno significativo_ di `*status` fosse `0`, allora la terminazione è **stata volontaria** e il _byte più significativo_ contiene lo stato di terminazione.

Per gestire `status` in modo astratto, la libreria `<sys/wait.h>` fornisce alcune _MACRO_:
- `WIFEXITED(status)`: restituisce `true` se è terminato volontariamente
- `WEXITSTATUS(status)`: restituisce lo stato di terminazione

## 3.3. Sostituzione di codice - `exec..()`

Un processo può **_sostituire il programma che sta eseguendo_** (_codice e dati_) eseguendo una _syscall_ della "famiglia" `exec()` (`excecl()`, `execle()`, `execclp()`, `execv()`, `execve()`, `execvp()`).

In particolare vediamo il comando `execl`:
```cpp
/**
* @param path rappresenta i percorso assoluto del comando che si vuole eseguire (es. "bin/ls")
* @param arg0 rappresenta il nome del programma da eseguire (es. "ls")
* @param arg1_argN rappresentano gli eventuali argomeni del comando
* @return solo in caso di fallimento
*/
int execl(char* path, char* arg0, ..., char* argN, NULL)
```

# 4. Interazione tra Processi

I processi `UNIX` aderiscono al modello ad **_ambiente locale_**, dove non c'è condivisione di variabili e ogni processo ha uno spazio di indirizzamento provato.

L'unica forma di interazione tra processi è la cooperazione, che può avvenire tramite:
- _Sincronizzazione_: imponendo vincoli temporali
- _Comunicazione_: attraverso scambio di messaggi

Le interazioni avvengono su **_astrazioni realizzate dal kernel_** interagendo mediante _system calls_.

## 4.1. 4.1 Sincronizzazione - `signal`

I segnali sono il meccanismo messo a disposizione dai sistemi `UNIX`/`Linux` per la sincronizzazione dei processi.
Permettono la notifica di eventi _asincroni_ da parte di un processo ad altri, e possono essere utilizzati anche dal _SO_ per notificare il verificarsi di eccezioni a un processo utente.

I segnali sono rappresentati dalle _**interrupt software**_.

La ricezione di un segnale ha tre possibili effetti sul processo:
1. Viene eseguita una funzione _handler_ definita dal programmatore
2. Viene eseguita un azione prefefinita dal sistema operativo attraverso un _default handler_
3. Il segnale viene ignorato

Nei primi due casi il processo si comporta **in modo asincrono rispetto al segnale**, interrompendo il processo in esecuzione per eseguire l'_handler_. Se, alla terminazione dell'_handler_, il processo non era precedentemente terminato, allora questo riprende dall'istruzione successiva all'ultima eseguita prima dell'interruzione.

Versioni diverse di `UNIX` possono avere segnali diversi. La lista di questi segnali si trova nel file di sistema `signal.h`. La pagina di manuale sui segnali si trova in:
```bash
man 7 signal
```

Ciascun segnale è identificato da **un intero** e un **nome simbolico**:

<div class="flexbox" markdown="1">

| Nome segnale | Numero Segnale | Descrizione                                                                                               |
| :----------: | :------------: | :-------------------------------------------------------------------------------------------------------- |
|   `SIGHUP`   |       1        | _Hang up_, indica che il terminale è stato chiuso                                                         |
|   `SIGINT`   |       2        | Interruzione del processo (`Ctrl+C`)                                                                      |
|  `SIGQUIT`   |       3        | Interruzione del processo e _core dump_ (`Ctrl+\`)                                                        |
|  `SIGKILL`   |       9        | Interruzione immediata. Non è ignorabile e il processo che lo riceve non può eseguire opzioni di chiusura |
|  `SIGTERM`   |       15       | Terminazione del programma                                                                                |
|  `SIGUSR1`   |       10       | Definito dall'utente. Di _default_ termina il processo                                                    |
|  `SIGUSR2`   |       12       | Definito dall'utente. Di _default_ termina il processo                                                    |
|  `SIGSEGV`   |       11       | Errore di segmentazione                                                                                   |
|  `SIGALRM`   |       14       | Indica terminazione del timer                                                                             |
|  `SIGCHLD`   |       17       | Processo figlio terminato, fermato o risvegliato. Di _default_ è ignorato                                 |
|  `SIGSTOP`   |       19       | Ferma temporaneamente l'esecuzione del processo. Non è ignorabile                                         |
|  `SIGTSTP`   |       20       | Sospende l'esecuzione del processo (`Ctrl+Z`)                                                             |
|  `SIGCONT`   |       18       | Il processo può continuare qual'ora fosse stato fermato da `SIGSTOP` o `SIGTSTP`                          |

</div>


### 4.1.1. System calls per i segnali

Vediamo adesso alcune _system calls_ per i segnali

#### 4.1.1.1. Signal

Così definita:
```cpp
typedef void (*sighander_t)(int);

sighandler_t signal(int sig, sighandler_t handler)
```

Permette di definire la funzione `handler` che dovrà gestire il segnale `sig`. La funzione `handler` dovrà prevedere **un parametro intero**, che al momento della ricezione del segnale ne conterrà il codice.

L'handler può valere delle macro:
- `SIG_IGN`: per ignorare il segnale
- `SIG_DFL`: per ripristinare l'azione di default

La funzione restituisce **_un puntatore al precedente handler del segnale_** o `SIG_ERR` in caso di errore.

Possiamo consultarla a:
```bash
man 2 signal
```

In caso di `fork()` il figlio **_eredita dal padre le informazioni relative alla gestione dei segnali_**. È importante sottolineare che eventuali _signal_ eseguite dal figlio **_non hanno effetto sul padre_**.

Le _syscall_ `exec..` **_non mantengono le associazioni segnale-handler_** (tranne per i segnali ignorati che continuano ad esserlo).

#### 4.1.1.2. Kill

Così definita:
```cpp
int kill(pid_t pid, int sig)
```

Invia il segnale `sig` al processo `pid`, dove se:
- `pid > 0`: il segnale viene inviato a `pid`
- `pid == 0`: il segnale viene inviato a _tutti i processi nello stesso_ `process group` del chiamante
- `pid == -1`: il segnale viene inviato a _tutti i processi nello stesso_ `process group` a cui il chiamante può inviare segnali
- `pid < -1`: il segnale viene inviato ai processi il cui `process group` è `-pid`

La funzione restituisce `0` in caso di successo.

Possiamo consultarla a:
```bash
man 2 kill
```

#### 4.1.1.3. Pause

Così definita:
```cpp
int pause(void)
```

Mette nello stato `sleeping` il processo fino alla ricezione di un segnale.

Se il gestore non terminasse l'esecuzione del processo, restituisce `-1`.

Possiamo consultarla a:
```bash
man pause
```

#### 4.1.1.4. Sleep

Così definita:
```cpp
unsigned int sleep(unsigned int seconds)
```

Mette nello stato `sleeping` il processo chiamante fino a che:
- Sono passati `seconds` secondi
- Arriva un segnale che non viene ignorato

All'accadere di uno dei due casi il processo viene risvegliato.

Ritorna `0` se è passato il tempo previsto, altrimenti il tempo rimanente allo scadere del timer nell'istante di arrivo del segnale.

Possiamo consultarla a:
```bash
man 3 sleep
```

#### 4.1.1.5. Alarm

Così definita:
```cpp
unsigned int alarm(unsigned int seconds)
```

Provoca la ricezione di un segnale `SIGALRM` dopo `seconds` secondi.

Di _default_ `SIGALRM` termina il processo, cancellando un eventuale allarme invocato precedentemente.

Se `seconds == 0` viene eliminato un eventuale allarme precedente.

La funzione ritorna `0` se non c'era un allarme programmato, altrimenti il numero di secondi mancanti all'ultimo allarme programmato

Possiamo consultarla a:
```bash
man alarm
```

## 4.2. Comunicazione - `pipe`

I processi possono comunicare anche sfruttando il meccanismo delle `pipe`.

Le `pipe` implementano un sistema di **comunicazione indiretta**, _senza naming esplicito_.

Realizza il concetto di _mailbox_ nella quale si possono accodare messaaggi in modo `FIFO`

La _pipe_ è un canale monodirezionale con due estremi:
- Uno per la lettura
- Uno per la scrittura


L'astrazione della `pipe` è realizzata **in modo omogeneo rispetto alla gestione dei file**.
A ciascun estremo è associato un `file descriptor`, risolvendo i problemi di sincronizzazione con primitive `read` e `write`.

I figli ereditano gli stessi `file descriptor` del padre, e li possono utilizzare per **_comunicare con i fratelli e con il padre_**.
È possibile implementare la comunicazone di processi che non si trovano nella stessa gerarchia attraverso i _socket_.

Possiamo consultare i `pipe` a:
```bash
man pipe
```

# 5. Gestione dei processi da terminale

Vi sono diversi comandi per gestire i processi direttamente dal terminale, ne vediamo qualcuno.

## 5.1. Invio segnali - `kill`

Il comando `kill` è un comando che permette l'invio di segnali a processi da terminale:
```bash
kill [options] pid [pid2...]
```

Il segnale di default inviato è `SIGTERM`, ma è possibile cambiarlo tramite opzione:
```bash
kill -SEGNALE pid		# invia il sengale SEGNALE
```

Per visualizzare la lista dei segnali disponibili è possibile eseguire:
```bash
kill -l
```

Un utente normale può inviare segnali **sono ai processi di cui è proprietario**. Un utente _root_ invece può inviare segnali **_a tutti i processi_**.

## 5.2. Visualizzazione Processi - `ps`

Il comando `ps` permette di visualizzare i processi in esecuzione al momento della chiamata.
```bash
ps [options...]
```

Alcune opzioni principali disponibili in `Linux` sono nella seguente tabella:

<div class="flexbox" markdown="1">

|   Opzione   | Descrizione                                                          |
| :---------: | :------------------------------------------------------------------- |
| `-u utente` | Visualizza i processi dell'utente sepcificato                        |
|     `u`     | Formato output utile all'analisi dell'utilizzo delle risorse         |
|     `a`     | Processi di tutti gli utenti                                         |
|     `x`     | Visualizza anche i processi che non sono stati generati da terminali |
|     `o`     | Mostra solo i campi specificati più avanti                           |
|    `-O`     | Mostra i campi specificati oltre ad altri di _default_               |

</div>

Gli stati principali dei processi sono:
- `S` - _sleep_
- `T` - _bloccato_
- `R` - _running_
- `Z` - _zombie_

Per vedere tutti i possibili stati:
```bash
man ps

ps aux 						# visualizza tutti i possibili processi
ps aux | grep "filtro"		# tra tutti i possibili processi, mostra solo le righe che contengono "filtro"
```

# 6. Gerarchia dei processi

I sistemi `UNIX`/`Linux` prevedono un `init system`, ovvero un **processo mandato in esecuzione dal _kernel_ durante il boot**. Questo è il primo processo ad andare in esecuzione (ha infatti `PID = 1`), ed è il **_padre di tutti gli altri processi_**.

In alcuni sistemi `Linux` come `Debian`/`Ubuntu` il gestore dei processi utilizzato è `systemd`.

Per visualizzare l'albero dei processi si utilizza il comando:
```bash
pstree
```
```
systemd─┬─ModemManager───3*[{ModemManager}]
    	├─NetworkManager───3*[{NetworkManager}]
    	├─accounts-daemon───3*[{accounts-daemon}]
    	├─at-spi-bus-laun─┬─dbus-daemon
    	│                 └─4*[{at-spi-bus-laun}]
    	├─at-spi2-registr───3*[{at-spi2-registr}]
    	├─avahi-daemon───avahi-daemon
    	├─colord───3*[{colord}]
    	├─cron
    	├─cups-browsed───3*[{cups-browsed}]
    	├─cupsd
    	├─dbus-daemon
    	├─gdm3─┬─gdm-session-wor─┬─gdm-wayland-ses─┬─dbus-run-sessio─┬─dbus-daemon
    	│      │                 │                 │                 └─gnome-session-b─┬─gnome-shell─┬─Xwayland
       ...    ...     			...			   	  ...								  ...			...
```

Un proccesso ha **_7 identificatori_**.

Tre identificatori sono relativi all'identificazione:
- `PID`: è l'ID univoco del processo
- `PPID`: è l'ID univoco del processo padre
- `PGID`: è l'ID univoco del _process group_ al quale appartiene il processo

Gli altri 4 identificatori che determinano i **permessi del processo** si dividono in _real_ e _effective_:
- `RUID` (_Real User ID_): è l'ID dell'utente che _ha mandato in esecuzione il processo_
- `RGID` (_Real Group ID_): è l'ID del gruppo primario dell'utente che _ha mandato in esecuzione il processo_
- `EUID` (_Effective User ID_)
- `EGID` (_Effective Group ID_)

`EUID` e `EGID` possono differire da `RUID` e `RGID` **solo se il comando eseguito ha _il suo bit `SUID` o `SGID` attivo_**. Sono spesso utilizzati per definire i privilegi di accesso alle risorse e di invocazione di _system call_ nel processo.

Infatti un processo utente (**_non root_**) può inviare segnali ad un altro processo **_solo se_** il suo `EUID`/`RUID` coincide con il `RUID` del processo destinatario,

## 6.1. Funzioni per gli Identificatori

Di seguito possiamo trovare alcune funzioni _get_ per recuperare gli identificatori:
```c
/* Recupero il mio PID */
pid_t getpid();

/* Recupero il PPID */
pid_t getppid();

/* Recupero il PGID*/
pid_t getpgrp();

/* Recupero il RUID */
uid_t getuid();

/* Recupero il RGID */
uid_t getgid();

/* Recupero il EUID */
uid_t geteuid();

/* Recupero il EGID */
uid_t getegid();
```

## 6.2. Gruppi di processi

I processi sono organizzati in **_gruppi_**. Quando un nuovo processo viene mandato in esecuzione da terminale gli viene associato **_un nuovo process group_**. Gli eventuali figli di questo processo, compresi quelli generati dalla _syscall_ `exec`, _erediteranno il process group_.

I gruppi permettono di mandare segnali ad una gerarchia di processi, e sono alla base del _job-control_ offerto dalla _shell_.

# 7. Priorità dei processi

Lo _scheduler_ `Linux` assegna la **CPU** ai processi tenendo conto di un **livello di priorità** assegnato a ciascun processo.
La priorità dipende principalmente dalla classe di scheduling del processo, e si divide tra _real-time_ e _normale_.

La priorità dei processi _normali_ può essere in parte controllata mediante il concetto di **_niceness_** e la relativa _system call_ `nice`.
Ad ogni processo infatti è associato un valore di _niceness_ nell'intervallo `[-20, 19]`, dove un valore più alto porta ad avere _minore priorità di esecuzione_.

In questo modo un processo eseguito in background, e quindi non interattivo, può lasciare più tempo di elaborazione agli altri processi.

Tramite il comando `nice`:
- L'utente può **solo aumentare la _niceness_ dei suoi processi**
- _Root_ può **sia aumentare che diminuire la _niceness_ dei suoi processi**

```bash
# Manda in esecuzione in background il processo del comando
#  bzip file &
# dandogli un valore di niceness specificato
nice -n valore_nice bzip2 file &

# Modifico la niceness di un processo già in esecuzione
renice valore_nice PID
```

# 8. Gestione dei processi da terminale

Con _job-control_ si intende la possibilità di sospendere e riattivare gruppi di processi, detti _jobs_, offerta dalla _shell_ mediante opportuni comandi.

Abbiamo già detto che la _shell_ associa un `JOB_ID` distinto ad ogni comando eseguito (alle _pipeline_ di comandi viene associato un solo job).
Questi _job_ sono salvati in una tabella specifica, visualizzabile tramite il comando `jobs`.

Un _job_ in esecuzione in **foreground** ha il controllo dello _standard input/output/error_, di fatto è come se "prendesse il controllo del terminale" restituendolo alla shell solo alla sua terminazione.

Per eseguire _job_ in **background** si utilizza il carattere `&` alla fine del comando:
```bash
comando &
```

In questo modo il processo **non ha più accesso allo standard input**, ma permettiamo all'utente di utilizzare la shell mentre il job viene eseguito in parallelo.

Nel caso avessimo già avviato un processo in _foreground_, per fermarlo possiamo inviare il segnale `SIGTSTP` attraverso la combinazione `Ctrl+Z`.

Per intervenire sui _job_ che sono stati fermati in questo modo si utilizza `jobs` per ottenerle l'identificatore `JOB_ID`, e successivamente:
```bash
# Per farlo ripartire in foreground
fg JOB_ID

# Per farlo ripartire in background
bg JOB_ID
```

Anche sui _job_ è possibile utilizzare il comando `kill`:
```bash
# Invio il segnale SIGTERM al job specificato
kill %JOB_ID

# Invio il segnale SIG al job specificato
kill -n SIG %JOB_ID
```

Poiché i _job_ ereditano il process group del terminale che li inizializza, se questo viene chiuso, ricevono il segnale `SIGHUP` e, di _default_, anche loro vengonno terminati.

Per fare in modo che il segnale `SIGHUP` non porti alla terminazione di un _job_ è possibile utilizzare due strumenti:
```bash
nohup comando
```

In questo modo il _job eseguito_ è immune a `SIGHUP`.
Tuttavia comporta due conseguenze per il _job_:
- **Non ha più accesso allo `stdin`**, in caso di lettura ottiene `EOF`
- Lo `stdout` viene **rediretto su un file chiamato** `nohup.out`

```bash
disown %JOB_ID
```

Può essere utilizzato per rendere immune a `SUGHUP` un _job_ **già in esecuzione**.

Il _job_ viene **_rimosso dalla tabella dei job_**. Di conseguenza la _shell_ non invierà più il segnale `SIGHUP` quando viene chiusa.

Il comando non influsice direttamente sulle modifiche relative alla lettura sullo `stdin` e/o scrittura sullo `stdout`, ed è opportuno modificarle per evitare errori durante l'esecuzione.

## 8.1. Monitor di sistema - `top`

Il comando `top` permette di visualizzare i processi e di effettuare operazioni su di essi in modo interattivo. Vengono visualizzate anche informazioni complessive sul sistema (_carico CPU_, _utilizzo della memoria_, ...)

I processi sono ordinati in **ordine di utilizzo decrescente della CPU**.
Dall'interfaccia che si genera dopo aver chiamato il comando è possibile inviare segnali ai processi e cambiarne il valore di _niceness_.

Di seguito possiamo vedere una serie di comandi interattivi utilizzabili dall'interfaccia:
- `h`: _help_
- `H`: permette di vedere i singoli _thread_
- `d`: intervallo di aggiornamento (_delay_)
- `k`: invio di un segnale
- `n`: numero di processi da visualizzare
- `r`: _renice_
- `u`: utente da specificare
- `q`: _quit_