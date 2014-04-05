> slide> titolo

buongiorno sono Federico e vi presento una tesi dal titolo mole.io, un sistema per la gestione centralizzata dei log applicativi.
i miei relatore e correlatore sono rispettivamente il prof. ernesto damiani e Emanuele del bono.

perché un tool per la centralizzazione dei log? per capirlo introduciamo immediatamente codiceplastico:

> slide> codiceplastico

codiceplastico è una azienda di bs che opera nel settore it e sviluppa applicazioni web, desktop e mobile per il mercato nazionale.
i clienti di codice plastico, e di conseguenza le applicazioni sviluppate, sono dislocate sul territorio nazionale, su server di diversi clienti o su dispositivi mobili o ancora all'interno di stabilimenti produttivi su pc industriali che monitorano lo stato di macchine di produzione.

gli sviluppatori del team di cp sono costrtti a collegarsi periodicamente alle macchine dei clienti per controllare lo stato delle applicazioni, quindi errori, situazioni anomale e anche statistiche di lavoro.

da qui l'esigenza di avere un tool in grado di ribaltare il paradigma della gestione dei log da pull a push. 

L'idea è semplice: metto in grado le applicazioni di inviare i propri messaggi verso un server centrale che le raccolgie, cataloga e organizza in modo da fornire agli sviluppatori un unico punto dove tenere sotto controllo lo stato di tutte le applicazioni in produzione. A questo punto, ovviamente diventa fondamentale che ogni applicaizone monitorata abbia accesso ad internet.

> slide> lista campi interessanti per i log

Quali sono le informazioni interessanti per gli sviluppatori?

- timestamp, per avere un riferimento del "quando"
- dati relativi all'os
- informazioni sull'utente che sta utilizzando l'applicazione
- dati relativi allo stato del sistema
- dati relativi allo stato dell'applicazione
- messaggio in linguaggio naturale 

prima di iniziare il progetto ci siamo guardati attorno per vedere cosa esisteva già. Per capire se avesse senso usare qualcosa di esistente o sviluppare qualcosa di nuovo. Abbiamo trovato varie applicazioni

> slide> carrellata applicazioni

queste sono le pù significative tra quelle esaminate

- Airbrake
- Log.io
- Rollbar
- Papertrail
- fluentd, elasticsearch kibana

ognuna di queste app presenta ovviamente pregi e difetti, abbiamo cercato di riassumerli in una tabella comparativa

> slide> tab comparativa

*nella quale sono riportare alcune delle funzioalità interessanti ai fini del nostro progetto e...*

dopo la comparativa ci siamo resi conto che le app presenti sul mercato erano ben fatte ma ad ognuna mancava qualcosa, inoltre l'azienda preferiva una soluzione sviluppata internamente e controllabile, da fornire anche come servizio di monitoring ai propri clienti

abbiamo quindi deciso di sviluppare una nuova applicazione: mole.io

> slide> logo

introduco mole.io con una similitudine che accompagna tutto il progetto ed ha aiutato a sviluppare i diversi componenti di mole.io.

> slide> logo mole in alto e schema insertion presentation

mole in inglese significa talpa, spia.
il compito di una talpa è infiltrarsi all'interno di organizzazioni avversarie e raccolgiere informazioni rispetto a notizie riservate e riportarle al proprio ufficio.

secondo questo paragone, abbiamo definito le organizzazioni avversarie come "source", sorgenti di informazioni che abbiamo chiamato whisper "sussurri".

chi fornisce a mole questi whisper? il suo contatto all'interno della source un mole-contact appunto. I mole contact si occupano di inviare gli whisper a mole, l'idea per il progetto finale è quella di avere un mole-contact per ogni linguaggio di programmazione, in modo da poter inserire un mole-contact in ogni applicazione sviluppata

i mole contact per altro sono molto semplici, si dovrebbero occupare esclusivamente di inviare messaggi in formato json verso un a interfaccia rest fornita da mole

una volta ricevuti i dati mole li salva all'interno di un database.

ogni spia che si rispetti è ben vestita, con giacca e cappello, come si vede dal logo. Anche noi abbiamo deciso di vestire in modo decorso la nostra applicazione costruendo mole-suit, l'interfaccia grafica per mole.io.

mole-suit si occupa di estrarre dati dal db e di presentarli all'utente attraverso una interfaccia web.

guardando il sistema dall'alto vediamo quindi un layer di insertion che si occupa di popolare una base dati e uyn layer di presentation che si occupa di estrarre i dati dal db

questi 2 layer sono ben disinti, per modellare questa perte del sistema , infatti ci siamo ispirati al modello CQRS ovver command query responsibility segregation che propone di disegnare un sistema nel quale le responsabilità di scrittura e lettura dei dati siano ben distinte e appartengano a componenti differenti.

Questo fornisce un vantaggio in quanto è possibile ottimizzare le procedure di lettura e scrittura in modo indipendente.

Prima di descrivere in dettaglio mole e mole-suit, vediamo quali tecnologie sono state utilizzate per la realizzazione del progetto.

la tecnologia che ha fatto da denominatore dcomune a tutto il progetto è node.js, infatti i server realizzati per Mole.io sono stati scritti utilizzando node.js. Node.js è un framework javascript per l sviluppo di server web applicativi che permette di realizzare applicazioni altamente performanti e scalabili che supportano un elevato numero di client, utilizzando un insieme limitato di risorse di sistema.

come fa?

> slide> node.js event loop

l'idea alla base di node è mantenere un thread nel quale gira un processo che accetta le richieste dai client e un pool di thread che fungono da worker. 

all'arrivo di una nuova richiesta, il main loop, la passa ad uno dei worker nel pool, e torna disponibile per la gestione di una nuova richiesta. in modo asincrono, quando una richiesta è gestita da uno dei worker, questo avverte il main loop,  il quale risponde al client. Il concetto che caratterizza node è l'asincronicità, quindi il client esegue una richiesta e riceverà la risposta dopo un periodo di tempo

*mole.io è più di una idea o di un progetto, è una applicazione vera, l'abbiamo realizzata utilizzando... E QUI NODE* 

> slide> mole

mole è un server node.js che si occupa di ricevere gli whisper e inserirli nel DB. Gli whisper (qui rappresentati con delle buste) partono dai mole contact e arrivano a mole.
mole esegue immediatamente un salvataggio del messaggio su db, dopo il salvataggio invia lo stesso messaggio su una coda rabbitmq configurata in modalità publish subscribe, e una serie di denormalizatori (i subscriber) ottengono il messaggio dalla coda, lo denormalizzano e lo salvano all'interno del DB

cosa è un denormalizzatore? è un ulteriore server node minimale che crea duplicazione nei dati, a quale scopo? allo scopo di aumentare la fruibilità del dato, cioé permettere a chi legge di avere un dato già pronto nella forma in cui questo necessita di trovarlo.

> slide> mole-suit

la freccia tratteggiata rappresenta le comunicazioni bidirezionali di molesuit, esse sono minime rispetto alla grande maggioranza delle operazioni, che sono esclusivamente di lettura (CQRS non implementato alla lettera, ma cmq efficace)

proporzione 1:1:1 tra denormalizzatori, plugin e widget

In mole-suit ogni plugin (rappresentati con la spina) si occupa di estrarre uno specifico tipo di dato, che gli è stato in precedenza denormalizzato da mole

la ui è composta da una serie di widget che contattano il proprio plugin, recuperano il dato denormalizzato di competenza e lo mostrano a video utilizzando una apposito componente grafico.

Ogni widget, all'occorenza, può contattare servizi esterni per integrare il dato da pubblicare. Ad esempio un plugin che visualizza punti su una mappa partendo dalle coordinate, può integrare i marker sulla mappa utilizzando un servizio di reverse geocoding

> slide> benchmark

mole.io è un sistema di raccolta dati, abbiamo quindi deciso di concentrare i test sulle operazioni di inserimento dati in mongodb e abbiamo cercato di capire come ottimizzare questa parte del sistema.

per i test abbiamo utilizzato



> slide> conclusioni

- supporto efficace: ribaltando il paradigma riusciamo a dare un supporto più efficace ai clienti e aumentiamo il valore dei prodotti percepito dai clienti stessi
- crescita personale: con questo progetto ho avuto modo di utilizzare tecnologie web su un proetto vero, prima le avevo usate solo per fare alcuni test o piccoli progetti personali e mai tutte insieme, ma in modo indipendente
- 























































---

#intro sui log

Le applicazioni, durante il loro flusso di lavoro, incontrano situazioni di errore. Gli sviluppatori (me compreso) sono abituati a tenere traccia di queste situazioni all'interno di file, chiamati log.

i dati che normalmente vengono tracciati sono 

- timestamp, per avere un riferimento del "quando"
- dati relativi all'os
- informazioni sull'utente che sta utilizzando l'applicazione
- dati relativi allo stato del sistema
- dati relativi allo stato dell'applicazione
- messaggio in linguaggio naturale 

la gestione dei log presenta un problema: normalmente si salvano in locale, all'interno del dispositivo server o pc nel quale gira l'applicazione.

questo, a lungo andare genera un problema di reperimento delle informazioni

per tenere monitorate diverse installazioni, infatti, gli sviluppatori sono costretti a contollare periodicamente lo stato dei server e i file di log.

oppure, ancora peggio, si trovano a dover correre ai ripari, dopo che il cliente ha rilevato un bug e lo ha segnalato.

*in presenza di diverse installazioni, infatti, gli sviluppatori sono costretti*

*i messaggi di log prodotti dall'app in locale, nella macchina sulla quale l'app lavora.*

*il che potrebbe essere accettabile nel caso di app desktop o siti web, non lo è se si tratta di app mobile*

#codice plastico

codiceplastico è una azienda di brescia che opera nel settore IT e sviluppa divere tipologie di applicazioni web app, app mobile, e applicazioni desktop.

Il team di CP si è trovato esattamente nella situazione illustrata poco fa, molte installazioni e difficoltà di ottenere una visione globale dello stato delle applicazioni e dei progetti.

Da qui è nata l'esigenza di un servizio in grado di raccogliere in un unico punto diversi messaggi e fornire una visualizzazione chiara e immediata dello stato di ogni applicazione. Con questo approccio si ottiene un vantaggio non indifferente, si ribalta il paradigma di accesso alle informazioni di log da da pull a push, quindi gli sviluppatori non devono più collegarsi ai server dei diversi clienti per controllare lo stato delle applicazioni, ma le applicazioni hanno la capacità di inviare il proprio stato ad un server che centralizza e organizza le informazioni raccolte. 

l'unico requisito richiesto alle diverse applicazioni da monitorare è, ovviamente, l'accesso a internet.



#app viste + comparativa

abbiamo preso spunto da alcune app esistenti e abbiamo cercato di creare un servizio con le funzionalità che ci piacevano delle app valutate e che fornisse funzionalità comode per CP.

qui potete vedere alcune delle applicazioni prese in esame

(screenshot in rapida successione)

- Airbrake
- Log.io
- Rollbar
- Papertrail
- fluentd, elasticsearch kibana

alla fine della ricerca abbiamo stilato una tabella contanante un insieme di feature interessanti e abbiamo valutato quali fossero presenti in ognuna delle app valutate

(tab comparativa)

una volta valutato lo stato dell'arte in fatto di applicazioni per la raccolta di log, abbiamo scelto alcune funzionalità interessanti per il nostro ambito allicativo e abbiamo deciso che, la nostra applicazione, non avrebbe dovuto esclusivamente raccogliere log, ma messaggi generici, in modo da poter essere molto più flessibile di quelle disponibili sul mercato.

Questo approccio permette di utilizzare il sistema anche come tool di BI, per la raccolta non solo di ti relativi a situazioni di errore, ma anche di informazioni sullo stato del sistema, ad esempio, vendite, visite accessi.

da qui l'idea di...

#mole.io

(1 slide solo logo)

l'applicazione 

struttura di mole.io

- mole
- mole-contact
- mole-suit

(questi vanno introdotti man mano con la storiella dei nomi)

#architettura

architettura dell'intero sistema con schema CQRS

schema di mole

schema di mole-suit

rilascio graduale di feature

accesso al dato raw solo tramite il dato denormalizzato, quindi mantengo sempre la consistenza, ance se non lavoro con un vero realtime.

#demo?

#benchmark

la parte di insertion è fondamentale per il sistema, quindi abbiamo privilegiato quella nei test

abbiamo deciso di eseguire test solo su mongo in quanto vero collo di bottiglia del sistema.

il nostro obiettivo è cercare di massimizzare la quantità di documenti inseriti nell'unità di tempo.

la documentazione di mongodb consiglia di utilizzare lo sharding per ottenere questo risultato:
cos'è lo sharding.
perché dovrebbe migliorare le performance
come si sharda: 2 tipi di chiave

come abbiamo fatto i test
3 VM mongos e config, mongod, mongod
2 tipi di test, con doc piccoli e grandi, aumentando progressivamente il numero di client che eseguono richieste in modo concorrente.

slides con i 2 grafici



#conclusioni

abbiamo scoperto alcuni aspetti interessanti riguardo allo sharding di mongo

- il balancer applica un overhead non trascurabile nel caso dei doc piccoli e più lo facciamo lavorare, peggio va il cluster
- in presenza di doc grandi, si saturano le code di write e il lavoro del balancer pesa meno nel tempo totale

per concludere posso dire che questo progetto mi è servito molto, perchè mi ha dato l'opportunità di applicare alcune tecnologie che già conoscevo e altre comletamente nuove ad un progetto "vero"! Prima avevo avuto modo solo di utilizzarle per alcuni test o piccolissimi progetti personali.

Le problematiche che sono state affrontate sono di ben altra entità. Possiamo dire che "qui si fa sul serio", ci sono performance da garantire, bisogna cercare di costruire un progetto stabile, estensibile e manutenibile.



#sviluppi futuri

- test più approfonditi sul layer di insertion, includendo nei test anche il server mole
- test su parte di presentation, cercando di capire come ottimizzare l'estrazione di dati massimizzando il throughput
- 
