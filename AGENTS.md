# AGENTS.md

Regeln für KI-Agenten in diesem Repository. Zweck, Aufbau und Befehle des Projekts stehen im README.

## Umgebung
- Lokal und in Codespaces läuft die Entwicklung im Devcontainer (`.devcontainer/`).
- Fehlt ein Werkzeug, benenne es im PR; die Ergänzung gehört in `.devcontainer/`.

## Checks vor Abschluss
- Maßgeblich sind die Schritte in `.github/workflows/ci.yml`. Führe Linter und Tests lokal genauso aus wie die CI. Der Sicherheitsscan läuft nur in der CI; behebe seine Funde.
- Linter und alle Tests müssen grün sein. Neue Funktionen und Fixes bekommen eigene Tests, soweit sie ohne Netzwerkzugriff prüfbar sind.

## Git und Pull Requests
- Jede Änderung gehört zu einem Redmine-Ticket. Arbeite in einem Branch `feature/<ticket>-<kurzbeschreibung>` oder `fix/<ticket>-<kurzbeschreibung>`, nie direkt auf `main`. Vergibt dein Werkzeug den Branch-Namen selbst (z. B. `copilot/…`), behalte ihn und nenne die Ticketnummer im PR-Titel. Fehlt die Ticketnummer, frage danach.
- Halte Änderungen klein und auf eine Aufgabe beschränkt. Gemergt wird per Squash, und zwar nur durch Menschen.
- Nenne die Ticketnummer im PR-Titel ohne „#“ (z. B. „… (Ticket 1234)“), verlinke das Ticket in der Beschreibung und beschreibe, was sich ändert und warum. Breaking Changes (z. B. geänderte Spalten oder Dateinamen in `output/`) werden zusätzlich in die Release Notes übernommen.
- Setze Hinweise aus dem Copilot-Review um oder begründe im PR, warum nicht.

## Abhängigkeiten
- Updates von Abhängigkeiten, GitHub Actions und Base Images übernimmt Dependabot. Ändere Versionen nur, wenn die Aufgabe es erfordert.
- Füge neue Abhängigkeiten nur hinzu, wenn die Aufgabe es erfordert. Begründe sie im PR und trage sie samt indirekter Abhängigkeiten mit fester Version (`==`) in `requirements.txt` bzw. `requirements-dev.txt` ein.
- Base Image in `.devcontainer/Dockerfile` immer mit festem Versionstag, nie `latest`.
- Fremdbibliotheken nur über den Paketmanager einbinden, nie als kopierte Datei ins Repo.

## Betrieb
- Das Projekt läuft nicht als Container. Es erzeugt die TSV-Dateien in `output/` (Enrichment Cache), die [dla-opac-transform](https://github.com/dla-marbach/dla-opac-transform) zur Anreicherung des Katalogs lädt.
- Produktiv wird `output/` von einem Release-Tag geladen, nicht von `main`. Versionen folgen SemVer (`vX.Y.Z`). Tags und Releases legen nur Menschen an.
- Spalten und Dateinamen in `output/` sind die Schnittstelle zu dla-opac-transform. Ändere sie nur auf ausdrückliche Anweisung; das ist ein Breaking Change.
- Neue Daten in `output/` kommen nur per Pull Request auf `main`, nie per Direkt-Commit.

## No-Gos
- Keine Secrets, Tokens oder Passwörter in Code, Tests, Logs oder Commits.
- Keine personenbezogenen Daten oder Produktivdaten in Code, Tests, Beispieldaten, Logs oder Commits. Test- und Beispieldaten sind synthetisch.
- Tests nicht abschwächen oder löschen und Checks oder Scans nicht deaktivieren oder unterdrücken, um die CI grün zu bekommen.
- Keine Einträge in `.trivyignore`. Ausnahmen für Sicherheitsfunde legen nur Menschen an.
- `.github/` und `.devcontainer/` nur auf ausdrückliche Anweisung ändern.

## Projektspezifisch
- Führe die Pipeline (`task`) nicht aus, außer die Aufgabe erfordert es: Sie lädt die gesamte GND über lobid-gnd und fragt Wikidata und Wikimedia Commons ab. Einzelne Tasks lassen sich getrennt ausführen, Zwischenergebnisse liegen in `data/` (nicht im Repo).
- Bearbeite `output/` nicht von Hand. Die Daten erzeugt der Workflow „Daten aktualisieren“ (`.github/workflows/update.yml`).
