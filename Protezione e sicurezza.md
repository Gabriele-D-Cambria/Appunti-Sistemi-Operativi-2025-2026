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
			- [2.4.1.2. Meccanismi `UNIX`](#2412-meccanismi-unix)
		- [2.4.2. Sicurezza Multilivello](#242-sicurezza-multilivello)

# 2. Protezione e Sicurezza

La **Protezione**:
> Riguarda l'insieme di attività che si preoccupano di garantire il **controllo dell'accesso** alle risorse logiche e fisiche da parte degli utenti all'interno di un sistema di calcolo

La **Sicurezza**:
> Garantisce l'autenticazione degli utenti, impedendo accessi non autorizzati al sistema e tentativi dolosi di alterazione/distruzione dei dati

Il livello di _controllo degli accessi_ della protezione è suddivisibile in tre concetti:
- **Modelli**: stabiliscono le relazioni tra le entità
- **Politiche**: stabilito un modello, regolamentano come assegnare i diritti di accesso
- **Meccanismi**: mettono in atto le scelte delle politiche durante l'esecuzione

## 2.1. Modelli di Protezione

Un modello di protezione definisce:
- **Soggetti**: sono _la parte attiva di un sistema_, cioè i processi che agiscono per conto degli utenti per accedere a determinati oggetti. In `UNIX` un soggetto è rappresentato da una tripla di interi `S = <UID, GID, PID>`
- **Oggetti**: costituiscono la parte passiva di un sistema, ovvero le _risorse_ (sia fisiche che logiche)
- **Diritti di Accesso**: definiscono le operazioni con le quali un soggetto può accedere ad un oggetto o ad un altro soggetto

Possiamo rappresentare un soggetto come la coppia **_(processo, dominio)_**, dove per dominio intendiamo:
> L'ambiente di protezione nel quale il soggetto sta eseguendo, ovvero l'insieme di diritti di accesso posseduti dal processo

I domini di protezione sono **unici per ogni soggetto**, mentre un processo può cambiare il dominio associato durante la sua esecuzione.

Il soggetto $S_i$ può rappresentare il processo $P$ mentre esegue in un dominio di protezione, e il soggetto $S_j$ rappresenta lo stesso processo mentre esegue un altro dominio.
Un esempio semplice di questo è il cambio di livello di esecuzione dovuto alle _system-call_, che cambiano i diritti del processo mantenendone però il `PID`.

## 2.2. Politiche di Protezione

Definiscono le **regole con le quali i soggetti possono accedere agli oggetti**.

Le politiche più famose e diffuse sono:
- **Discretionary Access Control** `DAC`: Il creatore di un oggetto ha pieno controllo sui diritti di accesso per quell'oggetto. È quella utilizzata da `UNIX`
- **Mandatory Access Control** `MAC`: I diritti di accesso vengono gestiti centralmente. È una politica utilizzata in installazioni di alta sicurezza come negli enti governativi
- **Roled-Based Access Control** `RBAC`: I diritti di accesso sulle risorse sono associati a determinati _ruoli_. Un utente può appartenere a diversi ruoli

Una caratteristica comune delle politiche di protezione è quella del **_principio del privilegio minimo_**:
> Ad un soggetto sono garantiti i diritti di accesso **solo agli oggetti strettamente necessari per la sua esecuzione**.

## 2.3. Meccanismi di Protezione

Sono gli strumenti messi a disposizione dal sistema di protezione e per **imporre una determinata politica**.

La politica definisce **_cosa va fatto_** mentre il meccanismo **_come va fatto_**.

I meccanismi di protezione devono essere sufficientemente generali per consentire l'applicazione di diverse politiche di protezione.

## 2.4. Dominio di protezione

Un dominio definisce un **insieme di oggetti** e i **_diritti di accesso a tali oggetti_**, ovvero i **tipi di operazioni che si possono eseguire su ciascuno**:
```cpp
	<oggetto, insieme diritti di accesso>
```

Più domini possono essere tra di loro **disgiunti** o **condividere uno o più diritti di accesso**.

Un soggetto può quindi accedere solo agli oggetti definiti nel dominio al quale è associato.

L'associazione può essere di due tipi:
- **Statica**: l'insieme delle risorse disponibili rimane fisso durante il _lifespan_ del soggetto. Questo comporta che l'insieme globale delle risorse che un processo potrà utilizzare non sarà diponibile prima delle sua esecuzione
- **Dinamica**: l'assocazione tra processo e dominio varia durante l'esecuzione. Implica l'esistenza di un _meccanismo che consenta il passaggio da un dominio ad un altro_.

In `UNIX`:
- Il dominio è rappresentato dalla coppia `UID`,`GID`
- Il soggetto è definito come `S = <PID, UID, GID>`
- Si utilizza una **associazione dinamica**.

Per cambiare dominio (passare da `system` a `user` e viceversa) si possono utilizzare due metodi:
- Il passaggio da un _gate_ della tabella `IDT` tramite le _system call_, (sfruttando le interruzioni).
- Utilizzando il comando `exec` in programmi con `SUID = 1`

### 2.4.1. Matrice degli Accessi

Un sistema di protezione può essere rappresentato utilizzando il modello della **_matrice degli accessi_**.

Questo modello mantiene le informazioni che specificano il tipo di accesso che i soggetti hanno per gli oggetti all'interno di una struttura tabellare. Ciò consente di rappresentare lo **stato di protezione** garantendo il rispetto dei vincoli.

Il _meccanismo_ associato a questa struttura ha il compito di **verificare se le richieste di accesso di un processo che opera in un certo dominio sono _consentite oppure no_**.

Il numero di oggetti e dei soggetti all'interno della matrice è _dinamicamente modificabile_, permettendo ai processi di cambiare dominio anche durante l'esecuzione.

Possiamo quindi modificare in _**modo controllato**_ il cambiamento dello stato di protezione, attraverso vere e proprie **_transizioni di stato_**.

Un esempio di _matrice degli accessi_ può essere il seguente, dove $D_i$ sono i domini e $O_i$ sono gli oggetti:
<div class="flexbox" markdown="1">

|       |       $O_1$       |      $O_2$      |   $O_3$   |        $D_1$        |      $D_2$       |    $D_3$    |
| :---: | :---------------: | :-------------: | :-------: | :-----------------: | :--------------: | :---------: |
| $D_1$ |      `read*`      |     `read`      | `execute` |                     |   `terminate`    |  `receive`  |
| $D_2$ |                   | `owner` `write` |           | `control` `receive` |                  | `terminate` |
| $D_3$ | `write` `execute` |                 |  `read`   |       `send`        | `send` `receive` |             |

</div>

Il _meccanismo_ verificherà quindi che il processo nel dominio $D_i$ possa accedere solo agli oggetti specificati nella riga $i$..
Quando nel dominio $D_i$ un operazione $M$ deve essere eseguita su un oggetto $O_j$, il _meccanismo_ controlla che **_$M$ sia tra le operazioni contentenute nell'elemento $(i, j)$ della tabella_**.

La politica del _Discretional Access Control_ (`DAC`) permette agli utenti di decidere il contenuto degli elementi della matrice.
In particolare, alla creazione di un nuovo oggetto $O$, questo sarà aggiunto alle colonne della tabella, e **l'utente `owner`** stabilirà come settare le intersezioni con le righe della tabella.

Sono consentite anche le modifiche ai diritti di accesso, attraverso un **_opportuno insieme di comandi_**.

Questi comandi sono stati stabiliti dai ricercatori _Graham_ e _Denning_, e si dividono in due tipi:
- Propagazione dei diritti di accesso
- Aggiunta e rimozione dei diritti di accesso in modo libero da parte dell'_owner_


La **propagazione** è subordinata alla seguente definizione:

> Un soggetto $S_i$ può propagare un diritto di accesso $\alpha$ per un oggetto $X$ ad un altro soggetto $S_j$ **se e solo se** $S_i$ ha accesso a $X$ e $\alpha$ ha il **_copy flag_**.

Il **_copy flag_** `*` determina quindi la possibilità di "copiare" un diritto di accesso da un dominio ad un altro.

È stato dimostrato che le regole definite da _Graham_ e _Denning_ danno luogo ad un sistema di protezione in grado di risolvere problemi come:
- **Confinement**
- **Trojan Horse**

#### 2.4.1.1. Realizzazione della matrice

Possiamo vedere la matrice per accessi come una serie di **_righe_** (**Capability List**) o di **_colonne_** (**Access Control List**).

La memorizzazione per colonne prende il nome di **_Access Control List_** (`ACL`), ed è quella utilizzata nei sistemi `UNIX`. In questo modo ad ogni oggetto è associata una lista che contiene tutti i soggetti che possono accedevi e i relativi diritti di accesso.

Il numero di `ACL` da salvare nel sistema sarà identico al numero di oggetti presenti. Poiché in `UNIX` gli oggetti sono _file_, le singole `ACL` sono conservate all'interno del **descrittore del file**, sottoforma dei **_9 bit di protezione_**:
- I primi `3 bit` rappresentano i diritti di un singolo soggetto, l'`owner`.
- I secondi `3 bit` rappresentano i diritti dei soggetti che sono nel **gruppo del proprietario**.
- Gli ultimi `3 bit` stabiliscono i diritti **per tutti gli altri soggetti**.

Un elemento della `ACL` assume la forma: `UID, GID: <diritti>`.
Subentra quindi il concetto di **ruolo** in quanto uno stesso utente può appartenere a gruppi diversi, avendo quindi diritti diversi sullo stesso oggetto. Per evitare conflitti tra regole diverse è quindi **_necessario specificare un gruppo di appartenenza_** quando l'utente vuole accede, al fine di evitare conflitti tra regole diverse.

Le entrate della `ACL` possono anche assumere forme più generali attraverso l'utilizzo del carattere `*`:
- `*, GID: <diritti>`: la regola è valida per qualsiasi utente che appartiene al gruppo.
- `UID, *: <diritti>`: la regola è valida **ad hoc** per un utente indipendentemente dal gruppo
- `*,* : <diritti>`: la regola è valida come base per chiunque non rispetti una delle altre regole

La memorizzazione per righe si dice invece **_Capability List_** (`CL`), e associa ad ogni soggetto una lista che contiene gli oggetti da lui accessibili i relativi diritti di accesso.
A differenza delle `ACL`, questa tecnica permette di avere **un unica struttura dati** sulla quale effettuare le ricerche da parte di un processo. I meccanismi di protezione che si basano su questa struttura sono quindi molto più efficaci, in quanto non è più necessario scorrere l'`ACL` dopo averla trovata per effettuare un _check_ sui diritti del processo.

Nasce però un altro problema, ovvero quello di dove salvare le `CL`. I soggetti, a differenza degli oggetti, _cambiano dinamicamente e frequentemente_, rendendo più complesso il salvataggio e la gestione della `CL` all'interno del sistema, andando a compensare il guadagno che ottenevamo sulla verifica a _runtime_.

Ipotizziamo infatti che volessimo cambiare i diritti di accesso ad un sottoinsieme dei soggetti per un dato oggetto $O$. Mentre nell'`ACL` ci basta accedere all'oggetto stesso e agire in maniera sequenziale, nel caso della `CL` sarà necessario **_aggiornare tutte le singole liste sul sistema_**, introducendo un enorme _overhead_.

#### 2.4.1.2. Meccanismi `UNIX`

In `UNIX`, oltre ad avere i **_bit di protezione_**, si mette in atto un altro meccanismo.

Quando un processo `PID` apre un _file_ tramite una `open`, specificando un azione, il sistema operativo verifica:
1. Che il file esista
2. Che i diritti combacino

La verifica di queste condizioni accade per step:
1. La `open` controlla se `UID` del processo corrisponde con quello dell'`owner` dell'oggetto. Se il soggetto è _owner_ controlla i **diritti di owner** per verificare se l'operazione è consentita o meno
2. Altrimenti, controlla se il `GID` corrisponde con quello dell'`group` dell'oggetto. Se corrispondono allora effettuerà il controllo di protezione sui **diritti del group owner**
3. Altrimenti effettua il controllo sui **diritti degli others**

Solamente quando si ha esito positivo su questi controlli, verrà inserito un nuovo _file descriptor_ `fd` nella **_Tabella dei File Aperti di Processo_**, che punterà alla **tabella dei file aperti di sistema**. La _tabella dei file aperti di processo_ agisce a tutti gli effetti come una `CL` dinamica del processo.

A livello statico quindi `UNIX` salva la matrice per `ACL`, ma quando un processo esegue si costruisce dinamicamente la propria `CL`, per avere i vantaggi di entrambe le soluzioni.

### 2.4.2. Sicurezza Multilivello

La maggior parte dei sistemi operativi permette ai singoli utenti di determinare chi possa leggere e scrivere i loro file e i loro oggetti, secondo il **_Controllo Discrezionale degli Accessi_**, o `DAC`.

In alcuni ambienti è però richiesta una _sicurezza più rigorosa e controllata_ (ospedali, aziende, ambienti militari, ...).
In questo caso sono stabilite delle **regole** su chi può vedere cosa, modificabili solo da chi **_possiede permessi speciali_**. Questa tecnica ha il nome di **_Controllo degli Accessi Obbligatorio_**, o `MAC`.

Un ulteriore criterio di sicurezza può essere quello di avere diversi **_livelli di protezione_**, ognuno con criteri di accesso diversi.
Questo modello si chiama **_Modello Bell-La Padula_**, progettato per gestire la sicurezza in ambiente militare.

Il **_Modello Bell-La Padula_** stabilisce 4 livelli di sicurezza:
1. Non classificato (pubblico)
2. Confidenziale
3. Segreto
4. Top-Secret (privato)

I livelli di sicurezza si applicano sia agli oggetti $O$ che **_ai soggetti_** $S$.

Il modello definisce una funzione $SC(\cdot)$ che rappresenta il livello di sicurezza di un entità, e la utilizza come criterio per stabilire come le informaizoni possono circolare.

Le regole che sfruttano la funzione:
- **_Proprietà di semplice sicurezza_**: un soggetto in esecuzione a livello di sicurezza $k$, può **leggere** solo _oggetti al suo livello o a livelli inferiori_, secondo la regola che $SC(S) \ge SC(O)$.
- **_Proprietà *_**: un soggetto in esecuzione al livello di sicurezza $k$, può **scrivere** solo _oggetti al suo livello o a livelli superiori_, secondo la regola che $SC(S) \le SC(O)$.

Per chiarire come opera il modello prendiamo la seguente tabella:

<div class="flexbox" markdown="1">

|       |    $O_1$ (file riservato)    | $O_2$ (trojan-horse) |   $O_3$ (file di appoggio)   |
| :---: | :--------------------------: | :------------------: | :--------------------------: |
| $S_1$ | `owner`<br>`read`<br>`write` |      `execute`       |           `write`            |
| $S_2$ |                              | `owner`<br>`execute` | `owner`<br>`read`<br>`write` |

</div>

Ipotizziamo che $O_2$ abbia accesso alle seguenti istruzioni:
- Aprire in lettura $O_1$
- Aprire in scrittura $O_3$

Se $O_2$ venisse eseguito da $S_2$ queso non potrebbe accedere in nessun modo ad $O_1$, in quanto $S_2$ non possiede alcun diritto sull'oggetto.

Se $O_2$ venisse invece eseguito da $S_2$, questo opererà con i diritti di $S_2$ potendo:
- Accedere in lettura a $O_1$
- Accedere in scrittura a $O_3$

Facendo così $S_2$ è in grado di **leggere indirettamente il contenuto di** $O_1$, attraverso la copia in $O_3$, effettuata da $O_2$ per conto di $S_1$.

Sfruttando invece la classificazione _Bell-La Padula_ possiamo classificare come **Pubblico** $(P)$ e **Privato** $(R)$ i soggetti e gli oggetti:
- $SC(S_1) = SC(O_1) = R$
- $SC(S_2) = SC(O_2) = SC(O_3) = P$


Se adesso $O_2$ venisse eseguito da $S_1$:
- Potrebbe accede in lettura per la **_Proprietà di semplice sicurezza_** &emsp; $SC(S_1) = R = SC(O_1)$
- Non potrebbe accedere in scrittura per la **_Proprietà *_** &emsp;$SC(S_1) = R \ge SC(O_3) = P$

È importante sottolineare che il modello _Bell-La Padula_ è stato concepito per __**mantenere segreti i dati_** e non **_per garantirne l'integrità_**.

Esiste infatti un altro modello, detto **_BiBa_**, che si preoccupa di garantire l'integrità dei dati. Questo modello di fatto inverte le regole del modello _Bell-La Padula_:
- La lettura è permessa solo su file di livello equo o superiore $SC(S) \le SC(O)$.
- La scrittura solo su file di livello equo o inferiore $SC(S) \ge SC(O)$.

Ovviemente i due modelli, _BiBa_ e _Bell-La Padula_, **_non sono compatibili_** e non possono essere realizzati contemporaneamente.