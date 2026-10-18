# Website der Gemeinde Tiddische

Offizielle Website der Gemeinde Tiddische (Ortschaften Tiddische und
Hoitlingen, Samtgemeinde Brome, Landkreis Gifhorn) –
**www.gemeinde-tiddische.de**

Statische Website, erstellt mit [Hugo](https://gohugo.io/), gehostet auf
GitHub Pages, Inhaltspflege über [Sveltia CMS](https://github.com/sveltia/sveltia-cms)
unter `/admin/`.

## Projektstruktur

```
content/          Alle Inhalte als Markdown (über das CMS bearbeitbar)
  aktuelles/        Meldungen
  veranstaltungen/  Termine (automatische Sortierung kommend/vergangen)
  protokolle/       Sitzungsprotokolle (PDF)
  satzungen/        Satzungen (PDF)
  ratsmitglieder/   Ratsmitglieder (erscheinen auf /gemeinderat/)
  vereine/          Vereine beider Ortschaften
data/gemeinde.yaml  Kontaktdaten & Zeiten (Fußzeile/Startseite)
data/weiterleitungen.yaml  Alte Adressen → neue Adressen
layouts/          Hugo-Vorlagen (HTML)
assets/css/       Stylesheet (Farbsystem aus dem Gemeindewappen)
static/           Bilder, PDFs, Schriften (selbst gehostet), CMS
  admin/            Sveltia CMS (config.yml = Redaktionsoberfläche)
.github/workflows/website.yml  Bauen & Veröffentlichen (GitHub Pages), Hugo-Version
```

## Technische Eckdaten

- **Barrierefreiheit:** semantisches HTML, Tastaturbedienung, Sprungmarke,
  geprüfte Kontraste, Schriftart „Atkinson Hyperlegible", Erklärung zur
  Barrierefreiheit unter `/barrierefreiheit/`
- **Datenschutz:** keine Cookies, kein Tracking, keine externen Ressourcen
  beim Seitenaufruf; Schriften self-hosted
- **Hell-/Dunkelmodus:** folgt der Systemeinstellung, manuell umschaltbar
- **RSS-Feeds:** `/aktuelles/index.xml` und `/veranstaltungen/index.xml`
- **Alte URLs** (`…/aktuelles.html` usw.) leiten auf die neuen Adressen
  weiter (`data/weiterleitungen.yaml`, ausgewertet von der 404-Seite)

## Lizenz

Quellcode unter [MIT-Lizenz](LICENSE). Inhalte (Texte, Bilder, Dokumente)
© Gemeinde Tiddische bzw. jeweilige Rechteinhaber.
