---
title: Classificazione delle Architetture
---

# 1. Indice

- [1. Indice](#1-indice)
- [2. Tassonomia di Flynn](#2-tassonomia-di-flynn)
	- [2.1. Macchine SISD](#21-macchine-sisd)
	- [2.2. Macchine SIMD](#22-macchine-simd)
	- [2.3. Macchine MISD](#23-macchine-misd)
	- [2.4. Macchine MIMD](#24-macchine-mimd)
		- [2.4.1. Macchine a Memoria Ditribuita DM-MIMD](#241-macchine-a-memoria-ditribuita-dm-mimd)
		- [2.4.2. Macchine a Memoria Condivisa SM-MIMD](#242-macchine-a-memoria-condivisa-sm-mimd)
	- [2.5. Confronto SIMD e MIMD](#25-confronto-simd-e-mimd)
- [3. Tipologie di interconnessione](#3-tipologie-di-interconnessione)
	- [3.1. Bus](#31-bus)
	- [3.2. Linear Array](#32-linear-array)
	- [3.3. Ring](#33-ring)
	- [3.4. Connessione Completa](#34-connessione-completa)
	- [3.5. Albero Binario](#35-albero-binario)
	- [3.6. Stella](#36-stella)
	- [3.7. Mesh (2D)](#37-mesh-2d)
	- [3.8. Toro (2D)](#38-toro-2d)
	- [3.9. Ipercubo](#39-ipercubo)
- [4. Metriche di Prestazione](#4-metriche-di-prestazione)
	- [4.1. Speed-up](#41-speed-up)
	- [4.2. Efficienza](#42-efficienza)
	- [4.3. Tempo Sequenziale](#43-tempo-sequenziale)
	- [4.4. Multitasking](#44-multitasking)


# 2. Tassonomia di Flynn

È un metodo di cclassificazione di sistemi di elaborazione da due punti di vista:
- In base alla capacità di avere più flussi di istruzioni
- In base alla capacità di avere più flussi di dati

<div class="flexbox" markdown="1">

|                                              | `SI` <br> (_Single Intruction stream_) | `MI` <br> (_Multiple Instruction stream_) |
| :------------------------------------------: | :------------------------------------: | :---------------------------------------: |
|  _**`SD`**_ <br> **(_Single Data stream_)**  |            Macchine `SISD`             |              Macchine `MISD`              |
| _**`MD`**_ <br> **(_Multiple Data stream_)** |            Macchine `SIMD`             |              Macchine `MIMD`              |


</div>

## 2.1. Macchine SISD

Sono macchine a singolo stream, che rappresentano le tradizionali macchine sequenziali basate sul modello di **Von Neumann** usata da tutti i calcolatori convenzionali.

<figure class="">
<img class="100" src="./images/Classificazione Architetture/SISD.png">
<figcaption>

Il cerchio rappresenta un circuito in grado di eseguire istruzioni e/o gestire dati.

</figcaption>
</figure>

## 2.2. Macchine SIMD

Si differenzia dalle macchine SISD per il numero di **_Data Processor_**, ciascuno dei quali possiede una propria **_Data Memory_**.

Questo permette a più unità di elaborazione di eseguire contemporaneamente la stessa istruzione, lavorando su flussi di dati differenti.


<figure class="">
<img class="100" src="./images/Classificazione Architetture/SIMD.png">
<figcaption>

Per permettere la condivisione a tutte le `DM` dell'istruzione, viene utilizzato uno _switch 1 a N_
</figcaption>
</figure>

La topologia di interconnessione tra i vari processori può essere sia _regolare_ che _creata ad hoc_.

Questa architettura permette comunicazioni regolari efficienti e poco costose, che non creano conflitti.

Il modello di computazione di queste macchine è **_Sincrono_**, ovvero gestito da un unica unità di controllo.
Questo permette due tipi di parallelismo:
- **Temporale** fasi diverse di un'unica istruzione sono eseguite in parallelo in differenti moduli connessi in cascata (_pipeline_)
- **Spaziale**: i medesimi passi sono eseguiti contemporaneamente su un arry di processori perfettamente uguali sincronizzaiti da un solo controllore

Alcuni esempi di architetture `SIMD`:
- Supercomputer vettoriali: tipici per lavorare su grandi matrici
- Vector Processor con caratteristiche _pipeline_
- Array Processor
- Systolic Array

I programmi che beneficiano dell'architettura `SIMD`, ad esempio per lavorare su grandi vettori, possono essere eseguiti, con opportune ma comunque piccole modifiche, da processori `SISD`.

Infatti avendo un operazione tra vettori `c = a + b`, il compilatore può tradurla in `for(...) c[i] = a[i] + b[i]`.


## 2.3. Macchine MISD

Queste macchine hanno più flussi di istruzioni che lavorano contemporaneamente su un unico fllusso di dati.

Molti considerano questa categoria ancora "vuota", ovvero senza esempi reali. Altri invece categorizzano i processori basati su _pipeline_ proprio come macchine `MISD`.

Per ulteriori informazioni sui processori moderni basati su pipeline consultare [gli appunti di Calcolatori dedicati](https://gabriele-d-cambria.github.io/Appunti-Calcolatori-Elettronici-2024-2025/Architettura%20Moderna%20CPU%20Intel#2-pipeline)

## 2.4. Macchine MIMD

In queste macchine abbiamo **tante unità di elaborazione** connesse a tante **untià di dati**. Abbiamo infatto più flussi di istruzioni in parallelo che elaborano insiemi di dati che possono essere _distinti_, _privati_ o _condivisi_.


Vediamo due categorie di macchien `MIMD`

### 2.4.1. Macchine a Memoria Ditribuita DM-MIMD

<figure class="">
<img class="100" src="./images/Classificazione Architetture/MD-MIMD.png">
<figcaption>

Non esiste memoria condivisa, ma ogni nodo esegue indipendentemente un flusso di istruzioni su un differente insieme di dati
</figcaption>
</figure>

Ogni coppia `IP`-`DP` (con le relative memoria) costituisce in pratica **una macchina** `SISD`.

Una qualsiasi **_rete di calcolatori_** rappresenta una macchina `DM-MIMD`. Infatti queste reti di interconnessione regolari permettono ai nodi di scambiare informazioni secondo il paradigma _message passing_. Queste reti permettono _algoritmi ad elevata località_ e un _elevata scalabilità_.

Nelle `DM-MIMD` troviamo due sottocategorie:
- `DM-MIMD MPP` (_Massively Parallel Processing_): L'elaborazione `MMP` è utilizzata in applicazioni scientifiche e in particolari ocntesti di calcolo commerciale-finanziario. Il sistema si basa su **migliaia di nodi** (**CPU** standard ogniuna con memoria e _SO_) e si una **rete di interconnessioen custom molto potente** (larga banda e bassa latenza). È inoltre necessario disporre di software _capace di partizionare il lavoro e i dati su vari processori_
- `DM-MIMD COW` (_Cluster Of Workstations_): Ha due caratteristiche principali: una **high-availability** che gli permette alla computazione di migrare da un nodo all'altro in caso di guasti, e un elevato **load-balancing** che permette di allocare i _task_ nel nodo con minor carico. Un esempio di questo tipo di connessioni è la **_Gigabit Ethernet_**

### 2.4.2. Macchine a Memoria Condivisa SM-MIMD

Sono macchine multiprocessore che permettono la condivisione della memoria tra processori attraverso delle **aree**.

Affinché questa architettura funzioni lo _switch NxN_ deve essere molto efficiente.

<figure class="">
<img class="100" src="./images/Classificazione Architetture/SM-MIMD.png">
<figcaption>

Se il numero di processori $N$ è "piccolo" $(N < 100)$, l'accoppiamento fra i nodi può essere _stretto_ (comunicazioni veloci)
</figcaption>
</figure>


A differenza delle `MD-MIMD` questa architettura ha una **scalabilità limitata**.

## 2.5. Confronto SIMD e MIMD

<div class="flexbox" markdown="1">

|                |                   `SIMD`                    |                              `MIMD`                               |
| :------------: | :-----------------------------------------: | :---------------------------------------------------------------: |
|   **_hardware_**   |     Poco, Unica **Unità di Controllo**      |                Molto, tante **Unità di Controllo**                |
|    **_costo_**     | Più costoso, hanno processori più specifici |         Meno costoso, hanno processori _general-purpose_          |
|   **_memoria_**    |  Poca, hanno una sola copia del programma   |               Molta, hanno più copie del programmma               |
| **_flessibilità_** |                    Poca                     | Alta flessibilità in termini di modelli computazionali supportati |

</div>


Possiamo vedere di seguito uno schema esteso che visualizza le principali tassonomie:

<img class="75" src="./images/Classificazione Architetture/Tassonomia Estesa.png">


# 3. Tipologie di interconnessione

Vediamo adesso diverse tipi di interconnessione, definnedo:
- **Grado della rete**: numero di connettori necessari per ogni nodo
- **Diametro della rete**: distanza tra una coppia di nodi misurata in link

<div class="grid2">
<div class="">

## 3.1. Bus

È la rete di interconnessione più semplice di tutte, composta da un unico _link_.

Ha alcune limitazioni di natura elettrica dovuta a interferenze elettromagnetiche, che possono degradare la velocità di connessione. Inoltre, nel caso di rottura del _link_ abbiamo zero tolleranza.

Dal punto di vista logico ha **competizione massima sull'accesso al mezzo**. È infatti necessario arbitrare gli accessi in caso di presenza di più _master_.

Questa rete ha:
- **_Grado 1_**
- **_Diametro 1_**

</div>
<div class="">
<img class="" src="./images/Classificazione Architetture/Bus.png">
</div>
<hr><hr>
<div class="">

## 3.2. Linear Array

È una rete composta da $N-1$ _link_, che collega un nodo al successivo.

Questo permette di **_ridurre al minimo la competizione_**. Nel caso ideale infatti possiamo avere $\frac{N}{2}$ **comunicazioni in contemporanea**.

In questa rete abbiamo nodi capaci di offrire servizi di _routing_, inoltrando informaizoni ai propri nodi adiacenti. La rottura di un nodo o di un _link_ crea due sottoreti che non possono comunicare.

Questa rete ha:
- **_Grado_** $1$ per il "primo" e l'"ultimo" nodo, $2$ per gli altri
- **_Diametro:_** $N-1$

</div>
<div class="">
<img class="" src="./images/Classificazione Architetture/Linear Array.png">
</div>
<hr><hr>
<div class="">

## 3.3. Ring

È la verisone migiorata del _Linear Array_, che aggiunge un _link_ (arrivando a $N$) tra il "primo" e l'"ultimo" nodo.

È inoltre tollerante a $1$ guasto, che lo rende un _Linear Array_

Questa rete ha:
- **_Grado:_** $2$
- **_Diametro:_** $\Bigl\lfloor\frac{N}{2}\Bigr\rfloor$
</div>
<div class="">
<img class="" src="./images/Classificazione Architetture/Ring.png">
</div>
<hr><hr>
<div class="">

## 3.4. Connessione Completa

È la soluzione più costosa, talmente tando da non essere scalabile. Il numero di _link_ con $N$ nodi è di $\frac{N\cdot (N-1)}{2}$.

Questa rete ha:
- **_Grado:_** $N-1$
- **_Diametro:_** $1$

</div>
<div class="">
<img class="" src="./images/Classificazione Architetture/Connessione Completa.png">
</div>
<hr><hr>
<div class="">

## 3.5. Albero Binario

Permette di liimtare il numero di _link_ a $N-1$.

È tuttavia una topologia non scalabile, poiché i rami alti hanno **_elevato rischio di congestione_**, poiché i nodi intermedi e la radice devono **_poter fare da router_**.

Inoltre la radice è un potenziale "punto debole", poiché la rottura di un suo link crea due sottoreti indipendenti.

Per migliorare la congestione e limitare la tolleranza ai guasti si più pensare ad una topologia detta **_fat-tree_**


Questa rete ha:
- **_Altezza_**: $h = \Bigl\lceil \log_2{N}\Bigr\rceil$
- **_Grado:_** $2$ per la radice, $1$ per le foglie e $3$ per gli altri
- **_Diametro:_** $2 \cdot (h-1)$
</div>
<div class="">
<img class="" src="./images/Classificazione Architetture/B-Tree.png">
</div>
<hr><hr>
<div class="">

## 3.6. Stella

È una rete con $N-1$ _link_, con tolleranza ai guasti **_fortemente dipendente dalla "robustezza" del nodo centrale_**, che fa da **_single point of failure_**.

Questa rete ha:
- **_Grado:_** $N-1$ per il nodo centrale, $1$ per gli altri
- **_Diametro:_** $2$
</div>
<div class="">
<img class="" src="./images/Classificazione Architetture/Star.png">
</div>
<hr><hr>
<div class="">

## 3.7. Mesh (2D)

Hanno come idea di base quella di creare delle matrici di nodi.

Possiede un numero di _link_ uguale a $2 \cdot N - 2 \cdot r$ dove $r = \sqrt{N}$.

Ha una buona tolleranza ai guasti, che va da $2$ a $4$ guasti.

Questa rete ha:
- **_Grado:_** $2$ per i vertici, $3$ per gli "spigoli" e $4$ per gli altri
- **_Diametro:_** $2 \cdot (r - 1)$
</div>
<div class="">
<img class="" src="./images/Classificazione Architetture/Mesh-2D.png">
</div>
<hr><hr>
<div class="">

## 3.8. Toro (2D)

Considerando sempre $r = \sqrt{N}$, migliora la struttura della _Mesh 2D_. 

Aumenta il numero totale di _link_ a $2N$, migliorano però notevolmente sia la resistenza ai guasti sia la scalabilità

Questa rete ha:
- **_Grado:_** $4$ per tutti i nodi
- **_Diametro:_** $2 \cdot \Bigl\lfloor\frac{r}{2}\Bigr\rfloor$
</div>
<div class="">
<img class="" src="./images/Classificazione Architetture/Torus-2D.png">
</div>
<hr><hr>
<div class="">

## 3.9. Ipercubo

Dato un ipercubo in $d$ dimensioni, abbiamo $N = 2^d$ nodi con $d \cdot \frac{N}{2} = d \cdot 2^{d-1}$ _link_.

Quetsa architettura è _scalabile_ **solo** con un numero di nodi che è potenza di $2$.

Questa rete ha:
- **_Grado:_** $d$
- **_Diametro:_** $d$

Dato l'elevato numero di nodi, questi sono numerati attraverso il **_Codice Binario di Gray_**.
</div>
<div class="">
<img class="" src="./images/Classificazione Architetture/Hypercube.png">
</div>
</div>

# 4. Metriche di Prestazione

Vediamo alcune metriche di prestazione, prendendo come riferimento un programma sequenziale eseguito su **_macchina `SISD`_**.

Abbiamo che:
- $T_1$: tempo di esecuzione su $1$ nodo
- $T_n$: tempo di esecuzione su $n$ nodi

## 4.1. Speed-up

Lo _speed-up_ è:
> Il rapporto tra l'esecuzione sequenziale con l'esecuzione con l'esecuzione su macchine `SIMD` o `MIMD`

$$
	S = \frac{T_1}{T_n}
$$

Ci aspettiamo che $S > 1$, ovvero di avere un guadagno di veocità all'aumentare dei nodi.

Idealmente, vorremmo avere uno _speed-up_ lineare $(S \in O(N))$ con il numero di processori usati nella macchina parallela. Nella realtà $S < N$, in particolare:
- `SIMD`: spesso $S \approx N$
- `MIMD`: è difficile far crescere $S$, poiché è legato al software specifico che deve cercare di far lavorare a pieno carico tutte le **CPU**

## 4.2. Efficienza

L'efficienza:
> È il rapporto tra lo _speed-up_ e il numero di processori

$$
	E = \frac{S}{N}
$$

Come abbiamo detto prima, l'ideale sarebbe avere $E = 1$, ma nella realtà $E < 1$

## 4.3. Tempo Sequenziale

Il _tempo sequenziale_:
> È il tempo impegato per eseguire istruzioni non parallelizzabili (operazioni I/O, costrutti condizionali, algoritmi intrinsecamente sequenziali)

Questo parametro è legato dalla **_Legge di Amdahl_**:
> Un parallelismo "perfetto" **_non è mai raggiungibile_** poiché saranno **_sempre presenti sequenze_** si _software_ intrinsecamente seriale

La legge ridefinisce lo _speed-up_:
$$
	S = \frac{T_1}{T_{seq}+\frac{T_1-T_{seq}}{N}} = \frac{N\cdot T_1}{T_1 + T_{seq}(N-1)}
$$

Possiamo notare che se $N \to \infty$:
$$
	\lim_{N\to\infty}S = \frac{T_1}{T_{seq}}
$$

Un esempio di algoritmo non parallelizzabile è il calcolo dell'$i$-esimo numero della sequenza di fibonacci:
$$
	f(i) = f(i-1) + f(i-2) \quad \wedge \quad \begin{cases}
		f(0) = f(1) = 1 \\
		i = 2, 3, 4, ...
	\end{cases}
$$

## 4.4. Multitasking

Ha un importanza notevole nelle macchine parallele per mantenere uno sfruttamento delle **CPU** _molto elevato_.

Per poter agire, **_deve rispettare il seguente vincolo_**:
$$
	P \gg N
$$