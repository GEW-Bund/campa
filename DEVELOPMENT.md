# Development Guide

> Technische Dokumentation für Entwickler

---

## 🏗️ Stack & Architektur

### Technologie-Stack

| Komponente | Technologie | Version |
|------------|-------------|---------|
| Static Site Generator | Hugo Extended | v0.100+ |
| CSS Framework | Tailwind CSS (CLI) | v3.4.x |
| JavaScript Runtime | Node.js | v20+ (v25 empfohlen) |
| Package Manager | npm | v9+ |
| Deployment | GitHub Webhook | - |

### Warum diese Technologien?

**Hugo:**
- Single Binary (~90 MB), keine Runtime-Dependencies
- Extrem schnell (Build in <1 Sekunde)
- Markdown-basiert
- Flexibles Template-System

**Tailwind CSS:**
- Utility-First CSS
- Kleines Production-Bundle (~100 KB)
- Kompiliert während Development

**Node.js:**
- Nur für Tailwind CSS Build
- Hugo selbst braucht kein Node.js

---

## 💻 Lokale Entwicklung

### Prerequisites

**Erforderlich:**
- Node.js v20+ ([nodejs.org](https://nodejs.org))
- Hugo Extended v0.100+ ([gohugo.io/installation](https://gohugo.io/installation/))

**Optional:**
- NVM (Node Version Manager) für einfaches Node-Versions-Management

### Installation

```bash
# Repository klonen
git clone https://github.com/GEW-Bund/campa.git
cd campa

# Node-Version aktivieren (falls NVM installiert)
nvm use 25

# Dependencies installieren
npm install
```

### Development Server starten

**Du brauchst 2 Terminals:**

```bash
# Terminal 1: Tailwind CSS Watch
npm run dev:css

# Terminal 2: Hugo Development Server
hugo server
```

**Öffne im Browser:** http://localhost:1313

**Was passiert:**
- Tailwind CSS kompiliert bei jeder Änderung in `assets/css/`
- Hugo rebuildet die Site bei jeder Änderung
- Browser lädt automatisch neu (Live Reload)

### Build für Production

```bash
# CSS kompilieren (minified)
npm run build:css

# Hugo Build
hugo --minify
```

**Output:** `public/` Verzeichnis (bereit für Deployment)

---

## 📁 Projektstruktur

### Vollständige Verzeichnisstruktur

```
campa/
├── assets/
│   └── css/
│       └── main.css                 # Tailwind-Quelle (editieren!)
├── content/
│   ├── _index.md                    # HAUPT-KONFIGURATION
│   └── schule-zeigt-haltung/        # Markdown-Inhalte
│       ├── verunsicherung.md
│       ├── forderungen.md
│       ├── faq.md
│       ├── impressum.md
│       └── ...
├── layouts/
│   ├── _default/
│   │   └── baseof.html              # HTML-Grundgerüst
│   ├── index.html                   # Homepage-Template (Block-System)
│   └── partials/
│       ├── block-textblock.html     # Textblock-Component
│       ├── block-slider.html        # Slider-Component
│       ├── block-faq.html           # FAQ-Component
│       └── ...
├── static/
│   ├── css/
│   │   ├── main.css                 # Kompilierte Tailwind-CSS (generiert!)
│   │   └── themes/
│   │       └── schule-zeigt-haltung.css   # Theme-Variablen
│   ├── fonts/                       # Lokale Webfonts
│   └── schule-zeigt-haltung/        # Kampagnen-Bilder
│       ├── header.jpg
│       ├── slide1-4.jpg
│       └── ...
├── hugo.toml                        # Hugo-Konfiguration
├── tailwind.config.js               # Tailwind-Konfiguration
├── package.json                     # Node.js Dependencies
├── README.md                        # Übersicht
├── DETAILS.md                       # Redaktions-Handbuch
├── DEVELOPMENT.md                   # Dieses Dokument
└── LLM.md                           # Vollständige Tech-Doku für LLMs
```

### Wichtige Dateien im Detail

#### `content/_index.md`
- **Hugo rendert IMMER diese Datei als Startseite**
- Enthält die komplette Seitenkonfiguration
- YAML Frontmatter + Markdown Content

#### `assets/css/main.css`
- Tailwind CSS Quelle
- Wird kompiliert zu `static/css/main.css`
- **Nur diese Datei editieren!** (nicht `static/css/main.css`)

#### `layouts/index.html`
- Homepage-Template
- Iteriert über `blocks:` Array
- Lädt Block-Partials dynamisch

#### `layouts/partials/block-*.html`
- Einzelne Block-Components
- Laden Content via `$page.Site.GetPage`

---

## 🎨 Theme anpassen

### CSS-Variablen

**Datei:** `static/css/themes/schule-zeigt-haltung.css`

```css
:root {
  /* Farben */
  --color-primary:     #1a3a5c;   /* Hauptfarbe (Buttons, Links) */
  --color-accent:      #e63946;   /* Akzentfarbe (Highlights) */
  --color-bg:          #f8f7f4;   /* Hintergrund */
  --color-text:        #1a1a1a;   /* Textfarbe */
  --color-text-light:  #555555;   /* Sekundärtext */
  --color-border:      #d0ccc4;   /* Linien */
  
  /* Fonts */
  --font-heading:      'Georgia', serif;
  --font-body:         system-ui, sans-serif;
}
```

### Eigenes Theme erstellen

1. Kopiere die Theme-Datei:
   ```bash
   cp static/css/themes/schule-zeigt-haltung.css \
      static/css/themes/neue-kampagne.css
   ```

2. Passe die CSS-Variablen an

3. Referenziere in `content/_index.md`:
   ```yaml
   theme_css: "neue-kampagne.css"
   ```

---

## 🔧 Hugo-Spezifika

### Content-Lookup

**Wichtig:** Hugo lädt Markdown-Partials über den Kampagnen-Namen:

```yaml
# content/_index.md
kampagne: "schule-zeigt-haltung"

blocks:
  - type: textblock
    src: forderungen  # ← Hugo sucht: content/schule-zeigt-haltung/forderungen.md
```

**Template-Code:**
```go
{{ $kampagne := $page.Params.kampagne }}
{{ $content := $page.Site.GetPage (printf "/%s/%s" $kampagne $block.src) }}
```

### Block-System

**Reihenfolge = Rendering-Reihenfolge:**

```yaml
blocks:
  - type: textblock    # ← Wird als 1. gerendert
  - type: slider       # ← Wird als 2. gerendert
  - type: cta          # ← Wird als 3. gerendert
```

**Template (vereinfacht):**
```html
{{ range .Params.blocks }}
  {{ if eq .type "textblock" }}
    {{ partial "block-textblock.html" (dict "block" . "page" $) }}
  {{ else if eq .type "slider" }}
    {{ partial "block-slider.html" (dict "Block" . "Page" $) }}
  {{ end }}
{{ end }}
```

### Anker-IDs

IDs **müssen** im `blocks:` Array stehen:

```yaml
# ❌ FALSCH
slider:
  id: bilder

# ✅ RICHTIG
blocks:
  - type: slider
    id: bilder  # ← Wird zu <section id="bilder">
```

---

## 🔄 Tailwind CSS Pipeline

### Wie funktioniert der Build?

```
assets/css/main.css
  (Quelle)
      ↓
  [Tailwind CLI]
      ↓
static/css/main.css
  (Kompiliert, ~100 KB)
      ↓
  [Hugo lädt es]
      ↓
  <link rel="stylesheet">
```

### npm Scripts

**`package.json`:**
```json
{
  "scripts": {
    "dev:css": "tailwindcss -i ./assets/css/main.css -o ./static/css/main.css --watch",
    "build:css": "tailwindcss -i ./assets/css/main.css -o ./static/css/main.css --minify"
  }
}
```

**Development:**
```bash
npm run dev:css
# → Watch-Modus, kompiliert bei jeder Änderung
```

**Production:**
```bash
npm run build:css
# → Einmaliger Build, minified
```

### Tailwind Config

**`tailwind.config.js`:**
```js
module.exports = {
  content: [
    "./layouts/**/*.html",
    "./content/**/*.md",
  ],
  theme: {
    extend: {},
  },
}
```

**Wichtig:** Tailwind scannt diese Dateien nach verwendeten CSS-Klassen!

---

## 🐛 Troubleshooting

### CSS-Änderungen greifen nicht

**Problem:** Änderungen in `assets/css/main.css` sind nicht sichtbar.

**Lösung:**
```bash
# 1. Läuft Tailwind Watch?
npm run dev:css

# 2. Existiert die kompilierte Datei?
ls -la static/css/main.css

# 3. Browser-Cache
# Hard-Refresh: Cmd+Shift+R (Mac) / Ctrl+Shift+R (Windows)
```

### Hugo findet Blöcke nicht

**Problem:** Block wird nicht angezeigt.

**Checkliste:**
- ✅ Steht der Block in `blocks:` Array in `content/_index.md`?
- ✅ Ist `type:` korrekt geschrieben? (`textblock` nicht `text-block`)
- ✅ Existiert die referenzierte Datei? (`src: forderungen` → `forderungen.md`)
- ✅ Ist der Kampagnen-Name korrekt? (`kampagne: "schule-zeigt-haltung"`)

**Debug:**
```bash
# Hugo mit Verbose-Logging
hugo server --verbose --debug
```

### Anker-Links funktionieren nicht

**Problem:** Navigation-Links springen nicht zum Block.

**Ursache:** ID fehlt oder ist falsch platziert.

**Fix:**
```yaml
nav_links:
  - label: "FAQ"
    url: "#faq"    # ← Muss mit Block-ID übereinstimmen!

blocks:
  - type: faq
    id: faq        # ← ID HIER, nicht woanders!
```

### Bilder werden nicht angezeigt

**Problem:** `<img>` lädt nicht.

**Checkliste:**
- ✅ Liegt das Bild in `static/`? (Nicht `assets/`!)
- ✅ Ist der Pfad relativ zur `static/`? (`/schule-zeigt-haltung/header.jpg`)
- ✅ Ist die Datei-Extension korrekt? (Groß-/Kleinschreibung!)

---

## 🚀 Deployment

### Automatisches Deployment

Die Live-Site wird **automatisch** aktualisiert wenn auf `main` gepusht wird.

**Workflow:**
1. Änderung committen
2. Push zu `main` Branch
3. GitHub triggert Webhook
4. Server baut die Site neu
5. ~15 Sekunden später ist die Site live

**Technische Details:**
- Webhook-basiertes Deployment
- Build läuft auf VPS
- Hugo + Tailwind CSS Build
- Static Files werden ausgeliefert

**Setup-Details befinden sich in privater Dokumentation** (nicht im Repo aus Sicherheitsgründen).

### Manueller Build

Falls du lokal einen Production-Build testen willst:

```bash
# CSS kompilieren
npm run build:css

# Hugo Build
hugo --minify

# Output ist in: public/
```

---

## 🔮 Zukünftige Kampagnen

Das System ist für **mehrere Kampagnen** ausgelegt, aktuell läuft aber nur eine.

### Neue Kampagne anlegen (für später)

1. **Content-Ordner erstellen:**
   ```bash
   mkdir -p content/neue-kampagne
   mkdir -p static/neue-kampagne
   ```

2. **Content-Dateien kopieren:**
   ```bash
   cp content/schule-zeigt-haltung/*.md content/neue-kampagne/
   ```

3. **`content/_index.md` anpassen:**
   ```yaml
   kampagne: "neue-kampagne"
   theme_css: "neue-kampagne.css"
   
   hero:
     image: "/neue-kampagne/header.jpg"
   ```

4. **Theme erstellen:**
   ```bash
   cp static/css/themes/schule-zeigt-haltung.css \
      static/css/themes/neue-kampagne.css
   ```

5. **Bilder hochladen:**
   - Nach `static/neue-kampagne/`

6. **Testen:**
   ```bash
   hugo server
   ```

---

## 📚 Weiterführende Dokumentation

- **[Hugo Documentation](https://gohugo.io/documentation/)** - Offizielle Hugo-Docs
- **[Tailwind CSS Docs](https://tailwindcss.com/docs)** - Tailwind Reference
- **[LLM.md](LLM.md)** - Vollständige technische Dokumentation (für LLMs/AI)

---

## 🤝 Contributing

**Pull Requests willkommen!**

1. Fork das Repo
2. Branch erstellen: `git checkout -b feature/mein-feature`
3. Änderungen committen
4. Push zum Branch
5. Pull Request öffnen

---

## 📞 Support

**Bei technischen Problemen:**
1. Prüfe dieses Dokument (Troubleshooting)
2. Schau in [LLM.md](LLM.md) für tiefere technische Details
3. Kontaktiere das Tech-Team
