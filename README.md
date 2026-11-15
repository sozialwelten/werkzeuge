# werkzeuge

Kleine Kommandozeilenwerkzeuge für den Alltag unter Linux.

| Werkzeug | Zweck | Braucht |
|---|---|---|
| `cr2` | wandelt alle CR2-Rohdateien im aktuellen Ordner in JPEG um, vorhandene JPEGs bleiben unangetastet | `darktable-cli` (Paket `darktable`) |
| `pdf_reduktion` | verkleinert eine PDF durch verlustbehaftete Neukomprimierung: `pdf_reduktion eingabe.pdf ausgabe.pdf [dpi]`, Standard 90 dpi | `gs` (Paket `ghostscript`) |

## Einrichtung

    git clone git@github.com:sozialwelten/werkzeuge.git
    cd werkzeuge
    ./einrichten

`einrichten` legt für jedes Werkzeug einen Verweis in `~/bin` an. Debian und Ubuntu nehmen `~/bin` beim Anmelden automatisch in den Suchpfad auf. Wurde `~/bin` gerade erst angelegt, einmal ab- und wieder anmelden.

Neue Werkzeuge: Datei ohne Endung in diesen Ordner legen, mit `chmod +x` ausführbar machen, committen und `./einrichten` erneut ausführen.

## Lizenz

- `pdf_reduktion`: shrinkpdf von Alfred Klomp, angepasst von Vivek Gite, BSD-3-Clause (Lizenztext in der Datei)
- `cr2`, `einrichten`: GPL-3.0-or-later
