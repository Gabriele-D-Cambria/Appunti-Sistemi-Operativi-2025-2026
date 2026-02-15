---
title: Thread
---

# 1. Indice

- [1. Indice](#1-indice)
- [2. Thread](#2-thread)
- [3. Pthreads](#3-pthreads)
	- [3.1. Mutua esclusione](#31-mutua-esclusione)
	- [3.2. Sincronizzazione](#32-sincronizzazione)
		- [3.2.1. `wait`, `signal` e `broadcast`](#321-wait-signal-e-broadcast)
		- [3.2.2. Esempi](#322-esempi)


# 2. Thread

Un thread è un flusso di esecuzione indipendente all'interno di un processo.

Un singolo processo può avere più _thread_ associati, che condividono risorse e spazio di indirizzamento (o almeno parte di esso).

I _thread_ sono anche detti "processi leggeri" in quanto le operazioni di creazione, distruzione e cambio contesto sono meno onerose rispetto a quelle di un processo.

I _thread_ portano alcuni vantaggi, come **interazioni più semplici ed efficaci** e la **minore onerosità dei passaggi di contesto**.
Tuttavia comportano anche altri svantaggi, in particolare è necessario gestire **la concorrenza fra thread**, dovendo scrivere codice _thread safe_ che non comporta _deadlock_.

In `Linux` i _thread_ sono **suportati nativamente a livello di kernel**. Infatti il _thread_ è  **l'unità di scheduling**.
Il processo tradizionale dei processi `UNIX` può essere visto come un _thread che non condivide le proprie risorse_.

# 3. Pthreads

Lo sandard `POSIX` definisce la libreria `pthreads` per la **programmazione di applicazioni multithreaded protabili**.

Per poterla utilizzare è necessario includere la libreria `<pthread.h>` e compilare aggiungendola esplicitamente:
```bash
gcc <options> file.c -l pthread		# alcuni compilatori non hanno bisogno della specifica

# DEBIAN
gcc <options> file.c -l pthread -std=99
```

Ed è possibile vedere la documentazione tramite:
```bash
man pthreads
```

Un _thread_ è identificato da un _id_ di tipo `pthread_t` recuperabile tramile la primitiva:
```c
pthread_t pthread_self(void)
```

`pthread_t` è un _tipo opaco_ (una `struct`), che può essere utilizzato solo mediante apposite funzioni, e non è convertibile semplicemente, ad esempio stampandolo a video.

Per confrontare due _id_ si utilizza:
```c
int pthread_equals(pthread_t tid1, pthread_t tid2);
```

L'esecuzione di un programma determina la creazione di un primo thread che esegue il codice del `main`.

Il thread iniziale può successivamente generare una gerarchia di _thread_ utilizzando:
```c
/**
* @param thread: puntatore ad identificatore di thread dove verrà scritto l'ID del thread creato
* @param attr: attributi del thread, NULL per usare valori di default
* @param start_routine: puntatore alla funzione che contiene il codice del nuovo thread
* @param arg puntatore che viene passato come argomento a `start_routine`
*
* @returns `0` in assenza di errore, un valore diverso altrimenti
*/
int pthread_create( pthread_t* thread,
					const pthread_attr_t* attr,
					void* (*start_routine)(void*),
					void* arg );
```

Un _thread_ può terminare volontariamente la sua esecuzione con il comando:
```c
/**
* L'esecuzione del thread termina e il sistema libera le risorse allocate
* Se un thread padre termina i figli **continuano la loro esecuzione**,
* tranne quelli nello stato `zombie`, che terminano a loro volta.
*
* @param retaval valore di ritorno del thread consultabile da altri thread utilizzando la @ref `pthread_join`
*/
void pthread_exit(void* retval);

/**
* Blocca un thread in attesa della terminazione di un thread specifico.
*
* @param thread ID del thread di cui attendere la terminazione
* @param retval puntatore al puntatore dove verrà salvato l'indirizzo restituito dal thread con la @ref `pthread_exit`.
*				Può essere impostato a `NULL` in caso volessimo ignorarlo.
*
*
* @returns `0` in caso di successo, altrimenti un codice di errore.
*/
int pthread_join(pthread_t thread, void** retval);
```


Di seguito possiamo vedere un esempio di creazione di un thread:
```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>

/* Corpo del thread */
void* tr_code(void* arg) {
	printf("Hello World! My arg is %d\n", *(int*)arg);
	free(arg);
	pthread_exit(NULL);
}

int main() {
	pthread_t t1, t2;
	int* arg1 = (int*)malloc(sizeof(int));
	int* arg2 = (int*)malloc(sizeof(int));

	*arg1 = 1;
	*arg2 = 2;

	int ret;

	/* Creo il primo thread passandogli `1` come argomento */
	ret = pthread_create(&tr1, NULL, tr_code, arg1);
	if (ret) {
		printf("Error: return code from pthread_create is %d\n", ret);
		exit(-1);
	}

	/* Creo il secondo thread passandogli `2` come argomento */
	ret = pthread_create(&tr2, NULL, tr_code, arg2);
	if (ret) {
		printf("Error: return code from pthread_create is %d\n", ret);
		exit(-1);
	}

	//! ATTENZIONE
	// La create non ha mandato in esecuzione i thread, quello è scelto dallo schedulatore
	pthread_exit(NULL);
}
```

Possiamo generalizzare la creazione di `NTHREADS` utilizzando una `#define`:
```c
// ... headers
#define NTHREADS 10

// ... thread code

int main() {
	pthread_t tr[NTHREADS];
	int* args[NTHREADS];
	int ret;
	for (int i = 0; i < NTHREADS; ++i) {
		args[i] = (int*)malloc(sizeof(int));
		args[i] = i;
		ret = pthread_create(&tr[i], NULL, tr_code, args[i]);
		if (ret) {
			printf("Error: return code from pthread_create is %d\n", ret);
			exit(-1);
		}
	}

	pthread_exit(NULL);
}
```

## 3.1. Mutua esclusione

La libreria pthread mette a disposizione **_l'astrazione della variabile di tipo `mutex`_**, analoga all'astrazione di un semaforo binario per risolvere problemi di mutua esclusione.

Infatti nella libreria è definito il tipo `pthread_mutex_t` che rappresenta:
- Lo stato del `mutex`
- La coda dove verranno sospesi i processi in attesa che il `mutex` sia libero

È un **_semaforo binario_**, quindi il suo stato può assumere due valori (libero o occupato):
```c
// Definire una variabile mutex
pthread_mutex_t M;

/**
* Permette di inizializzare la variabile mutex.
*
* @param M puntatore al mutex da inizializzare
* @param mattr puntatore a una struttura con attributi di inizializzazione. Se `NULL` vengono utilizzati i valori di default (mutex libero)
*
* @returns
*/
int pthread_mutex_init(pthread_mutex_t* M, const pthread_mutexattr_t* mattr);
```

La `wait` e la `signal` sul mutex sono realizzata con le primitive:
```c
// Entrambe restituiscono `0` in caso di successo, altrimenti un codice di errore

int pthread_mutex_lock(pthread_mutex_t* M);
int pthread_mutex_unlock(pthread_mutex_t* M);
```

Un classico utilizzo di un semaforo è il seguente:
```c
pthread_mutex_t M;

// ...

pthread_mutex_init(&M, NULL);

// Voglio utilizzare la risorsa
if (pthread_mutex_lock(&M)) {
	// Errore nella presa del lock
}

// Utilizzo la risorsa

// Rilascio la risorsa
if (pthread_mutex_unlock(&M)) {
	// Errore nel rilascio del lock
}
// Risorsa rilasciata
```

## 3.2. Sincronizzazione

Il semaforo di mutua esclusione permette una _sincronizzazione indiretta_ dei _thread_.

Per la **sincronizzazione diretta** dei _thread_ la libreria definisce le **_condition variables_**. Un thread può quindi sospendere in attesa del verificarsi di una determinata condizione, realizzando politiche avanzate di accesso alle risorse condivise e di sincronizzare i _thread_.

Una _condition variable_ è una **coda** con la quale i _thread_ possono sospendersi volontariamente in attesa di una condizione.

```c
pthread_cond_t C;

/**
* Inizializza una condition variable
* @param C puntatore alla condition variable da inizializzare
* @param attr attributi specificati per la condizione. Se `NULL` inizializzata a default
*/
int pthread_cond_init(pthread_cond_t* C, pthread_cond_attr_t* attr);
```

Un _thread_ può effettuare due "operazioni" su una _condition variable_:
- **Sospendersi** (`wait`): dopo aver verificato una determinata condizione, si sospende in attesa di essere risvegliato da un altro _thread_
- **Risvegliarsi** (`signal`/`broadcast`): può risvegliare uno (`signal`) o tutti (`broadcast`) i thread sospesi sulla _condition variable_

### 3.2.1. `wait`, `signal` e `broadcast`

La sospensione viene utilizzata al verificarsi di una particolare condizione logica ed è **_sempre bloccante_**.

```c
/*
* Si utilizza while perché il thread potrebbe essere risvegliato anche se la condizione logica non è stata modificata
* È quindi necessario ricontrollare la condizione dopo essere statis svegliati
*/
while (condizione_logica) {
	wait(condition_variable);
}
```

La `condizione_logica` è basata su una risorsa condivisa, quindi la sua verifica deve essere eseguita in _mutua esclusione_.

Tenendo conto di questo aspetto, la primitiva di `wait` permette di **_associare una variabile mutex a una variabile condition_**.
In questo modo il _lock_ della _mutua esclusione_ viene:
- Automaticamente rilasciato quando il _thread_ si sospende sulla `wait`
- Automaticamente preso quando il _thread_ viene risvegliato

Le primitive hanno quindi le seguenti forme:
```c
/**
* Sospende un thread nella coda associata a `C` e gestisce automaticamente
* il lock sulla risorsa `M`
*
* @param C variable condition sulla quale sospendersi
* @param M mutex associato alla condizione
*/
int pthread_cond_wait(pthread_cond_t* C, pthread_mutex_t* M);


/**
* Permette di risvegliare un thread sospeso su una condition variable `C`.
* Se non ci sono thread in attesa **non ha alcun effetto**.
* Se ci sono più thread in attesa ne viene scelto **uno a caso**
*
* @param C variable condition sulla quale risvegliare il processo
*/
int pthread_cond_signal(pthread_cond_t* C);

/**
* Permette di risvegliare **tutti** i thread sospesi su una condition variable `C`.
* Se non ci sono thread in attesa **non ha alcun effetto**.
*
* @param C variable condition sulla quale risvegliare i processo
*/
int pthread_cond_broadcast(pthread_cond_t* C);
```

La `pthread_cond_signal` segue una politica di tipo _signal&continue_, ovvero il thread che la invoca **_continua la propria esecuzione mantenendo il controllo dei `mutex`_**.

La `pthread_cond_broadcast` è utile qualora volessimo risvegliare **un thread specifico**.

Entrambe le `signal` e `broadcast` **_vanno invocate dentro la sessione critica per maggiore stabilità_**, così da avere la certezza che le condizioni vengano rispettate al momento della loro invocazione.

### 3.2.2. Esempi

Un primo esepio di utilizzo potrebbe essere l'accesso ad una risorsa condivisa, come un _ring-buffer_ dove:
- I _thread_ consumatori prelevano valori dal buffer
- I _thread_ produttori inseriscono nuovi valori dal buffer

La gestione ha due vincoli:
- Non si può inserire nel buffer pieno
- Non si può prelevare dal buffer vuoto

La risorsa può essere la seguente:
```c
typedef struct {
	int buffer[BUFFER_SIZE];
	itn readInd, writeInd;
	int cont;

	pthread_mutex_t M;

	pthread_cond_t FULL;
	pthread_cond_t EMPTY;
} risorsa;

risorsa r;
int main() {
	pthread_mutex_init(&r.M, NULL);

	pthread_cond_init(&r.FULL, NULL);
	pthread_cond_init(&r.EMPTY, NULL);

	r.readInt = r.writeInd = r.cont = 0;
	// ...
}
```

<div class="grid2">
<div class="top">
<p class="p">Thread Consumatore</p>

```c
// ...

ptrhead_mutex_lock(&r.M);

while (r.cont == 0) {
	pthread_cond_wait(&r.EMPTY, &r.M);
}

int var = r.buffer[r.readInd];
r.cont--;
r.readInt = (r.readInt + 1) % BUFFER_SIZE;

/*
* Adesso c'è spazio, quindi risvegliamo eventuali thread produttori in attesa
*  Usiamo signal perché abbiamo prelevato un valore, quindi possiamo garantire solamente
*  che si sia liberato uno slot, quindi non ha senso risvegliarli tutti i thread produttori
*/
pthread_cond_signal(&r.FULL);
pthread_mutex_unlock(&r.M);

// ...
```

</div>
<div class="top">
<p class="p">Thread Produttore</p>

```c
// ...

ptrhead_mutex_lock(&r.M);

while (r.cont == BUFFER_SIZE) {
	pthread_cond_wait(&r.FULL, &r.M);
}

r.buffer[r.writeInd] = val;
r.cont++;
r.writeInd = (r.writeInd + 1) % BUFFER_SIZE;

/*
* Adesso c'è un nuovo valore, quindi risvegliamo eventuali thread consumatori in attesa
*  Usiamo signal perché abbiamo inserito un valore, quindi possiamo garantire solamente
*  che si sia occupato uno slot, quindi non ha senso risvegliare tutti i thread consumatori
*/
pthread_cond_signal(&r.EMPTY);
pthread_mutex_unlock(&r.M);

// ...
```

</div>
</div>


Un altro esempio è quello di fornire **accesso limitato ad una risorsa**.

Immaginiamo di avere `NTHREADS` che utilizzano periodicamente una risorsa che può essere utilizzata **contemporaneamente** da un numero massimo di `MAX_T` _thread_:

```c
#define MAX_T 10

int n_users = 0;
pthread_cond_t FULL;
pthread_mutex_t M;

// ...
```

Nella fase di ingresso avremo:
```c
// ...

pthread_mutex_lock(&M);

while (n_users == MAX_T) {
	pthread_cond_wait(&FULL, &M);
}

n_users++;
pthread_mutex_unlock(&M);

// Utilizzo la risorsa
```

Nella fase di uscita invece:
```c
// ...

pthread_mutex_lock(&M);

n_users--;
pthread_cond_signal(&FULL);

pthread_mutex_unlock(&M);
```