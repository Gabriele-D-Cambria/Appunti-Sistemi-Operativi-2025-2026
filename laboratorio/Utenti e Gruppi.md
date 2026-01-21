---
title: Utenti e Gruppi
---

# 1. Indice

- [1. Indice](#1-indice)
- [2. Utenti e Gruppi](#2-utenti-e-gruppi)
- [3. Permessi di accesso al filesystem](#3-permessi-di-accesso-al-filesystem)
	- [3.1. Privilegi di un processo](#31-privilegi-di-un-processo)
	- [3.2. Cambiare proprietari](#32-cambiare-proprietari)
- [4. File di configurazione utenti](#4-file-di-configurazione-utenti)
- [5. Comandi per la gestione dei gruppi](#5-comandi-per-la-gestione-dei-gruppi)


# 2. Utenti e Gruppi

All'interno dei sistemi `Unix` ogni utente è definito da `username` e da `UID` (_User ID_). Un gruppo, analogamente, è definito da `groupname` e da `GID` (_Group ID_).

Ogni utente **_deve appartenere almeno ad un gruppo_**, detto _primary group_.

Esistono diversi comandi adibiti alla gestione degli utenti:
```bash
passwd 				# permette di cambiare la password sfruttando il permesso SUID

id [username]		# visualizza l'UID, il primary group e gli altri gruppi dell'utente corrente o di quello selezionato

groups [username]	# visualizza i nomi dei gruppi dell'utente corrente o di quello selezionato
```

Il comando `passwd` permette di cambiare file di sistema pur non avendo i privilegi di `su`.

È inoltre possibile aggiungere o rimuovere utenti dalla _shell_, avendo però i **privilegi di `root`**.

Per la gestione di un utente:
```bash
adduser username	# crea un nuovo utente

deluser username 	# elimina un utente già esistente
```

Per la gestione di un gruppo, **da root**
```bash
addgroup groupName

addgroup groupName
```

# 3. Permessi di accesso al filesystem

I file all'interno del filesistem sono presenti numerosi file sensibili, protetti dall'accesso degli utenti casuali. Il meccanismo dei permessi ne gestisce l'accesso.

Per ogni file/directory sono definiti:
- **Owner**: utente proprietario
- **Group Owner**: gruppo proprietario

Di conseguenza, per ogni file vi sono tre classi di utenti:
- **Owner**: il proprietario
- **Group Owners**: gli utenti appartenenti al gruppo proprietario
- **Others**: gli altri utenti

Dato un _file_, a ciascuna classe di utenti vengono applicati permessi di accesso specifici:
- `r`: _Read_ (lettura), permette di leggere il contenuto del file
- `w`: _Write_ (scrittura), permette di scrivere il contenuto del file. **Non permette di cancellare un file**
- `x`: _eXecute_ (esecuzione), permette di eseguire un file

Quando un utente prova ad utilizzare un file, vengono applicati i permessi specifici della categoria alla quale egli appartiene.

Stessa cosa accade per le directory:
- `r`: permette di leggere il contenuto di una cartella (elenco dei file). Se negato non è possibile utilizzare `ls`
- `w`: permette di modificare il contenuto di una cartella (aggiunta, rimozione e rinomina file).
- `x`: permette di attraversare una cartella. Se negato non è possibile utilizzare `cd` sulla directory

Per visualizzare i permessi di un file o di una directory, si utilizza il comando `ls -l`. Per ogni voce il formato sarà il seguente:

```
┌─── Tipo (d=directory, -=file, l=link)
│
│ ┌────────── Permessi Owner
│ │  ┌─────── Permessi Group Owner
│ │  │  ┌──── Permessi Others
│ │  │  │
│ │  │  │                 Dimensione (byte) ──┐           ┌── Data/ora ultima modifica
│┌┴┐┌┴┐┌┴┐                                   ┌┴┐  ┌───────┴──────┐
drwxr-xr-x  2  owner_name  group_owner_name  512  2008-11-04 16:58  nome
            │
            └── Numero di hard link
```

Le triple dei permessi sono codificate _cifre in base 8_, ottenute sommando:
- 1 se è permessa l'esecuzione
- 2 se è permessa la scrittura
- 4 se è permessa l'esecuzione

Ad esempio, se volessimo garantire tutti i permessi a tutti gli utenti utilizzeremo `777`, se invece volessimo dare tutti i diritti all'_owner_, solo esecuzione e scrittura al _group owner_ e niente a _other_ utilizzeremo `750`.

Per modificare i permessi relativi ad uno o più file si utilizza, da `root` o da _owner_, il comando `chmod`:
```bash
# chmod [who]|[how]|[which] fileName

chmod +x file.txt		  # aggiunge i permessi di esecuzione a TUTTI gli utenti per file.txt
chmod u-x file.txt		# rimuove i permessi di esecuzione all'OWNER per file.txt
chmod g-x file.txt		# rimuove i permessi di esecuzione al GROUP OWNER per file.txt
chmod o=x file.txt		# assegna SOLO PER OTHERS esclusivamente permessi di esecuzione

chmod -R XXX directory/   # aggiunge i permessi di esecuzione in maniera ricorsiva alla cartella directory/

chmod --reference=file1.txt file2.txt   # copia i permessi di file1.txt su file2.txt

chmod go-rwx file.txt	  # toglie tutti i permessi di accesso a `file` a GROUP OWNER e OTHERS
```

Oltre a questi permessi, ne esistono altri due "speciali" che vengono acquisiti durante l'esecuzione:
- `SUID`: il processo acquisisce i privilegi dell'**owner**. Tipicamente un processo acquisisce i privilegi di chi lo esegue.
- `SGID`: il processo acquisisce i privilegi del **group owner**. Tipicamente un processo acquisisce i privilegi del gruppo di chi lo esegue.

Per rappresentare questi permessi aggiuntivi si utilizzano le seguenti sintassi:
- `SUID`: invece del **permesso di esecuzione dell'OWNER** `x` si utilizza il permesso di **esecuzione con `SUID`** `s`
  - `ls -l /usr/bin/passwd` $\to$ `-rwsr-xr-x`
- `GUID`: analogo al `SUID`, ma si utilizza il campo relativo ai permessi in esecuzione del **group owner**

Per quanto riguarda l'ottale si aggiunge un ulteriore cifra antecedente a quelle note, sommando `4` se è attivo `SUID` e `2` se è attivo `GUID`. Ad esempio, il codice `6754` restituisce i permessi `rwsr-sr--`.

Nel caso siano assegnati i permessi speciali di esecuzione ma non quelli normali, si utilizza il carattere `S`.

## 3.1. Privilegi di un processo

I privilegi di un processo dipendono da due parametri:
- **Effective User ID** `EUID`
- **Effective Group ID** `EGID`

Quando un processo viene eseguito, normalmente `<EUID, GUID>` corrispondono rispettivamente all'`UID` del gruppo principale e all'utente che ha eseguito il processo stesso. Per permettergli di eseguire con privilegi diversi, è possibile impostare i permessi `SUID` e `GUID`.

## 3.2. Cambiare proprietari

Per cambiare il proprietario ad un file, **è necessario essere** `root`, e si fa così:
```bash
chown username file
```

Per cambiare invece il _group owner_:
```bash
chgrp groupname file		# da root o se si appartiene a groupname
```

# 4. File di configurazione utenti

Le informazioni degli utenti sono contenute in due file:
- `/etc/passwd`: contiene le informazioni **pubbliche**, leggibili da tutti gli utenti. È possibile editarlo con il comando `vipw` da `root`
- `/etc/shadow`: contiene le informazioni **private**, come la password. È accessibile e modificabile solo da root (`-rw-------`)

Analizzando questi file possiamo notare l'esistenza di numerosi **utenti di servizio**. Questi utenti sono generati dalle singole applicazioni, e vengono utilizzati per permettere loro di agire come tali. In particolare però, possiamo notare come queste non abbiano però l'utilizzo di `/bin/bash`, non permettendo loro di aprire una shell.

Il formato utilizzato per un utente è il seguente: 
```passwd
  username:password:UID:GID:info,aggiuntive,separate:homeDir:shellDir
```

La variabile di ambiente `$HOME` contiene il percorso assoluto della cartella `home` dell'utente. (tipicamente `/home/<nome_utente>`).

Per quanto riguarda la `shell` può essere impostata a `/sbin/nologin` o `/bin/false` per indicare che non è possibile fare login con tali utenti.

La password, se presente e criptata, si trova in `/etc/shadow`, nel seguente formato:
```shadow
	username:$hasingAlg$salt$hash(salt+passwd):ultimaModifica:etàMin:etàMax:campiAvviso
```

I singoli campi:
- `ultimaModifica`: è espressa in giorni dal `1970`
- `etàMin`: indica la durata minima della password
- `etàMax`: indica la durata massima della password
- `campiAvviso`: in particolare possiamo avere:
  - `periodoAvviso`: giorni prima della scadenza in cui l'utente viene avvisato
  - `periodoInattività`: giorni dopo la scadenza della password in cui questa è ancora accettata
  - `scadenza`: data scadenza account
  - `campoRiservato`: riservato per utilizzo futuro

Quando un utente inserisce `username` e `password` per fare un login il sistema operativo:
1. Cerca in `/etc/shadow` una riga che inizia con `username`
   1. Se non lo trova dà errore
2. Trovato il _record_ recupera l'algoritmo di `hash` e il `salt`
3. Procede a cifrare la password inserita con il `salt` attraverso l'algoritmo specificato
4. Confronta il risultato con il contenuto del record.
   1. Se combaciano permette il _login_
   2. Se non combaciano restituisce errore


# 5. Comandi per la gestione dei gruppi

Abbiamo già visto i comandi `addgroup` e `delgroup`, vediamo adesso i comandi `gpasswd` e `newgrp`.

Il comando `gpasswd` **_non si limita solo a cambiare la password di un gruppo_**:
```bash
gpasswd -a utente gruppo			# aggiunge utente al gruppo

gpasswd -d utente gruppo			# rimuove utente al gruppo

gpasswd -M utente1,utente2 gruppo	# Definisce i membri di un gruppo

gpasswd -A utente1,utente2 gruppo	# Definisce gli amministratori di un gruppo
```

Solo gli amministratori di un gruppo e **root** possono manipolare gli utenti di un gruppo. Solo **root** può cambiare gli amministratori di un gruppo.

```bash
gpasswd gruppo		# permette di impostare/cambiare la password di un gruppo

gpasswd -r gruppo	# rimuove la password di un gruppo
```

Se la password non è impostata, **solo i membri del gruppo possono averne i privilegi**.
Se invece è impostata, è per sua natura intrinseamnete poco sicura, poiché sono conosciute da più utenti. Se viene utilizzata **_da un altro utente_**, gli permette di **_acquisire temporaneamente i privilegi del gruppo_**. Per fare ciò si utilizza il comando `newgrp`.

La password non serve agli utenti del gruppo.

Il comando `newgrp` permette di impostare il nuovo gruppo primario dell'utente per la sessione di login corrente:
```bash
newgrp gruppo
```


Le informazioni pubbliche sui gruppi si trovano in `/etc/group`. In questo file, apribile con il comando `vigr` si trovano _record_ con il seguente formato: 
```group
	groupName:password:GID:lista,utenti,del,gruppo
```

Anche in questo caso la password, se presente, è **cifrata** e indicata con una `x`. La password si trova, insieme agli admin, in  `/etc/gshadow`. Questo file è apribile con `vigr -s` e contiene _record_: 
```gshadow
	groupName:pwd_cifrata:GID:admin1,admin2,admin3:membro1,membro2,membro3
```