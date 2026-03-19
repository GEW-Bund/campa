# Aktualisierte Dokumentation

Ich erstelle dir **zwei aktualisierte Versionen**:

---

## 1. **README.md** (Hauptdokumentation)

```markdown
# Campa — Modulares Kampagnen-Landingpage-System

> Dokumentation für Menschen und LLMs.
> Aktualisiert nach Bugfixes und Verbesserungen, März 2026.

---

## Was ist Campa?

Campa ist ein statisches Website-System für politische Kampagnen-Landingpages. Jede Kampagne ist eine eigene Seite mit eigenem Theme, eigenem Inhalt — aber gleichem technischen Fundament.

**Zielgruppe:** Technisch affine Person, die Inhalte selbst pflegt (via GitHub Web-Editor oder lokal), aber nicht für jede Kampagne von vorne anfangen will.

---

## Stack

| Komponente | Entscheidung | Begründung |
|---|---|---|
| Static Site Generator | Hugo | Kein Node auf dem Server nötig, sehr schnell |
| CSS | Tailwind v3 CLI (separat kompiliert) | Utility-first + kampagnenspezifische Themes |
| Fonts | Lokal (TTF in `static/fonts/`) | DSGVO-konform, kein Google Fonts |
| CI/CD | GitHub Actions → rsync | Einfach, kein Coolify (kollidiert mit SWAG+CrowdSec) |
| Hosting | Arch Linux VPS, SWAG + CrowdSec + Nginx Bouncer | Bestehende Infrastruktur |
| Node.js | Nur in GitHub Actions + lokal für CSS-Build, nie auf Server | Hugo braucht kein Node zur Laufzeit |

**Wichtig:** Tailwind v3 verwenden — v4 ist inkompatibel mit dem Setup.

---

## Projektstruktur

```
kampagnen-site/
├── .github/workflows/deploy.yml        ← CI/CD: Build + rsync zum Server
├── assets/css/main.css                 ← Tailwind-Quelldatei (Input für tailwindcss CLI)
├── static/css/
│   ├── main.css                        ← Generierte CSS (von npm run dev:css)
│   └── themes/kampagne-demo.css        ← CSS-Variablen pro Kampagne
├── layouts/
│   ├── _default/
│   │   ├── baseof.html                 ← HTML-Grundgerüst
│   │   ├── single.html                 ← Fallback für einzelne Seiten
│   │   └── legal.html                  ← Layout für Impressum/Datenschutz
│   ├── kampagnen/
│   │   └── legal.html                  ← Kopie (Hugo-Lookup-Fix)
│   ├── index.html                      ← Iteriert über blocks:-Liste aus _index.md
│   └── partials/
│       ├── nav.html
│       ├── footer.html
│       ├── hero.html
│       ├── back-to-top.html            ← ↑ Pfeil, wird in jeden Block eingebunden
│       ├── block-textblock.html
│       ├── block-slider.html           ← Scroll-Snap Slider mit Prev/Next + Dots
│       ├── block-faq.html
│       ├── block-cta.html              ← CTA-Banner mit flexibler Datenquelle
│       ├── block-logos.html            ← Logo-Leiste (dezent, grayscale)
│       ├── block-imagetext.html
│       ├── block-accordion.html
│       └── scripts.html                ← Slider-JS + FAQ-JS
├── content/
│   ├── _index.md                       ← HAUPT-KONFIGURATION (Hero, Blocks, Nav, Footer)
│   └── kampagnen/kampagne-demo/
│       ├── textblock1.md
│       ├── textblock2.md
│       ├── faq.md                      ← H3 = ein Aufklappelement
│       ├── impressum.md
│       └── datenschutz.md
├── static/
│   ├── fonts/                          ← TTF-Dateien (lokal gehostet)
│   └── images/kampagne-demo/           ← Bilder (header.jpg, slide1-4.jpg, logos)
├── hugo.toml                           ← KEIN theme="" (verursacht Fehler!)
├── tailwind.config.js
└── package.json                        ← tailwindcss@3 explizit!
```

---

## Lokale Entwicklung

```bash
# Node-Version setzen (v25+ ist inkompatibel mit Tailwind v3)
source /usr/share/nvm/init-nvm.sh
nvm use 20

# Abhängigkeiten installieren
npm install

# Terminal 1: CSS watch (kompiliert assets/css/main.css → static/css/main.css)
npm run dev:css

# Terminal 2: Hugo dev server (lädt CSS aus static/)
hugo server
```

Aufruf: http://localhost:1313

---

## Kampagne konfigurieren: `content/_index.md`

Die gesamte Kampagne wird über diese eine Datei gesteuert. Das ist die einzige Datei die Hugo als Startseite rendert.

```yaml
---
title: "Schule zeigt Haltung"
description: "Lehrkräfte stärken — Demokratie schützen."
kampagne: "kampagne-demo"           ← muss mit Ordnernamen übereinstimmen
theme_css: "kampagne-demo.css"      ← lädt static/css/themes/kampagne-demo.css

nav_links:
  - label: "Petition"
    url: "#petition"                ← Springt zu Block mit id: petition
  - label: "Forderungen"
    url: "#forderungen"

hero:
  eyebrow: "Mythos Neutralität"
  title: "Schule ist nicht neutral!"
  image: "/images/kampagne-demo/header.jpg"
  image_alt: "Header Bild"
  text: "Fließtext unter dem Titel."
  cta_label: "Petition unterzeichnen"
  cta_url: "https://weact.campact.de"

slider:
  title: "Die Petition"
  images:
    - src: "/images/kampagne-demo/slide1.jpg"
      alt: "Beschreibung"
      caption: "Bildunterschrift"

blocks:
  - type: textblock
    src: textblock1
    bg: light
    id: intro                       ← ID für Anker-Links (#intro)
  
  - type: slider
    id: petition                    ← ID muss hier stehen, nicht in slider:!
  
  - type: cta
    id: kontakt
    bg: accent                      ← Hintergrundfarbe wie bei anderen Blöcken
    title: "Jetzt Haltung zeigen"
    text: "Unterzeichnen Sie unsere Petition."
    button:
      label: "Zur Petition"
      url: "https://weact.campact.de"
      color: primary

footer_links:
  - label: "Impressum"
    url: "/kampagnen/kampagne-demo/impressum"
  - label: "Datenschutz"
    url: "/kampagnen/kampagne-demo/datenschutz"
footer_text: "© 2026 Schule zeigt Haltung"
---
```

---

## Block-System

Blöcke werden in der `blocks:`-Liste in `_index.md` definiert. Reihenfolge = Darstellungsreihenfolge.

### Verfügbare Block-Typen

| type | Parameter | Beschreibung |
|---|---|---|
| `textblock` | `src`, `bg`, `id` | Markdown-Datei als Textblock |
| `slider` | `id` | Bildslider aus `slider.images` in _index.md |
| `faq` | `src`, `title`, `bg`, `id` | FAQ aus Markdown (H3 = Frage) |
| `logos` | `title`, `bg`, `items[]` | Logo-Leiste (dezent, grayscale) |
| `cta` | `id`, `bg`, `title`, `text`, `button{}` | CTA-Banner (Daten im Block oder global) |
| `imagetext` | `src`, `image`, `bg`, `id`, `button{}` | Bild + Text nebeneinander |
| `accordion` | `title`, `bg`, `items[]` | Aufklapp-Elemente |

### Hintergrundfarben (`bg`)

| Wert | Farbe | Überschriften | Body |
|---|---|---|---|
| `light` | Warmes Grau | Lila | Dunkel |
| `white` | Reines Weiß | Lila | Dunkel |
| `dark` | Lila | Gelb | Weiß |
| `accent` | Gelb | Lila | Dunkelgrau |

### Anker-Links (Navigation)

**IDs müssen im `blocks:`-Array stehen:**

```yaml
# ✅ RICHTIG
blocks:
  - type: slider
    id: petition      # ← Wird zu <section id="petition">

nav_links:
  - label: "Petition"
    url: "#petition"  # ← Springt zur Section

# ❌ FALSCH
slider:
  id: petition        # ← Hugo findet das nicht!

blocks:
  - type: slider      # ← Keine ID!
```

---

## Theme-Variablen: `static/css/themes/kampagne-demo.css`

Jede Kampagne hat ihre eigene Theme-Datei mit CSS Custom Properties:

```css
@font-face {
  font-family: 'Permanent Marker';
  src: url('/fonts/PermanentMarker-Regular.ttf') format('truetype');
  font-display: swap;
}

:root {
  --color-primary:     #5815a0;   /* Lila — Hauptfarbe */
  --color-accent:      #e5e01d;   /* Gelb — Akzent */
  --color-bg:          #f8f7f4;   /* Warmes Grau */
  --color-text:        #1a1a1a;
  --color-text-light:  #555555;
  --color-border:      #d0ccc4;
  --font-heading:      'Permanent Marker', cursive;
  --font-body:         'Liter', system-ui, sans-serif;
}
```

---

## CSS-Pipeline verstehen

```
┌─────────────────────────────────────────────────────────────┐
│  assets/css/main.css                                         │
│  - Tailwind Direktiven (@tailwind base, components, etc.)   │
│  - Alle globalen Styles (.cta-banner, .section-*, etc.)     │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
              ┌────────────────────────┐
              │  npm run dev:css       │
              │  (Tailwind CLI v3)     │
              └────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  static/css/main.css                                         │
│  - Kompilierte CSS (Tailwind + Custom Styles)               │
│  - Diese Datei wird von Hugo geladen                         │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
              ┌────────────────────────┐
              │  baseof.html           │
              │  <link href="/css/     │
              │    main.css">          │
              └────────────────────────┘
```

**Wichtig:**
- `assets/css/main.css` ist die **Quelldatei** (editieren!)
- `static/css/main.css` ist **generiert** (nicht von Hand ändern!)
- Hugo lädt die CSS **direkt** aus `static/` (kein PostCSS in Hugo!)

---

## Inhalte bearbeiten

### Textblock ändern
Datei `content/kampagnen/kampagne-demo/textblock1.md` bearbeiten.
Markdown wird direkt gerendert. H2 = Abschnittstitel, H3 = Unterabschnitt.

### FAQ bearbeiten
Datei `content/kampagnen/kampagne-demo/faq.md` bearbeiten.
**Jedes H3 (`###`) wird zu einem Aufklappelement.**

```markdown
### Was bedeutet Neutralitätsgebot?
Antworttext hier...

### Darf ich politische Aussagen machen?
Weitere Antwort...
```

### Bilder tauschen
Datei mit gleichem Namen überschreiben in `static/images/kampagne-demo/`.
- `header.jpg` — Hero-Bild (Querformat empfohlen)
- `slide1.jpg` bis `slide4.jpg` — Slider-Bilder (4:5 Hochformat, z.B. 1200×1500px)
- Logos als PNG mit Transparenz (werden automatisch grayscale und auf dunklem BG invertiert)

---

## CTA-Block: Flexible Datenquellen

Der CTA-Block kann Daten aus **zwei Stellen** beziehen:

### Variante 1: Daten direkt im Block (empfohlen)
```yaml
blocks:
  - type: cta
    id: kontakt
    bg: accent
    title: "Jetzt Haltung zeigen"
    text: "Unterzeichnen Sie unsere Petition."
    button:
      label: "Zur Petition"
      url: "https://example.com"
      color: primary
```

### Variante 2: Separates globales `cta:`-Objekt
```yaml
cta:
  title: "Jetzt Haltung zeigen"
  text: "..."
  button_label: "Zur Petition"
  button_url: "https://example.com"
  button_color: "primary"

blocks:
  - type: cta
    id: kontakt    # Nur ID, keine Inhalte
```

**Das Partial prüft automatisch:** Gibt es `title` im Block? → Block hat Vorrang. Sonst: Nutze globales `cta:`.

---

## Deployment

### GitHub Actions Secrets

| Secret | Inhalt |
|---|---|
| `SSH_PRIVATE_KEY` | Privater SSH-Key (ed25519) |
| `REMOTE_HOST` | Server IP oder Hostname |
| `REMOTE_USER` | `campa` |
| `REMOTE_PATH` | `/var/www/campa/kampagne-demo/` |

### Server-Setup (einmalig)

```bash
# Dedizierter Deploy-User ohne sudo
useradd --system --no-create-home --shell /usr/sbin/nologin campa
groupadd campa
usermod -aG campa stef

# Verzeichnis
mkdir -p /var/www/campa/kampagne-demo
chown -R campa:campa /var/www/campa/
chmod -R 775 /var/www/campa/
chmod g+s /var/www/campa/

# SSH-Key hinterlegen (nur rsync erlaubt)
# authorized_keys mit command="" restriction
```

### SWAG Nginx-Config (Beispiel)
```nginx
server {
    listen 443 ssl;
    server_name kampagne.example.com;

    root /var/www/campa/kampagne-demo;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

## Bekannte Fallstricke & Lösungen

| Problem | Ursache | Fix |
|---|---|---|
| Hugo-Fehler `theme not found` | `theme = ""` in hugo.toml | Zeile komplett entfernen |
| Tailwind generiert kein CSS | Node v25+ inkompatibel | `nvm use 20` |
| CSS greift nicht | `npm run dev:css` vergessen | Terminal mit Watch starten |
| Änderungen in `assets/css/main.css` greifen nicht | `npm run dev:css` läuft nicht | Neu starten, warten bis "Done" |
| Nav-Link springt nicht | ID steht in `slider:` statt `blocks:` | ID in den Block verschieben |
| CTA wird nicht angezeigt | Daten fehlen oder falsches Format | Block-Daten haben Vorrang über globales `cta:` |
| CTA hat Streifen | `.cta-banner::before` in main.css | Pattern entfernen/auskommentieren |
| CTA-Abstände zu groß | `padding: 5rem 2rem` | Auf `2rem 2rem` oder `1.5rem 2rem` reduzieren |
| Slider zeigt mehrere Bilder | `overflow: hidden` fehlt | In main.css prüfen |
| Hugo findet `legal.html` nicht | Lookup-Reihenfolge bei Unterseiten | Kopie in `layouts/kampagnen/` anlegen |

---

## Debugging-Checkliste

### CSS wird nicht geladen?
1. Läuft `npm run dev:css`? → Check Terminal 1
2. Existiert `static/css/main.css`? → `ls static/css/`
3. Browser DevTools → Network → CSS-Dateien sichtbar?

### Anker-Link funktioniert nicht?
1. Browser DevTools → Elements → Suche nach `id="petition"`
2. Falls nicht gefunden: ID in `blocks:` verschieben (nicht in `slider:`)
3. Hugo neu starten

### Block wird nicht gerendert?
1. `blocks:`-Liste in `content/_index.md` prüfen
2. Typ richtig geschrieben? (`textblock` nicht `text-block`)
3. Hugo-Terminal: Fehlermeldungen?

---

## Neue Kampagne anlegen (Kurzanleitung)

1. `content/kampagnen/neue-kampagne/` anlegen mit allen MD-Dateien
2. `static/images/neue-kampagne/` anlegen, Bilder hochladen
3. `static/css/themes/neue-kampagne.css` anlegen (Farben + Fonts)
4. `static/fonts/` — Fonts hinterlegen falls neu
5. `content/_index.md` — `kampagne:` und `theme_css:` anpassen, gesamten Inhalt neu befüllen
6. GitHub Actions Secret `REMOTE_PATH` auf neues Verzeichnis setzen

---

## Für LLMs: Kontext und Entscheidungshistorie

Dieses System wurde sokratisch erarbeitet. Die wichtigsten Architekturentscheidungen:

- **Warum Hugo statt Next.js/Astro?** Kein Node auf dem Server, extrem schnell, einfaches Deployment via rsync.
- **Warum Tailwind CLI statt Hugo PostCSS?** Unabhängigkeit — CSS wird separat kompiliert, Hugo lädt nur fertiges CSS. Kein PostCSS-Setup in Hugo nötig.
- **Warum kein Coolify?** Kollidiert mit bestehendem SWAG+CrowdSec+Nginx Bouncer Stack.
- **Warum eine Root `_index.md` statt Kampagnen-Unterordner als Einstieg?** Hugo rendert immer `content/_index.md` als Startseite.
- **Warum Tailwind v3?** v4 hatte Inkompatibilitäten mit dem Node v20 Setup.
- **Warum lokale Fonts?** DSGVO — kein Laden externer Ressourcen.
- **Warum Scroll-Snap Slider statt JS-Slider?** Nativer Browser-Support, funktioniert auch ohne JS, besser auf Mobile.

Das Block-System ist bewusst einfach gehalten: Jeder Block-Typ ist ein separates Hugo-Partial. Neue Block-Typen einfach als `block-xyz.html` anlegen und in `layouts/index.html` registrieren.

### Lessons Learned (März 2026)

1. **IDs für Anker-Links:** Müssen immer im `blocks:`-Array stehen, nicht in den Daten-Objekten (`slider:`, `cta:`).
2. **Context-Übergabe:** Manche Partials nutzen `(dict "block" . "page" $)` (klein), andere `(dict "Block" . "Page" $)` (groß) — konsistent halten!
3. **CTA-Block:** Kann Daten aus Block ODER globalem `cta:`-Objekt beziehen — flexible Architektur.
4. **CSS-Pipeline:** `assets/` → Tailwind CLI → `static/` → Hugo lädt direkt. Kein PostCSS in Hugo!
5. **Styling-Defaults:** Nutzer wollen oft kompaktere Abstände — `padding: 2rem` statt `5rem` für Banners.
```

---

## 2. **Ergänzung zu KONTEXT.md** (Lessons Learned Section)

Am Ende der bestehenden KONTEXT.md anfügen:

```markdown
---

## Lessons Learned & Bugfixes (März 2026)

Diese Sektion dokumentiert Probleme, die im Dialog aufgetreten sind, und wie sie gelöst wurden.

### 1. Anker-Links funktionieren nicht

**Problem:** Navigation mit `url: "#petition"` springt nicht zum Slider.

**Ursache:** Die `id` stand im falschen Objekt:
```yaml
# ❌ FALSCH
slider:
  id: petition      # Hugo findet das nicht!

blocks:
  - type: slider    # Keine ID!
```

**Lösung:** IDs gehören in das `blocks:`-Array:
```yaml
# ✅ RICHTIG
blocks:
  - type: slider
    id: petition    # Wird zu <section id="petition">
```

**Pattern für Partials:**
```html
{{ $block := .Block }}
<section {{ with $block.id }}id="{{ . }}"{{ end }}>
```

---

### 2. CSS-Änderungen greifen nicht

**Problem:** Änderungen in `assets/css/main.css` erscheinen nicht im Browser.

**Ursache:** Zwei mögliche Gründe:
1. `npm run dev:css` läuft nicht → CSS wird nicht kompiliert
2. Hugo lädt alte CSS aus Cache oder falscher Quelle

**Lösung:**
1. **Prüfen:** Läuft `npm run dev:css` im Terminal?
2. **Warten:** Tailwind muss "Done" ausgeben nach Änderungen
3. **Browser:** Hard-Refresh (`Cmd+Shift+R` / `Ctrl+Shift+R`)
4. **Prüfen:** Existiert `static/css/main.css` und ist aktuell?

**CSS-Pipeline nochmal:**
```
assets/css/main.css  →  npm run dev:css  →  static/css/main.css  →  Browser
```

---

### 3. CTA-Block wird nicht angezeigt

**Problem:** Block vom Typ `cta` rendert nicht.

**Ursache:** Das Partial `block-cta.html` suchte nur nach `$page.Params.cta`, aber die Daten standen im Block selbst.

**Lösung:** Flexibles Daten-Pattern implementiert:
```html
{{ $cta := $page.Params.cta }}
{{ if $block.title }}
  {{ $cta = $block }}  {{/* Block-Daten haben Vorrang */}}
{{ end }}
```

Jetzt funktionieren beide Varianten:

**Variante 1: Daten im Block**
```yaml
blocks:
  - type: cta
    title: "..."
    text: "..."
    button:
      label: "..."
      url: "..."
```

**Variante 2: Globales cta:-Objekt**
```yaml
cta:
  title: "..."
  button_label: "..."

blocks:
  - type: cta    # Keine Inhalte, nutzt globales Objekt
```

---

### 4. CTA-Block hat Hintergrundstreifen und zu große Abstände

**Problem:** CTA sieht überladen aus (Diagonalstreifen-Pattern, große Padding-Werte).

**Lösung:**

**a) Pattern entfernen in `assets/css/main.css`:**
```css
/* .cta-banner::before { ... }  ← Auskommentiert */
```

**b) Padding reduzieren:**
```css
.cta-banner {
  padding: 2rem 2rem;  /* Statt 5rem 2rem */
}
```

**Empfehlung:** Standard-Padding für Banners auf `2rem` oder `3rem` setzen.

---

### 5. Context-Übergabe inkonsistent

**Problem:** Manche Partials nutzen `.block` (klein), andere `.Block` (groß).

**Beispiele:**
```html
{{/* Textblock, FAQ, etc. */}}
{{ partial "block-textblock.html" (dict "block" . "page" $) }}

{{/* Slider, CTA */}}
{{ partial "block-slider.html" (dict "Block" . "Page" $) }}
```

**Lösung:** Im Partial entsprechend anpassen:
```html
{{/* Für "block" (klein) */}}
{{ $block := .block }}
{{ $page := .page }}

{{/* Für "Block" (groß) */}}
{{ $block := .Block }}
{{ $page := .Page }}
```

**Empfehlung für neue Blöcke:** Immer **Kleinbuchstaben** verwenden (`"block" . "page"`), konsistent mit bestehenden Blöcken.

---

### 6. Hintergrundfarben im CTA-Block

**Problem:** CTA hatte feste Hintergrundfarbe, alle anderen Blöcke unterstützen `bg: light/white/dark/accent`.

**Lösung:** Wrapper-Pattern wie bei Textblocks:
```html
{{ $bg := $block.bg | default "light" }}

<div class="section-{{ $bg }}" {{ with $block.id }}id="{{ . }}"{{ end }}>
  <section class="cta-banner">
    <!-- Inhalt -->
  </section>
  {{ partial "back-to-top.html" . }}
</div>
```

**Wichtig:** ID verschiebt sich vom `<section>` zum Wrapper-`<div>`!

---

### 7. Warum static/css/main.css und nicht Hugo PostCSS?

**Frage:** Warum nicht Hugo's `resources.Get | postCSS`?

**Antwort:** Design-Entscheidung für dieses Setup:
- **Kein Node auf dem Server** gewünscht
- **Tailwind CLI separat** = unabhängig von Hugo
- **Einfacheres Deployment** = nur HTML/CSS/Images syncen

**Alternative (mit Hugo PostCSS):**
```html
<!-- baseof.html -->
{{ $styles := resources.Get "css/main.css" | postCSS | minify | fingerprint }}
<link rel="stylesheet" href="{{ $styles.RelPermalink }}">
```

**Voraussetzungen:**
- PostCSS + Plugins installiert (`npm install postcss postcss-cli autoprefixer`)
- `postcss.config.js` im Root
- Hugo muss PostCSS binary finden

**Unser Setup:** Simpler und funktioniert zuverlässig ohne Hugo-PostCSS-Integration.

---

### 8. Button-Struktur-Varianten

**Problem:** Manche Blöcke nutzen `button.label`, andere `button_label`.

**Aktuell unterstützt:**

**Neue Struktur (bevorzugt):**
```yaml
button:
  label: "Text"
  url: "https://..."
  color: primary
```

**Alte Struktur (kompatibel):**
```yaml
button_label: "Text"
button_url: "https://..."
button_color: "primary"
```

**Partials prüfen beide:**
```html
{{ if .button }}
  <a href="{{ .button.url }}">{{ .button.label }}</a>
{{ else if .button_label }}
  <a href="{{ .button_url }}">{{ .button_label }}</a>
{{ end }}
```

**Empfehlung:** Neue Kampagnen sollten nested `button:`-Struktur nutzen.

---

## Für künftige Bugfixes: Debug-Pattern

### 1. Hugo rendert Block nicht?
- Prüfe `blocks:`-Liste in `_index.md`
- Ist `type:` richtig geschrieben?
- Partial existiert in `layouts/partials/`?
- Hugo-Terminal: Fehlermeldungen?

### 2. ID wird nicht gesetzt?
- Browser DevTools → Elements → Suche `id="xyz"`
- Falls fehlt: Steht ID im Block (nicht im Daten-Objekt)?
- Context-Übergabe korrekt? `(dict "Block" . "Page" $)`
- Partial nutzt `{{ with $block.id }}id="{{ . }}"{{ end }}`?

### 3. CSS-Änderung greift nicht?
- `npm run dev:css` läuft?
- Änderung in `assets/css/main.css` (nicht `static/css/main.css`)?
- Tailwind hat kompiliert ("Done" im Terminal)?
- Browser Hard-Refresh?

### 4. Neue Block-Typen anlegen?
- Partial in `layouts/partials/block-xyz.html` erstellen
- In `layouts/index.html` registrieren:
  ```html
  {{ else if eq $type "xyz" }}
    {{ partial "block-xyz.html" (dict "block" . "page" $) }}
  ```
- Context-Pattern von bestehenden Blöcken übernehmen
- ID-Support einbauen: `{{ with $block.id }}i