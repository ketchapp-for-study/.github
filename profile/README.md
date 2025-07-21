# Guida all’avvio del progetto KetchApp
_Progetto Universitario unico realizzato da Alessandro Bruno & Alessandra Di Bella, che comprende gli esami dei corsi: Sviluppo del Software, Sistemi Cloud, Sistemi Mobile._

## Prerequisiti

Per eseguire correttamente il progetto, assicurati di avere installato sul tuo sistema:

- **Docker**  
- **Docker Compose**
> Questi strumenti sono necessari per gestire e avviare i container del progetto in modo semplice e portabile.

## Avvio del progetto

1. Scaricare l'ultima [Release](https://github.com/ketchapp-for-study/releases/releases) che si trova nell Repo Git [releases](https://github.com/ketchapp-for-study/releases).

### Windows

2. Apri **Git Bash** nella cartella principale del progetto.
3. Esegui il comando:
   ```bash
   ./build.sh
   ```

### Linux & macOS

2. Apri il **Terminale** nella cartella principale del progetto.
3. Esegui il comando:
   ```bash
   ./build.sh
   ```

## Note
- Il file `build.sh` si occuperà di costruire e avviare i container necessari tramite Docker Compose.
- Assicurati che Docker sia in esecuzione prima di avviare lo script.
- In caso di problemi, verifica che Docker e Docker Compose siano correttamente installati e aggiornati.

# Importante
- Momentaneamente allo stato attuale (21/07/2025) il [FrontEnd in Flutter](https://github.com/ketchapp-for-study/KetchApp-Flutter) non è possibile utilizzarlo a pieno poichè per gli ultimi esami di Sviluppo del Software e Sistemi Cloud, sono state dovute effettuare delle modifiche per poter rispettare le richieste degli esami. In caso si voglia provare ad avviarlo contattare [@Dibbiii](https://github.com/Dibbiii) o [@alessandrobrunoh](https://github.com/alessandrobrunoh) cosi da sistemare le modifiche e renderlo eseguibile.

_2024-2025_
