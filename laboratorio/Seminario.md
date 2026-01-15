---
title: Seminario
---

# 1. Indice

- [1. Indice](#1-indice)
- [2. Come si compila Linux](#2-come-si-compila-linux)
	- [2.1. Compilare un singolo modulo](#21-compilare-un-singolo-modulo)
- [3. VirtIO](#3-virtio)
- [4. Contibuire a Linux](#4-contibuire-a-linux)

# 2. Come si compila Linux

Innanzitutto dobbiamo capire di quale Linux stiamo parlando.

Se infatti accedessimo al sito [kernel.org](kernel.org) noteremmo che esistono diverse versioni di linux.

La principale versione di `Linux` è la `Linux mainline`, che rilascia le **_versioni ufficiali_** e le **_release candidates_**, ovvero le versioni candidate al rilascio.

Quello che abbiamo invece scaricato tendenzialmente sono le `linux stable`, che effettua delle backport dei fix all'interno della linea principale.

`Linux` viene compilato attraverso l'utilizzo dei `makefile`. È necessario specificare un file di _configurazione_, questo perché `Linux` supporta tantissime architetture diverse tra di loro, ed è quindi necessario effettuare dei _tweak_ per ogni versione affinché possa funzionare.

QUando eseguiamo il comando `make fileconfig` si aprirà una schermata dove possiamo **_personalizzare il nostro sistema con tutte le configurazioni che vogliamo inserire_**. 
Le varie opzioni possono essere caricati, non caricati oppure _caricati come moduli_, ovvero eseguibili che vengon ocaricati solo quando necessario.

Quando scarichiamo una versione di Linux nella cartella `/boot/` si troverà un file di configurazioni globali che verranno utilizzati quando si compilerà il kernel.

Per installare il kernel esistono diverse opzioni.

Una prima opzione è quella di usare il comando `make binrpm-pkg`, cos' da generare dei **pacchetti `rpm` per il _kernel_**.
Per installare il kernel sulla macchina stessa si utilizza il comando `make install` e `make module_install` (per i moduli).

Se invece volessimo scaricarlo su un altro sistema è possibile copiare i file `.rpm` all'interno della macchina designata e successivamente spacchettarli attraverso `dnf`.
Quando avrà finito con un semplice `reboot` potremo selezionare il kernel appena compilato.

All'interno del file `module.alias` troviamo tutta una serie di _alias_ che permettono di capire quali moduli caricare qualora venisse rilevato un dispositivo.

## 2.1. Compilare un singolo modulo

Il primo metodo per compilare un singolo modulo (file `.ko`) è quello di compilare l'intero kernel e selezionare solo i moduli desiderati.

Questo metodo è dispendioso, quidni si preferisce una **_compilazione out-of-tree_**.
In questa versione si prende una versione **_già compilata del kernel_** e si pacchettizza solamente uno dei moduli accedibili (eventualmente dopo le desiderate modifiche al modulo).

A questo punto è possibile utilizzare e distribuire solamente il singolo modulo.

# 3. VirtIO

È uno standard per la creazione di macchine virtuali.

Le macchine virtuali non hanno infatti necessità stretta di emulare le componenti in modo preciso, ma è possibile per loro utilizzare delle tecniche per ottimizzare.

Per capire se ci troviamo su una macchina virtuale uno dei metodi è proprio quello di visualizzare le periferiche (`lspci`) e vedere se sono periferiche emulate o meno.

Essitono diversi :
- Virio-Block: disci
- Virio-Crypto: dispositivi criptati
- Virio-Sound: dispositivi audio

In particolare capiamo cos'è `Virtio-vsock`.
Questo driver permette di  effettuare **_comunicazioni tra macchine virtuali e macchine fisiche_** attraverso una _virtualizzazione dei socket_.
Permette la comunicazione attraverso le `socket API` utilizzando semplicemente degli ID assegnate alle varie macchine virtuali.

Un esempio di utilizzo di `vsock` sono le _guest-addition_ di _virtualbox_ che permettono di fare copia-incolla tra macchina _guest_ e _host_.

Attraverso il software _netconnect_ è possibile connettere attraverso un _vsock_ le due macchine:
```bash
# Macchina fisica
nc --vsock -l 12345

# Macchina Virtuale
nc --vsock 2 12345
```

# 4. Contibuire a Linux

`Linux` è un progetto open-source alla quale è possibile contribuire in modo libero.

In pratica i maggiori contributori (e in realtà gli unici) sono le **_grandi società_** come `Intel`, `Google`, `RedHat`, `Linaro`, `SUSE`, `AMD`, `NVIDIA`, `Qualcomm`, `Meta`, ...

Tuttavia si può accedere alla repo [github.com/torvalds/linux](www.github.com/torvalds/linux) e utilizzare le _pull request_ anche se **_non è il modo ufficiale_**.

IL modo ufficiale per l'invio delle patch è **_per email_**.
Al sito [lore.kernel.org](lore.kernel.org) è possibile visualizzare una raccolta delle email inviate per effettuare delle modifiche e caricare il codice modificato attraverso il comando `diff`.

Dietro ai motivi della scelta delle email si celano due problemi diversi: il single-point-of-failure che introduce un unico metodo come le _pull request_ di `github`, e poi il fatto stesso che `Github` è **_un azienda privata posseduta dalla microsoft_**.

Attraverso il comando:
```bash
git format-patch -1
```

Si genera un file `.patch` che contiene il commit delle modifiche fatte. Questo comando prende le informaizoni trovate in `~/.gitconfig` per la mail.

Per inviare una mail si utilizza il comando:
```bash
git send-email
```

Chi effettuera il controllo della patch risponde alla mail, tendenzialmente utilizzando client mail da terminale.

Questi utenti si chiamano **_maintainer_**. Il **_main maintainer_** è Torvalds in persona che revisiona tutte le patch che arrivano a lui. Sotto si hanno dei _cluser_ di _maintainer_, divisi per moduli divisi ad albero.

La lista dei maintainers dei vari moduli si trova nel file `MAINTAINERS` all'interno della repo di _Torvalds_.