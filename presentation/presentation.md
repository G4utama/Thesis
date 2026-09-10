# Presentazione

## Slide 1 — Titolo
Buongiorno a tutti. 

Il lavoro che presento riguarda lo sviluppo di un Middleware basato su Spring per l'arricchimento di dati, realizzato durante il tirocinio presso SyncLab.

Vi racconterò il contesto del progetto, le scelte fatte, e i risultati ottenuti.

## Slide 2 — L'azienda
Il tirocinio si è svolto presso SyncLab, in modalità prevalentemente remota con presenza settimanale in sede. 

L'SyncLab si occupa principalmente di soluzioni informatiche, ricerca e sviluppo e integrazioni di sistemi.

Per l'appunto il lavoro consisteva nello sviluppare un Middleware per l'arricchimento di dati, ovvero un software che si colloca tra due o più sistemi, occupandosi di farli comunicare senza che questi debbano conoscersi direttamente.

## Slide 3 — Perché serve un Middleware?
Perché serve un Middleware? Partiamo dal problema. In un sistema con più applicazioni che devono scambiarsi dati, se ogni sistema comunica direttamente con tutti gli altri, il numero di connessioni cresce in modo quadratico, come si vede nello schema a sx, con quattro sistemi servono già sei collegamenti diretti, e ognuno deve conoscere il formato dati dell'altro. 

Introducendo un Middleware come intermediario, come si vede nello schema a dx, ogni sistema comunica solo con esso: le connessioni diventano lineari, non più dirette, e i sistemi restano disaccoppiati, cioè indipendenti l'uno dall'altro. 

È esattamente questo il problema che il Middleware sviluppato risolve, occupandosi anche di dati incompleti che devono essere arricchiti con informazioni aggiuntive prima di essere ridistribuiti.

## Slide 4 — Content Enricher
Dal punto di vista architetturale, il Middleware implementa un pattern noto come Content Enricher, uno degli Enterprise Integration Patterns: un catalogo di soluzioni standard per problemi ricorrenti nell'integrazione tra sistemi. 

Il Content Enricher riceve in ingresso un messaggio con informazioni parziali, lo arricchisce interrogando una fonte dati esterna (come per esempio un database), e produce in uscita un messaggio completo.

## Slide 5 — Dead Letter Channel
Il secondo pattern adottato è il Dead Letter Channel, anch'esso parte degli Enterprise Integration Patterns.

Quando un messaggio non può essere arricchito correttamente, ad esempio perché il dato collegato non esiste, questo viene instradato su un canale separato, il Dead Letter Channel, dedicato esclusivamente alla gestione degli errori.

Questo consente al flusso principale di proseguire senza interruzioni, mantenendo al contempo traccia dei messaggi problematici per un'analisi o un riprocessamento successivo.

## Slide 6 — Stack tecnologico
Concretamente, questi pattern sono stati implementati con Java come linguaggio e Maven per la gestione del progetto e delle sue dipendenze. 

Come framework applicativo ho usato Spring (basato appunto su Java), in particolare tre suoi moduli: Spring Boot (che semplifica la configurazione e l'avvio dell'applicazione), Spring Data JPA (per l'accesso al database e la gestione dei dati) e Spring Kafka (per la lettura e scrittura dei messaggi). 

Infine, come insfrastrutture di supporto, Docker per containerizzare (cioè eseguire in ambienti isolati e riproducibili), PostgreSQL (come database per contenere i dati per arricchire) e Apache Kafka (per la gestione dei topic).

## Slide 7 — Il progetto
Questo schema riassume l'intera architettura: 
- l'applicazione Spring, in alto, legge, tramite il Consumer, dal topic Kafka orders-input (un topic è semplicemente un canale su cui vengono pubblicati e letti i messaggi); 
- poi tramite l'Enrichment, appunto arricchisce il dato interrogando PostgreSQL per ottenere i dati aggiuntivi;
- infine, tramite il Producer pubblica il risultato su un altro topic Kafka orders-output, oppure, in caso di errore come visto dal pattern Dead Letter Channel, sul topic orders-input-dlq. 

Tutta l'infrastruttura in basso, è containerizzata tramite Docker Compose.

## Slide 8 — Implementazione: un esempio
Per darvi un'idea concreta del codice, questa è l'entità che rappresenta la classe cliente.

Si può notare l'uso delle annotazioni (un modo, tipico di Java, di aggiungere informazioni al codice senza scriverne di aggiuntivo): 
- quelle di Jakarta Persistence (Entity, Table, Id, Column), sono uno standard Java per definire come una classe si collega a una tabella del database; 
- quelle di Lombok (i metodi Getter, Setter e i costruttori), generano automaticamente in fase di compilazione codice ripetitivo che altrimenti andrebbe scritto a mano, mantenendo la classe concisa.

## Slide 9 — Ambiente di sviluppo
L'ambiente di sviluppo è organizzato attorno a tre topic Kafka.

Ogni topic ha un numero di partizioni diverso a seconda del ruolo, in particolare i topic del flusso principale ne hanno di più per consentire il parallelismo in lettura/scrittura, la singola partizione per DLQ indica un volume atteso più basso.

Il tutto è orchestrato integralmente con Docker, insieme a PostgreSQL e a un'interfaccia web (grazie alla dipendenza Kafka UI) per ispezionare i messaggi, altrimenti visibili tramite riga di comando.

## Slide 10 — Vantaggi ottenuti
I vantaggi principali ottenuti sono:
- Disaccoppiamento: ovvero i sistemi coinvolti restano disaccoppiati
- Nessuna perdita dati: nessun dato viene perso grazie alla persistenza di Kafka
- Errori isolati: gli errori vengono isolati automaticamente tramite il DLQ
- Ambiente riproducibile: ambiente è riproducibile su qualsiasi macchina grazie all'uso di Docker

## Slide 11 — Sviluppi futuri
Gli sviluppi futuri principali sono due: 
- l'implementazione dello stesso progetto tramite Apache Flink, un motore di stream processing che, a differenza di Spring (che elabora un messaggio alla volta non appena arriva) è pensato per elaborare flussi continui di dati in modo nativo. Per limiti di tempo, questo confronto è stato condotto solo a livello teorico e progettuale, senza un'implementazione concreta.
- l'esecuzione dei test di performance attraverso la metodologia definita nella prossima slide

## Slide 12 - Valutazione prestazioni
Ho definito una metodologia per valutare le prestazioni del sistema, seguendo le seguenti metriche:
- throughput, cioè quanti messaggi vengono elaborati al secondo
- latenza end-to-end, ovvero il tempo tra la pubblicazione di un evento grezzo sul topic sorgente (orders-input) e la comparsa del corrispondente evento arricchito sul topic di destinazione (orders-output)
- consumer lag, che indica quanto il sistema è indietro rispetto ai messaggi in arrivo 
- costo interrogazione, quindi il tempo di risposta della fonte dati esterna

Anche in questo caso l'esecuzione concreta dei test non è stata fatta, quindi l'implementazione di Apache Flink e l'esecuzione dei test di performance restano il prossimo passo da eseguire.

## Slide 14 — Fine
Vi ringrazio per l'attenzione, la mia discussione finisce qui.