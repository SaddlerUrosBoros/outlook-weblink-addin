# Outlook Web-Link – Add-in

Ein Button in Outlook (neues Outlook für Mac, Outlook im Web, Windows), der zur geöffneten Mail bzw. zum geöffneten Termin einen Link für Outlook im Web erzeugt. Im Bereich gibt es „Link kopieren“, „Als Markdown-Link kopieren“ und „In Outlook im Web öffnen“.

## Inhalt

- `manifest.xml`: die Beschreibung des Add-ins für Outlook
- `taskpane.html`: die eigentliche Oberfläche und Logik
- `assets/`: Icons

## 1. Dateien per HTTPS bereitstellen (GitHub Pages)

Outlook lädt Add-ins nur von einer HTTPS-Adresse. Am einfachsten geht das mit GitHub Pages:

1. Auf github.com ein neues **öffentliches** Repository anlegen, z. B. `outlook-weblink`.
2. `taskpane.html` und den Ordner `assets/` hochladen (die `manifest.xml` muss nicht ins Repo).
3. Im Repository unter *Settings → Pages* bei „Branch“ `main` und `/ (root)` wählen und speichern.
4. Nach etwa einer Minute ist die Seite erreichbar unter `https://DEIN-NAME.github.io/outlook-weblink/taskpane.html`. Einmal im Browser aufrufen und prüfen.

Die Seite enthält keine Daten von dir. Sie läuft nur innerhalb von Outlook und liest dort die ID des geöffneten Elements.

## 2. Manifest anpassen

In `manifest.xml` jedes `https://BASIS-URL` ersetzen durch deine Adresse **ohne** Schrägstrich am Ende, also z. B. `https://deinname.github.io/outlook-weblink`. Im Terminal:

```bash
sed -i '' 's#https://BASIS-URL#https://deinname.github.io/outlook-weblink#g' manifest.xml
```

## 3. In Outlook installieren

1. Im Browser `https://aka.ms/olksideload` öffnen. Das öffnet Outlook im Web mit dem Dialog für Add-ins.
2. *Meine Add-ins* → ganz unten *Benutzerdefinierte Add-ins* → *Benutzerdefiniertes Add-in hinzufügen* → *Aus Datei hinzufügen* → `manifest.xml` wählen.
3. Das Add-in synchronisiert sich ins neue Outlook für Mac. Gegebenenfalls Outlook einmal neu starten.

Fehlt der Punkt „Benutzerdefiniertes Add-in hinzufügen“, hat die IT das Hinzufügen eigener Add-ins gesperrt. Dann kann ein Admin das Manifest zentral bereitstellen.

## 4. Benutzen

Mail oder Termin öffnen, dann in der Symbolleiste bzw. im „…“-Menü („Apps“) auf **Web-Link** klicken. Mit der Stecknadel lässt sich der Bereich bei Mails anheften. Er aktualisiert sich dann beim Wechsel der Mail automatisch.

## Wenn ein Link nicht öffnet

Unter *Einstellungen* im Bereich lässt sich zwischen zwei Link-Formaten und drei Outlook-Adressen wechseln. Die Auswahl wird gespeichert.

- **Klassisch** entspricht dem Format, das Microsoft Graph als `webLink` liefert. Das ist die Voreinstellung.
- **Modern** nutzt die neueren Pfade `/mail/deeplink/read/…` bzw. `/calendar/item/…`.

Hinweise: Der Link funktioniert nur für Personen mit Zugriff auf dasselbe Postfach, in der Regel also nur für dich selbst. Wird eine Mail verschoben, bleibt die ID meist gültig. Bei Serienterminen zeigt der Link auf das jeweils geöffnete Vorkommen.

## Änderungen

Bei Änderungen an `taskpane.html` reicht es, die Datei im Repo zu aktualisieren. Bei Änderungen am Manifest die `<Version>` erhöhen, das Add-in entfernen und neu hinzufügen.
