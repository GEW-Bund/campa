# Campa — Modulares Kampagnen-Landingpage-System

> Static Site Generator für politische Kampagnen. Gebaut mit Hugo + Tailwind CSS.

---

## Für Redakteure: Inhalte bearbeiten

### Text ändern

1. Öffne das Repo auf [github.com](https://github.com)
2. Navigiere zu `content/kampagnen/deine-kampagne/`
3. Klicke auf die gewünschte Datei (z.B. `textblock1.md`)
4. Klicke auf den **Bleistift** oben rechts (Edit this file)
5. Ändere den Text
6. **Commit changes** → Commit bestätigen
7. **Fertig** — die Seite wird automatisch aktualisiert (~15 Sekunden)

### Diese Dateien kannst du bearbeiten:

| Datei | Inhalt |
|-------|--------|
| `_index.md` | Haupt-Konfiguration (Hero, Navigation, Blocks) |
| `textblock1.md` | Textblock-Inhalte |
| `faq.md` | FAQ — jede `###`-Überschrift wird ein Aufklappelement |
| `impressum.md` | Impressum |
| `datenschutz.md` | Datenschutzerklärung |

### Bilder austauschen

1. Benenne das neue Bild **exakt wie das alte** (z.B. `slide1.jpg`)
2. Navigiere zu `static/images/deine-kampagne/`
3. Klicke auf **Add file** → **Upload files**
4. Lade das neue Bild hoch (GitHub überschreibt automatisch)
5. **Commit** → fertig

### FAQ-Einträge bearbeiten

In `faq.md` erzeugt jede `###`-Überschrift ein Aufklappelement:

```markdown
### Meine Frage hier?
Die Antwort kommt direkt darunter als normaler Text.

### Nächste Frage?
Nächste Antwort.
```

---

## Für Entwickler: Stack & Architektur

### Stack

| Komponente | Technologie |
|------------|-------------|
| Static Site Generator | Hugo (Extended) |
| CSS Framework | Tailwind CSS v3 (CLI) |
| Fonts | Lokal gehostet (DSGVO) |
| Deployment | GitHub Webhook → Host Script |
| Node.js | v20+ (für CSS Build) |

### Deployment-Architektur (High-Level)

```
GitHub Push (main)
    ↓ Webhook (HTTPS POST)
Server: Webhook-Service
    ↓ triggert
deploy.sh:
  - git fetch + reset (force update)
  - npm run build:css (Tailwind)
  - hugo --minify
    ↓
Web-Server liefert Static Files
```

**Für Details zum Server-Setup:** Siehe `DEPLOYMENT.md` (wird lokal vorgehalten, nicht im Repo)

---

## Lokale Entwicklung

### Voraussetzungen

- Node.js v20+ (v25 funktioniert einwandfrei!)
- Hugo Extended
- NVM (empfohlen)

### Setup

```bash
# Node-Version aktivieren (falls NVM installiert)
nvm use 25  # Oder 20, 22 - alle funktionieren!

# Abhängigkeiten installieren
npm install

# Terminal 1: Tailwind watch
npm run dev:css

# Terminal 2: Hugo dev server
hugo server
```

**Öffne:** http://localhost:1313

---

## Kampagne konfigurieren

Die gesamte Kampagne wird über **eine Datei** gesteuert: `content/_index.md`

### Minimalbeispiel

```yaml
---
title: "Meine Kampagne"
description: "Kurzbeschreibung"
kampagne: "meine-kampagne"
theme_css: "meine-kampagne.css"

nav_links:
  - label: "Über uns"
    url: "#intro"

hero:
  title: "Hauptüberschrift"
  image: "/images/meine-kampagne/header.jpg"
  text: "Beschreibung"
  cta_label: "Jetzt mitmachen"
  cta_url: "https://example.com"

blocks:
  - type: textblock
    src: intro
    bg: light
    id: intro

  - type: cta
    id: kontakt
    title: "Kontaktiere uns"
    button:
      label: "E-Mail schreiben"
      url: "mailto:info@example.com"
---
```

### Verfügbare Block-Typen

| Block-Typ | Parameter | Beschreibung |
|-----------|-----------|------------|
| `textblock` | `src`, `bg`, `id` | Markdown-Textblock |
| `slider` | `id` | Bildslider (Bilder aus `slider.images`) |
| `faq` | `src`, `title`, `bg`, `id` | FAQ (H3 = Frage) |
| `cta` | `id`, `bg`, `title`, `text`, `button` | Call-to-Action Banner |
| `logos` | `title`, `bg`, `items` | Logo-Leiste |
| `imagetext` | `src`, `image`, `bg`, `id`, `button` | Bild + Text nebeneinander |
| `accordion` | `title`, `bg`, `items` | Aufklapp-Elemente |

### Anker-Links für Navigation

**IDs müssen im `blocks:`-Array stehen:**

```yaml
# ✅ RICHTIG
nav_links:
  - label: "FAQ"
    url: "#faq"

blocks:
  - type: faq
    id: faq  # ← Erzeugt <section id="faq">
```

---

## Theme anpassen

Erstelle eine CSS-Datei: `static/css/themes/meine-kampagne.css`

```css
:root {
  --color-primary:     #1a3a5c;   /* Hauptfarbe */
  --color-accent:      #e63946;   /* Akzentfarbe */
  --color-bg:          #f8f7f4;   /* Hintergrund */
  --color-text:        #1a1a1a;   /* Text */
  --color-text-light:  #555555;   /* Sekundärtext */
  --color-border:      #d0ccc4;   /* Linien */
  --font-heading:      'Georgia', serif;
  --font-body:         system-ui, sans-serif;
}
```

In `content/_index.md`:
```yaml
theme_css: "meine-kampagne.css"
```

---

## Projektstruktur

```
campa/
├── assets/css/main.css            ← Tailwind-Quelle
├── static/
│   ├── css/
│   │   ├── main.css               ← Generiert (von npm run build:css)
│   │   └── themes/                ← Kampagnen-spezifische CSS
│   ├── fonts/                     ← Lokale Fonts
│   └── images/                    ← Kampagnen-Bilder
├── layouts/
│   ├── _default/baseof.html       ← HTML-Grundgerüst
│   ├── index.html                 ← Block-System
│   └── partials/                  ← Block-Templates
├── content/
│   ├── _index.md                  ← HAUPT-KONFIGURATION
│   └── kampagnen/*/               ← Markdown-Inhalte
├── hugo.toml
├── tailwind.config.js
└── package.json
```

---

## Troubleshooting

### CSS-Änderungen greifen nicht
```bash
# Läuft Tailwind-Watch?
npm run dev:css

# CSS-Datei existiert?
ls -la static/css/main.css

# Hard-Refresh im Browser
Cmd+Shift+R / Ctrl+Shift+R
```

### Hugo findet Blöcke nicht
- Prüfe `blocks:`-Liste in `content/_index.md`
- Typ korrekt geschrieben? (`textblock` nicht `text-block`)

### Anker-Links funktionieren nicht
- ID muss im Block stehen, nicht im Daten-Objekt!
```yaml
# ❌ FALSCH
slider:
  id: bilder

# ✅ RICHTIG
blocks:
  - type: slider
    id: bilder
```

---

## Neue Kampagne anlegen

1. **Ordner erstellen**
   ```bash
   mkdir -p content/kampagnen/neue-kampagne
   mkdir -p static/images/neue-kampagne
   ```

2. **Dateien kopieren**
   ```bash
   cp content/kampagnen/demo/* content/kampagnen/neue-kampagne/
   ```

3. **`_index.md` anpassen**
   - `kampagne: "neue-kampagne"`
   - `theme_css: "neue-kampagne.css"`
   - Hero, Blocks, Navigation anpassen

4. **Theme erstellen** (optional)
   ```bash
   cp static/css/themes/demo.css static/css/themes/neue-kampagne.css
   ```

5. **Bilder hochladen** nach `static/images/neue-kampagne/`

6. **Lokal testen:** `hugo server`

---

## Lizenz & Credits

- **Hugo:** Apache 2.0 License
- **Tailwind CSS:** MIT License
- **System:** Entwickelt für politische Kampagnenarbeit

**Projekt-Repository:** https://github.com/GEW-Bund/campa
