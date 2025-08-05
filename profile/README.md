# Relazione Solution Design KetchApp | Corso Sistemi Cloud A.A. 2024/2025 | Laurea in Tecnologie dei Sistemi Informatici - UNIBO
_By Alessandro Bruno & Alessandra Di Bella_

# 1. Introduzione

Questo documento descrive il Solution Design di "KetchApp", un'applicazione mobile progettata per ottimizzare la produttività e le abitudini di studio degli utenti attraverso il metodo Pomodoro, ovvero una tecnica di gestione del tempo che divide il lavoro in intervalli, chiamati "pomodori", separati da brevi pause. L'obiettivo primario di KetchApp è fornire un sistema personalizzato di pianificazione dello studio, integrando funzionalità avanzate come l'organizzazione intelligente delle sessioni basata su intelligenza artificiale, il monitoraggio delle statistiche di studio, un sistema di obiettivi (achievements) e classifiche globali.

Con questa relazione descriveremo l'architettura di KetchApp, le decisioni di design adottate e l'integrazione con altri microservizi.

# 2. Raccolta Requisiti e Obiettivi

## Richieste del committente:

*Requisiti non funzionali aggiuntivi per il corso Sistemi Cloud.*

1. Sviluppare un’architettura a microservizi
2. Utilizzare Docker
3. Utilizzare Spring Boot
4. Utilizzare Kafka per l’invio asincrono delle email
5. Sviluppare un’autenticazione che sfrutta un token JWT con chiave pubblica e privata
6. Mettere alcuni microservizi in cloud

# 3. Analisi

## Requisiti Funzionali

I requisiti funzionali descrivono le funzionalità che l'applicazione deve offrire agli utenti dopo che questi ultimi hanno completato il processo di autenticazione.

- **Pianificazione dello Studio Personalizzata:**
    - Gli utenti devono poter inserire le materie da studiare e la durata desiderata per ciascuna.
    - Gli utenti devono poter definire la durata delle proprie sessioni di studio ("pomodori") e delle pause.
    - Gli utenti devono poter inserire impegni e orari specifici durante la giornata (es. appuntamenti, lezioni).
    - Un'intelligenza artificiale deve elaborare queste informazioni per generare un piano di studio ottimizzato, suddividendo il tempo di studio in "pomodori" e rispettando gli impegni inseriti.
- **Sessioni di Focus:** L'applicazione deve consentire l'avvio e la gestione di sessioni di studio basate sul metodo Pomodoro, con timer dedicati per il focus e le pause.
- **Statistiche di Studio:** L'applicazione deve presentare un sistema di statistiche che visualizzi le ore totali di studio per ogni singola materia dell'utente in un intervallo di date selezionabile.
    - Visualizzazione tramite grafico a barre delle ore di studio giornaliere.
    - Possibilità di selezionare un giorno specifico dal grafico per visualizzare statistiche dettagliate (materie studiate e tempo impiegato per ciascuna).
- **Sistema di Achievements:** Gli utenti devono poter visualizzare gli obiettivi completati giornalmente e quelli ancora da raggiungere, incentivando la costanza nello studio.
- **Classifiche Globali:** L'applicazione deve visualizzare una classifica globale dei primi 100 utenti con il maggior numero di ore di studio totali, promuovendo la competitività.

## Requisiti Non Funzionali

I requisiti non funzionali definiscono le qualità del sistema e i vincoli operativi.

- **Usabilità:** L'interfaccia utente (frontend in Flutter) deve essere intuitiva e facile da usare per garantire un'esperienza utente ottimale.
- **Affidabilità:** Il sistema deve garantire la corretta gestione e persistenza dei dati utente, delle pianificazioni e delle statistiche, minimizzando la perdita di dati.
- **Scalabilità:** Il backend (Java con Spring Boot) deve essere progettato per gestire un numero crescente di utenti e richieste, mantenendo prestazioni elevate.
- **Sicurezza:** L'applicazione deve implementare meccanismi di autenticazione robusti, inclusa la validazione di token JWT tramite un middleware di sicurezza.
- **Sistema di Messaggistica Asincrona:** L'applicazione deve utilizzare un broker di messaggi (Kafka) per gestire l'invio asincrono di email (es. promemoria inizio sessione), garantendo scalabilità e affidabilità del sistema di notifica.

## Analisi delle Soluzioni Esistenti

Prima di iniziare la fase di progettazione della nostra applicazione, abbiamo analizzato alcune soluzioni esistenti riguardanti la produttività e lo studio, in particolare quelle che implementano il metodo Pomodoro o funzionalità di tracciamento dello studio. Questo ci ha permesso di identificare le funzionalità più comuni, i punti di forza e le funzionalità che avremmo potuto migliorare o aggiungere con la nostra applicazione.

### **Funzionalità Comuni nelle App Esistenti:**

- **Timer Pomodoro:** La maggior parte delle app offre un timer configurabile per le sessioni di studio e le pause.
- **Tracciamento del Tempo:** Funzionalità per registrare il tempo di studio per materia o progetto.
- **Statistiche:** Visualizzazione di statistiche aggregate come il tempo totale di studio giornaliero o settimanale.
- **Liste di Attività/To-Do:** Integrazione con liste di cose da fare per organizzare le sessioni.

### **Punti di Forza delle Soluzioni Esistenti:**

- **Semplicità d'uso:** Molte app sono progettate per essere immediate e facili da avviare.
- **Focus sul Pomodoro:** Efficaci nel mantenere l'utente concentrato durante le sessioni.

### **Lacune e Opportunità di Differenziazione per KetchApp:**

Analizzando le applicazioni esistenti abbiamo notato diverse aree in cui sono presenti delle limitazioni e dove KetchApp può offrire un valore aggiunto:

- **Pianificazione Intelligente:** Molte app si limitano a fornire timer o a permettere l'inserimento manuale delle sessioni. Mancano di una logica avanzata per ottimizzare la pianificazione in base a impegni, preferenze e durata delle materie. **KetchApp si distingue per l'integrazione di un Intelligenza Artificiale**, che permette la **generazione automatica e personalizzata di piani di studio ottimizzati**, un aspetto raramente presente nelle app attuali.
- **Classifiche:** Le **classifiche globali** sono una funzionalità poco comune nel contesto delle app di studio. KetchApp introduce questo elemento per promuovere la competitività e la motivazione tra gli utenti, incentivando la costanza nello studio.

# 3. Progettazione della Soluzione

## KetchApp Context Diagram (Alto Livello)

```
@startuml
!include https://raw.githubusercontent.com/kirchsth/C4-PlantUML/extended/C4_Container.puml

title KetchApp - Context Diagram

Person(user, "User", "Utente che crea e gestisce i propri piani di studio")
Container_Boundary(ketchapp, "KetchApp Application") {
    System(frontend, "FrontEnd", "Interfaccia utente che permette di visualizzare e modificare i piani di studio")
    Container(backend, "BackEnd", "Punto di accesso unico che instrada le richieste dell'app mobile ai microservizi interni")
}
System_Ext(email, "Invio Notifiche Email", "Invia notifiche e comunicazioni via email all'utente.")

Rel_D(user, frontend, "Interagisce con Applicazione")
Rel_R(frontend, backend, "Comunica con le Api")
Rel_D(backend, email, "Invia Email con dettagli sul Piano Studio")

@enduml
```

[![test-KetchApp___Context_Diagram.png](Relazione%20Solution%20Design%20-%20KetchApp%2024268c8ca0e9806aaa29c47556226de5/test-KetchApp___Context_Diagram.png)](https://uml.planttext.com/plantuml/png/PL9BRnen4BxlhvXoAP4QkFJKKn8WKjI7KAASqM0Fx0YllVA39bJzx_sy443gIfQzzeo_b-zIvfXBvvfFweqhLgZDkK_VfjF8loTRpMrPboJH19_5vua9tNIdqRGfjzFPrzkbOvzVV_wOoXlEhLXihcOePlKI9tszkicQdT1toQQzxtkwfLg01fehRoDtJREKc88VLwGlD7h5DAfpXHGoBKOK9g6jfAp922drCVGHax9Niaef5wjXTDESqLSFeNZByNsygz3SzxZpp0B3gU6imOzkw4z3-5xUKvPkl8c3MelonxfSU5lfF64pRjWOPIeNQht4JZ9-G6AlgR3Jmu6ZW6uNh6u04WV2_p6ja6UsupMRtH7q0QiJvhBu76eJO2MbGCMh2GEk-fGZMoPu6zMq2cz0Gfpx3Ad0NYjEMAbJ4mCitUj1qGKHNe7-jpdANItUybWwLZTet6kWNo5NtW1PrFtHwO39dm-WFNIL31_WO5MEV6enCGWSxPa0fNaMjvgVahqoTZ2JjqEOU5mVSTJB16srw_agY8ivinFiq0M1zxsUwkSN2w_by574K6yH56yTCYYEbWWaqCj76iqTMl5U_m40)

## KetchApp Diagram Context (Basso Livello)

```
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Component.puml

title KetchApp - Diagram Context

Person(user, "Utente", "Crea e gestisce i propri piani di studio e le relative attività.")


Container_Boundary(ketchapp_application, "KetchApp Application") {
    Component(frontend, "FrontEnd UI ", "Flutter", "Permette all' utente di gestire i propri piani di studio e le relative attività.")

    Container_Boundary(bffcompose, "Bff Compose") {
        Container(bffapi, "BackEnd Bff Api", "Spring Boot", "Gestisce le richieste dell' utente e richiama i microservizi interssati.")
    }

    Container_Boundary(appcompose, "App Compose") {
        Container(appapi, "App Api", "Spring Boot", "Gestisce le richieste dell' utente e richiama i microservizi interssati.")
    }

    Container_Boundary(authcompose, "Auth Compose") {
        Container(authapi, "Auth Api", "Actix", "Gestisce l' autenticazione e l' autorizzazione degli utenti.")
        ContainerDb(authdb, "Auth Database", "PostgreSQL", "Memorizza i dati relativi agli utenti e alle loro credenziali.")
    }

    Container_Boundary(kafkacompose, "Kafka Compose") {
        Container(kafka_broker, "Kafka Broker", "Apache Kafka", "Gestisce la ricezione e l'invio dei messaggi tra i microservizi.")
        Container(kafka_zookeeper, "Zookeeper", "Apache Zookeeper", "Gestisce la configurazione e il coordinamento dei microservizi.")
    }
    
    Container_Boundary(kafkaconsumercompose, "Kafka Consumer Compose") {
        Container(kafkaengine, "Kafka Engine", "Spring Boot Kafka", "Gestisce la ricezione e l' invio dei messaggi tra i microservizi.")
    }
  

    Container_Boundary(supabase, "Supabase") {
        ContainerDb(appdb, "App Database", "PostgreSQL", "Gestisce la memorizzazione dei dati relativi agli utenti e alle loro interazioni con l' applicazione.")
    }
}

System_Ext(smtp, "SMTP Service", "Servizio di invio email per notifiche e comunicazioni.")
System_Ext(mail, "Gmail Service", "Servizio di invio su Gmail per notifiche e comunicazioni.")

Rel_R(user, frontend, "Interagisce con l' applicazione")
Rel_D(frontend, bffapi, "Richiede dati e servizi")
Rel_D(bffapi, appapi, "Richiede dati e servizi")
Rel_R(bffapi, authapi, "Richiede autenticazione e autorizzazione")
Rel_D(authapi, authdb, "Memorizza e recupera dati utente")
Rel_D(appapi, appdb, "Memorizza e recupera dati utente")
Rel_L(appapi, kafka_broker, "Invia messaggi")
Rel_D(kafka_broker, kafkaengine, "Consuma messaggi")
Rel_D(kafka_broker, kafka_zookeeper, "Gestisce la configurazione e il coordinamento")
Rel_R(kafkaengine, smtp, "Invia Email via SMTP")
Rel_D(smtp, mail, "Invia notifiche via Email")

@enduml
```
# **Spiegazione dei Componenti:**

[![test-KetchApp___Diagram_Context.png](Relazione%20Solution%20Design%20-%20KetchApp%2024268c8ca0e9806aaa29c47556226de5/test-KetchApp___Diagram_Context.png)](https://uml.planttext.com/plantuml/png/nLPDJnin4Btlht2v4AH2BZtrn3T2W9I6SkabSdOdcr5sRSlsKaZ5V-b_wH_hZBtnPY50wAL84DkPvtdp_3pcq7bfVLDNsITKUTKK8ERU6_TrEBJovq69VjvC6mSsryg3yWUvheUcaeen-yuN5Kw79r_sHxnqTtCzhANpO6bfSg9henLZ3E-VUVGLY2lm-Vp86B4lJb6MLjRYXBT-y5as0kkq6d2wpsBdZlF13erEB4W1eWJdqUKWK1YhZQKFb0f5WSBvfa1DCPJ1GYKzteEGdZxG__bziBENPHcdaQZ0Jevremff7mSB9YEDcT1lXJd1jA9yYUDHjxop9tvbWdvIMOEP3PeKX3ZduPagnDsbOCRdLUD91XvIKJNGHCYgsXLDA8efXshiFnOJYRoePpgRvKpF0IK-diqYMGU9V0_7uT8WXyfymUmPScIGQTyI9LMAOwqzJozMsZCdpET8Soe3rcfglv2rf9fgp2qbjlUuH87qjNME2c7opE7frHBe9BeI-0pUBe72OmdnkFulzSRFrxZJx3to5DAov-2M_b7kyQ5FV5V8m9Iyk2GlCk6ufYqkb-rQ0MM5iQ94jfVlT1eo5jDLmbFfvLIwSCT6sldImkstQvxTG1st9ZqAAdxbGnIoIoA2fucAjbha5WfGIvJL5aejv6mXEwckUFgELW4ocLgz2CqXGex3D8XcP3wdxiBBVUqadorqig6wfyjL01qqqC6M9Gfldv_xHlrQ3ajDIS446ZzMupKElRLr7jH2PrWsDfqXLhIcRO5ArYHeose3ZQVmznqrbMjgxjFFLOthsyWBggH9Wfw5wRCRjOt4uaCQSt6lEiKr9bYKIRJ3pHMmkOs9tgPMyBgrrsdNAvklRj2sRWzj8Q2GZpNSnlXMX9swqkWEt3vIlwadPmz-u6flk9ARxoDnoohaKTueYERc7wM3Mf8vo4D2QOypP5y1PQeRrUO8ygrjpG2kBW3VtDirucAxxRCnL9Dn-nwllNQNeVeogBYXUa8oy7JjWKmlpJWqOFel8-WCelL2Wgm2Krz_6p3k0AcN9iIBdjblc2bdWgRksBK_Ve7pXfIIaK1yC3feItDblIs1rmduhArTqWd9T6bIcdvK_wx62xuTgDUxFjIQajwzvAsV8-kpO2eUiRyJZHZJsZD6TgPBE7RR8Lc5_bFy2m00)
1. **Smartphone Utente (App KetchApp - Flutter):** L'applicazione mobile, interfaccia utente finale, da cui l'utente interagisce con il sistema.
2. **PC Fisso Nostro (Backend-Fe - Frontend che espone le API al client):** Un Backend For Frontend (BFF) che agisce come gateway per le richieste provenienti dall'applicazione mobile. Questo componente semplifica l'interazione del client con i vari microservizi backend e gestisce l'inoltro dei token JWT.
3. **KetchApp Auth API (Rust - Ambiente Cloud):** Un microservizio dedicato esclusivamente alla gestione dell'autenticazione e autorizzazione.
    - **Generazione JWT:** Dopo un login riuscito da parte dell'utente (richiesta dal BFF), la Rust Auth API genera e firma digitalmente un JSON Web Token (JWT). Questo token, contenente le informazioni sull'utente, viene inviato al BFF e poi al client.
    - **Validazione JWT:** È anche responsabile della validazione dei token JWT per le richieste di autenticazione.
    - Utilizza un **Auth Database (PostgreSQL)** per memorizzare i dati degli utenti.
4. **KetchApp API (Java - Ambiente Cloud):** Il microservizio principale del backend, scritto in Java con Spring Boot, che implementa la logica di business dell'applicazione (pianificazione, statistiche, achievements, classifiche).
    - Accede e persiste i dati nel **KetchApp Database (PostgreSQL)**.
5. **Java Kafka API (Java - Ambiente Cloud):** Un microservizio separato che gestisce le comunicazioni asincrone all'interno dell'applicazione.
    - **Gestisce Servizi Asincroni:** In particolare, è responsabile della produzione e del consumo di messaggi da e verso il broker Kafka.
    - Interagisce con **Kafka (Apache Kafka)**, che funge da Message Broker per la gestione dei messaggi.
6. **Email Inviata (Servizio esterno di invio email):** Un servizio esterno che consuma i messaggi dal topic Kafka relativi alle notifiche (es. piani di studio creati) e si occupa dell'invio effettivo delle email agli utenti.
   
## **Flusso del Token JWT**
```
@startuml
title KetchApp - Token JWT Flow

actor Utente as U
participant "FrontEnd" as Client
participant "Bff BackEnd" as Bff
participant "Auth BackEnd" as Auth

== Login e JWT ==
U -> Client: Login (credenziali)
Client -> Bff: Richiesta di login(credenziali)
activate Bff
Bff -> Auth: Richiesta di login(credenziali)
deactivate Bff
activate Auth
Auth -> Auth: Valida le credenziali
Auth -> Auth: Genera il JWT
Auth -> Client: Restituisce il JWT
deactivate Auth
note right of Client: Client memorizza il JWT

' Fase 2: Uso del token per accedere a una risorsa
== Accesso API protetta ==
Client -> Bff: Richiesta API\n(Header: Authorization: Bearer JWT)
note right of Client: Prende il Token Memorizzato
activate Bff
Bff -> Bff: Valida il JWT
Bff --> App: Risposta API (dati)
deactivate Bff
activate App
App -> App: Esegue Logica della Api
App -> Bff: Risposta API(dati)
deactivate App
activate Bff
Bff -> Client: Risposta API(dati)
deactivate Bff
Client -> U: Visualizza dati

@enduml
```

[![test-KetchApp___Token_JWT_Flow.png](Relazione%20Solution%20Design%20-%20KetchApp%2024268c8ca0e9806aaa29c47556226de5/test-KetchApp___Token_JWT_Flow.png)](https://uml.planttext.com/plantuml/png/ZLF1JXin4BsljFymuj9muC9nfLGa5IbGAY74jfTUBEya6U7ObZqBKhwU6MUpWOYWtjOxxysyDy--ocmIznjNLqpi47uXs_KiHhY0f_2C7kx-Fi72XTUwgYjZEIHe6JqZc0njNKMXawLeFCFP8WNFrxuxq-8FHu8xGioNIvWR-xm7oVa8CUjv_G6YFtJuT0htOKKUi6YQJkkgXOllmvncA4vimWxzbeoZyxhQ5HKcanfu9Bic5C_G4JWb7E758RqOSLU4gLgXgeB_u7RuaJsUTWwAiR7R7-5q1cJZxveSWtwYnsI0d3e-5FUE7qKDSK_PueXv9s8trWVvJBHQCuJbI1tsii5DIBJT7cRKrJTOc8nmsK2R0tJeWCj5Y9Z0M2jYauGFlJVIDeUKJSbc9gKiXDd3BSGaGrasfI6TZ42G__pa1erqR8fQrMAOWczWZYR9GD5qVih3GqBV5UUxc_fxRuR3YI2BW67pUyEbf1kFKSNb60PjCEb4ovUvnYYXw5iPwDSPLpsMcsYDhiv9fqWZQ5Z0OSOd8qhJJzMFiN_Te50EIs_5CULUJ6lC2jUGhsHrykhV0000)

# Definizione delle Decisioni Architetturali

### Adozione dell'Architettura a Microservizi

- **Contesto:** Il progetto KetchApp prevede lo sviluppo di un'applicazione mobile scalabile e robusta.
- **Problema:** Un'architettura monolitica avrebbe potuto portare a difficoltà di scalabilità, tempi di deployment lunghi e sarebbe stata difficile da mantenere.
- **Decisione:** Abbiamo suddiviso l'applicazione in microservizi.
- **Motivazioni:**
    - **Scalabilità Indipendente:** Ogni microservizio può essere scalato orizzontalmente in base al proprio carico, ottimizzando l'uso delle risorse.
    - **Manutenibilità:** La suddivisione in servizi più piccoli e autonomi rende il codice più facile da comprendere, sviluppare e mantenere.
    - **Flessibilità Tecnologica:** Permette l'utilizzo di tecnologie diverse per servizi specifici (es. Rust per l'autenticazione, Java Spring Boot per la logica principale).

### Gestione dell'Autenticazione tramite JWT Middleware

- **Contesto:** Necessità di un meccanismo di sicurezza robusto per proteggere le API del backend, garantendo che solo gli utenti autenticati possano accedere alle risorse.
- **Problema:** Le sessioni tradizionali sono state scartate in quanto sono stateful e questo contrasta i principi stateless delle API REST.
- **Decisione:** Implementare un middleware di sicurezza basato su Spring Security configurato per la validazione dei token JWT tramite una chiave pubblica RSA.
- **Motivazioni:**
    - **Statelessness:** I JWT sono auto-contenuti, eliminando la necessità per il server di mantenere lo stato della sessione in un database.
    - **Centralizzazione della Logica:** Il middleware intercetta ogni richiesta HTTP in entrata, centralizzando la logica di validazione e impostando il contesto di sicurezza di Spring.
    - **Efficienza:** La validazione avviene rapidamente senza interpellare un database di sessioni ad ogni richiesta.

### Comunicazione Asincrona con Apache Kafka per Notifiche

- **Contesto:** Necessità di inviare notifiche (es. email sui piani di studio, promemoria) in modo affidabile e scalabile, senza bloccare il flusso principale dell'applicazione.
- **Problema:** L'invio sincrono di email potrebbe introdurre latenza nel processo di creazione del piano di studio e rendere il sistema vulnerabile a fallimenti del servizio di invio email.
- **Decisione:** Utilizzare Apache Kafka come broker di messaggi per gestire l'invio asincrono delle email. Un microservizio dedicato (Java Kafka API) si occupa di consumare questi messaggi.
- **Motivazioni:**
    - **Disaccoppiamento:** Il servizio che crea il piano di studio non dipende direttamente dal servizio di invio email.
    - **Affidabilità:** I messaggi vengono mantenuti in Kafka, garantendo che le notifiche vengano inviate anche se il servizio di invio email è temporaneamente non disponibile.

### Persistenza Dati con PostgreSQL

- **Contesto:** Necessità di un database relazionale robusto e affidabile per la persistenza dei dati strutturati dell'applicazione (utenti, achievements, tomatoes, activities).
- **Problema:** Bisognava scegliere il database più adatto per garantire la consistenza, l'integrità e le prestazioni dei dati in base alle nostre disponibilità e conoscenze.
- **Decisione:** Utilizzare PostgreSQL come database relazionale principale per entrambi i microservizi (Autenticazione e Applicazione principale).
- **Motivazioni:**
    - **Affidabilità e Consistenza:** PostgreSQL è un database relazionale robusto che supporta ACID (Atomicità, Consistenza, Isolamento, Durabilità) e ha la capacità di gestire dati complessi.
    - **Open Source:** Non ha costi di licenza.
    - **Ecosistema:** Ampio supporto della community e integrazione con Spring Data JPA.

### Integrazione AI per la Pianificazione dello Studio

- **Contesto:** Necessità di fornire una pianificazione dello studio personalizzata e ottimizzata basata sugli input dell'utente.
- **Problema:** Implementare una logica di pianificazione complessa manualmente sarebbe molto difficile e poco flessibile.
- **Decisione:** Integrare le Gemini API (Google) per sfruttare le capacità di intelligenza artificiale nella generazione del piano di studio.
- **Motivazioni:**
    - **Ottimizzazione:** Permette di generare piani di studio intelligenti che tengono conto di materie, ore di studio e impegni, massimizzando l'efficienza.
    - **Personalizzazione:** Offre un'esperienza utente altamente personalizzata.
    - **Riduzione della Complessità:** Delega la logica complessa di pianificazione a un servizio AI esterno, semplificando la logica di business interna.

### Deployment e Gestione con Docker e Docker Compose

- **Contesto:** Necessità di un ambiente di sviluppo e deployment coerente, isolato e portabile per i microservizi.
- **Problema:** La gestione delle dipendenze e degli ambienti per ogni microservizio può essere complessa e soggetta a errori.
- **Decisione:** Utilizzare Docker per containerizzare ogni microservizio e Docker Compose per gestire l'ambiente di sviluppo locale e i servizi correlati.
- **Motivazioni:**
    - **Isolamento:** Ogni servizio opera nel proprio ambiente isolato, garantendo coerenza tra gli ambienti di sviluppo, test e produzione.
    - **Portabilità:** I container possono essere eseguiti su qualsiasi sistema che supporti Docker.
    - **Semplificazione del Deployment:** Docker Compose semplifica l'avvio e la gestione di più servizi interconnesi.
    - **Efficienza delle Risorse:** I container sono più leggeri delle VM.

### Tecnologie Utilizzate

- **Frontend:** Flutter.
- **Backend (Applicazione & Kafka):** Java 21 con Spring Boot.
- **Backend (Autenticazione):** Rust.
- **Database:** PostgreSQL.
- **Messaggistica Asincrona:** Apache Kafka.
- **Intelligenza Artificiale:** Google Gemini API.
- **Testing:** JUnit 5, Mockito, Spring MockMvc.
- **Documentazione API:** Swagger/OpenAPI.
- **Deployment/Containerizzazione:** Docker, Docker Compose.

# 4. Implementazione e Test

## Test Di Integrazione

Nel nostro progetto, abbiamo scelto di sottoporre a test di integrazione il livello di routing, `PlanBuilderRoutes`, che gestisce le richieste HTTP in entrata relative alla creazione di piani di studio. Questo componente si occupa di ricevere le richieste, validare i dati e indirizzarle al controller appropriato.

### Test Implementati

Nei test implementati abbiamo coperto i seguenti scenari:

1. **testCreatePlanBuilder():** Verifica la creazione di un piano con dati validi aspettandosi che una richiesta valida porti a una risposta 200 OK e che il controller venga invocato con i dati corretti.
2. **testCreatePlanBuilder_InternalServerError():** Verifica la gestione di un errore interno del server (HTTP 500).
3. **testCreatePlanBuilder_BadRequest():** Testa una richiesta con un corpo JSON vuoto o incompleto, aspettandosi una risposta 400 Bad Request, che conferma il funzionamento della validazione dei dati a livello di routing.
4. **testCreatePlanBuilder_PartialBadRequest():** Verifica la risposta a una richiesta con DTO con dati insufficienti verificando che venga comunque generato un 400 Bad Request.
5. **testCreatePlanBuilder_InvalidValuesBadRequest():** Verifica la risposta a una richiesta con valori non validi aspettandosi che vengano rifiutati con un 400 Bad Request.
6. **testCreatePlanBuilder_NotFound():** Verifica la risposta a una richiesta verso un endpoint inesistente, aspettandosi un HTTP 404.

Per realizzare questi test abbiamo utilizzato il framework JUnit 5 in combinazione con Mokito per la simulazione delle dipendenze e con Spring MockMvc per la simulazione delle richieste HTTP. In particolare:

- **JUnit5:** è Il framework di test principale, esteso con `MockitoExtension` per abilitare l'integrazione con Mockito
- **Mokito:** permette di sostituire le dipendenze della classe che stiamo testando con delle versioni fittizie chiamate Mock che possono essere programmate per agire in un determinato modo simulando l’oggetto reale.
- **Spring MockMvc:** è uno strumento fornito da Spring Test per simulare richieste HTTP verso i controller Spring senza dover avviare un server HTTP completo.

In questo modo i test vengono eseguiti in modo automatico verificando la correttezza del flusso delle richieste API e la validazione degli input.

### Pattern

Il metodo di test che abbiamo usato segue il patter Arrange, Act, Assert (AAA), ovvero nella fase di Arrange prepariamo i dati di input (come i DTO di richiesta) e definiamo il comportamento dei mock, nella fase di Act eseguiamo l’operazione da testare, ovvero in questo caso l’invio di una richiesta HTTP simulata grazie a Spring MockMvc e infine nella fase di Assert verifichiamo la correttezza dei risultati.

## Test Unitari

Abbiamo implementato una serie di test unitari per la classe **`KafkaProducer`**, il cui compito è serializzare un oggetto `PlanBuilderRequestDto` in una stringa JSON e inviarlo in un topic Kafka.

### Test Implementati

Nei test implementati abbiamo coperto i seguenti scenari:

- **`testSendToKafkaSuccess()`:** Verifica il caso di successo, in cui un oggetto `PlanBuilderRequestDto` valido viene correttamente serializzato in JSON e inviato al topic Kafka corretto.
- **`testSendToKafkaJsonProcessingException()`:** Verifichiamo un caso di errore. Simuliamo una `JsonProcessingException` da parte dell'`ObjectMapper` e verifichiamo che il nostro producer non tenti di inviare il messaggio a Kafka, confermando che l'eccezione è gestita in modo corretto.
- **Test di gestione dei valori nulli e vuoti:** Abbiamo sviluppato una serie di test per verificare il comportamento del producer in caso valori non validi o mancanti, come un oggetto `plan` vuoto (`testSendToKafkaWithEmptyPlan`), nullo (`testSendToKafkaWithNullPlan`), incompleto (`testSendToKafkaWithIncompletePlan`), o email vuote e nulle (`testSendToKafkaWithEmptyEmail`, `testSendToKafkaWithNullEmail`). Per ogni scenario, abbiamo verificato che il componente gestisse correttamente questi casi, inviando comunque un messaggio formattato in modo appropriato, delegando la validazione dei dati a un livello superiore.

# 5. Distribuzione e Manutenzione

Il nostro applicativo non andrà in produzione, ma se dovessimo pubblicarlo eseguiremmo prima dei test per valutarne la sicurezza e le eventuali vulnerabilità. Per questo abbiamo scannerizzato i codici sorgenti dei nostri microserivzi tramite i tool SonarQube e Qodana per identificare eventuali falle di sicurezza e questo é il risultato:

## Qodana

### Bff Api

![image.png](Relazione%20Solution%20Design%20-%20KetchApp%2024268c8ca0e9806aaa29c47556226de5/36f9fcad-6871-4775-bd73-392fa4889eef.png)

### App Api

![image.png](Relazione%20Solution%20Design%20-%20KetchApp%2024268c8ca0e9806aaa29c47556226de5/716fd2f6-3e2d-4cfd-b919-be5e76914a27.png)

### Kafka Engine

![image.png](Relazione%20Solution%20Design%20-%20KetchApp%2024268c8ca0e9806aaa29c47556226de5/e665340d-2598-4b43-993e-e7dce6d35719.png)

## SonarQube

### Bff Api

![Screenshot 2025-08-04 at 18.23.34.png](Relazione%20Solution%20Design%20-%20KetchApp%2024268c8ca0e9806aaa29c47556226de5/Screenshot_2025-08-04_at_18.23.34.png)

### App Api

![Screenshot 2025-08-04 at 18.23.27.png](Relazione%20Solution%20Design%20-%20KetchApp%2024268c8ca0e9806aaa29c47556226de5/Screenshot_2025-08-04_at_18.23.27.png)

### Kafka Engine

![Screenshot 2025-08-04 at 18.23.50.png](Relazione%20Solution%20Design%20-%20KetchApp%2024268c8ca0e9806aaa29c47556226de5/Screenshot_2025-08-04_at_18.23.50.png)

> In generale vediamo che tutti i microservizi mostrano uno stato "Passed" nelle scansioni SonarQube, il che è un buon punto di partenza, tuttavia ci sono alcune vulnerabilitá da sistemare, in particolare in KetchApp-App-Api. Nel caso volessimo mandare in produzione il nostro applicativo dovremmo sistemare prima queste problematiche.
>


TODO: Deployment Diagram
