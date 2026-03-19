# Kampagnen-Site — Dokumentation

Ein modulares System für Kampagnen-Landingpages.
Gebaut mit Hugo + Tailwind CSS. Deployt via GitHub Actions auf deinen eigenen Server.

---

## Für Nicht-Techniker: Inhalte bearbeiten

### Text ändern

1. Öffne das Repo auf github.com
2. Navigiere zu `content/kampagne-demo/`
3. Klicke auf die gewünschte Datei (z.B. `textblock1.md`)
4. Klicke auf den **Bleistift** oben rechts (Edit this file)
5. Ändere den Text
6. Klicke auf **Commit changes** → **Commit changes** bestätigen
7. Fertig — die Seite wird automatisch aktualisiert (ca. 2 Minuten)

**Diese Dateien kannst du bearbeiten:**

| Datei | Inhalt |
|-------|--------|
| `textblock1.md` | Erster Textblock (Bündnis, Einleitung) |
| `textblock2.md` | Zweiter Textblock (Forderungen) |
| `faq.md` | FAQ — jede `###`-Überschrift wird ein Aufklappelement |
| `impressum.md` | Impressum |
| `datenschutz.md` | Datenschutzerklärung |

### Bild austauschen

1. Benenne das neue Bild **exakt wie das alte** (z.B. `slide1.jpg`)
2. Navigiere zu `static/images/kampagne-demo/`
3. Klicke auf **Add file → Upload files**
4. Lade das neue Bild hoch — GitHub überschreibt automatisch das alte
5. Commit → fertig

### FAQ-Einträge bearbeiten

In `faq.md` erzeugt jede `###`-Überschrift ein Aufklappelement:

```markdown
### Meine Frage hier?

Die Antwort kommt direkt darunter als normaler Text.
Mehrere Absätze sind möglich.

### Nächste Frage?

Nächste Antwort.
```

---

## Für Entwickler: Neue Kampagne anlegen

```bash
# 1. Neuen Kampagnen-Ordner anlegen
cp -r content/kampagnen/kampagne-demo content/kampagnen/neue-kampagne

# 2. Bilder-Ordner anlegen
mkdir static/images/neue-kampagne

# 3. _index.md anpassen
#    → kampagne: "neue-kampagne"
#    → Farben in static/css/themes/neue-kampagne.css
#    → hero, slider, cta, nav_links, footer_links

# 4. Optional: eigenes Theme
cp static/css/themes/demo.css static/css/themes/neue-kampagne.css
# CSS Custom Properties anpassen (Farben, Fonts)
```

### Theme anpassen

Lege eine Datei unter `static/css/themes/meine-kampagne.css` an:

```css
:root {
  --color-primary:    #1a3a5c;  /* Hauptfarbe */
  --color-accent:     #e63946;  /* Akzentfarbe */
  --color-bg:         #f8f7f4;  /* Hintergrund */
  --color-text:       #1a1a1a;  /* Fließtext */
  --color-text-light: #555555;  /* Sekundärtext */
  --color-border:     #d0ccc4;  /* Trennlinien */
  --font-heading: 'Source Serif 4', Georgia, serif;
  --font-body:    'Source Sans 3', system-ui, sans-serif;
}
```

Dann in `_index.md` der Kampagne:
```yaml
theme_css: "meine-kampagne.css"
```

---

## GitHub Actions — Secrets einrichten

Unter **Settings → Secrets and variables → Actions** folgende Secrets anlegen:

| Secret | Inhalt |
|--------|--------|
| `SSH_PRIVATE_KEY` | Inhalt des privaten SSH-Keys (z.B. `~/.ssh/id_ed25519`) |
| `REMOTE_HOST` | IP oder Hostname deines Servers |
| `REMOTE_USER` | SSH-Benutzername |
| `REMOTE_PATH` | Pfad auf dem Server (z.B. `/var/www/kampagne/`) |

### SSH-Key für Deploy anlegen

```bash
# Auf deinem Rechner
ssh-keygen -t ed25519 -C "github-actions-deploy" -f ~/.ssh/deploy_kampagne

# Public key auf den Server kopieren
ssh-copy-id -i ~/.ssh/deploy_kampagne.pub user@dein-server

# Private key als Secret in GitHub hinterlegen
cat ~/.ssh/deploy_kampagne
```

---

## Lokale Entwicklung

```bash
# Abhängigkeiten installieren
npm install

# Tailwind + Hugo gleichzeitig starten
npm run dev:css &
hugo server -D

# Seite öffnen
open http://localhost:1313
```

---

## Projektstruktur

```
kampagnen-site/
├── .github/workflows/deploy.yml    ← GitHub Actions Pipeline
├── assets/css/main.css             ← Tailwind + CSS Custom Properties
├── layouts/
│   ├── _default/baseof.html        ← HTML-Grundgerüst
│   ├── index.html                  ← One-Pager Aufbau
│   ├── partials/
│   │   ├── nav.html                ← Navigation
│   │   ├── footer.html             ← Footer
│   │   ├── hero.html               ← Hero-Block
│   │   ├── slider.html             ← Bildslider
│   │   ├── faq.html                ← FAQ-Accordion
│   │   └── scripts.html            ← JavaScript
├── content/kampagnen/
│   └── kampagne-demo/
│       ├── _index.md               ← Konfiguration der Kampagne
│       ├── textblock1.md           ← Erster Textblock
│       ├── textblock2.md           ← Zweiter Textblock
│       ├── faq.md                  ← FAQ-Inhalte
│       ├── impressum.md            ← Impressum
│       └── datenschutz.md          ← Datenschutz
└── static/
    ├── css/themes/                 ← Ein CSS-File pro Kampagne
    └── images/kampagne-demo/       ← Bilder der Kampagne
```
