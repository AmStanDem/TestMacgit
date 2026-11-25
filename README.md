# Test manuale dei JAR su macOS

Questo repository serve a provare i JAR su un runner macOS tramite GitHub
Actions. `jars/` contiene i JAR; il codice Java presente in `src/` non è
necessario per eseguire questi test.

## Preparazione

I nomi attesi sono:

- `jars/theknife-server.jar`
- `jars/theknife-client-macos.jar`

Il server usa PostgreSQL. Non occorre installarlo sul PC Windows né ricreare
un'immagine Docker dell'applicazione: il workflow prepara PostgreSQL sul runner
macOS. Il server JAR contiene le migrazioni Flyway, che creano e popolano lo
schema al primo avvio. Il workflow prepara database e utente con i valori
predefiniti del server (`theknife`, `admin` / `admin`).

Per limitare l'accesso SSH, registra una chiave pubblica SSH nel tuo account
GitHub (**Settings → SSH and GPG keys**) prima di avviare il workflow.

## Pubblicare modifiche

Dalla cartella del progetto, aggiungi i file e pubblicali sul branch `master`:

```powershell
git add .gitignore README.md .github/workflows/test-macos-gui.yml jars/theknife-server.jar jars/theknife-client-macos.jar
git commit -m "Prepare macOS JAR test"
git push origin master
```

Se hai già creato un commit, puoi saltare `git commit` e fare il push. I JAR in
`jars/` sono tracciati da Git; gli altri JAR e i file di build locali restano
ignorati.

## Provare una versione diversa dei JAR

Per mantenere la versione corrente, crea un branch e sostituisci i JAR
mantenendo gli stessi nomi. Per esempio, da PowerShell:

```powershell
git switch master
git switch -c test/nuovi-jar
Copy-Item C:\percorso\server-nuovo.jar jars\theknife-server.jar
Copy-Item C:\percorso\client-macos-nuovo.jar jars\theknife-client-macos.jar
git add jars/theknife-server.jar jars/theknife-client-macos.jar
git commit -m "Test new JAR versions"
git push -u origin test/nuovi-jar
```

Avviando il workflow e scegliendo `test/nuovi-jar` nel selettore del branch,
GitHub testerà i JAR di quel branch. `master` rimane invariato; puoi creare
branch separati per altre versioni.

## Avviare il test

1. Su GitHub apri **Actions → Test macOS GUI → Run workflow**.
2. Scegli `master` oppure il branch con i JAR da testare e avvia il workflow.
3. Nei log del job copia il comando SSH mostrato da Upterm e collegati.
4. Nella sessione SSH esegui questi comandi dalla root del repository:

```sh
cd "$GITHUB_WORKSPACE"
(printf '\n\n\n\n\n' | java -jar jars/theknife-server.jar) > server.log 2>&1 < /dev/null &

for attempt in {1..30}; do
  if lsof -nP -iTCP:1099 -sTCP:LISTEN >/dev/null; then
    break
  fi
  sleep 1
done

if ! lsof -nP -iTCP:1099 -sTCP:LISTEN; then
  cat server.log
  exit 1
fi

java -jar jars/theknife-client-macos.jar > client.log 2>&1 &
sleep 8
screencapture -x client-screenshot.png
```

Le cinque righe vuote forniscono al server i valori di configurazione
predefiniti; il controllo sulla porta `1099` verifica che il servizio RMI sia
in ascolto. Se il server non parte, consulta `server.log`; per il client
consulta `client.log`.

5. Dopo la cattura, esegui `exit` per chiudere la sessione SSH e consentire al
   workflow di terminare.
6. Scarica `macos-gui-verification` dalla sezione **Artifacts** dell'esecuzione
   GitHub Actions: contiene lo screenshot e i log, se generati.

Il workflow usa Upterm per l'accesso SSH: il relay pubblico di tmate è stato
dismesso e l'azione `mxschmitt/action-tmate` non può più creare la sessione.
