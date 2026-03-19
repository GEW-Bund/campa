
# Campa — Modulares Kampagnen-Landingpage-System

> Static Site Generator für politische Kampagnen. Gebaut mit Hugo + Tailwind CSS.
> Deployment via GitHub Webhooks auf eigenen Server.

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
| Hosting | Linux VPS + Nginx |
| Node.js | Lokal + auf Server (für CSS Build) |

### Deployment-Architektur

```
GitHub Push (main)
    ↓ Webhook (HTTPS POST)
Server: Webhook-Service (systemd)
    ↓ triggert
deploy.sh (auf Host):
  - git pull
  - npm run build:css (Tailwind)
  - hugo --minify
    ↓
Nginx liefert Static Files aus
```

**Alle Build-Tools laufen auf dem Host, nicht in Containern!**

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
|-----------|-----------|--------------|
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

## Deployment auf VPS

### Architektur

```
1. Redakteur committet auf GitHub
2. GitHub sendet Webhook-POST
3. Server empfängt via Webhook-Service (systemd)
4. Script führt aus: git pull → CSS build → Hugo build
5. Nginx liefert aktualisierte Seite aus
```

### Server-Voraussetzungen

- Linux-Server (Arch/Ubuntu/Debian)
- Git
- Hugo Extended
- Node.js + NVM
- Nginx (oder Apache)
- webhook binary

### Setup-Schritte (Kurzfassung)

1. **Verzeichnisse anlegen**
   ```bash
   mkdir -p ~/repos ~/webhook/{config,scripts}
   ```

2. **Repository klonen**
   ```bash
   cd ~/repos
   git clone https://github.com/DEINE-ORG/kampagnen-site.git
   ```

3. **Webhook-Secret generieren**
   ```bash
   openssl rand -hex 32
   ```

4. **Webhook-Config** (`~/webhook/config/hooks.json`)
   ```json
   [{
     "id": "deploy-kampagne",
     "execute-command": "/home/$USER/webhook/scripts/deploy.sh",
     "trigger-rule": {
       "match": {
         "type": "payload-hmac-sha256",
         "secret": "DEIN_SECRET",
         "parameter": {
           "source": "header",
           "name": "X-Hub-Signature-256"
         }
       }
     }
   }]
   ```

5. **Deploy-Script** (`~/webhook/scripts/deploy.sh`)
   ```bash
   #!/bin/bash
   set -e
   cd ~/repos/kampagnen-site
   git pull origin main
   source ~/.nvm/nvm.sh  # Oder: /usr/share/nvm/init-nvm.sh
   nvm use 25
   npm install
   npm run build:css
   hugo --minify -d /var/www/meine-site
   ```

6. **systemd-Service** einrichten
7. **Nginx konfigurieren** (Webhook-Endpoint + Static Files)
8. **GitHub Webhook** einrichten

**Vollständige Anleitung:** Siehe DEPLOYMENT.md

---

## Projektstruktur

```
kampagnen-site/
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

**Projekt-Repository:** [GitHub Link hier]
```

---

## DEPLOYMENT.md (Neu - Vollständige Anleitung)

```markdown
# Deployment-Anleitung

> Vollständige Anleitung zum Deployment der Campa-Site auf einem VPS

---

## Übersicht

### Architektur

```
GitHub Repository
    ↓ Push auf main-Branch
GitHub sendet Webhook (HTTPS POST)
    ↓
VPS: Nginx empfängt Request
    ↓ proxy_pass
VPS: Webhook-Service (systemd)
    ↓ validiert Secret
    ↓ führt aus
deploy.sh:
  1. git pull origin main
  2. npm install && npm run build:css
  3. hugo --minify -d /var/www/
    ↓
Nginx liefert Static Files aus
```

**Alle Build-Tools auf Host, kein Docker für Builds!**

---

## Server-Voraussetzungen

### System
- Linux-Server (Arch Linux, Ubuntu, Debian)
- Root-Zugriff oder sudo
- Domain mit DNS auf Server-IP zeigend

### Software

```bash
# Arch Linux
sudo pacman -S git hugo nodejs npm nvm nginx webhook

# Ubuntu/Debian
sudo apt install git nginx
# Hugo: siehe https://gohugo.io/installation/
# NVM: siehe https://github.com/nvm-sh/nvm
# webhook: siehe https://github.com/adnanh/webhook
```

---

## Setup Schritt für Schritt

### 1. Verzeichnisstruktur anlegen

```bash
# Als dein normaler User (nicht root!)
mkdir -p ~/repos
mkdir -p ~/webhook/{config,scripts}
mkdir -p /var/www/deine-site  # Oder wo auch immer Nginx Root ist
```

**Verzeichnisbaum:**
```
/home/$USER/
├── repos/
│   └── kampagnen-site/          ← Git-Repo
└── webhook/
    ├── config/
    │   ├── hooks.json           ← Webhook-Config
    │   └── deploy.log           ← Deploy-Logs
    └── scripts/
        └── deploy.sh            ← Deploy-Script
```

---

### 2. Repository klonen

```bash
cd ~/repos
git clone https://github.com/DEINE-ORG/kampagnen-site.git

# Test
cd kampagnen-site
git status
```

---

### 3. Node.js einrichten

```bash
# NVM aktivieren (Pfad kann variieren!)
source ~/.nvm/nvm.sh
# Oder auf Arch: source /usr/share/nvm/init-nvm.sh

# Node v25 installieren (oder v20, v22 - alle funktionieren!)
nvm install 25
nvm alias default 25

# Prüfen
node --version  # Sollte v25.x.x zeigen

# NVM dauerhaft laden
echo 'source ~/.nvm/nvm.sh' >> ~/.bashrc  # Oder ~/.zshrc
```

---

### 4. Webhook-Secret generieren

```bash
openssl rand -hex 32
```

**Beispiel-Output:**
```
8f3a9b2c1d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1
```

**⚠️ WICHTIG:** Notiere dieses Secret sicher!
- Brauchst du für `hooks.json` (Server)
- Brauchst du für GitHub Webhook Settings

---

### 5. hooks.json erstellen

```bash
nano ~/webhook/config/hooks.json
```

**Inhalt:**
```json
[
  {
    "id": "deploy-kampagne",
    "execute-command": "/home/$USER/webhook/scripts/deploy.sh",
    "command-working-directory": "/tmp",
    "response-message": "Deployment gestartet!",
    "trigger-rule": {
      "and": [
        {
          "match": {
            "type": "payload-hmac-sha256",
            "secret": "HIER_DEIN_SECRET_EINFÜGEN",
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

**Anpassen:**
- `$USER` durch deinen Linux-Username ersetzen
- `HIER_DEIN_SECRET_EINFÜGEN` durch das generierte Secret

---

### 6. deploy.sh erstellen

```bash
nano ~/webhook/scripts/deploy.sh
```

**Inhalt:**
```bash
#!/bin/bash
set -e

REPO_DIR="$HOME/repos/kampagnen-site"
WEB_DIR="/var/www/deine-site"  # ← ANPASSEN!
LOG="$HOME/webhook/config/deploy.log"

echo "========================================" >> "$LOG"
echo "$(date): Deploy gestartet" >> "$LOG"

# Git pull
cd "$REPO_DIR"
git pull origin main >> "$LOG" 2>&1

# Node/NPM (NVM-Pfad kann variieren!)
source ~/.nvm/nvm.sh  # Oder: /usr/share/nvm/init-nvm.sh
nvm use 25 >> "$LOG" 2>&1
npm install >> "$LOG" 2>&1
npm run build:css >> "$LOG" 2>&1

# Hugo build
hugo --minify -d "$WEB_DIR" >> "$LOG" 2>&1

echo "$(date): Deploy erfolgreich!" >> "$LOG"
echo "" >> "$LOG"
```

```bash
chmod +x ~/webhook/scripts/deploy.sh
```

**Anpassen:**
- `WEB_DIR`: Nginx Web-Root-Pfad
- NVM-Pfad: Wo ist `nvm.sh` bei dir? (`find / -name "nvm.sh" 2>/dev/null`)

---

### 7. Manueller Test

```bash
# Script direkt ausführen
~/webhook/scripts/deploy.sh

# Log prüfen
cat ~/webhook/config/deploy.log

# Output prüfen
ls -la /var/www/deine-site/
# Sollte index.html, css/, images/ enthalten
```

**Bei Fehlern:** Log zeigt genau wo es hakt!

---

### 8. systemd-Service einrichten

```bash
sudo nano /etc/systemd/system/webhook.service
```

**Inhalt:**
```ini
[Unit]
Description=GitHub Webhook Server
After=network.target

[Service]
Type=simple
User=$USER
Group=$USER
WorkingDirectory=/home/$USER/webhook
ExecStart=/usr/bin/webhook -hooks /home/$USER/webhook/config/hooks.json -verbose -port 9000
Restart=always
RestartSec=5

StandardOutput=journal
StandardError=journal
SyslogIdentifier=webhook

[Install]
WantedBy=multi-user.target
```

**Anpassen:**
- `$USER` durch deinen Linux-Username ersetzen (3 Stellen!)

**Service aktivieren:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable webhook
sudo systemctl start webhook

# Status prüfen
sudo systemctl status webhook

# Sollte zeigen: "Active: active (running)"
```

**Logs ansehen:**
```bash
sudo journalctl -u webhook -f
```

---

### 9. Nginx konfigurieren

#### a) Static Files Config

```bash
sudo nano /etc/nginx/sites-available/deine-site.conf
```

**Minimale Config:**
```nginx
server {
    listen 443 ssl http2;
    server_name deine-domain.de;

    # SSL-Zertifikate (mit Let's Encrypt/Certbot erstellt)
    ssl_certificate /etc/letsencrypt/live/deine-domain.de/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/deine-domain.de/privkey.pem;

    root /var/www/deine-site;
    index index.html;

    # Static Files
    location / {
        try_files $uri $uri/ =404;
    }

    # Asset-Caching
    location ~* \.(jpg|jpeg|png|webp|svg|css|js|woff2|ttf)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Webhook-Endpoint (versteckter Pfad!)
    location /webhook-DEIN-GEHEIMER-PFAD/ {
        proxy_pass http://127.0.0.1:9000/hooks/deploy-kampagne;
        proxy_set_header Host $host;
        proxy_set_header X-Hub-Signature-256 $http_x_hub_signature_256;
    }
}

# HTTP → HTTPS Redirect
server {
    listen 80;
    server_name deine-domain.de;
    return 301 https://$host$request_uri;
}
```

**Anpassen:**
- `deine-domain.de`: Deine Domain
- `/var/www/deine-site`: Dein Web-Root
- `/webhook-DEIN-GEHEIMER-PFAD/`: Zufälligen Pfad wählen!

**Geheimen Pfad generieren:**
```bash
echo "webhook-$(openssl rand -hex 16)"
# z.B.: webhook-a3f9e2c8d1b4f7e6c2a5b8d3e9f1a4b7
```

#### b) Config aktivieren

```bash
# Symlink erstellen
sudo ln -s /etc/nginx/sites-available/deine-site.conf /etc/nginx/sites-enabled/

# Config testen
sudo nginx -t

# Neu laden
sudo systemctl reload nginx
```

#### c) SSL-Zertifikat holen

```bash
# Certbot installieren (falls noch nicht vorhanden)
sudo apt install certbot python3-certbot-nginx

# Zertifikat holen
sudo certbot --nginx -d deine-domain.de

# Auto-Renewal prüfen
sudo certbot renew --dry-run
```

---

### 10. GitHub Webhook einrichten

**Gehe zu:**
```
https://github.com/DEINE-ORG/kampagnen-site/settings/hooks/new
```

**Fülle aus:**

| Feld | Wert |
|------|------|
| **Payload URL** | `https://deine-domain.de/webhook-DEIN-GEHEIMER-PFAD/` |
| **Content type** | `application/json` |
| **Secret** | Das generierte Secret (aus Schritt 4) |
| **SSL verification** | ✅ Enable SSL verification |
| **Which events?** | Just the push event |
| **Active** | ✅ |

**Klick "Add webhook"**

---

### 11. Test!

#### Editiere eine Datei auf GitHub.com

1. Öffne `content/_index.md` im Browser
2. Klick auf Bleistift (Edit)
3. Ändere z.B. den Titel
4. **Commit changes** → **Commit directly to main**

#### Logs verfolgen

**Terminal 1: Webhook-Service**
```bash
sudo journalctl -u webhook -f
```

**Terminal 2: Deploy-Log**
```bash
tail -f ~/webhook/config/deploy.log
```

#### Was sollte passieren:

1. GitHub sendet POST
2. Nginx leitet weiter an Webhook-Service
3. Webhook validiert Secret ✅
4. `deploy.sh` wird ausgeführt
5. Nach ~15 Sekunden: Website aktualisiert!

**Im Browser:** Refresh → Änderung ist sichtbar! 🎉

---

## Debugging

### Webhook empfängt nichts

```bash
# Service läuft?
sudo systemctl status webhook

# Logs
sudo journalctl -u webhook -n 50

# Port erreichbar?
curl http://localhost:9000/hooks/deploy-kampagne

# GitHub Webhook Delivery
# → GitHub → Settings → Webhooks → Recent Deliveries
```

### Webhook empfängt, aber triggert nicht

**Logs zeigen:** `parameter node not found: X-Hub-Signature-256`
→ Nginx Config prüfen (`proxy_set_header`)

**Logs zeigen:** `trigger rules were not satisfied`
→ Secret falsch oder Branch nicht main

### Script läuft nicht

```bash
# Pfad korrekt?
stat ~/webhook/scripts/deploy.sh

# Ausführbar?
ls -la ~/webhook/scripts/deploy.sh

# Manuell testen
~/webhook/scripts/deploy.sh
```

### Build fehlschlägt

```bash
# Deploy-Log ansehen
cat ~/webhook/config/deploy.log

# Häufige Fehler:
# - "nvm: command not found" → NVM-Pfad falsch
# - "hugo: command not found" → Hugo nicht installiert
# - "npm ERR!" → Node-Version oder package.json Problem
```

---

## Sicherheit

### ✅ Gut umgesetzt

- Webhook validiert Secret (HMAC SHA256)
- Nur main-Branch triggert Deploy
- Versteckter URL-Pfad (nicht leicht zu erraten)
- Service läuft als non-root User
- Kein SSH-Key in GitHub

### ⚠️ Best Practices

- **Secret regelmäßig rotieren** (alle 6-12 Monate)
- **Webhook-URL geheim halten** (nicht in öffentliche Repos!)
- **Logs regelmäßig prüfen** auf ungewöhnliche Requests
- **Firewall einrichten** (nur Port 80/443 nach außen)

---

## Wartung

### Logs rotieren

Deploy-Log kann groß werden:

```bash
# Alte Logs löschen
> ~/webhook/config/deploy.log

# Oder: Automatisch mit logrotate
sudo nano /etc/logrotate.d/webhook-deploy
```

### Hugo/Node updaten

```bash
# Hugo
sudo pacman -S hugo  # Arch
# oder: https://gohugo.io/installation/

# Node
nvm install 26  # Wenn neue Version verfügbar
nvm alias default 26
```

### Webhook-Service neu starten

```bash
sudo systemctl restart webhook
```

---

## Mehrere Kampagnen

**Option A:** Mehrere Repos, mehrere Hooks

```json
[
  {"id": "deploy-kampagne-a", "execute-command": "/path/to/deploy-a.sh"},
  {"id": "deploy-kampagne-b", "execute-command": "/path/to/deploy-b.sh"}
]
```

**Option B:** Ein Repo, mehrere Sites (Hugo Multi-Site)

Siehe: https://gohugo.io/configuration/multisite/

---

## Rollback bei Fehler

**Aktuell kein automatisches Rollback!**

**Manuell:**
```bash
cd ~/repos/kampagnen-site
git log  # Letzter guter Commit
git reset --hard COMMIT-HASH
~/webhook/scripts/deploy.sh
```

**Zukünftig:** Git-Tags + Deploy-Script erweitern

---

## Performance-Optimierungen

### Incremental Builds

```bash
# In deploy.sh
hugo --gc --minify --cleanDestinationDir -d "$WEB_DIR"
```

### npm install nur bei package.json-Änderung

```bash
# In deploy.sh
if git diff --name-only HEAD@{1} HEAD | grep -q "package.json"; then
  npm install >> "$LOG" 2>&1
fi
```

### Build-Benachrichtigungen

```bash
# Am Ende von deploy.sh
curl -X POST "https://hooks.slack.com/..." \
  -d '{"text":"Deployment erfolgreich!"}'
```

---

## Support

Bei Problemen:
- **Deploy-Log:** `~/webhook/config/deploy.log`
- **Service-Log:** `sudo journalctl -u webhook -n 100`
- **Nginx-Log:** `/var/log/nginx/error.log`
- **Hugo-Docs:** https://gohugo.io/documentation/

---

**Geschafft!** Deine Kampagnen-Site deployed sich jetzt automatisch! 🚀
