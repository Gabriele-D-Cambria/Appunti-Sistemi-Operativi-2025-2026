---
title: Protezione e sicurezza
---

# 1. Indice

- [1. Indice](#1-indice)
- [2. Protezione e Sicurezza](#2-protezione-e-sicurezza)
	- [2.1. Modelli di Protezione](#21-modelli-di-protezione)
	- [2.2. Politiche di Protezione](#22-politiche-di-protezione)
	- [2.3. Meccanismi di Protezione](#23-meccanismi-di-protezione)
	- [2.4. Dominio di protezione](#24-dominio-di-protezione)
		- [2.4.1. Matrice degli Accessi](#241-matrice-degli-accessi)
			- [2.4.1.1. Realizzazione della matrice](#2411-realizzazione-della-matrice)
		- [2.4.2. Sicurezza Multilivello](#242-sicurezza-multilivello)

# 2. Protezione e Sicurezza

La **Protezione**:
> Riguarda l'insieme di attività che si preoccupano di garantire all'interno di un sistema di calcolo il **controllo dell'accesso** alle risorse logiche e fisiche da parte degli utenti

La **Sicurezza**:
> Garantisce l'autenticazione degli utenti, impedendo accessi non autorizzati al sistema, tentativi dolosi di alterazione e distruzione dei dati


Il livello di _controllo degli accessi_ della protezione è suddivisibile in tre livelli concettuali:
- **Modelli**: deve stabilire le relazioni tra le entità
- **Politiche**: stabilito un modello, ci dice in che modo assegnare i diritti di accesso
- **Meccanismi**: a _runtime_ mettono in atto le scelte delle politiche

## 2.1. Modelli di Protezione

Un modello di protezione definisce i **soggetti** e gli **oggetti** ai quali i soggetti hanno accesso e i **diritti di accesso**, ovvero le operazioni con le quali si può accedere agli oggetti.

I **soggetti** sono _la parte attiva di un sistema_, cioè i processi che agiscono per conto degli utenti per accedere a determinati oggetti. In `UNIX` un soggetto è rappresentato da una tripla di interi `S = <UID, GID, PID>`

Gli **oggetti** costituiscono la parte passiva di un sistema, ovvero le _risorse_ (sia fisiche che logiche)

I **diritti di accesso** possono riguardare anche altri soggetti. In questo caso stabiliscono come un osggetto può controllarne un altro.

Un soggetto può quindi essere considerato come una coppia **_(processo, dominio)_**.

Definiamo dominio:
> L'ambiente di protezione nel quale il soggetto sta eseguendo, ovvero l'insieme di diritti di accesso posseduti dal processo

Un dominio di protezione è **unico** per un soggetto, mentre un processo può cambiare il dominio associato durante la sua esecuzione.

Il soggetto $S_i$ può rappresentare il processo $P$ mentre esegue in un dominio di protezione, e il soggetto $S_j$ rappresenta lo stesso processo mentre esegue un altro dominio.
Un esempio semplice di questo è il passaggio del _gate_ dato dalle _system calls_, che cambiano i diritti del processo, senza cambiare il `PID` del processo.


## 2.2. Politiche di Protezione

Definiscono le **regole con le quali i soggetti posson oaccedere agli oggetti**.

Le politiche più famose e diffuse sono:
- **Discretionary Access Control** `DAC`: Il creatore di un oggetto controlla i diritti di accesso per quell'oggetto. È quella utilizzata da `UNIX`
- **Mandatory Access Control** `MAC`: I diritti di accesso vengono gestiti centralmente. Sono utilizzate in installazioni di alta sicurezza come negli enti governativi
- **Roled-Based Access Control** `RBAC`: Ad un ruolo sono assegnati specifici diritti di accesso sulle risorse. Un utente può appartenere a diversi ruoli

Una caratteristica comune delle politiche di protezione è quella del **_principio del privilegio minimo_**:
> Ad un soggetto sono garantiti i diritti di accesso **solo agli oggetti strettamente necessari per la sua esecuzione**.

## 2.3. Meccanismi di Protezione

SOno gli strumenti messi a disposizione dal sistema di protezione e per **imporre una determinata politica**.

La politica definisce **_cosa va fatto_** mentre il meccanismo **_come va fatto_**.

I meccanismi di protezione devono essere sufficientemente generali per consentire l'applicazione di diverse politiche di protezione.

## 2.4. Dominio di protezione

Un dominio definisce un **insieme di oggetti** e i **tipi di operazioni che si possono eseguire su ciascun oggetto**, detti **_diritti di accesso_**:
<div class="p"><code>&lt;oggetto, insieme diritti di accesso&gt;</code></div>

I vari domini possono essere **disgiunti** oppure **con diritti di accesso in comune**.

Un soggetto può quindi accedere solo agli oggetti definito nel dominio al quale è associato.
L'associazione può essere:
- **statica**: l'insieme delle risorse disponibili rimane fisso durante il suo tempo di vita. Questo comporta che l'insieme globale delle risorse che un processo potrà utilizzare non sarà diponibile prima delle sua esecuzione
- **dinamica**: l'assocazione tra processo e dominio varia durante l'esecuzione. Implica l'esistenza di un _meccanismo che consenta il passaggio da un dominio ad un altro_.

In `UNIX` il dominio è rappresentato da `UID + GID`, definendo quindi il soggetto come `S = <PID, UID, GID>`, e si ha un **associazione dinamica**.

Per cambiare dominio (passare da system a user e viceversa) si possono utilizzare due metodi:
- Il passaggio da un _gate_ della tabella `IDT` tramite le _system call_, che sfrittano le interruzioni.
- Utilizzando il comando `exec` in programmi con `SUID = 1`.

### 2.4.1. Matrice degli Accessi

Un sistema di protezione può essere rappresentato utilizzando il modello della **_matrice degli accessi_**.

Questo modello mantiene tutta l'informazione che specifica il tipo di accessi che i soggetti hanno per gli oggetti.

Consente di rappresentare lo **stato di protezione** garantendo il rispetto dei vincoli di accesso per ogni tentatico di accesso di un sogetto ad un oggetto

Il _meccanismo_ associato ha il compito di **verificare se una richiesta di accesso di un processo che opera in un certo dominio è _consentita oppure no_**.

Dinamicamente possiamo modificare il numero degli oggetti e dei soggetti all'interno della matrice, consentendo ai processi di cambiare dominio anche durante l'esecuzione.

Possiamo quindi modificare in _**modo controllato**_ il cambiamento dello stato di protezione, sancendo una **_transizione di stato_**.

Un esempio:
<div class="flexbox" markdown="1">

|       |       $X_1$       |      $X_2$      |   $X_3$   |        $S_1$        |      $S_2$       |    $S_3$    |
| :---: | :---------------: | :-------------: | :-------: | :-----------------: | :--------------: | :---------: |
| $S_1$ |      `read*`      |     `read`      | `execute` |                     |   `terminate`    |  `receive`  |
| $S_2$ |                   | `owner` `write` |           | `control` `receive` |                  | `terminate` |
| $S_3$ | `write` `execute` |                 |  `read`   |       `send`        | `send` `receive` |             |

</div>

In questo modo, il meccanismo consente di assicurare che un processo nel dominio $D_j$ possa accedere solo agli oggetti specificati nella riga $i$ e solo con i diritti di accesso indicati.

Quando un operazione $M$ deve essere eseguita nel dominio $D_i$ sull'oggetto $O_j$ il meccanismo consente di controllare che **_$M$ sia contentenuta nell'elemento $(i, j)$_**.

Attraverso il _Discretional Access Control_ (`DAC`) sono gli utenti a decidere il contenuto degli elementi della matrice.
Alla creazione di un nuovo oggetto $O$, questo sarà aggiunto alle colonne della tabella. Sarà quindi l'utente `owner` a stabilire come settare le intersezioni con le altre righe della tabella.

La modifica può essere ottenuta attraverso un **_opportuno insieme di comandi_**, stabiliti dai ricercatori _Graham_ e _Denning_.
I comandi si dividono in due tipi:
- Propagazione dei diritti di accesso
- Aggiunta e rimozione dei diritti di accesso in modo libero da parte dell'_owner_


La propagazione:

> Un soggetto $S_i$ può trasferire un diritto di accesso $\alpha$ per un oggetto $X$ ad un altro soggetto $S_j$ **solo se** $S_i$ ha accesso a $X$ e $\alpha$ ha il **_copy flag_**.

Il **_copy flag_** `*` determina la possibilità di copiare un diritto di accesso da un dominio ad un altro della matrice di accesso.

È stato dimostrato che le regole definite da _Graham_ e _Denning_ danno luogo ad un sistema di protezione in grado di risolvere problemi come:
- **Confinement**:
- **Trojan Horse**

#### 2.4.1.1. Realizzazione della matrice

Possiamo vedere la matrice per accessi come una serie di **_righe_** o di **_colonne_**.

Se decidiamo di memorizzare la tabella per colonne si dice **_Access Control List_** `ACL`: ad ogni oggetto è associata una lista che contiene tutti i soggetti che possono accedere all'oggetto e i relativi diritti di accesso.


In questo caso avremmo un `ACL` per ogni oggetto. In `UNIX` gli oggetti sono _file_, e le singole `ACL` sono conservate all'interno del **descrittore del file**.
In particolare le `ACL` sono rappresentate dai 9 **_bit di protezione_**.
I primi `3 bit` rappresentano un singolo soggetto, l'`owner`. I `3 bit` successivi rappresentano i soggetti che sono nel **gruppo del proprietario**. Gli ultimi `3 bit` invece stabiliscono i **_diritti per tutti gli altri soggetti_**.

Se invece la memorizziamo per righe si dice **_Capability List_** `CL`: ad ogni soggetto è associata una lista che contiene gli oggetti accessibili dal soggetto ed i relativi diritti di accesso.
Questa tecnica permette di avere un unica struttura dati sulla quale effettuare le ricerche da parte di un processo. Questo permette meccanismi di protezione a _runtime_ molto più efficaci, in quanto non è più necessario scorrere l'`ACL` dopo averla trovata per effettuare un _check_ sui diritti del processo.

Nasce però un problema diverso, quello di dove salvare le `CL`. Infatti i soggetti _cambiano dinamicamente e frequentemente_ in un sistema.
Ciò rende più complesso come salvarli in maniera efficace all'interno del sistema, andando a compensare il guadagno che ottenevamo sulla verifica a _runtime_.

Ipotizziamo però che volessimo cambiare i diritti di accesso ad un sottoinsieme dei soggetti per un dato oggetto $O$.
In questo caso sarà quindi necessario **_aggiornare tutte le loro singole `CL`_**.

In `UNIX`, oltre ad avere i **_bit di protezione_**, si mette in atto un altro meccanismo.

Quando un processo `PID` apre un file tramite una `open`, specificando un azione, il sistema operativo verifica che il file esista e che i diritti combacino prima di creare un nuovo `fd` che punta alla tabella dei file aperti di sistema.

La verifica accade per step:
1. La `open` controlla se `UID` del processo corrisponde con quello dell'`owner` dell'oggetto. Se è l'owner controlla i diritti di owner per verificare se l'operazione è consentita o meno
2. Si controlla se il `GID` corrisponde con quello dell'`group` dell'oggetto. Se corrispondono effettuerà il controllo di protezione sui bit del _owner group_
3. Altrimenti effettua il controllo sui bit per gli `others`.

Solo al termine di questi controlli senza errori viene inserito il `fd` nella **_Tabella dei File Aperti di Processo_**, che a tutti gli effetti agisce come `CL` dinamica del processo.

A livello statico quindi `UNIX` salva la matrice per `ACL`, ma quando un processo esegue si costruisce dinamicamente la propria `CL`.

Un elemento di della `ACL` quindi può avere la forma: `UID, GID: <diritti>`.
Subentra quindi il concetto di **ruolo**: lo stesso utente può appartenere a gruppi diversi e quindi avere diritti diversi.
Infatti è possibile permettere elementi `*, GID: <diritti>`, dove l'asterisco stabilisce che la regola è valida per qualsiasi utente.

Quando un utente accede deve perciò **_specificare il gruppo di appartenenza_** così da evitare conflitti tra regole diverse.

Inoltre è possibile sancire diritti **ad hoc** per un utente indipendentemente dal gruppo: `UID, *: <diritti>`, oppure stabilire i diritti base per chiunque non rispetti una delle altre regole `*,* : <diritti>`.

### 2.4.2. Sicurezza Multilivello

La maggior parte dei sistemi operativi permette ai singoli utenti di determinare chi possa leggere e scrivere i loro file e i loro oggetti, secondo il **_Controllo Discrezionale degli Accessi_**, o `DAC`.

In alcuni ambienti è però richiesta una _sicurezza più rigorosa e controllata_ (ospedali, aziende, ambienti militari, ...).
In questo caso sono stabilite delle **regole** su chi può vedere cosa, modificabili solo dpop aver **_ottenuto permessi speciali_**. Questa tecnica ha il nome di **_Controllo degli Accessi Obbligatorio_**, o `MAC`.

Un ulteriore criterio di sicurezza può essere quello di avere diversi **_livelli di protezione_**, ognuno con criteri di accesso diversi.
Questo modello si chiama **_Modello Bell-La Padula_**, progettato per grstire la sicurezza in ambiene militare, che stabilisce 4 livelli di sicurezza:
1. Non classificato
2. Confidenziale
3. Segreto
4. Top-Secret

Oltre che agli oggetti $O$, questi livelli di sicurezza sono stabiliti **_anche ai soggetti_** $S$.
Inoltre si definisce la funzione $SC(\cdot)$ che rappresenta il livello di sicurezza di un entità.

Il modello si basa su due regole su come le informaizoni possono circolare:
- **_Proprietà di semplice sicurezza_**: un soggetto in esecuzione a livello $k$, può **leggere** solo oggetti al suo livello o a livelli inferiori, secondo la regola che $SC(S) \ge SC(O)$. 
- **_Proprietà *_**: un soggetto in esecuzio al livello di sicurezza $k$ può **scrivere** solamente oggitti al suo livello o a quelli superiori, secondo la regola che $SC(S) \le SC(O)$.

Immaginiamo di avere la seguente tabella:

<div class="flexbox" markdown="1">

|       |    $O_1$ (file riservato)    | $O_2$ (trojan-horse) |   $O_3$ (file di appoggio)   |
| :---: | :--------------------------: | :------------------: | :--------------------------: |
| $S_1$ | `owner`<br>`read`<br>`write` |      `execute`       |           `write`            |
| $S_2$ |                              | `owner`<br>`execute` | `owner`<br>`read`<br>`write` |

</div>

Ipotizziamo che $O_2$ abbia un istruzione che gli permetta di aprire $O_1$ e leggerlo e un altra che gli permetta di aprire in scrittura $O_3$.

Se $S_1$ eseguisse $O_2$ questo:
- Potrebbe accedere in lettura a $O_1$ in quanto $S_1$ è _owner_, quindi il processo può leggerlo
- Potrebbe accedere in scrittura a $O_3$ in quanto $S_1$ ha i permessi

In questo modo $S_2$, che invece ha gli accessi in lettura a $O_3$ può leggere il contenuto copiato da $O_1$.

Con la classificazione _Bell-La Padula_ possiamo classificare come **Pubblico** $(P)$ e **Privato** $(R)$ i soggetti e gli oggetti:
- $SC(S_1) = SC(O_1) = R$
- $SC(S_2) = SC(O_2) = SC(O_3) = P$

In questo caso l'operazione di lettura viene permessa, mentre la **_Proprietà * vieta l'operazione di scrittura_**, in quanto $O_3$ ha un livello di protezione _inferiore a quello del soggetto_.

Il modello _Bell-La Padula_ è stato concepito per mantenere i segreti e non **_per garantire l'integrità dei dati_**.

Esiste un altro modello detto **_BiBa_** che permette la garanzia dell'integrità dei dati, che di fatto inverte le regole del modello _Bell-La Padula_, leggendo solo i file di livello equo o superiore e scrivendo solo file di livello equo o inferiore.

Ovviemente i due modelli, _BiBa_ e _Bell-La Padula_, **_non sono compatibili_** e non si possono realizzare contemporaneamente.