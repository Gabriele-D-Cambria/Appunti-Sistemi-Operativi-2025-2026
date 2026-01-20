---
title: Introduzione ai comandi UNIX
---

# 1. Indice

- [1. Indice](#1-indice)
- [2. Informazioni generali](#2-informazioni-generali)
	- [2.1. Shell](#21-shell)
	- [2.2. Redirizione di IO](#22-redirizione-di-io)
	- [2.3. Comandi privilegiati](#23-comandi-privilegiati)
- [3. Comandi `vi`](#3-comandi-vi)
- [4. Strumenti per la gestione dei file](#4-strumenti-per-la-gestione-dei-file)
	- [4.1. Find](#41-find)
	- [4.2. Locate](#42-locate)
	- [4.3. Grep](#43-grep)
- [5. Archiviazione e Compressione](#5-archiviazione-e-compressione)
	- [5.1. TAR](#51-tar)

# 2. Informazioni generali

Durante questo corso utilizzeremo una macchina virtuale con distribuzione `Ubuntu 24.04`.

Il sistema operativo si basa su una serie di file _montati_ tramite un unico _filesystem virtuale_.
La directory principale è `/` e da lì:
- `home/` contiene le _home directory_ degli utenti
- `dev/` device attaccati al sistema
- `lib/` librerie utilizzate dal sistema
- `bin/` cartella con i birnari delle applicazioni scaricate.
Per descrivere un _path_ si possono utilizzare:
- **Percorsi Assoluti**: `/home/utente/Documents/todolists/groceries.txt`
- **Percorsi relarivi**: a partire dalla directory dove ci si trova: `./Documents/todolists/groceries.txt`

Esistono diversi caratteri speciali:
- `~`: la **home directory** dell'utente
- `.`: indica la **directory corrente**
- `..`: indica la **directory padre**

## 2.1. Shell

È un interprete dei comandi che consente all'utente di richiedere informazioni e servizi al sistema operativo.
Ne esistono di due tipi:
- **Shell grafica** (_GUI_): più intuitiva e facile dausare
- **Shell Testuale** (_CLI_): più efficace se ci conoscono bene i comandi

Noi utilizzeremo la **shell testuale**. In questa shell:
- Viene mostrato un prompt
- Si legge il comando digitato dall'utente terminato con `\r\n`
- Viene eseguito il comando
  - Se non si è in grado di completarlo si segnala un'errore
  - Il comando viene completato e si reinizia il ciclo

In `bash` il prompt ha questa forma:
```bash
username@host:directoryAttuale$
# Il dollaro compare solo se si è utente semplice
# Per utente di root si ha il carattere `#`
```

Per effettuare il logout dalla shell testuale si può utilizzare la combinazione `Ctrl+D` o utilizzare il comando:
```bash
exit
```

Alcune funzioni utili sono:
- Autocompletamento con il tasto `TAB`
- Cronologia dei comandi recenti con le freccie $\uparrow$ e $\downarrow$
- Ricerca nella cronologia tramite `Ctrl+R`
- Nuovi terminali con la shortcut `Ctrl+Alt+T` (non su _Manjaro_ per qualche motivo)

Per spegnere e riavviare il pc, il `root` (su _Manjaro_ anche non il root):
```bash
sudo shutdown -h now		# Spegnere adesso
sudo shutdown -r now		# Riavviare il sistema
```

Per cambiare directory:
```bash
cd /lib		# Path assoluto
cd ./lib	# Path relativo, lo elabora dalla locazione attuale
cd ..		# Torno indietro (tranne se già nella cartella di root `/`)
cd 			# Torno a `~` (va bene anche `cd ~`)
```

Per ottenere il percorso assoluto della directory corrente si può utilizzare il comando:
```bash
pwd
```

Per mostrare il contenuto della directory specificata:
```bash
ls						# directory corrente
ls qualche_directory	# contenuto di qualche_directory
```

La maggior parte delle shell utilizza colori diversi per elencare diversi tipi di file. Lo standard per shell bash:
- **Blu**: cartelle
- **Bianco**: file
- **Verde**: eseguibili
- **Azzurro**: link

Il comando `ls` permette alcune opzioni (che possono essere concatenate):
- `-l` mostra più dettagli sui file
- `-a` mostra tutti i file, anche quelli nascosti (su sistemi `UNIX` sono quelli che iniziano con il carattere `.`)
- `-lH` mostra più dettagli e scrive le dimensioni in maniera _human readable_
- `-ll`: equivalente a `-la`

Per indicare i path è possibile utilizzare delle _wildcards_, ovvero caratteri _regex_ che permettono di generalizzare la ricerca:
- `*` sostituisce zero o più caratteri
- `?` sostituisce un carattere
- `[a,b,c]`: sostituisce un carattere con quelli specificati
- `[a-z]`: sostituisce un carattere con quello nel range `a-z`

Per visualizzare il funzionamente di un comando:
```bash
man nome_comando
```

Il manuale è diviso in sezioni, e funziona per:
1. Comandi
2. Funzioni del kernel
3. Funzioni delle librerie C
4. File di Configurazione

Se ci sono ambiguità si utilizza l'indice numerico:
```bash
man printf			# comando bash
man 3 printf		# funzione C
```

Per visualizzare la descrizione breve di una pagina del manuale si può utilizzare `whatis`.

Per gestire file (`file`) e directory (`dir`):
```bash
mkdir nome_cartella			# crea una directory
rmdir nome_cartella			# elimina una directory SOLO SE vuota

cp file dst					# copia file in dst
cp file1 file2 ... dst_dir 	# copia file1, file2, ... nella directory dst_dir
cp -r dir1 dir2 ... dst_dir # copia le directory dir1, dir2, ... nella directory dst_dir

mv src dst					# sposta src in dst se esiste come directory, altrimenti lo rinomina
mv src1 src2 ... dst_dir	# sposta src1, src2, ... in nella directory dst_dir

touch file					# Aggiorna il timestamp di accesso e modifica di un file. Se il file non esiste lo crea

cat file1 file2 ...			# Concatena il contenuto di più file e lo stampa nello standard output

rm file1 file2 ...			# Elimina file
rm -r dir1 dir2 ...			# Elimina le cartelle e tutto il suo contenuto

less						# Mostra un file "un po' alla volta" interattivamente
head						# Mostra la prima parte di uno o più file
tail						# Mostra l'ultima parte di uno o più file
```

`head` e `tail` prendono i modificatori:
- `-c`: permette di specificare il numero di Byte
- `-n`: permette di specificare il numero di righe

## 2.2. Redirizione di IO

I processi hanno tre canali di input/output standard:
- `stdin`: input da tastiera
- `stdout`: output su schermo
- `stderr`: messaggi di errore su schermo

È possibile deviare l'output di un comando verso un file tramite operatore `>`:
```bash
ls -l > filelist.txt				# Se non esiste è creato, se esiste è completamente sovrascritto
ls -l 2> filelist.txt				# Come sopra, ma per lo stderr
ls -l &> filelist.txt				# Come sopra, ma per entrambi

# I seguenti sono equivalenti a quelli sopra, ma effettuano append invece di overwrite
ls -l >> filelist.txt
ls -l 2>> filelist.txt
ls -l &>> filelist.txt

# Si possono combinare
ls -l > filelist.txt  2> errors.txt
```

Oppure prendere l'input da un file tramite operatore `<`:
```bash
sort < list.txt						# Ordina gli elementi dati gli input nel file
```

I due operatori di ridirezione si possono combinare:
```bash
sort < list.txt > sortedList.txt
```


Per collegare l'output di un comando all'input del successivo si utilizza l'oepratore di _pipe_ `|`:
```bash
ls -l mydir | less

cat *.txt | sort > result-file.txt
```


## 2.3. Comandi privilegiati

Il primo è `su`, che permette di accede al terminale di un'altro utente, previo inserimento della **password dell'utente desiderato**:
```bash
su alice				# terminale di alice

su						# terminale di root
```

Per lanciare un comando come un altro utente si utilizza il comando `sudo` (_super user do_). Richiede la **password dell'utente corrente**:
```bash
sudo utente comando

sudo comando
```

Questo comando può essere utilizzato solo da utenti nel gruppo `sudoers` (li primo utente vi è inserito automaticamente).

# 3. Comandi `vi`

Ecco un lista di comandi per l'editor di testo da terminale `vi`/`vim`:

<div class="flexbox" markdown="1">

|   Comando   | Descrizione                                                              |
| :---------: | :----------------------------------------------------------------------- |
|    `Esc`    | Passa in modalità comandi                                                |
|     `i`     | Passa in modalità inserimento nella posizione corrente                   |
|     `v`     | Passa in modalità visualizzazione. Permette di selezionare più caratteri |
|     `o`     | Inserisce una nuova linea dopo quella corrente                           |
|     `x`     | Cancella il carattere corrente                                           |
|     `u`     | Annulla l'ultimo comando sulla linea corrente                            |
|    `r?`     | Sostituisce con `?` il carattere su cui si trova il cursore              |
|    `dd`     | Cancella la riga corrente                                                |
|    `ndd`    | Cancella `n` righe a partire da quella corrente                          |
|    `yy`     | Copia una riga                                                           |
|    `nyy`    | Copia `n` righe a partire da quella corrente                             |
|     `p`     | Incolla la selezione nella riga sotto il cursore                         |
|   `/word`   | Ricerca nel testo la parola `word`                                       |
|     `n`     | Si posizione sull'occorenza successiva (nella ricerca)                   |
|     `N`     | Si posizione sull'occorenza precedente (nella ricerca)                   |
|    `:q`     | Esce (se non si sono fatte modifiche)                                    |
| `:wq` o `x` | Salva ed esce                                                            |
|    `:q!`    | Esce senza salvare                                                       |
|   `:help`   | Apre l'aiuto in linea                                                    |

</div>

# 4. Strumenti per la gestione dei file

## 4.1. Find

Il primo strumento che andiamo a vedere è il comando `find`.
Questo comando permette di trovare file e cartelle all'interno del sistema.

Utilizza una sistassi relativamente complessa, ma questo gli permette di effettuare la ricerca combinando dei test sulle proprietà dei file, che siano _filename_, _file type_, _owner_, _permessi_, _timestamp_,...

È importante evidenziare che la ricerca **non è influenzata dal contenuto del file**.

Il comando `find` permette di eseguire delle _actions_ (comandi) sui file trovati.

La sintassi è la seguente:
```bash
find [path1...] [espressione]
```

Il `path` permette di specificare uno o più percorsi. La ricerca avverrà **soltanto nei percorsi specificati**. L'espressione descrive come vengono trovati i file e quali azioni devono essere eseguite su di essi.

Le espressioni sono composte da una sequenza di elementi:
- **Test**: valutazione di una proprietà del file. Ritorna `true` o `false`
- **Azioni**: sono delle azioni da effettuare sui file "trovati". Ritornano `true` se hanno successo
- **Opzioni Globali**: influenziano l'esecuzione di test o di azioni, ritornando sempre `true`
- **Opzioni posizionali**: influenziano solo le azioni o i test che seguono, ritornando sempre `true`


Gli elementi di una espressioni sono collegati da **operatori**, ad esempio `-o` indica `OR` e `-a` indica `AND`. In caso non siano specificati operatori, l'utilizzo dell'operatore `AND` è **implicito per collegare due espressioni**. Per negare una espressione il carattere `!` rappresenta il `NOT`.

Vediamo alcuni **_test_**  di utilizzo:
- `find . -name pattern`: ricerca basata sul nome del file. Il pattern può includere i metacaratteri oppure le parentesi e, per evitarne l'espansione, è neccesario scriverli tra apici.
- `find . -type dfl`: ricerca basata sul **tipo di file**. `d` indica le _directory_, `f` i _regular files_ e `l` i _symbolic link_
- `find . -size [+-]n[ckMG]`: permette di effettuare ricerche basate sulla dimensione del file. Il prefisso `[+-]` indica se il file deve essere maggiore o minore della dimensione specificata. `n` indica la dimensione e `[ckMG]` indica l'unità di misura utilizzata. In ordine `byte`, `kilobyte`, `megabyte` e `gigabyte`
- `find . -user utente`: si cercano i file appartenenti ad un `utente` come `UID` o come `username`
- `find . -group gruppo`: si cercano i file appartenenti ad un `gruppo` come `GID` o come `groupname`
- `find . -perm [-/] mode`: si cercano i file a seconda dei permessi del file:
  - `mode`: i permessi devono essere **esattamente** quelli specificati
  - `-mode`: almeno i permessi indicati devono essere presenti
  - `/mode`: almeno uno dei permessi indicati deve essere presente

Prima di vedere le **_azioni_**, è importante sottolineare che questi comandi vanno inseriti **dopo i test**, altrimenti avranno effetto su tutti i file:
- `-delete`: elimina i file trovati. Ritorna `true` in caso di successo
- `-exec command \;`: esegue il comando `command` specificato sui file trovati. Tutti gli argomenti specificati dopo `command` vengono considerati come _argomenti del comando_, fino al carattere `\;`. La stringa `{}` è utilizzata per **specificare il nome del file attualmente processato**. Il comando viene eseguito a partire dal _percorso di partenza_.

Un esempio di `find` che:
- Cerca i file che hanno dimensione di almeno 10MB con permessi di scrittura per il proprietario che appartengono all'utente `pippo`
- Inserisce la lista dei percorsi in un file `list.txt`

```bash
find . -size 10M -perm -u=w -user pippo -exec echo {} >> list.txt \;
```

## 4.2. Locate

```bash
locate [options] file1 file2 ....
```

Permette di ricercare un file specificato sfruttando un database aggiornato periodicamente dal sistema. È possibile forzare l'aggiornamento del database tramite comando `sudo updatedb`.

È più semplice da utilizzare e più veloce rispetto alla `find`. Tuttavia, mentre la `find` fornisce sempre risultati aggiornati, la `locate` potrebbe non avere ancora aggiornato il database, non fornendo alcun _match_.

Inoltre `find`, oltre a fornire supporto per test e azioni, è installato di _default_, mentre `locate` no.

## 4.3. Grep

Il comando `grep` (_General Regular Expression Print_) permette di cercare il uno o più file di testo le linee che **corrispondono ad espressioni regolari** o **stringhe letterali**.

```bash
grep [opzioni] [-e] modello [-e modello2...] file1 [file2...]
```

Se si vuole specificare più di un modello si deve utilizzare `-e` prima di ciascun modello, incluso il primo.

<div class="flexbox" markdown="1">

| Opzione |                    Significato                    |
| :-----: | :-----------------------------------------------: |
|  `-i`   | Ignora le distininzioni tra minuscole e maiuscole |
|  `-v`   | Mostra le linee che non contengono l'espressione  |
|  `-n`   |             Mostra il numero di linea             |
|  `-c`   |   Riporta solo in conteggio delle linee trovate   |
|  `-w`   |             Trova solo parole intere              |
|  `-x`   |                   Linee intere                    |

</div>

È possibile specificare dove la stringa/espressione deve trovarsi all'interno di una riga:
- `^`: deve trovarsi a inizio riga
- `$`: deve trovarsi a fine riga
- `[]`: permettono di definire set di caratteri ammessi
- `.`: indica qualsiasi carattere
- `*`: indica che può essere ripetuta zero o più volte
- `\`: carattere di _escape_

# 5. Archiviazione e Compressione

## 5.1. TAR

Il comando `tar` (_Tape ARchive_) permette di archiviare/estrarre una raccolta di file e cartelle
```bash
tar modalità[opzioni] [file1...]
```
La modalità specifica il modo in cui il comando deve operare, le opzioni permettono di fornire ulteriori dettagli sul comportamento tecnica di compressione (nome dell'archivio, ...)

La lista di file/cartelle indica quali devono essere archiviati o estratti

Il formato del file creato dipende dalla compressione eventualmente utilizzata:
- `.tar`: non è stata utilizzata compressione
- `.tar.gz`: l'archivio è stato compresso con `gz`
- `.tar.bz2`: l'archivio è stato compresso con `bzip2`

Subito dopo il comando `tar` deve essere specificata la modalità:

<div class="flexbox" markdown="1">

| Simbolo Modalità | Significato                                                                                    |
| :--------------: | :--------------------------------------------------------------------------------------------- |
|       `A`        | Aggiunge file `.tar` all'archivio                                                              |
|       `c`        | Crea un nuovo archivio                                                                         |
|       `d`        | Trova le differenze tra l'archivio e il _filesystem_                                           |
|    `--delete`    | Cancella un file all'archivio                                                                  |
|       `r`        | Aggiunge un file all'archivio                                                                  |
|       `t`        | Elenca i file di un archivio                                                                   |
|       `U`        | Aggiunge deu file all'archivio, ma solo se differiscono dalla copia eventualmente già presente |
|       `x`        | Estrae i file dall'archivio                                                                    |

</div>

Le opzioni invece permettono di definire il modo in cui il comando deve operare:

<div class="flexbox" markdown="1">

| Simbolo Modalità | Significato                                   |
| :--------------: | :-------------------------------------------- |
|       `v`        | Modalità _verbose_                            |
|       `z`        | Compressione con `gzip`                       |
|       `j`        | Compressione con `bzip2`                      |
|       `f`        | Permette di specificare il nome dell'archivio |

</div>

Alcuni esempi:
```bash
tar cvf archivio.tar percorso     	# crea un archivio di nome `archivio.tar` con il contenuto `percorso` in modalità verbose

tar czf archivio.tar.gz percorso  	# crea un archivio compresso di nome `archivio.tar.gz` con il contenuto `percorso`

tar tf archivio.tar             	# mostra il contenuto dell'archivio `archivio.tar`

tar xvf archivio.tar file     		# estrae `file` da `archivio.tar` in modalità verbose
```

Se si devono comprimere file o archivi creati precedentemente con tar possiamo utilizzare:
```bash
gzip file1 file2				# comprime e salva i file con lo stesso nome e estensione `.gz` senza eliminare gli originali
gunzip file1.gz file2.gz		# estraei file compressi in file con lo stesso nome (senza estensione della compressione). Gli origniali vengono eliminati
```

`bzip2` e `bunzip2` utilizzano la stessa sintassi ma utilizzano l'algoritmo `bzip2`
