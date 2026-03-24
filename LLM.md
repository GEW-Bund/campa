# Campa — LLM Context & Technical Documentation

> Vollständige technische Dokumentation für Large Language Models.
> Dokumentiert alle Architektur-Entscheidungen, Lessons Learned und Debugging-Patterns.
> Stand: März 2026 | Aktualisiert nach Kampagne-Refactoring

---

## Executive Summary

**Campa** ist ein Hugo-basiertes Static Site Generator System für politische Kampagnen-Landingpages. 

**Aktuelle Kampagne:** "Schule zeigt Haltung" (GEW)

**Kern-Philosophie:** Hybrid-Architektur mit Build-Tools auf Host und nur Services in Docker.

**Deployment-Methode:** GitHub Webhook → Host systemd-Service → Shell-Script (git pull + CSS build + Hugo build)

---

## 🆕 Wichtige Änderungen (März 2026)

### Content-Struktur Vereinfachung

**Alt (bis März 2026):**
```
content/kampagnen/kampagne-demo/
static/images/kampagne-demo/
```

**Neu (ab März 2026):**
```
content/schule-zeigt-haltung/
static/schule-zeigt-haltung/
```

**Grund:** Vereinfachung der URL-Struktur. Die aktuelle Kampagne läuft unter Root (`/`), nicht unter `/kampagnen/`.

---

### Template-Pfad-Migration (März 2026)

**Problem:** Block-Partials hatten hardcodierten Pfad `/kampagnen/%s/%s`

**Betroffene Dateien:**
- `layouts/partials/block-textblock.html` (Zeile 6)
- `layouts/partials/block-imagetext.html` (Zeile 5)
- `layouts/partials/block-faq.html` (Zeile 8)
- `layouts/partials/block-accordion.html` (Zeile 11)

**Fix:**
```go
// ❌ ALT
{{ $content := $page.Site.GetPage (printf "/kampagnen/%s/%s" $kampagne $src) }}

// ✅ NEU
{{ $content := $page.Site.GetPage (printf "/%s/%s" $kampagne $src) }}
```

**Grund:** Unterstützt neue flache Content-Struktur ohne `/kampagnen/` Zwischenebene.

**Auswirkung:** Hugo findet Content-Dateien jetzt unter `content/schule-zeigt-haltung/` statt `content/kampagnen/kampagne-demo/`.

---

## Finale Architektur (März 2026)

### Stack

| Komponente | Technologie | Wo | Warum |
|------------|-------------|-----|-------|
| Static Site Generator | Hugo Extended | Host | Single Binary, keine Dependencies |
| CSS Framework | Tailwind CSS v3 (CLI) | Host (via Node) | Kompiliert assets/ → static/ |
| Node.js | v25 (via NVM) | Host | Für Tailwind CSS Build |
| Git | System Git | Host | Repository Management |
| Webhook-Listener | webhook binary (Go) | Host (systemd) | Empfängt GitHub POST |
| Web-Server | Nginx (optional SWAG) | Docker oder Host | Liefert Static Files |
| Deployment-Trigger | GitHub Webhook | Cloud | HTTPS POST bei Push |

### Deployment-Flow

```
GitHub.com Edit → Push to main
         ↓
GitHub Webhook (HMAC-signiert)
         ↓
Nginx (SSL) → Port 9000 (localhost)
         ↓
systemd webhook.service
         ↓
deploy.sh ausführen:
  - git pull
  - npm run build:css
  - hugo --minify
         ↓
Static Files in /var/www/
         ↓
Nginx liefert aus
```

**Reaktionszeit:** ~10-20 Sekunden

---

## Verzeichnisstruktur (Production)

```
/home/$USER/
├── repos/
│   └── kampagnen-site/                 ← Git-Repo (HTTPS clone)
│       ├── content/
│       │   ├── _index.md               ← Hauptkonfiguration (wird als / gerendert!)
│       │   └── schule-zeigt-haltung/   ← Content-Partials (via src: ...)
│       ├── layouts/partials/           ← Block-Templates
│       ├── static/
│       │   ├── schule-zeigt-haltung/   ← Bilder
│       │   └── css/themes/schule-zeigt-haltung.css
│       └── assets/css/main.css         ← Tailwind-Quelle
│
├── webhook/
│   ├── config/
│   │   ├── hooks.json                  ← Webhook-Konfiguration
│   │   └── deploy.log                  ← Deploy-Logs
│   └── scripts/
│       └── deploy.sh                   ← Deploy-Script
│
└── .nvm/                               ← NVM-Installation
    └── versions/node/v25.8.1/

/var/www/kampagnen-site/                ← Hugo Output (public/)
    ├── index.html
    ├── css/main.css                    ← Kompilierte Tailwind-CSS
    ├── schule-zeigt-haltung/           ← Images
    └── fonts/

/etc/systemd/system/
└── webhook.service                     ← systemd-Service

/etc/nginx/
└── sites-available/
    └── kampagne.conf                   ← Nginx-Config
```

---

## Hugo-Spezifika

### content/_index.md ist die Startseite!

**Hugo rendert `content/_index.md` als Startseite** (`/index.html`).

**Wichtig:** Andere `_index.md` Dateien in Unterordnern (z.B. `content/schule-zeigt-haltung/_index.md`) werden **NICHT** als separate Seiten gerendert, sondern nur als **Datenquelle** für Metadaten.

**Content-Struktur:**
```
content/
├── _index.md                         ← WIRD GERENDERT als /
└── schule-zeigt-haltung/
    ├── _index.md                     ← Nur Metadaten (nicht gerendert)
    ├── verunsicherung.md             ← Partial (via src: verunsicherung)
    ├── forderungen.md                ← Partial (via src: forderungen)
    └── ...                           ← Weitere Partials
```

### Content Lookup in Block-Partials

**Alle Block-Partials nutzen:**
```go
{{ $kampagne := $page.Params.kampagne | default "schule-zeigt-haltung" }}
{{ $src := $block.src }}
{{ $content := $page.Site.GetPage (printf "/%s/%s" $kampagne $src) }}
```

**Beispiel:**
- In `content/_index.md`: `kampagne: "schule-zeigt-haltung"`
- Block definiert: `src: "verunsicherung"`
- Hugo lädt: `content/schule-zeigt-haltung/verunsicherung.md`

### Block-System

**Reihenfolge in `blocks:`-Liste = Darstellungsreihenfolge auf der Seite!**

```yaml
# In content/_index.md
blocks:
  - type: textblock
    src: verunsicherung     # ← Hugo lädt content/schule-zeigt-haltung/verunsicherung.md
    bg: white
    
  - type: slider
    id: petition
    
  - type: cta
    title: "Jetzt Haltung zeigen!"
```

### Anker-IDs für Navigation

**IDs müssen im `blocks:`-Array stehen:**

```yaml
nav_links:
  - label: "Petition"
    url: "#petition"        # ← Springt zu Block mit id: petition

blocks:
  - type: slider
    id: petition            # ← Diese ID wird angesprungen!
```

**Nicht** in den Datenstrukturen wie `slider:` oder `hero:`!

---

## Tailwind CSS Pipeline

```
assets/css/main.css (Quelle)
         ↓
npm run dev:css (Tailwind CLI Watch)
         ↓
static/css/main.css (Generiert, ~100KB)
         ↓
Hugo lädt direkt (kein PostCSS!)
```

**Wichtig:**
- ✅ `assets/css/main.css` = Quelle (editieren)
- ❌ `static/css/main.css` = Output (NICHT editieren, automatisch generiert)

---

## Architektur-Entscheidungen

### 1. Webhook auf Host statt Docker

**Grund:** Build-Tools (Git, Hugo, Node) sind besser auf Host als in Container.

**Vorteile:**
- Direkter Zugriff auf alle Tools
- Keine Docker Socket Risks
- Einfacher zu debuggen (journalctl)

### 2. Hugo auf Host statt Container

**Hugo = Single Binary (~90 MB), KEINE Runtime-Dependencies!**

**Community-Practice:** Hugo nativ auf Host, Docker nur für CI/CD.

### 3. GitHub Webhook statt GitHub Actions

**Vorteil:** Kein SSH-Key in GitHub Secrets, VPS hat volle Kontrolle.

**Use-Case:** Redakteure können auf GitHub.com im Web-Editor arbeiten.

---

## Kritische Dateien (Platzhalter-Beispiele)

### hooks.json

**Pfad:** `~/webhook/config/hooks.json`

```json
[
  {
    "id": "deploy-kampagne",
    "execute-command": "/absoluter/pfad/zu/deploy.sh",
    "response-message": "Deployment gestartet!",
    "trigger-rule": {
      "and": [
        {
          "match": {
            "type": "payload-hmac-sha256",
            "secret": "SECRET_STRING",
            "parameter": {
              "source": "header",
              "name": "X-Hub-Signature-256"
            }
          }
        },
        {
          "match": {
            "type": "value",
            "value": "refs/heads/main",
            "parameter": {
              "source": "payload",
              "name": "ref"
            }
          }
        }
      ]
    }
  }
]
```

**Wichtig:** 
- `execute-command` muss **absoluter Pfad** sein
- `secret` wird mit `openssl rand -hex 32` generiert (NICHT im Repo!)

### deploy.sh

**Pfad:** `~/webhook/scripts/deploy.sh`

```bash
#!/bin/bash
set -e

REPO_DIR="$HOME/repos/kampagnen-site"
WEB_DIR="/var/www/kampagnen-site"
LOG="$HOME/webhook/config/deploy.log"

echo "$(date): Deploy gestartet" >> "$LOG"

cd "$REPO_DIR" || exit 1
git pull origin main >> "$LOG" 2>&1

source ~/.nvm/nvm.sh || source /usr/share/nvm/init-nvm.sh
nvm use 25 >> "$LOG" 2>&1

npm install >> "$LOG" 2>&1
npm run build:css >> "$LOG" 2>&1

hugo --minify -d "$WEB_DIR" >> "$LOG" 2>&1

echo "$(date): Deploy erfolgreich!" >> "$LOG"
```

### systemd-Service

**Pfad:** `/etc/systemd/system/webhook.service`

```ini
[Unit]
Description=GitHub Webhook Server
After=network.target

[Service]
Type=simple
User=$USER
WorkingDirectory=/home/$USER/webhook
ExecStart=/usr/bin/webhook -hooks /home/$USER/webhook/config/hooks.json -verbose -port 9000
Restart=always

StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

### Nginx Config

**Webhook-Proxy (Beispiel):**
```nginx
location /webhook-GEHEIMER-PFAD/ {
    proxy_pass http://127.0.0.1:9000/hooks/deploy-kampagne;
    proxy_set_header Host $host;
    proxy_set_header X-Hub-Signature-256 $http_x_hub_signature_256;
}
```

**Wenn Nginx in Docker:**
```nginx
location /webhook-GEHEIMER-PFAD/ {
    proxy_pass http://172.17.0.1:9000/hooks/deploy-kampagne;  # Docker Bridge IP
    proxy_set_header X-Hub-Signature-256 $http_x_hub_signature_256;
}
```

**Wichtig:** 
- `172.17.0.1` = Standard Docker Bridge IP (öffentlich bekannt, kein Secret)
- Header `X-Hub-Signature-256` **muss** weitergeleitet werden!

---

## Lessons Learned

### 1. Docker ist nicht immer die Lösung

**Erkenntnis:** Docker für **Services** (24/7), nicht für Build-Tools (5 Sekunden).

### 2. Node v25 funktioniert mit Tailwind v3

**Test-Ergebnis:** v25.8.1 + Tailwind v3.4.x = ✅ funktioniert einwandfrei

### 3. Hugo ist ein Single Binary

**Konsequenz:** Kein Grund für Docker-Container, Binary auf Host ist simpler.

### 4. Template-Pfade müssen flexibel sein

**Lesson:** Hardcodierte Pfade (`/kampagnen/%s/`) machen Refactoring schwer.  
**Fix:** Dynamische Pfade (`/%s/%s`) erlauben flexible Content-Struktur.

### 5. Content-Lookup via GetPage ist performant

**Hugo cacht GetPage-Results**, daher keine Performance-Probleme bei vielen Blöcken.

### 6. NVM muss explizit geladen werden

**In non-interactive Shell:** `source ~/.nvm/nvm.sh` ist Pflicht!

### 7. Nginx-Header MÜSSEN weitergeleitet werden

**Ohne `proxy_set_header X-Hub-Signature-256`:** Webhook-Validierung schlägt fehl!

---

## Debugging-Strategien

### Webhook empfängt nichts
```bash
sudo systemctl status webhook
sudo ss -tlnp | grep 9000
curl http://localhost:9000/hooks/deploy-kampagne
```

### Webhook empfängt, aber triggert nicht
```bash
sudo journalctl -u webhook -n 50
# Suche nach "trigger rules were not satisfied"
```

### Build fehlschlägt
```bash
cat ~/webhook/config/deploy.log
# Häufig: "nvm: command not found" oder "hugo: command not found"
```

### Content wird nicht angezeigt
```bash
# Prüfe content/_index.md:
# - kampagne: "schule-zeigt-haltung"
# - blocks: src: "dateiname" (ohne .md)

# Prüfe dass Datei existiert:
ls -la content/schule-zeigt-haltung/dateiname.md
```

---

## Sicherheitsüberlegungen

### ✅ Gut implementiert

1. **HMAC SHA256 Secret-Validierung** (Webhook signiert)
2. **Branch-Whitelist** (nur `main` triggert)
3. **Versteckter URL-Pfad** (min. 32 Zeichen, zufällig)
4. **Non-Root Execution** (User-Service)
5. **Kein SSH-Key in GitHub** (VPS pullt Code)

### ⚠️ Potenzielle Risiken

1. **Secret-Leak** → Mitigation: Regelmäßig rotieren
2. **Webhook-URL-Leak** → Mitigation: Nicht im Repo committen
3. **Deploy-Script hat User-Rechte** → Mitigation: Separater Deploy-User
4. **Kein Rollback-Mechanismus** → Mitigation: Git-Tags + manuell

### 🔐 Best Practices

```bash
# Secret rotieren (alle 6 Monate)
openssl rand -hex 32

# Separater Deploy-User (empfohlen)
sudo useradd -m -s /bin/bash deploy
sudo chown -R deploy:deploy /var/www/kampagnen-site

# Logs prüfen
sudo journalctl -u webhook --since "1 week ago"
```

---

## Weitere Dokumentation

Für detaillierte Informationen siehe:

- **[README.md](README.md)** - Schnelleinstieg & Überblick
- **[DETAILS.md](DETAILS.md)** - Ausführliche Redaktions-Anleitung
- **[DEVELOPMENT.md](DEVELOPMENT.md)** - Entwickler-Guide & Troubleshooting

---

**Stand:** März 2026  
**Repository:** https://github.com/GEW-Bund/campa  
**Aktuelle Kampagne:** Schule zeigt Haltung (GEW)
