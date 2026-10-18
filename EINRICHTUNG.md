# Einrichtung & technische Betreuung

**Für: die technisch betreuende Person (aktuell oder zukünftig).**
Dieses Dokument beschreibt die einmalige Einrichtung der Website und alle
wiederkehrenden Wartungsaufgaben. Ziel: Jede technisch versierte Person
(oder Agentur) kann die Betreuung ohne Vorwissen übernehmen.

## Überblick: Wie die Website funktioniert

| Baustein | Dienst | Kosten | Zweck |
|---|---|---|---|
| Quellcode & Inhalte | GitHub (öffentliches Repository) | 0 € | Speichert alles; jede Änderung ist protokolliert |
| Website-Generator | Hugo (Version in `.github/workflows/website.yml` festgepinnt) | 0 € | Baut aus den Inhalten statische HTML-Seiten |
| Hosting | GitHub Pages + GitHub Actions | 0 € | Baut bei jeder Änderung automatisch neu und liefert die Seiten aus (kostenlos für öffentliche Repositories, ohne Begrenzung der Veröffentlichungen) |
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

### 2. GitHub Pages aktivieren

1. Repository → **Settings → Pages** → „Build and deployment" →
   Source: **GitHub Actions**.
2. Mehr ist nicht einzustellen: Die Datei `.github/workflows/website.yml`
   baut und veröffentlicht die Seite bei jeder Änderung auf `main`.
3. Reiter **Actions** im Repository öffnen: Der Lauf „Website
   veröffentlichen" muss nach 1–2 Minuten einen grünen Haken zeigen.
   Falls noch kein Lauf existiert: „Website veröffentlichen" →
   **Run workflow**.

### 3. Domain verbinden

> **Reihenfolge beachten:** Domain immer **zuerst in GitHub** eintragen,
> **dann** DNS ändern – sonst könnte jemand anderes die Domain auf GitHub
> für sich beanspruchen.

1. **Domain für die Organisation verifizieren** (Schutz vor Übernahme):
   Organisation → **Settings → Pages → Add a domain** →
   `gemeinde-tiddische.de` → den angezeigten TXT-Eintrag
   (`_github-pages-challenge-gemeinde-tiddische.gemeinde-tiddische.de`)
   beim Domain-Anbieter anlegen → **Verify**. Den TXT-Eintrag dauerhaft
   stehen lassen.
2. Repository → **Settings → Pages → Custom domain**:
   `www.gemeinde-tiddische.de` → **Save**. (Die Meldung „DNS check
   unsuccessful" ist normal, bis Schritt 3 erledigt ist.)
3. Beim Domain-Anbieter die DNS-Einträge setzen und **alle alten Einträge
   für `@` und `www` entfernen**:

   | Name | Typ | Wert |
   |---|---|---|
   | `@` | A | `185.199.108.153` |
   | `@` | A | `185.199.109.153` |
   | `@` | A | `185.199.110.153` |
   | `@` | A | `185.199.111.153` |
   | `@` | AAAA | `2606:50c0:8000::153` |
   | `@` | AAAA | `2606:50c0:8001::153` |
   | `@` | AAAA | `2606:50c0:8002::153` |
   | `@` | AAAA | `2606:50c0:8003::153` |
   | `www` | CNAME | `gemeinde-tiddische.github.io` |

4. Sobald der DNS-Check in den Pages-Einstellungen grün ist (Minuten bis
   max. 24 Stunden): **Enforce HTTPS** aktivieren. GitHub leitet
   `gemeinde-tiddische.de` automatisch auf `www` um.
5. `gemeinde-hoitlingen.de`: GitHub Pages bedient nur eine Domain pro
   Website. Beim Domain-Anbieter eine **Weiterleitung** auf
   `https://www.gemeinde-tiddische.de` einrichten.

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
      Cloudflare-Konto, Domain-Registrar – je mit
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
Nur nötig, wenn neue Funktionen gebraucht werden. In
`.github/workflows/website.yml` die `HUGO_VERSION` erhöhen, lokal testen
(`hugo server`), committen. Bei Problemen: Versionsnummer einfach
zurückdrehen.

### Sveltia CMS
`static/admin/index.html` lädt automatisch die aktuelle Version. Sollte
das CMS nach einem Update Probleme machen: in dieser Datei die
auskommentierte **feste Versionsnummer** eintragen (Anleitung steht direkt
in der Datei). Sveltia ist konfigurationskompatibel zu Decap CMS – im
Notfall kann die Script-Zeile auch auf Decap umgestellt werden, die
`config.yml` bleibt gleich.

### Lokale Vorschau (für größere Änderungen)
```bash
# Hugo "extended" installieren: https://gohugo.io/installation/
# (Version siehe .github/workflows/website.yml)
git clone https://github.com/gemeinde-tiddische/website.git
cd website
hugo server          # Vorschau unter http://localhost:1313
```
Das CMS kann lokal ohne Anmeldung getestet werden: `hugo server` starten,
dann http://localhost:1313/admin/ öffnen und „Work with Local Repository"
wählen (Chrome/Edge).

### Änderungen erscheinen nicht online?
Reiter **Actions** im Repository öffnen. Jede Änderung erzeugt dort einen
Lauf „Website veröffentlichen":
- **Grüner Haken:** Die Änderung ist online (ggf. Browser-Cache leeren).
- **Rotes X:** Lauf anklicken – die Fehlermeldung steht im Protokoll des
  fehlgeschlagenen Schritts. Nach der Korrektur startet automatisch ein
  neuer Lauf; ein Lauf lässt sich auch mit „Re-run jobs" wiederholen.
- **Gar kein Lauf:** Settings → Pages prüfen (Source muss „GitHub Actions"
  sein) und ob Actions für das Repository erlaubt sind.

### Etwas ist kaputtgegangen?
Jede Änderung ist ein Git-Commit. Auf GitHub unter „Commits" die letzte
funktionierende Version finden → „Revert" → die reparierte Fassung wird
automatisch neu gebaut und veröffentlicht.

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
- **GitHub Pages statt Netlify** (Umstellung Oktober 2026): Netlifys
  kostenloser Plan rechnet mit Credits – jede Veröffentlichung kostet 15
  von 300 Credits pro Monat, also höchstens rund 20 Speicherungen im CMS.
  Danach werden Änderungen nicht mehr veröffentlicht und die Seite wird
  schließlich abgeschaltet. Für eine von Laien gepflegte Seite ungeeignet.
- **Weiterleitungen alter Adressen** laufen über die Fehlerseite
  (`layouts/404.html`, Liste in `data/weiterleitungen.yaml`), weil
  GitHub Pages keine Weiterleitungen auf dem Server kennt. Eigene
  HTTP-Header sind auf GitHub Pages ebenfalls nicht möglich.
- **`buildFuture = true`** (`hugo.toml`): Beiträge mit Datum in der Zukunft
  erscheinen sofort. Eine statische Seite baut sich am Stichtag nicht von
  selbst neu – ohne diese Einstellung blieben solche Beiträge unsichtbar.
- **Ratsmitglieder erzeugen keine eigenen Unterseiten** (bewusst; sie
  erscheinen nur auf der Gemeinderats-Seite).
