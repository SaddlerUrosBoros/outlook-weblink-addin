# Projekt: Outlook Web-Link Add-in

Kontext für Claude Code. Dieses Projekt wurde in einem claude.ai-Chat begonnen und hier übernommen.

## Ziel

Im **neuen Outlook für Mac** (Client) soll man zu einer geöffneten Mail bzw. einem geöffneten Termin per Klick den Deep Link für **Outlook im Web** bekommen, also den Link, den man sonst nur im Browser erhält. Ausgabe: Link in die Zwischenablage, optional als Markdown-Link mit dem Betreff als Text.

## Warum ein Office-Add-in

- Das neue Outlook für Mac hat praktisch keine AppleScript/JXA-Unterstützung (`selected objects` liefert nichts). Microsoft hat das Roadmap-Feature RM88537 („Support for AppleScript in the New Outlook for Mac“) gestrichen. Ein Skript per Hotkey scheidet damit aus.
- Office-Add-ins (Office.js) laufen auch im neuen Outlook für Mac und sehen das geöffnete Element. Das war die gewählte Variante.
- Verworfene Alternative: Microsoft Graph (`webLink`-Property). Dafür wäre eine App-Registrierung in Entra ID nötig, und das Skript kennt die Auswahl in Outlook nicht.

## Aufbau

| Datei | Zweck |
|---|---|
| `manifest.xml` | XML-Manifest (MailApp, VersionOverrides 1.1). Button auf `MessageReadCommandSurface`, `AppointmentAttendeeCommandSurface`, `AppointmentOrganizerCommandSurface`. Permission `ReadItem`, Mindestversion Mailbox 1.3. Add-in-GUID: `49261295-5ad5-46ed-a26b-1777da03c689` (beibehalten). |
| `taskpane.html` | Komplette UI und Logik inline (kein Build-Schritt). Lädt `office.js` vom Microsoft-CDN. |
| `assets/icon-{16,32,64,80,128}.png` | Icons (blaues Quadrat mit Kettengliedern). |
| `README.md` | Einrichtungsanleitung für den Nutzer, auf Deutsch. |

### Logik in `taskpane.html`

1. ID holen: `item.itemId` (Lesemodus) oder `item.getItemIdAsync` (Organisator, Mailbox 1.8). Ist das Element ungespeichert, wird ein Fehler angezeigt.
2. `Office.context.mailbox.convertToRestId(id, v2_0)`. Fallback: `/`→`-`, `+`→`_`.
3. Link bauen, Format umschaltbar:
   - **owa** (Default, entspricht dem Format von Graph `webLink`):
     Mail `{host}/owa/?ItemID={id}&exvsurl=1&viewmodel=ReadMessageItem`,
     Termin `{host}/owa/?itemid={id}&exvsurl=1&path=/calendar/item`
   - **modern**: Mail `{host}/mail/deeplink/read/{id}`, Termin `{host}/calendar/item/{id}`
   - `host`: `outlook.office.com` (Default), `outlook.office365.com` oder `outlook.cloud.microsoft`
4. Einstellungen landen in `Office.context.roamingSettings`.
5. Kopieren erst über `navigator.clipboard`, als Fallback `execCommand('copy')`. Öffnen über `Office.context.ui.openBrowserWindow`.
6. Handler für `ItemChanged`, damit ein angeheftetes Taskpane mitläuft.

## Platzhalter

`manifest.xml` enthält überall `https://BASIS-URL`. Das muss durch die echte HTTPS-Basis-URL ersetzt werden, ohne Schrägstrich am Ende, z. B. `https://<user>.github.io/<repo>`.

## Hosting

Geplant ist GitHub Pages aus dem angebundenen Repo (Branch `main`, Root). Ausgeliefert werden `taskpane.html` und `assets/`; das Manifest darf mit im Repo liegen. `.nojekyll` anlegen.

## Aufgaben für Claude Code

1. Dateien ins Repo übernehmen, sinnvoller Commit.
2. GitHub-Pages-URL ermitteln (Repo-Owner/Name), Platzhalter in `manifest.xml` ersetzen.
3. GitHub Pages aktivieren, falls noch nicht aktiv (z. B. `gh api -X POST repos/{owner}/{repo}/pages -f "source[branch]=main" -f "source[path]=/"`), und prüfen, ob `taskpane.html` und die Icons per HTTPS erreichbar sind.
4. Optional: Manifest mit `npx office-addin-manifest validate manifest.xml` prüfen.
5. Dem Nutzer das fertige `manifest.xml` nennen und die Installation beschreiben: `https://aka.ms/olksideload` → Meine Add-ins → Benutzerdefiniertes Add-in hinzufügen → Aus Datei.

## Offene Punkte / noch ungetestet

- Welches Link-Format (owa/modern) und welcher Host im Tenant des Nutzers zuverlässig öffnen. Nach dem Test als Default festlegen.
- Ob `navigator.clipboard` im WebView des neuen Outlook für Mac funktioniert, sonst greift der Fallback.
- Ob der Organisator-Modus ein Element schon mit ID liefert.
- Ob der Tenant benutzerdefinierte Add-ins erlaubt. Falls nicht, ist eine zentrale Bereitstellung durch die IT nötig.
- Bei Manifest-Änderungen `<Version>` erhöhen und das Add-in neu hinzufügen.

## Konventionen

- UI-Texte und Doku auf Deutsch.
- Kein Build-Tooling, alles bleibt statisch und in einer HTML-Datei.
- Light- und Dark-Mode über CSS-Variablen.
