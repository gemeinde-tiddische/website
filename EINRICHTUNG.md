# Einrichtung & technische Betreuung

**Für: die technisch betreuende Person (aktuell oder zukünftig).**
Dieses Dokument beschreibt die einmalige Einrichtung der Website und alle
wiederkehrenden Wartungsaufgaben. Ziel: Jede technisch versierte Person
(oder Agentur) kann die Betreuung ohne Vorwissen übernehmen.

## Überblick: Wie die Website funktioniert

| Baustein | Dienst | Kosten | Zweck |
|---|---|---|---|
| Quellcode & Inhalte | GitHub (öffentliches Repository) | 0 € | Speichert alles; jede Änderung ist protokolliert |
| Website-Generator | Hugo (Version in `netlify.toml` festgepinnt) | 0 € | Baut aus den Inhalten statische HTML-Seiten |
| Hosting | Netlify (Free Plan) | 0 € | Baut bei jeder Änderung automatisch neu und liefert die Seiten aus |
| Inhaltsverwaltung | Sveltia CMS (`/admin/`) | 0 € | Web-Oberfläche für Redakteure, schreibt direkt ins Repository |
| CMS-Anmeldung | Cloudflare Worker (sveltia-cms-auth) | 0 € | Vermittelt den GitHub-Login für das CMS |
| Domain | gemeinde-tiddische.de (+ gemeinde-hoitlingen.de) | ~10 €/Jahr | Einziger Kostenpunkt; zahlt die Gemeinde |

Es gibt **keinen Server, keine Datenbank, keine Updates-Pflicht**. Die
Website besteht aus statischen Dateien.

---

## A. Einmalige Einrichtung (ca. 1–2 Stunden)

### 1. GitHub-Organisation anlegen

1. Auf github.com ein **persönliches Konto** für die Gemeinde-Verwaltung
   anlegen (falls nicht vorhanden) – oder das eigene nutzen, um die Org zu
   erstellen (Eigentümerschaft kann später übertragen werden).
2. **Organisation erstellen**: github.com/organizations/plan → Free →
   Name: `gemeinde-tiddische`.
3. **Wichtig (Grundsatz „funktioniert ohne Einzelperson"):** Mindestens
   **zwei Personen als Owner** eintragen, z. B. die betreuende Person und
   ein Konto, dessen Zugangsdaten versiegelt im Gemeindebüro hinterlegt sind.
4. Repository anlegen: `gemeinde-tiddische/website`, **öffentlich**.
5. Dieses Projekt hochladen:
   ```bash
   cd website-projekt
   git init && git add . && git commit -m "Neue Website der Gemeinde Tiddische"
   git branch -M main
   git remote add origin https://github.com/gemeinde-tiddische/website.git
   git push -u origin main
   ```

### 2. Netlify einrichten

1. Auf netlify.com ein Konto anlegen – **per „Sign up with GitHub"** mit
   einem Konto der Organisation.
2. **Add new site → Import an existing project → GitHub** →
   `gemeinde-tiddische/website` auswählen.
3. Die Build-Einstellungen werden automatisch aus `netlify.toml` gelesen
   (Hugo-Version, Befehl, Verzeichnis) – nichts ändern, **Deploy** klicken.
4. Nach 1–2 Minuten ist die Seite unter einer `*.netlify.app`-Adresse
   erreichbar. Alles prüfen.
5. **Team-Zugang:** Unter Team settings eine zweite Person einladen
   (gleiches Prinzip wie bei GitHub).

### 3. Domain verbinden

1. Netlify → Site → **Domain management → Add custom domain** →
   `www.gemeinde-tiddische.de` (und `gemeinde-tiddische.de` als Redirect
   auf www).
2. Beim bisherigen Domain-Anbieter die von Netlify angezeigten DNS-Einträge
   setzen (CNAME für www auf die Netlify-Adresse, A/ALIAS für die
   Hauptdomain auf Netlifys Load Balancer – Netlify zeigt die genauen
   Werte an).
3. HTTPS aktiviert Netlify automatisch (Let's Encrypt), sobald die
   DNS-Einträge greifen.
4. `gemeinde-hoitlingen.de`: ebenfalls als Domain-Alias hinzufügen oder
   beim Registrar eine Weiterleitung auf www.gemeinde-tiddische.de
   einrichten.

> **Hinweis Domainwechsel:** Erst die neue Seite vollständig prüfen, dann
> die DNS-Einträge umstellen. Die alte Website bleibt bis dahin erreichbar.

### 4. CMS-Anmeldung einrichten (OAuth-Worker)

Das CMS unter `/admin/` meldet Redakteure über GitHub an. Dafür braucht es
einen kleinen, kostenlosen Vermittler-Dienst („OAuth-Worker") bei
Cloudflare. Anleitung des Projekts:
**https://github.com/sveltia/sveltia-cms-auth** (README, 10 Minuten).

Kurzfassung:

1. Kostenloses Cloudflare-Konto anlegen (mit der Gemeinde-Mail).
2. Auf der sveltia-cms-auth-Seite den Button **„Deploy to Cloudflare
   Workers"** nutzen – der Worker wird automatisch eingerichtet.
3. In GitHub eine **OAuth App** anlegen
   (Organisation → Settings → Developer settings → OAuth Apps → New):
   - Homepage URL: `https://www.gemeinde-tiddische.de`
   - Authorization callback URL: `https://<WORKER-NAME>.workers.dev/callback`
4. Client-ID und Client-Secret der OAuth App als Variablen
   (`GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`) im Worker hinterlegen;
   bei `ALLOWED_DOMAINS` die Domain `www.gemeinde-tiddische.de` eintragen.
5. In `static/admin/config.yml` zwei Zeilen anpassen:
   - `repo:` → `gemeinde-tiddische/website`
   - `base_url:` → `https://<WORKER-NAME>.workers.dev`
6. Änderung committen/pushen → nach dem automatischen Deploy unter
   `https://www.gemeinde-tiddische.de/admin/` den Login testen.

### 5. Letzte Handgriffe

- [ ] In `content/datenschutz.md` und `content/barrierefreiheit.md` die
      **Entwurfs-Hinweise prüfen/entfernen** und Platzhalter ausfüllen
      (Prüfdatum!). Idealerweise vom Datenschutzbeauftragten der
      Samtgemeinde gegenlesen lassen.
- [ ] In `data/gemeinde.yaml` den GitHub-Link prüfen.
- [ ] „Zugangs-Zettel" fürs Gemeindebüro erstellen: GitHub-Org-Owner-Konto,
      Netlify-Konto, Cloudflare-Konto, Domain-Registrar – je mit
      Benutzername, hinterlegter E-Mail und Hinweis auf den Passwort-Ort.
- [ ] Alte Website-Hosting kündigen (erst nach erfolgreichem Umzug!).

---

## B. Neue Redakteurin / neuen Redakteur hinzufügen (15–30 Min)

1. Gemeinsam ein **GitHub-Konto** für die Person anlegen
   (github.com/signup, dienstliche oder private Mail der Person).
2. Die Person zur Organisation einladen:
   github.com/orgs/gemeinde-tiddische/people → **Invite member**.
   Rolle „Member" genügt; beim Repository `website` **Write**-Zugriff
   geben (Repository → Settings → Collaborators and teams).
3. Einladung annehmen lassen, dann gemeinsam
   **www.gemeinde-tiddische.de/admin/** öffnen und einmal den kompletten
   Anmelde-Ablauf durchspielen.
4. Eine Test-Meldung anlegen und wieder löschen – so verliert die
   Oberfläche ihren Schrecken.
5. Der Person die **ANLEITUNG.md** (oder den Ausdruck davon) mitgeben.

## C. Redakteur entfernen

GitHub-Organisation → People → Person entfernen. Fertig – der CMS-Zugang
erlischt damit automatisch.

---

## D. Wartung (selten nötig)

**Grundsatz:** Die Seite läuft ohne Eingriffe weiter. Folgende Punkte nur
bei Bedarf:

### Hugo aktualisieren
Nur nötig, wenn neue Funktionen gebraucht werden. In `netlify.toml` die
`HUGO_VERSION` erhöhen, lokal testen (`hugo server`), committen. Bei
Problemen: Versionsnummer einfach zurückdrehen.

### Sveltia CMS
`static/admin/index.html` lädt automatisch die aktuelle Version. Sollte
das CMS nach einem Update Probleme machen: in dieser Datei die
auskommentierte **feste Versionsnummer** eintragen (Anleitung steht direkt
in der Datei). Sveltia ist konfigurationskompatibel zu Decap CMS – im
Notfall kann die Script-Zeile auch auf Decap umgestellt werden, die
`config.yml` bleibt gleich.

### Lokale Vorschau (für größere Änderungen)
```bash
# Hugo installieren: https://gohugo.io/installation/ (Version siehe netlify.toml)
git clone https://github.com/gemeinde-tiddische/website.git
cd website
hugo server          # Vorschau unter http://localhost:1313
```
Das CMS kann lokal ohne Anmeldung getestet werden: `hugo server` starten,
dann http://localhost:1313/admin/ öffnen und „Work with Local Repository"
wählen (Chrome/Edge).

### Etwas ist kaputtgegangen?
Jede Änderung ist ein Git-Commit. Auf GitHub unter „Commits" die letzte
funktionierende Version finden → „Revert" → Netlify baut automatisch die
reparierte Fassung. Alternativ in Netlify unter „Deploys" einen früheren
Deploy mit einem Klick wiederherstellen („Publish deploy").

---

## E. Bewusste Entscheidungen (damit niemand sie versehentlich rückbaut)

- **Kein Kontaktformular** – bewusst nur E-Mail-Adresse (kein Spam, keine
  Abhängigkeit von Formulardiensten).
- **Keine Cookies, kein Tracking, keine externen Einbindungen** beim
  Seitenaufruf – Schriften liegen lokal (`static/fonts/`). Bitte niemals
  Google Fonts, YouTube-Embeds o. Ä. direkt einbinden (DSGVO!).
- **Großer PDF-Bestand im Repository** ist beabsichtigt (Transparenz,
  Vollständigkeit). Neue PDFs möglichst unter 10 MB halten; Scans vorher
  komprimieren. Die Bebauungspläne wurden beim Umzug bereits von 68 MB auf
  27 MB komprimiert (Originale liegen bei der Gemeinde/Samtgemeinde).
- **Hugo-Version festgepinnt** – verhindert, dass ein automatisches Update
  den Build bricht.
- **Ratsmitglieder erzeugen keine eigenen Unterseiten** (bewusst; sie
  erscheinen nur auf der Gemeinderats-Seite).
