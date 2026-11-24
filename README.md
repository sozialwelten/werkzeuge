# werkzeuge

Kleine Kommandozeilenwerkzeuge für den Alltag unter Linux.

| Werkzeug | Zweck | Braucht |
|---|---|---|
| `bestand` | Bestandsaufnahme eines Archivordners: erfasst jede Datei mit Größe, SHA-256 und, wo vorhanden, Aufnahmedatum und Kamera in einer SQLite-Datei, schreibt einen Strukturbericht (Dateiarten, Zeiträume, RAW-JPG-Paare, Doppelungen innerhalb und zwischen Ordnern) und prüft andere Ordner gegen den Katalog. Ein abgebrochener Lauf macht beim nächsten Aufruf weiter. Aufruf siehe unten | `python3`, für die Aufnahmedaten `exiftool` (Paket `libimage-exiftool-perl`) |
| `trialog_abende` | Katalog je Trialog-Abend: ordnet Fotos aus dem Katalog von `bestand`, Abendordner, gescannte Blätter und Beiträge einer WordPress-Sicherung den Abenden zu und listet Tage mit vielen Fotos, die keinem Abend zugeordnet sind. Aufruf siehe unten | `python3` |
| `obsidian_ordnen` | benennt und verschiebt Notizen eines Obsidian-Vaults nach einer Plan-Datei, zieht dabei alle Links nach, wandelt YAML-Köpfe in eine schlichte Zeile um und setzt Hashtags. Zeigt ohne `--ausfuehren` nur an, was geschehen würde, und legt vor dem Ausführen eine Sicherung an. Aufruf siehe unten | `python3` |
| `kontoauswertung` | wertet CSV-Exporte eines Girokontos (Santander/Openbank) aus: erkennt wiederkehrende Zahlungen und zeigt jede im Monat ihrer tatsächlichen Fälligkeit, auch jährliche, rechnet den Kontostand für die kommenden Monate voraus und liefert Tilgungspläne, Kontoverlauf, Ausgaben je Kategorie und nicht zugeordnete Zahlungen. Persönliche Zuordnungen stehen in einer eigenen Regeldatei, ein Muster liegt als `regeln.beispiel.toml` bei. Aufruf siehe unten | `python3` ab 3.11 |
| `notiz` | liest, legt an und ändert Notizen in einem Obsidian-Vault, Text aus der Zwischenablage oder über die Standardeingabe. Gedacht für die Arbeit mit einer Assistenz im Chat: Sie gibt fertige Befehle, der Mensch führt sie aus. `karte` zeigt alle Notizen mit Stand und erstem Satz, `zeige --kopieren` legt Notizen in die Zwischenablage, `-z` liest neuen Text aus ihr und fragt nach einer Prüfung, ob geschrieben wird, `--probe` prüft nur. Vor jeder Änderung eine Sicherung. Aufruf siehe unten | `python3`, für die Zwischenablage `wl-copy` und `wl-paste` (Paket `wl-clipboard`), `xclip` oder `xsel` |
| `post` | durchsucht die lokale Mailablage von Thunderbird nach Absender, Empfänger, Betreff, Inhalt und Zeitraum, zeigt Mails als Text oder legt sie in die Zwischenablage, speichert Anhänge und .eml-Dateien und öffnet fertige Entwürfe mit Text aus der Zwischenablage und Anhängen. Liest nur, braucht keine Zugangsdaten, abgeschickt wird von Hand in Thunderbird. Aufruf siehe unten | `python3` ab 3.9, Thunderbird mit offline gespeicherten Mails, für die Zwischenablage `wl-copy` und `wl-paste` (Paket `wl-clipboard`) |
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
    notiz neu ORDNER/NAME  [-z] [--probe]  < text.md
    notiz ersetze NAME     [-z] [--probe]  < text.md
    notiz aendere NAME     [-z] [--probe]  < blöcke.txt

Den Vault nennt die Umgebungsvariable `OBSIDIAN_VAULT`. `neu` legt keine Ordner an und lehnt vergebene Namen ab. `aendere` liest Blöcke aus `@@@ alt`, altem Text, `@@@ neu`, neuem Text und `@@@ ende`. Jeder alte Text muss genau einmal vorkommen, sonst ändert sich nichts. Sicherungen liegen unter `~/.local/state/notiz/`.

Mit `-z` (`--zwischenablage`) kommt der Text aus der Zwischenablage statt aus der Standardeingabe. So bleibt langer Inhalt aus dem Terminal heraus, in dem Bash beim Einfügen vieler Zeilen die Anzeige durcheinanderbringt. Die Reihenfolge ist wichtig, weil das Kopieren des Befehls die Zwischenablage überschreibt:

1. Den kurzen Befehl kopieren und starten, etwa `notiz aendere "Finanzen" -z`.
2. `notiz` wartet. Jetzt den Inhalt kopieren und Enter drücken. Steht noch der Befehl in der Zwischenablage, sagt `notiz` das und wartet weiter.
3. `notiz` prüft und zeigt eine Übersicht: bei `aendere` für jeden Block, ob er genau einmal passt, bei `neu` und `ersetze` Ziel, Zeilenzahl, erste und letzte Zeile und die Hinweise zu YAML-Kopf, Überschrift, Stand-Zeile und Hashtags.
4. `notiz` fragt `Schreiben? [j/N]`. Nur `j` schreibt.

Passt bei `aendere` ein Block nicht, nennt `notiz` alle fehlerhaften Blöcke und schreibt nichts.

`--probe` macht dieselbe Prüfung und schreibt nie, mit der Standardeingabe wie mit `-z`. Damit lassen sich vorbereitete Änderungen testen, etwa von einer Assistenz, die die Datei nicht vorher gesehen hat.

## post

    post ordner
    post suche [BEGRIFF …] [--von X] [--an X] [--betreff X] [--text X] [--ordner X] [--seit TT.MM.JJJJ] [--bis TT.MM.JJJJ] [-n 20]
    post zeige ID [--kopieren]
    post anhaenge ID [--ziel ORDNER]
    post eml ID [--ziel ORDNER]
    post entwurf --an ADRESSE [--cc ADRESSE] --betreff TEXT [--anhang PFAD …] (-z | --text DATEI)

Gedacht für die Arbeit mit einer Assistenz im Chat, wie `notiz`: Einzelne Mails lassen sich gezielt in die Zwischenablage legen, Entwürfe kommen als Text aus der Zwischenablage zurück. `post` liest die mbox-Dateien von Thunderbird nur und schreibt nie in das Profil. Gefunden werden Profile unter `~/snap/thunderbird/current`, `~/.thunderbird` und im Flatpak-Pfad, ein anderes setzt die Umgebungsvariable `POST_PROFIL`. Konten erscheinen mit ihrem Namen aus `prefs.js`, Ordnernamen in modifiziertem UTF-7 wie `Entw&APw-rfe` lesbar.

Beim ersten Aufruf liest `post` alle Kopfzeilen und legt einen Index unter `~/.cache/post/index.json` an, bei rund einem Gigabyte Mails dauert das ein bis zwei Minuten. Danach liest es nur geänderte Ordner neu, bei neu angehängten Mails nur den neuen Teil. Gelöschte, aber noch nicht komprimierte Mails werden übersprungen.

`suche` zeigt die neuesten Treffer zuerst, jede Mail mit einer sechsstelligen ID, die `zeige`, `anhaenge` und `eml` nehmen. Ohne Filter durchsucht `suche` Von, An, Cc und Betreff, `--text` zusätzlich den Inhalt, das ist langsamer.

`entwurf -z` läuft wie bei `notiz`: erst den Befehl starten, dann den Mailtext kopieren und Enter drücken. `post` prüft, ob die Anhänge existieren, zeigt Empfänger, Betreff, Anhänge und den Anfang des Textes und öffnet den Entwurf in Thunderbird. Text und Kopien der Anhänge legt es dafür in `~/snap/thunderbird/common/post-entwurf` ab, weil ein Snap-Thunderbird weder `/tmp` noch versteckte Ordner lesen kann. Kommas in Dateinamen werden dabei ersetzt, weil Thunderbird sie als Trenner liest, Apostrophe in Betreff und Adressen lehnt `post` ab. Thunderbird startet im Hintergrund ohne Ausgabe, das Terminal bleibt frei.

## ablage_bericht

    ablage_bericht --aus DATEI [--name NAME] [--eingehaengt PFAD] [--verweis NOTIZ] [--probe] ORDNER [ORDNER …]

Schreibt eine Obsidian-Notiz mit der obersten Ebene der angegebenen Ordner: je Eintrag Größe und letzte Änderung, dazu Größe und freier Platz des Laufwerks. Den Stand merkt sich das Programm unter `~/.local/state/ablage_bericht/`. Ab dem zweiten Lauf steht oben, was neu ist, was verschwunden ist und was um mindestens 1 GB und ein Viertel größer oder kleiner wurde. Jede dieser Änderungen kommt außerdem in einen Verlauf am Ende der Notiz, damit sie auch nach weiteren Läufen sichtbar bleibt.

`--eingehaengt` markiert Ordner, die ein eingehängtes Laufwerk sein müssen. Fehlt das Laufwerk, steht oben eine Warnung, und der alte Stand bleibt für den Vergleich erhalten, statt dass alles als verschwunden gilt. `lost+found` wird ausgelassen, die Größen ermittelt `du`. Die Notiz wird bei jedem Lauf ganz ersetzt. `--probe` gibt den Bericht nur aus.

Entstanden, nachdem auf einem Rechner Ordner gelöscht wurden, die eine Notiz danach noch als vorhanden beschrieb. Eine Notiz sagt, wozu die Ordner da sind, der Bericht zeigt, was tatsächlich da ist.

## aliase

Kleine Shell-Aliase, die auf allen Rechnern gelten sollen. Einbinden mit einer Zeile in `~/.bash_aliases`:

    [ -f ~/Arbeitsmappe_Lokal/Code/werkzeuge/aliase ] && . ~/Arbeitsmappe_Lokal/Code/werkzeuge/aliase

Gebraucht werden je nach Alias `nmcli`, `cava`, `arecord`, `toot` und `play` (Paket `sox`).

## trialog_abende

    trialog_abende --db bestand.sqlite --aus abende.md [--katalog scans.csv] [--liste abende.md] [--website ordner] [--dateien zuordnung.csv]

Abende kommen aus Ordnern, deren Name ein Datum JJMMTT trägt, mit oder ohne vorangestelltes `∆`, `∆1`, `∆C` oder `∆Ort` und mit oder ohne Zyklus, etwa `∆1_090624_1.1`, `∆_090708` oder `121017_1.23`, und aus Markdown-Tabellen mit einer Spalte „Datum“; `--liste` lässt sich mehrfach angeben. Abende werden über das Datum zusammengeführt, zwei Abende am selben Tag bleiben getrennt, wenn ihr Zyklus verschieden ist. Weicht das Datum der Liste von dem der Kennung ab, meldet der Bericht das unter „Zu prüfen“, ebenso Nummern wie `#161`, die nicht zur Reihenfolge der Daten passen. Abende, die nicht aus der Liste stammen, werden einem Nachbarn zugeschlagen, wenn derselbe Zyklus innerhalb von drei Wochen schon vorkommt; Abende aus Ordnernamen auch am selben Tag oder ohne Zyklus und ohne eigene Fotos am Tag nach einem Abend. Auch das steht unter „Zu prüfen“. `--website` liest eine WordPress-Sicherung als JSON (`posts_*.json` und `categories_*.json`, etwa aus `/wp-json/wp/v2/posts`) und übernimmt je Beitrag Kennung, Nummer, Bildtitel, Ort und Link. Passt das Datum einer Kennung nicht zur Veröffentlichung, wird der Abend über die Adresse des Beitrags oder ein berichtigtes Jahr eingeordnet und gemeldet, ebenso Nummern, die auf der Website doppelt vergeben sind. Fotos zählen zu einem Abend, wenn sie ein Kameramodell in den Bilddaten haben und zwischen 12 Uhr und 6 Uhr am Morgen danach aufgenommen sind, gleiche Inhalte einmal. Scanner nimmt `--ohne-kamera` aus, voreingestellt ist `oj_g510n`. Aus einem Scankatalog kommen Blätter mit passendem Datum oder passender Kennung dazu, vermutete Daten werden als unsicher markiert. Der Bericht enthält nur Struktur und Titel. `--dateien` schreibt alle Zuordnungen als CSV.

## Einrichtung

    git clone git@github.com:sozialwelten/werkzeuge.git
    cd werkzeuge
    ./einrichten

`einrichten` legt für jedes Werkzeug einen Verweis in `~/bin` an. Debian und Ubuntu nehmen `~/bin` beim Anmelden automatisch in den Suchpfad auf. Wurde `~/bin` gerade erst angelegt, einmal ab- und wieder anmelden.

Neue Werkzeuge: Datei ohne Endung in diesen Ordner legen, mit `chmod +x` ausführbar machen, committen und `./einrichten` erneut ausführen.

## Lizenz

- `pdf_reduktion`: shrinkpdf von Alfred Klomp, angepasst von Vivek Gite, BSD-3-Clause (Lizenztext in der Datei)
- `bestand`, `obsidian_ordnen`, `kontoauswertung`, `notiz`, `post`, `cr2`, `einrichten`, `aliase`, `trialog_abende`: GPL-3.0-or-later, Lizenztext in `LICENSE`
