# Presentazione

## Slide 1 — Titolo
Buongiorno a tutti. 

Il lavoro che presento riguarda lo sviluppo di un Middleware basato su Spring per l'integrazione di flussi di dati, realizzato durante un tirocinio. 

Vi racconterò il contesto del progetto, le scelte fatte, e i risultati ottenuti.

## Slide 2 — L'azienda
Il tirocinio si è svolto presso SyncLab S.r.l., in modalità prevalentemente remota con presenza settimanale in sede. 

L'azienda si occupa principalmente di soluzioni informatiche, ricerca e sviluppo e integrazioni di sistemi.

Il lavoro consiteva nello sviluppare un Middleware, ovvero un software che si colloca tra due o più sistemi, occupandosi di farli comunicare senza che questi debbano conoscersi direttamente, per l'arrichimento di dati.

## Slide 3 — Perché serve un Middleware?
Partiamo dal problema. In un sistema con più applicazioni che devono scambiarsi dati, se ogni sistema comunica direttamente con tutti gli altri, il numero di connessioni cresce in modo quadratico, come si vede nello schema a sinistra, con quattro sistemi servono già sei collegamenti diretti, e ognuno deve conoscere il formato dati dell'altro. 

Introducendo un Middleware come intermediario, ogni sistema comunica solo con esso: le connessioni diventano lineari, non più dirette, e i sistemi restano disaccoppiati, cioè indipendenti l'uno dall'altro. 

È esattamente questo il problema che il Middleware sviluppato risolve, occupandosi anche di dati incompleti che devono essere arricchiti con informazioni aggiuntive prima di essere ridistribuiti.

## Slide 4 — Content Enricher
Dal punto di vista architetturale, il Middleware implementa un pattern noto come Content Enricher, uno degli Enterprise Integration Patterns: un catalogo di soluzioni standard per problemi ricorrenti nell'integrazione tra sistemi. 

Il Content Enricher riceve un messaggio con informazioni parziali, lo arricchisce interrogando una fonte dati esterna (come per esempio un database), e produce in uscita un messaggio completo.

## Slide 5 — Dead Letter Channel
Il secondo pattern adottato è il Dead Letter Channel, anch'esso un Enterprise Integration Patterns.

Quando un messaggio non può essere arricchito correttamente, ad esempio perché il dato collegato non esiste, invece di bloccare l'intero flusso o perderlo silenziosamente, questo viene instradato su un canale separato, dedicato esclusivamente alla gestione degli errori.

## Slide 6 — Stack tecnologico
Concretamente, questi pattern sono stati implementati con Java come linguaggio e Maven per la gestione del progetto e delle sue dipendenze. 

Come framework applicativo ho usato Spring (basato appunto su Java), in particolare tre suoi moduli: Spring Boot (che semplifica la configurazione e l'avvio dell'applicazione), Spring Data JPA (per l'accesso al database senza scrivere query manualmente) e Spring Kafka (per la lettura e scrittura dei messaggi). 

Infine, Docker per containerizzare (cioè eseguire in ambienti isolati e riproducibili), PostgreSQL (come database per contenere i dati per arricchire) e Apache Kafka (per la gestione dei topic).

## Slide 7 — Il progetto
Questo schema riassume l'intera architettura: 
- l'applicazione Spring, in alto, legge, tramite il Consumer, dal topic Kafka orders-input (un topic è semplicemente un canale su cui vengono pubblicati e letti i messaggi); 
- poi tramite l'Enrichment, appunto arricchisce il dato interrogando PostgreSQL;
- infine, tramite il Producer pubblica il risultato su un altro topic Kafka orders-output, oppure, in caso di errore, sul topic orders-input-dlq. 

Tutta l'infrastruttura, in basso, gira containerizzata tramite Docker Compose.

## Slide 8 — Implementazione: un esempio
Per darvi un'idea concreta del codice, questa è l'entità che rappresenta un cliente.

Si può notare l'uso delle annotazioni (un modo, tipico di Java, di aggiungere informazioni al codice senza scriverne di aggiuntivo): 
- quelle di Jakarta Persistence (Entity, Table, Id, Column), sono uno standard Java per definire come una classe si collega a una tabella del database; 
- quelle di Lombok (Getter, Setter e i costruttori), generano automaticamente in fase di compilazione codice ripetitivo che altrimenti andrebbe scritto a mano, mantenendo la classe concisa.

## Slide 9 — Ambiente di sviluppo
L'ambiente di sviluppo è organizzato attorno a tre topic Kafka in modalità KRaft, che permette ai broker (responsabile della gestione di un sottoinsieme di partizioni) di garantisce tolleranza ai guasti.

Ogni topic ha un numero di partizioni diverso a seconda del ruolo, in particolare i topic del flusso principale ne hanno di più per consentire il parallelismo in lettura/scrittura, la singola partizione per DLQ indica un volume atteso più basso.

Il tutto è orchestrato integralmente con Docker, insieme a PostgreSQL e a un'interfaccia web (grazie alla dipendenza Kafka UI) per ispezionare i messaggi, altrimenti visibili tramite riga di comando.

## Slide 10 — Comparazione: Apache Flink
Uno degli obiettivi desiderabili era il confronto con Apache Flink, un motore di stream processing che, a differenza di Spring (che elabora un messaggio alla volta non appena arriva) è pensato per elaborare flussi continui di dati in modo nativo. 

Per limiti di tempo, questo confronto è stato condotto solo a livello teorico e progettuale, senza un'implementazione concreta.

## Slide 11 — Valutazione prestazioni
Un obiettivo facoltativo ma anch'esso preso in considerazione sono i test di performance.

Allo stesso modo, ho definito una metodologia per valutare le prestazioni del sistema, seguendo le seguenti metriche:
- throughput, cioè quanti messaggi vengono elaborati al secondo
- latenza end-to-end, ovvero il tempo tra la pubblicazione di un evento grezzo sul topic sorgente (orders-input) e la comparsa del corrispondente evento arricchito sul topic di destinazione (orders-output)
- consumer lag, che indica quanto il sistema è indietro rispetto ai messaggi in arrivo 
- costo interrogazione, quindi il tempo di risposta della fonte dati esterna

Anche in questo caso l'esecuzione concreta dei test non è stata fatta

## Slide 12 — Vantaggi ottenuti
Nonostante questi limiti, il sistema realizzato raggiunge gli obiettivi principali.

I vantaggi principali sono:
- i sistemi coinvolti restano disaccoppiati
- nessun dato viene perso grazie alla persistenza di Kafka
- gli errori vengono isolati automaticamente
- ambiente è riproducibile su qualsiasi macchina

## Slide 13 — Limiti e sviluppi futuri
I limiti principali restano quindi principalmente due: 
- l'implementazione Flink
- l'esecuzione dei test di performance

Entrambi già progettati a livello teorico, ma non eseguiti. 

Sono il naturale passo successivo per un eventuale sviluppo futuro.

## Slide 14 — Fine
Vi ringrazio per l'attenzione, sono a disposizione per eventuali domande.