# Ablauf bei neuem Feature oder Fix

Die verbindlichen Grundsätze (Branching, Prüfungen, Secrets, Betrieb) stehen in der [DevOps-Vorlage](https://github.com/dla-marbach/devops). Regeln für KI-Agenten stehen in der `AGENTS.md`.

Das Projekt läuft nicht als Container. Release, Deployment und Rollback sind deshalb nach „Projekte ohne Container-Betrieb“ in der DevOps-Vorlage beschrieben. Die Daten in `output/` lädt [dla-opac-transform](https://github.com/dla-marbach/dla-opac-transform) in den Katalog.

## Entwicklung

1. **Ticket:** Redmine-Ticket anlegen oder übernehmen. Darin die geplante Änderung, wesentliche Schritte und den Abschluss dokumentieren.
2. **Branch anlegen:** `git switch main && git pull && git switch -c feature/1234-kurzbeschreibung` (bzw. `fix/…`, mit Ticketnummer). Branches, die KI-Agenten selbst anlegen (z. B. `copilot/…`), behalten ihren Namen; die Ticketnummer steht dann im PR-Titel.
3. **Im Devcontainer entwickeln:** Codespace auf dem Branch oder lokal „Reopen in Container“. Tests ergänzen, Linter und Tests lokal ausführen wie in `.github/workflows/ci.yml`: `ruff check .` und die übrigen Schritte des Jobs „Linter und Tests“. Die gesamte Pipeline (`task`) nur ausführen, wenn die Änderung es erfordert, sie lädt die gesamte GND. Der Sicherheitsscan läuft in der CI.
4. **Pushen und Pull Request nach `main` öffnen:**
   - Titel mit Ticketnummer ohne „#“, z. B. „Neues Feld für Erscheinungsverlauf (Ticket 1234)“. `#1234` würde auf GitHub verlinken statt auf Redmine.
   - Beschreibung: Link zum Ticket, was sich ändert und warum, Breaking Changes (z. B. geänderte Spalten oder Dateinamen in `output/`).
5. **Copilot-Review anfordern** (unter „Reviewers“). Hinweise umsetzen oder im PR begründen, warum nicht.
6. **CI abwarten:** Alle Checks müssen grün sein, bei Funden nachbessern. Ist `main` weitergegangen, „Update branch“ und erneut abwarten.
7. **Squash-Merge:** Prüfen, dass die Ticketnummer in der Commit-Nachricht steht. Danach lokal aufräumen: `git switch main && git pull --prune && git branch -D feature/1234-kurzbeschreibung`

## Daten aktualisieren

Neue Daten kommen nie per Direkt-Commit auf `main`, sondern per Pull Request.

1. **Workflow starten:** „Actions“ → „Daten aktualisieren“ → „Run workflow“. Der Workflow führt `task` aus; die Task `output` bricht ab, wenn eine Datei deutlich kleiner wird als bisher. Gibt es Änderungen, legt er einen Branch `daten/JJJJ-MM-TT` mit Pull Request an und startet dafür die CI.
2. **Prüfen:** Diff-Statistik im Pull Request auf Plausibilität prüfen (in der Regel überwiegend neue Zeilen), CI grün.
3. **Squash-Merge**, danach ein Patch-Release.

Automatische Läufe (z. B. ein Scheduler im DLA) committen ebenfalls nicht auf `main`. Sie starten den Workflow (`gh workflow run update.yml`) oder legen wie er einen Branch `daten/JJJJ-MM-TT` mit Pull Request an.

## Release

Releases und Tags legen nur Menschen an.

1. **CI prüfen:** Der letzte Lauf auf `main` muss grün sein.
2. **Version festlegen:** Maßgeblich ist die größte Änderung seit dem letzten Release.
   - Patch (`v1.2.1`): neue Daten und Fixes, Format von `output/` unverändert
   - Minor (`v1.3.0`): neue Spalten, Felder oder Dateien in `output/`, die dla-opac-transform nicht anpassen muss
   - Major (`v2.0.0`): Breaking Changes, also umbenannte oder entfernte Spalten oder Dateien oder geänderte Werteformate, die eine Anpassung in dla-opac-transform erfordern
3. **Release anlegen:** „Releases“ → „Draft a new release“, neuen Tag auf `main`, „Generate release notes“. Breaking Changes und nötige Anpassungen in dla-opac-transform ergänzen.
4. **Veröffentlichen:** „Publish release“. Tags werden nie verschoben oder wiederverwendet. Ein fehlerhaftes Release wird durch ein neues Patch-Release ersetzt.

## Deployment

dla-opac-transform lädt `output/` produktiv von einem Release-Tag, nicht von `main`, z. B. `https://github.com/dla-marbach/dla-opac-gnd-enrichment/raw/refs/tags/v1.3.0/output/…`. Zuerst in der Testumgebung, danach mit derselben Version in der Produktion:

1. **Vorbereiten:** Release Notes lesen. Bei Breaking Changes zuerst dla-opac-transform anpassen (eigenes Ticket und PR dort).
2. **Testumgebung:** In dla-opac-transform die neue Version für die Testumgebung setzen, die Transformation laufen lassen und das Ergebnis im Testkatalog stichprobenartig prüfen.
3. **Produktion:** Läuft die Version in der Testumgebung fehlerfrei, dieselbe Version für die Produktion setzen. Sonst in der Testumgebung zurückrollen und den Fix als neues Patch-Release ausliefern.

## Rollback

1. **Vorherige Version setzen:** In dla-opac-transform die zuletzt funktionierende Version setzen und die Transformation erneut laufen lassen, erst in der Testumgebung, dann in der Produktion.
2. **Sicherung:** Die Daten jeder Version bleiben unter ihrem Tag abrufbar, weil Tags nie verschoben werden. Eine eigene Sicherung ist für dieses Repo deshalb nicht nötig. Sicherung und Wiederherstellung des Katalogindex richten sich nach dla-opac-transform.
3. **Beheben:** Fehler per Ticket und Fix-PR beheben und als neues Patch-Release ausliefern.

## Projektspezifisch

- Datenaktualisierungen über den Workflow „Daten aktualisieren“ brauchen kein eigenes Ticket und nutzen Branches `daten/JJJJ-MM-TT`. Begründung: Sie wiederholen sich regelmäßig und sind gleichartig, ähnlich wie Dependabot-PRs. Fällt beim Prüfen etwas auf, wird dafür ein Ticket angelegt.
