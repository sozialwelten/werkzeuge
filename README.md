# werkzeuge

Kleine Kommandozeilenwerkzeuge für den Alltag unter Linux.

| Werkzeug | Zweck | Braucht |
|---|---|---|
| `bestand` | Bestandsaufnahme eines Archivordners: erfasst jede Datei mit Größe, SHA-256 und, wo vorhanden, Aufnahmedatum und Kamera in einer SQLite-Datei, schreibt einen Strukturbericht (Dateiarten, Zeiträume, RAW-JPG-Paare, Doppelungen innerhalb und zwischen Ordnern) und prüft andere Ordner gegen den Katalog. Ein abgebrochener Lauf macht beim nächsten Aufruf weiter. Aufruf siehe unten | `python3`, für die Aufnahmedaten `exiftool` (Paket `libimage-exiftool-perl`) |
| `trialog_abende` | Katalog je Trialog-Abend: ordnet Fotos aus dem Katalog von `bestand`, Abendordner und gescannte Blätter den Abenden zu und listet Tage mit vielen Fotos, die keinem Abend zugeordnet sind. Aufruf siehe unten | `python3` |
| `obsidian_ordnen` | benennt und verschiebt Notizen eines Obsidian-Vaults nach einer Plan-Datei, zieht dabei alle Links nach, wandelt YAML-Köpfe in eine schlichte Zeile um und setzt Hashtags. Zeigt ohne `--ausfuehren` nur an, was geschehen würde, und legt vor dem Ausführen eine Sicherung an. Aufruf siehe unten | `python3` |
| `kontoauswertung` | wertet CSV-Exporte eines Girokontos (Santander/Openbank) aus: erkennt wiederkehrende Zahlungen und zeigt jede im Monat ihrer tatsächlichen Fälligkeit, auch jährliche, rechnet den Kontostand für die kommenden Monate voraus und liefert Tilgungspläne, Kontoverlauf, Ausgaben je Kategorie und nicht zugeordnete Zahlungen. Persönliche Zuordnungen stehen in einer eigenen Regeldatei, ein Muster liegt als `regeln.beispiel.toml` bei. Aufruf siehe unten | `python3` ab 3.11 |
| `notiz` | liest, legt an und ändert Notizen in einem Obsidian-Vault, Text über die Standardeingabe. Gedacht für die Arbeit mit einer Assistenz im Chat: Sie gibt fertige Befehle, der Mensch führt sie aus. `karte` zeigt alle Notizen mit Stand und erstem Satz, `zeige --kopieren` legt Notizen in die Zwischenablage. Vor jeder Änderung eine Sicherung. Aufruf siehe unten | `python3`, für `--kopieren` `wl-copy` (Paket `wl-clipboard`) oder `xclip` |
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

Die Plan-Datei hat je Notiz eine Zeile mit drei durch Tabulatoren getrennten Spalten: alter Pfad, neuer Pfad (leer heißt unverändert) und Hashtags, Pfade relativ zum Vault und ohne `.md`. Links der Formen `[[Name]]`, `[[Name|Text]]`, `[[Name#Abschnitt]]`, `![[Name]]` und `[[Ordner/Name]]` werden auf den neuen Namen umgestellt, Dateinamen in Backticks wie `` `name.md` `` werden zu `[[neuer Name]]`. Ein YAML-Kopf wird entfernt, Termin, Stand, Beteiligte und Quelle bleiben als eine Zeile am Anfang der Notiz, bei einer vorhandenen Überschrift direkt darunter. Die Hashtags stehen danach in der letzten Zeile; eine vorhandene Hashtag-Zeile am Ende wird ergänzt. Der Plan wird vorher geprüft: fehlende Dateien, doppelte Ziele und doppelte Dateinamen brechen ab, bevor etwas geändert wird. Obsidian sollte währenddessen geschlossen sein.

## kontoauswertung

    kontoauswertung CSV-ORDNER --regeln regeln.toml --aus bericht.md [--monate 12]

Liest alle CSV-Exporte im Ordner, auch überlappende, und schreibt einen Markdown-Bericht. Die Regeldatei vergibt lesbare Namen und Kategorien, ergänzt Verträge, die das Programm nicht selbst erkennt, und trägt Termine und Kredite ein. Sie enthält persönliche Daten und gehört deshalb nicht in dieses Repo, sondern zu den Kontoauszügen.

## notiz

    notiz liste [ORDNER]
    notiz karte [ORDNER]
    notiz zeige NAME [NAME …] [--kopieren]
    notiz neu ORDNER/NAME  < text.md
    notiz ersetze NAME     < text.md
    notiz aendere NAME     < blöcke.txt

Den Vault nennt die Umgebungsvariable `OBSIDIAN_VAULT`. `neu` legt keine Ordner an und lehnt vergebene Namen ab. `aendere` liest Blöcke aus `@@@ alt`, altem Text, `@@@ neu`, neuem Text und `@@@ ende`. Jeder alte Text muss genau einmal vorkommen, sonst ändert sich nichts. Sicherungen liegen unter `~/.local/state/notiz/`.

## aliase

Kleine Shell-Aliase, die auf allen Rechnern gelten sollen. Einbinden mit einer Zeile in `~/.bash_aliases`:

    [ -f ~/Arbeitsmappe_Lokal/Code/werkzeuge/aliase ] && . ~/Arbeitsmappe_Lokal/Code/werkzeuge/aliase

Gebraucht werden je nach Alias `nmcli`, `cava`, `arecord`, `toot` und `play` (Paket `sox`).

## trialog_abende

    trialog_abende --db bestand.sqlite --aus abende.md [--katalog scans.csv] [--liste abende.md] [--dateien zuordnung.csv]

Abende kommen aus Ordnern, deren Name ein Datum JJMMTT trägt, mit oder ohne vorangestelltes `∆`, `∆1`, `∆C` oder `∆Ort` und mit oder ohne Zyklus, etwa `∆1_090624_1.1`, `∆_090708` oder `121017_1.23`, und aus Markdown-Tabellen mit einer Spalte „Datum“; `--liste` lässt sich mehrfach angeben. Abende werden über das Datum zusammengeführt, zwei Abende am selben Tag bleiben getrennt, wenn ihr Zyklus verschieden ist. Weicht das Datum der Liste von dem der Kennung ab, meldet der Bericht das unter „Zu prüfen“, ebenso Nummern wie `#161`, die nicht zur Reihenfolge der Daten passen. Abende, die nur aus Ordnernamen stammen, werden einem Nachbarn zugeschlagen: am selben Tag, bei gleichem Zyklus in derselben Woche oder ohne Zyklus und ohne eigene Fotos am Tag nach einem Abend. Auch das steht unter „Zu prüfen“. Fotos zählen zu einem Abend, wenn sie ein Kameramodell in den Bilddaten haben und zwischen 12 Uhr und 6 Uhr am Morgen danach aufgenommen sind, gleiche Inhalte einmal. Scanner nimmt `--ohne-kamera` aus, voreingestellt ist `oj_g510n`. Aus einem Scankatalog kommen Blätter mit passendem Datum oder passender Kennung dazu, vermutete Daten werden als unsicher markiert. Der Bericht enthält nur Struktur und Titel. `--dateien` schreibt alle Zuordnungen als CSV.

## Einrichtung

    git clone git@github.com:sozialwelten/werkzeuge.git
    cd werkzeuge
    ./einrichten

`einrichten` legt für jedes Werkzeug einen Verweis in `~/bin` an. Debian und Ubuntu nehmen `~/bin` beim Anmelden automatisch in den Suchpfad auf. Wurde `~/bin` gerade erst angelegt, einmal ab- und wieder anmelden.

Neue Werkzeuge: Datei ohne Endung in diesen Ordner legen, mit `chmod +x` ausführbar machen, committen und `./einrichten` erneut ausführen.

## Lizenz

- `pdf_reduktion`: shrinkpdf von Alfred Klomp, angepasst von Vivek Gite, BSD-3-Clause (Lizenztext in der Datei)
- `bestand`, `obsidian_ordnen`, `kontoauswertung`, `notiz`, `cr2`, `einrichten`, `aliase`, `trialog_abende`: GPL-3.0-or-later, Lizenztext in `LICENSE`
