# Campa — Kampagnen-Landingpage "Schule zeigt Haltung"

> Hugo-basierte Landingpage für die GEW-Kampagne "Schule zeigt Haltung"  
> Statische Website mit modularem Block-System

---

## 🎯 Was ist das?

**Campa** ist ein Static Site Generator für politische Kampagnen-Landingpages.

- **Technologie:** Hugo + Tailwind CSS
- **Bearbeitung:** Direkt auf GitHub.com (Web-Editor)
- **Deployment:** Automatisch bei Änderungen (~15 Sekunden)

---

## 📱 Für Redakteure: Inhalte bearbeiten

### Schnellanleitung

1. Öffne [github.com/GEW-Bund/campa](https://github.com/GEW-Bund/campa)
2. Navigiere zu `content/_index.md` oder `content/schule-zeigt-haltung/*.md`
3. Klicke auf **Bleistift-Symbol** (Edit this file)
4. Ändere den Text
5. **Commit changes** → Fertig!

Die Website wird automatisch aktualisiert.

### Die wichtigsten Dateien

| Datei | Zweck |
|-------|-------|
| `content/_index.md` | Hauptkonfiguration (Hero, Navigation, Blöcke) |
| `content/schule-zeigt-haltung/*.md` | Textblöcke (Impressum, FAQ, etc.) |
| `static/schule-zeigt-haltung/*.jpg` | Bilder |
| `static/css/themes/schule-zeigt-haltung.css` | Theme-Farben |

**📖 Ausführliche Anleitung:** [DETAILS.md](DETAILS.md)

---

## 💻 Für Entwickler: Lokale Entwicklung

### Quick Start

```bash
# 1. Repository klonen
git clone https://github.com/GEW-Bund/campa.git
cd campa

# 2. Dependencies installieren
npm install

# 3. Dev-Server starten (2 Terminals)
npm run dev:css    # Terminal 1: Tailwind CSS Watch
hugo server        # Terminal 2: Hugo Dev Server
```

**Öffne:** http://localhost:1313

### Prerequisites

- Node.js v20+ (v25 empfohlen)
- Hugo Extended v0.100+
- NVM (optional)

**📖 Technische Details:** [DEVELOPMENT.md](DEVELOPMENT.md)

---

## 📚 Dokumentation

| Datei | Für wen? | Inhalt |
|-------|----------|--------|
| **[DETAILS.md](DETAILS.md)** | Redakteure | Ausführliche Bearbeitungs-Anleitung |
| **[DEVELOPMENT.md](DEVELOPMENT.md)** | Entwickler | Setup, Architektur, Troubleshooting |
| **[LLM.md](LLM.md)** | AI/LLMs | Vollständige technische Dokumentation |

---

## 🏗️ Struktur (Überblick)

```
campa/
├── content/
│   ├── _index.md                      # Hauptkonfiguration
│   └── schule-zeigt-haltung/          # Markdown-Inhalte
├── static/
│   ├── schule-zeigt-haltung/          # Bilder
│   └── css/themes/                    # Theme-CSS
├── layouts/                           # Hugo-Templates
└── assets/css/                        # Tailwind-Quelle
```

---

## 📄 Lizenz & Credits

- **Hugo:** Apache 2.0 License
- **Tailwind CSS:** MIT License
- **Entwickelt für:** Politische Kampagnenarbeit (GEW)

**Repository:** https://github.com/GEW-Bund/campa
