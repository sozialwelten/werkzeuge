# werkzeuge

Kleine Kommandozeilenwerkzeuge für den Alltag unter Linux.

| Werkzeug | Zweck | Braucht |
|---|---|---|
| `bestand` | Bestandsaufnahme eines Archivordners: erfasst jede Datei mit Größe, SHA-256 und, wo vorhanden, Aufnahmedatum und Kamera in einer SQLite-Datei, schreibt einen Strukturbericht (Dateiarten, Zeiträume, RAW-JPG-Paare, Doppelungen innerhalb und zwischen Ordnern) und prüft andere Ordner gegen den Katalog. Ein abgebrochener Lauf macht beim nächsten Aufruf weiter. Aufruf siehe unten | `python3`, für die Aufnahmedaten `exiftool` (Paket `libimage-exiftool-perl`) |
| `cr2` | wandelt alle CR2-Rohdateien im aktuellen Ordner in JPEG um, vorhandene JPEGs bleiben unangetastet | `darktable-cli` (Paket `darktable`) |
| `pdf_reduktion` | verkleinert eine PDF durch verlustbehaftete Neukomprimierung: `pdf_reduktion eingabe.pdf ausgabe.pdf [dpi]`, Standard 90 dpi | `gs` (Paket `ghostscript`) |

## bestand

    bestand erfassen  ORDNER --db katalog.sqlite
    bestand bericht   --db katalog.sqlite --aus bericht.md [--tiefe 2] [--beispiele 30]
    bestand vergleich ANDERER_ORDNER --db katalog.sqlite --aus vergleich.md

`erfassen` liest jede Datei einmal ganz, um die Prüfsumme zu bilden; bei einer Festplatte sind das grob 100 bis 150 MB/s. Ein zweiter Lauf erfasst nur neue oder geänderte Dateien und entfernt verschwundene aus dem Katalog. Die Katalogdatei gehört neben den erfassten Ordner, nicht hinein.

`bericht` gibt nur Struktur aus, keine Dateiinhalte: je Ordner bis zur gewählten Tiefe Dateizahl, Größe, häufigste Dateiarten, Zeitraum, Kameramodelle, RAW-JPG-Paare und Doppelungen, dazu eine Tabelle der Ordnerpaare mit gemeinsamen Inhalten, die größten Doppelungen mit Pfaden und eine Übersicht der Aufnahmen nach Monat, in der Tage mit vielen Aufnahmen hervorgehoben sind. Zeiträume stammen aus den Kameradaten; fehlen sie, aus dem Änderungsdatum der Datei, dann mit * markiert, weil das bei Kopien oft das Kopierdatum ist.

`vergleich` bildet die Prüfsummen eines anderen Ordners und meldet, welche Dateien inhaltsgleich im Katalog liegen und wo, und welche fehlen.

## Einrichtung

    git clone git@github.com:sozialwelten/werkzeuge.git
    cd werkzeuge
    ./einrichten

`einrichten` legt für jedes Werkzeug einen Verweis in `~/bin` an. Debian und Ubuntu nehmen `~/bin` beim Anmelden automatisch in den Suchpfad auf. Wurde `~/bin` gerade erst angelegt, einmal ab- und wieder anmelden.

Neue Werkzeuge: Datei ohne Endung in diesen Ordner legen, mit `chmod +x` ausführbar machen, committen und `./einrichten` erneut ausführen.

## Lizenz

- `pdf_reduktion`: shrinkpdf von Alfred Klomp, angepasst von Vivek Gite, BSD-3-Clause (Lizenztext in der Datei)
- `bestand`, `cr2`, `einrichten`: GPL-3.0-or-later, Lizenztext in `LICENSE`
