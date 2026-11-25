# Test manuale su macOS

Avvia il workflow **Test macOS GUI** da GitHub Actions e connettiti alla sessione
SSH mostrata nei log. Nella sessione esegui i comandi seguenti dalla root del
repository:

```sh
cd "$GITHUB_WORKSPACE"
java -jar jars/theknife-server.jar > server.log 2>&1 &
java -jar jars/theknife-client-macos.jar > client.log 2>&1 &
sleep 5
screencapture -x client-screenshot.png
```

Il server e il client vengono avviati in background; il client resta attivo
durante la cattura. Controlla `server.log` e `client.log` in caso di errori.
