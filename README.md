# Strategienavigator continuous deployment

Dieses Repository deployed die eingestellte Version vom Strategienavigator auf den konfigurierten Server.

## Version aktualisieren

Um die Version auf dem Server zu aktualisieren, muss die passende `versions` Datei angepasst werden.

In dieser Datei kann der `FRONTEND_HASH` und/order der `BACKEND_HASH` angepasst werden.
Der Wert sollte nur auf einen commit verweisen, wie zum beispiel der commit hash oder ein Tag.
Auf keinen Fall sollte ein Branch name gewählt werden.

Auch wenn nur das frontend oder backend geändert wurde, wird beides auf dem Server aktualisiert.

### Fehler beim Aktualisieren

Falls der Workflow beim deployen fehlschlägt, wird auf dem Server eine Datei erstellt, damit nachfolgende Jobs keine
Änderungen mehr vornehmen.

Um nachfolgende Jobs wieder möglich zu machen, muss die Datei `deploy-error` aus dem Heimverzeichnis gelöscht werden. 

Falls die gewünscht, kann anschließend der Workflow Job neugestartet werden.

## Workflow Dokumentation

Der Workflow besteht aus 2 Jobs `generate-matrix` und `deploy`.

### Generate Matrix

Generate matrix besteht nur aus einem
step: [hellofresh/action-changed-files](https://github.com/hellofresh/action-changed-files).

Dieser liest alle Änderungen des letzten push und erstellt auf Basis dieser Änderungen eine job matrix.

Das Argument: `pattern: (?P<environment>[^/]+)/versions` bewirkt dabei,
dass Einträge mit dem Namen `enviroment` und als Wert den namen des Ordners in dem die `versions` Datei geändert wurde,
zur job matrix hinzugefügt wird.

### Deploy

Der `deploy` job installiert die Konfigurierte version vom Front und Backend auf den Server.

Beim Job ist das `environment: production` und `concurrency: production` eingestellt.

`environment` sorgt dafür, dass die secrets und variablen von dem gegebenen Environment geladen und verfügbar sind.
`concurrency` sorgt dafür, dass nur ein Job, der den gegebenen Wert eingestellt hat, gleichzeitig ausgeführt wird.

Hier eine kleine erklärung zu den wichtigsten Workflow Steps:

- `Check for Error Marker` prüft ob auf dem Server eine Datei existiert, die erstellt wurde als ein vorheriger Job
  fehlgeschlagen ist.
- `Frontend Zip Name` erstellt dabei den Dateinamen der Zip Datei des Frontends. Der Name enthält das aktuelle
  Datum und die Sekunden seit 01.01.1970.
- `Install sshpass` wird benötigt, um das passwort eines ssh users automatisch einzugeben.
- `Configure SSH` erstellt eine ssh config Datei um auf den server zuzugreifen und fügt alle keys des servers
  in `known_hosts` hinzu.
- `Load versions` lädt dabei die `versions` geänderte versions datei definiert durch die job matrix.
- `build frontend` baut das Frontend und erstellt ein Zip Archiv mit dem gebauten frontend.
- `Copy Frontend to Server` kopiert das Zip Archiv aus `build frontend` auf den Server in den Backup-Ordner.
- `Update backend` update backend
- `Install frontend` führt den `frontend_patcher` aus.
- `Cache views` cached im backend die blade views.
- `Create Error Marker` erstellt eine Datei, falls der Job fehlgeschlagen ist, um nachfolgende Jobs daran zu hindern
  durchzulaufen.
