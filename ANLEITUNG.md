# Trainingsplan als iPhone-App einrichten

Einmalig ca. 15 Minuten. Danach liegt die App mit eigenem Icon auf dem Home-Bildschirm und läuft offline.

## Schritt 1 — GitHub-Konto

Auf github.com registrieren, falls noch nicht vorhanden. Kostenlos, keine Zahlungsdaten nötig.

## Schritt 2 — Repository anlegen

1. Oben rechts auf **+** → **New repository**
2. Name: `trainingsplan`
3. Sichtbarkeit: **Public** (bei Private funktioniert GitHub Pages im kostenlosen Tarif nicht)
4. **Create repository**

## Schritt 3 — Dateien hochladen

1. Im leeren Repository auf **uploading an existing file** klicken
2. Diese sieben Dateien hochladen — **einzeln, nicht als ZIP-Ordner**:
   - `index.html`
   - `manifest.webmanifest`
   - `sw.js`
   - `apple-touch-icon.png`
   - `icon-192.png`
   - `icon-512.png`
   - `icon-512-maskable.png`
3. Unten auf **Commit changes**

Die Datei `ANLEITUNG.md` musst du nicht hochladen, sie stört aber auch nicht.

## Schritt 4 — GitHub Pages aktivieren

1. Im Repository auf **Settings** (oben rechts)
2. Links im Menü auf **Pages**
3. Unter *Source*: **Deploy from a branch**
4. Branch: **main**, Ordner: **/ (root)** → **Save**
5. Ein bis zwei Minuten warten, dann die Seite neu laden

Oben erscheint die Adresse, etwa:
`https://DEINNAME.github.io/trainingsplan/`

## Schritt 5 — Auf dem iPhone installieren

1. Adresse in **Safari** öffnen (nicht Chrome — nur Safari kann auf iOS installieren)
2. Auf das **Teilen-Symbol** unten in der Mitte tippen
3. Nach unten scrollen → **Zum Home-Bildschirm**
4. Name bestätigen → **Hinzufügen**

Fertig. Das Icon liegt jetzt auf dem Home-Bildschirm, die App startet im Vollbild ohne Safari-Leisten.

## Offline-Betrieb

Beim ersten Start lädt der Service Worker alle Dateien in den Gerätespeicher. Danach funktioniert die App ohne Internetverbindung — auch im Flugmodus.

## Änderungen einspielen

Neue Dateiversion im Repository hochladen (gleicher Dateiname überschreibt), dann in `sw.js` die Zeile

```
const CACHE = "trainingsplan-v1";
```

auf `v2`, `v3` und so weiter hochzählen. Ohne diese Änderung zeigt das iPhone weiter die alte, zwischengespeicherte Version.

## Falls es nicht klappt

- **Weiße Seite:** Ein bis zwei Minuten warten, GitHub Pages braucht nach dem ersten Aktivieren etwas Zeit.
- **Icon fehlt:** Prüfen, ob `apple-touch-icon.png` wirklich im Hauptverzeichnis liegt und nicht in einem Unterordner.
- **Kein Ton:** Der erste Signalton kommt erst nach dem ersten Tippen auf einen Button. Das ist eine Schutzregel aller Browser, kein Fehler.
- **Bildschirm geht aus:** Wake Lock greift nur, wenn die Seite über HTTPS läuft — über GitHub Pages ist das der Fall.
