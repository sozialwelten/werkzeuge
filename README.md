# werkzeuge

Kleine Kommandozeilenwerkzeuge für den Alltag unter Linux.

| Werkzeug | Zweck | Braucht |
|---|---|---|
| `bestand` | Bestandsaufnahme eines Archivordners: erfasst jede Datei mit Größe, SHA-256 und, wo vorhanden, Aufnahmedatum und Kamera in einer SQLite-Datei, schreibt einen Strukturbericht (Dateiarten, Zeiträume, RAW-JPG-Paare, Doppelungen innerhalb und zwischen Ordnern) und prüft andere Ordner gegen den Katalog. Ein abgebrochener Lauf macht beim nächsten Aufruf weiter. Aufruf siehe unten | `python3`, für die Aufnahmedaten `exiftool` (Paket `libimage-exiftool-perl`) |
| `obsidian_ordnen` | benennt und verschiebt Notizen eines Obsidian-Vaults nach einer Plan-Datei, zieht dabei alle Links nach, wandelt YAML-Köpfe in eine schlichte Zeile um und setzt Hashtags. Zeigt ohne `--ausfuehren` nur an, was geschehen würde, und legt vor dem Ausführen eine Sicherung an. Aufruf siehe unten | `python3` |
| `kontoauswertung` | wertet CSV-Exporte eines Girokontos (Santander/Openbank) aus: erkennt wiederkehrende Zahlungen und zeigt jede im Monat ihrer tatsächlichen Fälligkeit, auch jährliche, rechnet den Kontostand für die kommenden Monate voraus und liefert Tilgungspläne, Kontoverlauf, Ausgaben je Kategorie und nicht zugeordnete Zahlungen. Persönliche Zuordnungen stehen in einer eigenen Regeldatei, ein Muster liegt als `regeln.beispiel.toml` bei. Aufruf siehe unten | `python3` ab 3.11 |
| `cr2` | wandelt alle CR2-Rohdateien im aktuellen Ordner in JPEG um, vorhandene JPEGs bleiben unangetastet | `darktable-cli` (Paket `darktable`) |
| `pdf_reduktion` | verkleinert eine PDF durch verlustbehaftete Neukomprimierung: `pdf_reduktion eingabe.pdf ausgabe.pdf [dpi]`, Standard 90 dpi | `gs` (Paket `ghostscript`) |

## bestand

    bestand erfassen  ORDNER --db katalog.sqlite
    bestand bericht   --db katalog.sqlite --aus bericht.md [--tiefe 2] [--beispiele 30]
    bestand vergleich ANDERER_ORDNER --db katalog.sqlite --aus vergleich.md

`erfassen` liest jede Datei einmal ganz, um die Prüfsumme zu bilden; bei einer Festplatte sind das grob 100 bis 150 MB/s. Ein zweiter Lauf erfasst nur neue oder geänderte Dateien und entfernt verschwundene aus dem Katalog. Die Katalogdatei gehört neben den erfassten Ordner, nicht hinein.

`bericht` gibt nur Struktur aus, keine Dateiinhalte: je Ordner bis zur gewählten Tiefe Dateizahl, Größe, häufigste Dateiarten, Zeitraum, Kameramodelle, RAW-JPG-Paare und Doppelungen, dazu eine Tabelle der Ordnerpaare mit gemeinsamen Inhalten, die größten Doppelungen mit Pfaden und eine Übersicht der Aufnahmen nach Monat, in der Tage mit vielen Aufnahmen hervorgehoben sind. Zeiträume stammen aus den Kameradaten; fehlen sie, aus dem Änderungsdatum der Datei, dann mit * markiert, weil das bei Kopien oft das Kopierdatum ist.

`vergleich` bildet die Prüfsummen eines anderen Ordners und meldet, welche Dateien inhaltsgleich im Katalog liegen und wo, und welche fehlen.

## obsidian_ordnen

    obsidian_ordnen VAULT PLAN.tsv                 Vorschau
    obsidian_ordnen VAULT PLAN.tsv --ausfuehren    Sicherung, dann ausführen

Die Plan-Datei hat je Notiz eine Zeile mit drei durch Tabulatoren getrennten Spalten: alter Pfad, neuer Pfad (leer heißt unverändert) und Hashtags, Pfade relativ zum Vault und ohne `.md`. Links der Formen `[[Name]]`, `[[Name|Text]]`, `[[Name#Abschnitt]]`, `![[Name]]` und `[[Ordner/Name]]` werden auf den neuen Namen umgestellt, Dateinamen in Backticks wie `` `name.md` `` werden zu `[[neuer Name]]`. Ein YAML-Kopf wird entfernt, Termin, Stand, Beteiligte und Quelle bleiben als eine Zeile unter der Überschrift. Die Hashtags stehen danach in der letzten Zeile; eine vorhandene Hashtag-Zeile am Ende wird ergänzt. Der Plan wird vorher geprüft: fehlende Dateien, doppelte Ziele und doppelte Dateinamen brechen ab, bevor etwas geändert wird. Obsidian sollte währenddessen geschlossen sein.

## kontoauswertung

    kontoauswertung CSV-ORDNER --regeln regeln.toml --aus bericht.md [--monate 12]

Liest alle CSV-Exporte im Ordner, auch überlappende, und schreibt einen Markdown-Bericht. Die Regeldatei vergibt lesbare Namen und Kategorien, ergänzt Verträge, die das Programm nicht selbst erkennt, und trägt Termine und Kredite ein. Sie enthält persönliche Daten und gehört deshalb nicht in dieses Repo, sondern zu den Kontoauszügen.

## aliase

Kleine Shell-Aliase, die auf allen Rechnern gelten sollen. Einbinden mit einer Zeile in `~/.bash_aliases`:

    [ -f ~/Arbeitsmappe_Lokal/Code/werkzeuge/aliase ] && . ~/Arbeitsmappe_Lokal/Code/werkzeuge/aliase

Gebraucht werden je nach Alias `nmcli`, `cava`, `arecord`, `toot` und `play` (Paket `sox`).

## Einrichtung

    git clone git@github.com:sozialwelten/werkzeuge.git
    cd werkzeuge
    ./einrichten

`einrichten` legt für jedes Werkzeug einen Verweis in `~/bin` an. Debian und Ubuntu nehmen `~/bin` beim Anmelden automatisch in den Suchpfad auf. Wurde `~/bin` gerade erst angelegt, einmal ab- und wieder anmelden.

Neue Werkzeuge: Datei ohne Endung in diesen Ordner legen, mit `chmod +x` ausführbar machen, committen und `./einrichten` erneut ausführen.

## Lizenz

- `pdf_reduktion`: shrinkpdf von Alfred Klomp, angepasst von Vivek Gite, BSD-3-Clause (Lizenztext in der Datei)
- `bestand`, `obsidian_ordnen`, `kontoauswertung`, `cr2`, `einrichten`, `aliase`: GPL-3.0-or-later, Lizenztext in `LICENSE`
