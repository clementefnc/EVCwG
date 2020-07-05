# Controllo versione semplice con Git

## Licenza

Questo lavoro è rilasciato sotto licenza **Creative Commons Attribution-NonCommercial-ShareAlike 3.0 Unported**. Per vedere una copia di questa licenza visita http://creativecommons.org/licenses/by-nc-sa/3.0/ o invia una lettera a Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.

![licenza](/Users/clementefnc/Documents/Git/img/licenza.png)

Questo lavoro è basato sulla seconda edizione del libro **Pro Git** di *Scott Chacon e Ben Straub* edito da *Apress*. Il libro è disponibile per il download gratuito dal [sito ufficiale](https://git-scm.com/) di Git.



## Prefazione

Ho deciso di lanciarmi nella scrittura di questo manuale perchè volevo qualcosa di rapido ma esaustivo che mi permettesse, e che permettesse a chiunque avesse poi intenzione di leggerlo, di fare una carrellata di informazioni fondamentali su Git in poche righe e poco tempo e soprattutto in italiano.

Spero che chiunque utilizzerà questo testo possa trovarlo utile ed esaustivo.

Per qualsiasi segnalazione, suggerimento, lamentela mandatemi liberamente una mail all'indirizzo [clementefnc@gmail.com](mailto:clementefnc@gmail.com). Su OpenPGP potete trovare [la mia chiave GPG](https://keys.openpgp.org/search?q=clementefnc%40gmail.com).



## Riguardo i sistemi di controllo versione (VCS)

Un sistem di controllo versione è un sistema che si occupa di memorizzare le modifiche effettuate ad un file o un insieme di file nel tempo di modo che si possa analizzare questi stati intermedi in un momento futuro ed eventualmente capire dove le cose possono essere andate storte.

### Tipologie

#### Sistemi di controllo versione locali (LVCS)

Molte persone sfruttano cartelle etichettate con date contententi un progetto  aggiornato a quel giorno; fondamente delle ridondanze infinite organizzate in cartelle.

Il limite è che basta poco per aprire la cartella sbagliata e rovinare tutto.

Per questo sono stati inventati i sistemi di controllo versione.

Un esempio di VCS che lavora sul nostro filesystem è RCS che lavora memorizzando le differenze tra le varie versione dei file in uno specifico formato; quindi è in grado di spostarsi tra i vari stadi di un progetto aggiungendo o togliendo ciò che è stato modificato nel frattempo.

#### Sistemi di controllo versione centralizzati (CVCS)

Quando si lavora in gruppo però, come gestire il fatto che le persone posseggono le loro copie del progetto? Ci vuole qualcuno che poi si preoccupi di mettere tutto insieme ed eviti di far casini.

Per questo sono nati i sistemi di controllo versione centralizzati, come ad esempio Subversion e Perforce, che possiedono un singolo server che contiene tutte le versioni dei file. A questi server si connettono un certo numero di clients che scaricano la versione più aggiornata del progetto adl server e poi ricaricano la versione su cui hanno fatto le modifiche.

La criticità di questo sistema è che se il server va down, finchè non viene ripristinato, nessuno può collaborare; se il server viene corrotto o violato, tutto il lavoro è perso eccetto ciò che le persone hanno sulle loro macchine.

#### Sistemi di controllo versione distribuiti (DVCS)

In un sistema di controllo versione distribuito, come Git, Mercurial, Bazaar o Darcs, il client non solo scarica l'ultimo snapshot del progetto, ma crea una copia completa di ciò che è nella repository, quindi tutta la sua storia. Così nel caso in cui un nodo qualsiasi venisse meno, chiunque potrebbe diventare il nuovo "server" e tutti potrebbero riscaricarsi il progetto e la sua storia.

**Ogni client possiede un clone del server e quindi un backup completo dei dati.**

Oltretutto questi sistemi lavorano senza problemi con più di una repository remota, in questo modo è possibile collaborare con diversi gruppi di persone in diversi modi, simultaneamente, sullo stesso progetto.

### Cos'è Git e come lavora?

Git è un sistema di controllo centralizzato che pensa in modo completamente innovativo ai suoi dati. La maggior parte dei sistemi di controllo versione pensano alle varie versioni di un progetto come a delle differenze rispetto alla versione precedente (*delta-based version control*), Git pensa ai suoi dati come ad una serie di **snapshot** (istantanee) di un piccolo filesystem.

Ogni volta che si effettua un commit o si salva lo stato del progetto, Git effettua una fotografia dello stato attuale dei files. Per essere efficiente, se i file non sono cambiati, non li salva nuovamente, aggiunge soltanto un link alla versione precedente.

**Git pensa ai suoi dati come ad una sequenza di snapshots.**

La maggior parte delle operazioni effettuate da Git sono locali proprio perchè si ha l'intera storia del progetto memorizzata sul filesystem. Questo significa che si può continuare a lavorare anche quando si va offline.

Ogni cosa salvata in Git viene processata da un algoritmo di checksum; ciò implica che è impossibile cambiare il contenuto di un file o cartella senza che se ne accorga.

Questo rende impossibile corrompere il database senza che Git se ne accorga.

Il meccanismo di checksum utilizzato da git è basato sull'algoritmo di hashing SHA-1. L'algoritmo consta nel processare i file e produrre per ciascuno una stringa esadecimale di 40 caratteri che ne rappresenta una sorta di impronta digitale.

Queste stringhe vengono utilizzate da Git per rappresentare il proprio database; ogni cosa è memorizzata non attraverso il suo nome ma attraverso l'hash del suo contenuto.

##### I tre stati

Questa è la cosa più importante da capire quando si lavora con Git di modo da non avere perplessità quando si effettua una qualunque operazione.

Git possiede tre stati principali in cui un file può trovarsi:

- Committed: significa che il dato è stato memorizzato in modo sicuro nel database locale
- Modified: significa che il file ha delle modifiche rispetto alla versione precedente ma non è stato ancora fatto il commit
- Staged: significa che le modifiche della versione attuale rispetto alla precedente sono state segnalate e che questa versione è pronta per essere inserita nel prossimo commit

![Pro_Git-012](/Users/clementefnc/Documents/Git/img/Pro_Git-012.jpg)

Questo ci porta a parlare anche delle tre sezioni principali di un progetto Git: la git directory, il working tree e la staging area.

- Git directory: è dove Git memorizza i metadati e gli oggetti del database per il nostro progetto; è la parte più importante per Git ed è ciò che viene copiato quando si *clona* una repository da un altro computer
- Working tree: è un singolo checkout di una versione del progetto; questi file sono estratti da un database compresso nella git directory e posizionate sul disco per essere utilizzati o modificati
- Staging area: è un file, generalmente contenuto nella git directory, che memorizza informazioni circa cosa verrà inserito nel prossimo commit; in gergo è detto *index*

Generalmente il workflow con Git è il seguente:

- Si modifica un file nel working tree
- Si selezionano solo i file le cui modifiche si vogliono inserire nel prossimo commit; questo aggiunge **solo** questi cambiamenti alla staging area
- Si effettua il commit che prende i file della staging area e ne memorizza uno snapshot nella git directory

Una versione particolare di un file che è nella git directory, è **committed**. Se è stata modificata ed aggiunta alla staging area allora è **staged**. Se sono stati fatte delle modifiche ma non è stato aggiunto alla staging area allora è **modified**.

### La riga di comando

Ci sono vari modi per usare Git. È stato però pensato per essere utilizzato da riga di comando, nonostante esistano ormai svariati tool grafici che ne rendono l'uso più semplice pre i meno avvezzi al terminale.

D'ora in avanti si da per scontato che ci si sappia quantomeno destreggiare con i comandi basi di una shell bash.

### Intallare Git

Prima di iniziare ad utilizzarlo, è necessario installare Git. In alcuni casi è già installato. Nel caso in cui non fosse così fate riferimento al vostro package manager se siete su Linux; se siete su MacOS lo trovate nei developer tool che, se non avete già installato, potete installare semplicemente digitando `git` e premendo invio di modo che vi verrà chiesto di installare i developer tools; per installarlo su windows far riferimento alle istruzioni sul [sito ufficiale](http://git-scm.com/download/win); infine è possibile anche compilarselo autonomamente.

### Primo avvio e setup

Ora che abbiamo installato Git sul nostro sistema è necessario fare un paio di cose per configurare il nostro ambiente. Questa procedura si deve fare soltanto una volta su ogni pc su cui installiamo git.

Git ha un tool chiamato `git config` che permette di visualizzare ed impostare le variabili che controllano ogni aspetto relativo alle modalità in cui git opera. Queste variabili possono essere memorizzate in tre punti diversi ed hanno un significato diverso a seconda di dove sono posizionate.

- `/etc/gitconfig`: contiene i parametri che si applicano ad ogni utente del sistema. Si manipola con l'opzione `--system` passata a `git config`.
- `~/.gitconfig` o `~/.config/git/config`: questo file contiene i valori specifici di un singolo utente. Si manipola con l'opzione `--global` passata a `git config`.
- `config`: questo file si trova nella Git directory ed è specifico della repository corrente

Ogni livello sovrascrive ciò che è nel livello precedente, per cui la configurazione nella Git directory, se presente, è la principale.

### La nostra identità

La prima cosa da fare è configurare il nostro nome ed indirizzo email. Questo è importante perchè ogni commit conterrà questa informazione. Ora diamo i seguenti comandi sostituendo nome e mail con le nostre informazioni personali

```bash
$ git config --global user.name "clementefnc"
$ git config --global user.email clementefnc@gmail.com
```

Fatto la prima volta non sarà più necessario rifarlo, a meno che non si vogliano modificare o ci si trovi su un nuovo computer.

Fatto ciò, volendo si può impostare anche l'editor di default che ci verrà mostrato da git ogni volta che sarà necessario inserire un messaggio. Se non viene configurato, usa l'editor di default del sistema. Nel caso in cui ne volessimo uno differente come ad esempio Vim possiamo dare il seguente comando

```bash
$ git config --global core.editor vim
```

Per controllare lo stato attuale della configurazione basterà utilizzare il comando `git config --list` che produrrà un output simile al seguente

```bash
$ git config --list
user.name=clementefnc
user.email=clementefnc@gmail.com
core.editor=vim
```

Volendo si può anche vedere uno specifico valore

```bash
$ git config user.name
clementefnc
```

### Ottenere aiuto

