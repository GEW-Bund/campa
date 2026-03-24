# Redaktions-Handbuch: Inhalte bearbeiten

> Ausführliche Anleitung für Redakteure zur Bearbeitung der Kampagnen-Website

---

## 📝 Text auf GitHub.com ändern

### Schritt-für-Schritt-Anleitung

1. **GitHub öffnen**
   - Gehe zu [github.com/GEW-Bund/campa](https://github.com/GEW-Bund/campa)
   - Logge dich ein (falls nötig)

2. **Datei finden**
   - Navigiere zu `content/_index.md` (Hauptkonfiguration)
   - ODER zu `content/schule-zeigt-haltung/` (Textblöcke)
   - Klicke auf die gewünschte Datei

3. **Bearbeiten**
   - Klicke auf das **✏️ Bleistift-Symbol** oben rechts ("Edit this file")
   - Ändere den Text im Editor
   - **Vorschau:** Klicke auf "Preview" um deine Änderungen zu sehen

4. **Speichern**
   - Scrolle nach unten zu "Commit changes"
   - Schreibe eine **kurze Beschreibung** was du geändert hast
   - Klicke **"Commit changes"**

5. **Fertig!**
   - Die Website wird automatisch aktualisiert (~15 Sekunden)
   - Prüfe die Live-Site

---

## 📂 Datei-Übersicht

### Hauptkonfiguration: `content/_index.md`

Diese Datei steuert die **gesamte Startseite**:

- **Hero-Section** (Kopfbereich mit großem Bild)
- **Navigation** (Menü-Links)
- **Block-Reihenfolge** (Welche Blöcke in welcher Reihenfolge?)
- **Slider-Bilder**
- **Logo-Leiste**

**Beispiel:**
```yaml
---
title: "Schule zeigt Haltung"
kampagne: "schule-zeigt-haltung"

hero:
  title: "Schule ist nicht neutral"
  image: "/schule-zeigt-haltung/header.jpg"
  text: "Schulen sind keine neutralen Räume..."
  cta_label: "Petition unterzeichnen"
  cta_url: "https://example.com"
---
```

---

### Textblöcke: `content/schule-zeigt-haltung/*.md`

Diese Dateien enthalten die **Inhalte** der einzelnen Blöcke:

| Datei | Zweck |
|-------|-------|
| `verunsicherung.md` | Textblock "Verunsicherung" |
| `unterstuetzung.md` | Textblock "Unterstützung" |
| `forderungen.md` | Textblock "Forderungen" |
| `broschuere.md` | Text zur Broschüre |
| `kontakt.md` | Kontakt-Informationen |
| `faq.md` | FAQ (Häufige Fragen) |
| `faq-lit.md` | Literatur-Liste |
| `impressum.md` | Impressum |
| `datenschutz.md` | Datenschutzerklärung |
| `bildquellen.md` | Bildquellen |

**Beispiel `verunsicherung.md`:**
```markdown
---
title: "Verunsicherung im Schulalltag"
---

## Lehrkräfte erleben Verunsicherung

Viele Lehrkräfte sind unsicher, wie sie mit...
```

---

## 🎨 Block-System verstehen

### Was sind Blöcke?

Die Website besteht aus **Blöcken** die untereinander angeordnet sind:

```
┌────────────────────┐
│   Hero (Kopf)      │
├────────────────────┤
│   Textblock 1      │  ← type: textblock
├────────────────────┤
│   Bildslider       │  ← type: slider
├────────────────────┤
│   CTA-Button       │  ← type: cta
├────────────────────┤
│   FAQ              │  ← type: faq
└────────────────────┘
```

### Verfügbare Block-Typen

| Block-Typ | Was macht er? | Parameter |
|-----------|---------------|-----------|
| `textblock` | Textinhalt aus `.md` Datei | `src`, `bg`, `id` |
| `slider` | Bildkarussell | `id` |
| `cta` | Call-to-Action Button | `title`, `text`, `button` |
| `faq` | FAQ (Aufklappbar) | `src`, `title`, `bg`, `id` |
| `imagetext` | Bild + Text nebeneinander | `src`, `image`, `bg`, `button` |
| `logos` | Logo-Leiste | `items` |
| `accordion` | Aufklapp-Elemente | `items` |

### Reihenfolge ändern

Die Reihenfolge der Blöcke steht in `content/_index.md` unter `blocks:`:

```yaml
blocks:
  - type: textblock
    src: verunsicherung   # ← Wird als erstes angezeigt
    
  - type: slider          # ← Dann der Slider
    id: petition
    
  - type: cta             # ← Dann der CTA
    title: "Jetzt Haltung zeigen!"
```

**Um die Reihenfolge zu ändern:** Einfach die Blöcke verschieben!

### IDs für Anker-Links

Um in der Navigation zu Blöcken zu springen, brauchen sie eine `id`:

```yaml
nav_links:
  - label: "Petition"
    url: "#petition"      # ← Springt zu Block mit id: petition

blocks:
  - type: slider
    id: petition          # ← Diese ID wird angesprungen!
```

---

## 🖼️ Bilder verwalten

### Bild tauschen

1. **Neues Bild vorbereiten**
   - Benenne es **exakt wie das alte** (z.B. `header.jpg`)
   - Empfohlene Größen siehe unten

2. **Auf GitHub hochladen**
   - Gehe zu `static/schule-zeigt-haltung/`
   - Klicke **"Add file"** → **"Upload files"**
   - Wähle dein Bild
   - **Commit changes**

3. **GitHub überschreibt automatisch** das alte Bild

### Neues Bild hinzufügen

**Schritt 1:** Bild hochladen (wie oben)

**Schritt 2:** In `content/_index.md` referenzieren:

```yaml
hero:
  image: "/schule-zeigt-haltung/mein-neues-bild.jpg"
```

### Bildgrößen-Empfehlungen

| Verwendung | Breite | Format |
|------------|--------|--------|
| Hero-Bild | 1920px | JPG |
| Slider-Bilder | 1200px | JPG |
| Logos | 300px | WebP/PNG |
| Thumbnails | 600px | WebP/JPG |

**Formate:**
- **JPG** für Fotos
- **WebP** für kleinere Dateigröße
- **PNG** nur für Logos/Transparenz

---

## ❓ FAQ bearbeiten

### Markdown-Syntax

FAQ-Einträge in `content/schule-zeigt-haltung/faq.md` werden so formatiert:

```markdown
### Was ist das Beutelsbach-Kontroversitätsgebot?

Das Kontroversitätsgebot besagt, dass...

### Darf ich als Lehrkraft politische Themen ansprechen?

Ja, politische Bildung ist Teil des Bildungsauftrags...
```

**Wichtig:** 
- Jede Frage = `###`-Überschrift
- Antwort direkt darunter (normaler Text)
- Eine Leerzeile zwischen Fragen

### Neue Frage hinzufügen

Einfach am Ende der Datei hinzufügen:

```markdown
### Meine neue Frage hier?

Die Antwort kommt hier hin.
```

---

## 🔧 Häufige Änderungen

### Hero-Text ändern

**Datei:** `content/_index.md`

```yaml
hero:
  title: "Hier deinen neuen Titel"
  text: "Hier deinen neuen Text"
  cta_label: "Button-Text"
  cta_url: "https://deine-url.de"
```

### Navigation anpassen

**Datei:** `content/_index.md`

```yaml
nav_links:
  - label: "Petition"
    url: "#petition"
  - label: "Forderungen"  
    url: "#forderungen"
  - label: "Neuer Link"    # ← Hinzufügen
    url: "#neu"
```

### CTA-Button ändern

**Datei:** `content/_index.md`

Such nach `type: cta` in den `blocks`:

```yaml
- type: cta
  title: "Neuer Titel!"
  text: "Neuer Text"
  button:
    label: "Button-Text"
    url: "https://neue-url.de"
    target: "_blank"
```

### Footer-Text ändern

**Datei:** `content/_index.md`

Ganz unten:

```yaml
footer_text: "© 2026 Dein neuer Footer-Text"
```

---

## 🎨 Farben & Theme ändern

**Datei:** `static/css/themes/schule-zeigt-haltung.css`

```css
:root {
  --color-primary:     #1a3a5c;   /* Hauptfarbe (z.B. Buttons) */
  --color-accent:      #e63946;   /* Akzentfarbe (z.B. Highlights) */
  --color-bg:          #f8f7f4;   /* Hintergrundfarbe */
  --color-text:        #1a1a1a;   /* Textfarbe */
}
```

**Einfach die Hex-Codes ändern!**

---

## 🐛 Häufige Probleme

### "Meine Änderungen sind nicht sichtbar"

**Lösung:**
1. Warte 15-20 Sekunden nach dem Commit
2. Hard-Refresh im Browser: `Cmd+Shift+R` (Mac) oder `Ctrl+Shift+R` (Windows)
3. Cache leeren

### "Block wird nicht angezeigt"

**Checkliste:**
- ✅ Steht der Block in `blocks:` Liste?
- ✅ Ist `type:` richtig geschrieben? (z.B. `textblock` nicht `text-block`)
- ✅ Existiert die referenzierte Datei? (z.B. `src: forderungen` → `forderungen.md`)

### "Anker-Link funktioniert nicht"

**Lösung:** `id` muss im Block stehen, nicht woanders!

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

## 📞 Hilfe & Support

**Bei Problemen:**
1. Prüfe dieses Handbuch
2. Schau in [DEVELOPMENT.md](DEVELOPMENT.md) (Troubleshooting)
3. Kontaktiere das Tech-Team

**Für Entwickler:** Siehe [DEVELOPMENT.md](DEVELOPMENT.md) und [LLM.md](LLM.md)
