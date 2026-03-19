# 📝 LLM.md (Komplett überarbeitet)

```markdown
# Campa — LLM Context & Technical Documentation

> Vollständige technische Dokumentation für Large Language Models.
> Dokumentiert alle Architektur-Entscheidungen, Lessons Learned und Debugging-Patterns.
> Stand: März 2026

---

## Executive Summary

**Campa** ist ein Hugo-basiertes Static Site Generator System für politische Kampagnen-Landingpages. 

**Kern-Philosophie:** Hybrid-Architektur mit Build-Tools auf Host und nur Services in Docker.

**Deployment-Methode:** GitHub Webhook → Host systemd-Service → Shell-Script (git pull + CSS build + Hugo build)

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
┌─────────────────────────────────────────────────────────────┐
│  1. Redakteur editiert auf GitHub.com                        │
│     - Klickt auf .md Datei                                   │
│     - Edit → Commit changes                                  │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  2. GitHub sendet Webhook POST                               │
│     URL: https://domain.de/webhook-SECRET-PATH/              │
│     Header: X-Hub-Signature-256 (HMAC SHA256)                │
│     Body: {"ref":"refs/heads/main", ...}                     │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  3. Nginx empfängt (Port 443)                                │
│     - SSL-Terminierung                                       │
│     - proxy_pass → localhost:9000                            │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  4. Webhook-Service (systemd, Port 9000)                     │
│     - Liest hooks.json                                       │
│     - Validiert HMAC SHA256 Secret                           │
│     - Prüft: ref == "refs/heads/main"                        │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  5. deploy.sh wird ausgeführt (als normaler User)            │
│     a) cd ~/repos/kampagnen-site                             │
│     b) git pull origin main                                  │
│     c) source nvm → nvm use 25                               │
│     d) npm install                                           │
│     e) npm run build:css (Tailwind CLI)                      │
│     f) hugo --minify -d /var/www/site/                       │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  6. Output landet in Web-Root                                │
│     - index.html                                             │
│     - css/main.css                                           │
│     - images/                                                │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  7. Nginx liefert aktualisierte Seite aus                    │
│     - Static Files                                           │
│     - Asset-Caching (1 Jahr)                                 │
└─────────────────────────────────────────────────────────────┘
```

**Reaktionszeit:** ~10-20 Sekunden (abhängig von Repo-Größe)

---

## Architektur-Entscheidungen

### Entscheidung 1: Webhook auf Host statt Docker

**Problem:** Ursprünglich war geplant, Webhook + Hugo + Node in Docker-Containern laufen zu lassen.

**Herausforderungen:**
1. Webhook-Container kann nicht auf Host-Git zugreifen
2. Webhook-Container braucht Docker Socket für `docker compose run` (Security-Risk!)
3. Custom Dockerfile mit Git + Hugo + Node → Große Images, schwer wartbar
4. Komplexität steigt exponentiell

**Recherche-Ergebnis:** Standard-Practice für Build-Tools ist Host-Installation.

**Reddit/Community-Consensus:**
> "Native installations for build tools, containers for deployment services."

**Hugo ist kein Service!** Es läuft 5 Sekunden, dann ist es fertig. Docker ist Overkill.

**Finale Entscheidung:** Webhook als systemd-Service auf Host.

**Vorteile:**
- ✅ Direkter Zugriff auf Git, NVM, Hugo
- ✅ Kein Docker Socket Risk
- ✅ Einfacher zu debuggen (journalctl)
- ✅ Weniger Komponenten = weniger Fehler

---

### Entscheidung 2: Hugo auf Host statt Container

**Hugo = Single Binary (~90 MB), KEINE Runtime-Dependencies!**

**Recherche-Ergebnis:**
- Community nutzt Hugo primär nativ
- Docker-Images vor allem für CI/CD (reproduzierbare Umgebung)
- Für VPS-Deployment: Binary auf Host ist Standard

**Zitat aus Blog:**
> "How I went from an overengineered Docker + Tilt.dev setup to just running Hugo directly, and why sometimes simpler is better."

**Entscheidung:** Hugo via Package-Manager installieren (`pacman -S hugo`)

**Vorteil:** Updates via `pacman -Syu`, kein Container-Management.

---

### Entscheidung 3: Node v25 statt v20

**Ursprüngliche Annahme:** "Node v25+ ist inkompatibel mit Tailwind v3"

**Test-Ergebnis:** Node v25.8.1 + Tailwind v3.4.x = **funktioniert einwandfrei!**

```bash
$ node --version
v25.8.1

$ npm run build:css
Rebuilding...
Done in 830ms
✅ Kein Fehler!
```

**Entscheidung:** Node v25 nutzen (neueste Version).

**Lesson Learned:** Alte Warnings in Dokumentation waren veraltet. Immer testen!

---

### Entscheidung 4: GitHub Webhook statt GitHub Actions

**Ursprünglicher Plan:** GitHub Actions baut in der Cloud, rsync zu VPS.

**Problem:** SSH-Key muss in GitHub Secrets → Security-Concern.

**Alternative Patterns recherchiert:**

| Pattern | Vorteile | Nachteile |
|---------|----------|-----------|
| GitHub Actions + rsync | Build in Cloud, VPS spart Ressourcen | SSH-Key in GitHub |
| Git Post-Receive Hook | Einfach, direkt | Kein GitHub Web-Editor möglich |
| **Webhook + VPS Build** | VPS kontrolliert, kein SSH-Key in GitHub | Build auf VPS nötig |

**Use-Case:** Redakteure sollen auf GitHub.com editieren können (Web-UI!).

**Entscheidung:** Webhook + VPS Build

**Begründung:**
- ✅ GitHub nur für Collaboration & Web-Editor
- ✅ Kein gefährlicher SSH-Key in GitHub
- ✅ VPS hat volle Kontrolle über Deployment
- ✅ Hugo Forum empfiehlt dieses Pattern explizit

---

## Verzeichnisstruktur (Production)

```
/home/$USER/
├── repos/
│   └── kampagnen-site/                 ← Git-Repo (HTTPS clone)
│       ├── .git/
│       ├── content/
│       ├── layouts/
│       ├── static/
│       ├── assets/css/main.css         ← Tailwind-Quelle
│       ├── hugo.toml
│       ├── package.json
│       └── tailwind.config.js
│
├── webhook/
│   ├── config/
│   │   ├── hooks.json                  ← Webhook-Konfiguration
│   │   └── deploy.log                  ← Deploy-Logs (stdout+stderr)
│   └── scripts/
│       └── deploy.sh                   ← Deploy-Script
│
└── .nvm/                               ← NVM-Installation (optional)
    └── versions/node/v25.8.1/

/var/www/
└── kampagnen-site/                     ← Hugo Output (public/)
    ├── index.html
    ├── css/
    │   └── main.css                    ← Kompilierte Tailwind-CSS
    ├── images/
    └── fonts/

/etc/systemd/system/
└── webhook.service                     ← systemd-Service-Definition

/etc/nginx/                             ← Oder: Docker-Volume
└── sites-available/
    └── kampagne.conf                   ← Nginx-Config
```

---

## Kritische Dateien im Detail

### hooks.json

**Pfad:** `~/webhook/config/hooks.json`

**Schema:**
```json
[
  {
    "id": "deploy-kampagne",
    "execute-command": "/absoluter/pfad/zu/deploy.sh",
    "command-working-directory": "/tmp",
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

**Wichtige Details:**
- `execute-command` muss **absoluter Pfad** sein!
- `command-working-directory` ist irrelevant wenn absoluter Pfad genutzt wird
- `secret` wird mit `openssl rand -hex 32` generiert
- Webhook binary erkennt Änderungen automatisch (hotreload, kein Restart nötig!)
- `trigger-rule.and[]` = ALLE Bedingungen müssen erfüllt sein (Secret UND Branch)

**Fehlerquellen:**
- Relativer Pfad → `stat: no such file or directory`
- Secret falsch → `trigger rules were not satisfied`
- Header fehlt → `parameter node not found: X-Hub-Signature-256` (Nginx Config!)

---

### deploy.sh

**Pfad:** `~/webhook/scripts/deploy.sh`

**Best Practices:**
```bash
#!/bin/bash
set -e  # ← Stoppt bei erstem Fehler!

REPO_DIR="$HOME/repos/kampagnen-site"
WEB_DIR="/var/www/kampagnen-site"
LOG="$HOME/webhook/config/deploy.log"

echo "========================================" >> "$LOG"
echo "$(date): Deploy gestartet" >> "$LOG"

# Git pull
cd "$REPO_DIR" || exit 1
git pull origin main >> "$LOG" 2>&1  # ← stdout + stderr ins Log!

# NVM laden (Pfad kann variieren!)
source ~/.nvm/nvm.sh || source /usr/share/nvm/init-nvm.sh
nvm use 25 >> "$LOG" 2>&1

# CSS Build
npm install >> "$LOG" 2>&1
npm run build:css >> "$LOG" 2>&1

# Hugo Build
hugo --minify -d "$WEB_DIR" >> "$LOG" 2>&1

echo "$(date): Deploy erfolgreich!" >> "$LOG"
echo "" >> "$LOG"
```

**Wichtige Details:**
- `set -e` → Script stoppt bei Fehler (wichtig!)
- Alle Befehle mit `>> "$LOG" 2>&1` → stdout + stderr werden geloggt
- `source nvm` → NVM muss explizit geladen werden (non-interactive Shell!)
- `nvm use 25` → Node-Version explizit setzen (nicht auf $PATH verlassen)
- `|| exit 1` bei kritischen Befehlen → Verhindert Folge-Fehler
- Absoluter Pfad für `WEB_DIR` → Verhindert versehentliches Löschen falscher Ordner

**Fehlerquellen:**
- `nvm: command not found` → NVM-Pfad falsch
- `hugo: command not found` → Hugo nicht installiert oder nicht in $PATH
- `Permission denied` → deploy.sh nicht ausführbar (`chmod +x`)
- Relativer Pfad → Kann in falschem Verzeichnis landen

---

### systemd-Service

**Pfad:** `/etc/systemd/system/webhook.service`

```ini
[Unit]
Description=GitHub Webhook Server
After=network.target

[Service]
Type=simple
User=$USER                              # ← Normaler User, nicht root!
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

**Wichtige Details:**
- `User=$USER` → Service läuft als normaler User (Security!)
- `After=network.target` → Startet erst wenn Netzwerk verfügbar
- `Restart=always` + `RestartSec=5` → Auto-Restart bei Crash
- `StandardOutput=journal` → Logs gehen in journald (nicht in Datei!)
- `WorkingDirectory` → Basis für relative Pfade (aber wir nutzen absolute!)
- `-verbose` → Webhook loggt jeden Request
- `-port 9000` → Localhost-Port (nicht nach außen exponiert!)

**Logs ansehen:**
```bash
# Live-Logs
sudo journalctl -u webhook -f

# Letzte 50 Zeilen
sudo journalctl -u webhook -n 50

# Seit heute
sudo journalctl -u webhook --since today
```

---

### Nginx Config

**Zwei Teile:**

#### 1. Static Files
```nginx
server {
    listen 443 ssl http2;
    server_name kampagne.domain.de;
    
    root /var/www/kampagnen-site;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    # Asset-Caching
    location ~* \.(jpg|jpeg|png|webp|svg|css|js|woff2|ttf)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

#### 2. Webhook-Proxy
```nginx
location /webhook-GEHEIMER-PFAD/ {
    proxy_pass http://127.0.0.1:9000/hooks/deploy-kampagne;
    proxy_set_header Host $host;
    proxy_set_header X-Hub-Signature-256 $http_x_hub_signature_256;
}
```

**Wenn Nginx in Docker (z.B. SWAG):**
```nginx
location /webhook-GEHEIMER-PFAD/ {
    proxy_pass http://172.17.0.1:9000/hooks/deploy-kampagne;  # ← Docker Bridge IP
    proxy_set_header Host $host;
    proxy_set_header X-Hub-Signature-256 $http_x_hub_signature_256;
}
```

**Wichtige Details:**
- `172.17.0.1` = Standard Docker Bridge IP zum Host
- `proxy_set_header X-Hub-Signature-256` → **MUSS** gesetzt werden! Sonst Webhook-Validierung schlägt fehl
- `/webhook-GEHEIMER-PFAD/` → Sollte min. 32 Zeichen haben, zufällig generiert
- `proxy_pass` → **Vollständiger Pfad** inkl. Hook-ID!

**Alternativen zu 172.17.0.1:**
- `host.docker.internal` (bei Docker Desktop)
- Host-Network für Nginx (nicht empfohlen)
- Webhook auch in Docker (dann braucht es Docker Socket → Security!)

---

## Hugo-Spezifika

### content/_index.md ist ALLES!

**Hugo rendert IMMER `content/_index.md` als Startseite.**

Kampagnen-spezifische `_index.md` in Unterordnern werden **nicht** als Startseite gerendert!

**Korrekte Struktur:**
```
content/
├── _index.md                    ← HAUPT-KONFIGURATION (wird gerendert!)
└── kampagnen/kampagne-demo/
    ├── textblock1.md            ← Content-Dateien (via src: textblock1)
    ├── faq.md
    └── impressum.md
```

**Wichtig:** Alle Kampagnen-Settings stehen in der **Root** `_index.md`:
- `hero:`
- `slider:`
- `blocks:`
- `nav_links:`

---

### Block-System

**Reihenfolge in `blocks:`-Liste = Darstellungsreihenfolge auf der Seite!**

```yaml
blocks:
  - type: textblock    # ← Wird als erstes gerendert
    src: intro
    bg: light
  
  - type: slider       # ← Dann der Slider
    id: bilder
  
  - type: cta          # ← Dann CTA
```

**Hugo-Template (`layouts/index.html`):**
```html
{{ range .Params.blocks }}
  {{ $type := .type }}
  
  {{ if eq $type "textblock" }}
    {{ partial "block-textblock.html" (dict "block" . "page" $) }}
  
  {{ else if eq $type "slider" }}
    {{ partial "block-slider.html" (dict "Page" $ "Block" .) }}
  
  {{ end }}
{{ end }}
```

**Wichtig:** Context-Übergabe unterscheidet sich!
- Einige Blöcke: `(dict "block" . "page" $)` (Kleinbuchstaben)
- Andere: `(dict "Block" . "Page" $)` (Großbuchstaben)

**Im Partial dann:**
```html
{{ $block := .block }}  <!-- oder: .Block -->
{{ $page := .page }}    <!-- oder: .Page -->
```

**Empfehlung:** Konsistenz wahren! Alle neuen Blöcke sollten Kleinbuchstaben nutzen.

---

### Anker-IDs müssen im Block stehen!

**❌ FALSCH:**
```yaml
slider:
  id: bilder  # ← Hugo findet das nicht!

blocks:
  - type: slider
```

**✅ RICHTIG:**
```yaml
blocks:
  - type: slider
    id: bilder  # ← Wird zu <section id="bilder">
```

**Grund:** `layouts/index.html` iteriert über `blocks:`, nicht über `slider:`!

---

### Tailwind CSS Pipeline

```
┌─────────────────────────────────────┐
│  assets/css/main.css                │  ← Quelldatei (editieren!)
│  - @tailwind base                   │
│  - @tailwind components             │
│  - Custom CSS (.cta-banner { ... }) │
└──────────────┬──────────────────────┘
               │
               ▼
      ┌────────────────────┐
      │  npm run dev:css   │  ← Tailwind CLI v3
      │  (Terminal 1)      │
      └────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  static/css/main.css                │  ← Generiert (nicht editieren!)
│  - Kompilierte CSS (~100 KB)        │
└──────────────┬──────────────────────┘
               │
               ▼
      ┌────────────────────┐
      │  Hugo lädt direkt  │
      │  (kein PostCSS!)   │
      └────────────────────┘
```

**Wichtig:**
- `assets/css/main.css` = Quelle
- `static/css/main.css` = Output (automatisch generiert)
- Hugo lädt CSS aus `static/` (NICHT aus `assets/`!)
- Kein PostCSS in Hugo nötig!

**package.json Scripts:**
```json
{
  "scripts": {
    "dev:css": "tailwindcss -i ./assets/css/main.css -o ./static/css/main.css --watch",
    "build:css": "tailwindcss -i ./assets/css/main.css -o ./static/css/main.css --minify"
  }
}
```

---

## Lessons Learned

### 1. Docker ist nicht immer die Lösung

**Ursprünglicher Plan:** Alles in Docker!

**Realität:** Für Build-Tools ist Host besser.

**Erkenntnis:**
- Docker für **Services** die 24/7 laufen (Nginx, PostgreSQL, Redis)
- **Nicht** für Tools die 5 Sekunden laufen (Git, Hugo, npm)

**Community-Konsens:** "Native for build, containers for deployment"

---

### 2. Node v25 funktioniert mit Tailwind v3

**Alte Doku sagte:** "v25+ inkompatibel"

**Test-Ergebnis:** Funktioniert einwandfrei!

**Lesson:** Immer selbst testen, Warnings können veraltet sein.

---

### 3. Webhook-Container kann nicht auf Host zugreifen

**Problem:** Container will Script auf Host ausführen, das Host-Tools braucht.

**Lösungsversuche:**
1. Docker Socket mounten → Security-Risk ❌
2. Alle Tools im Container → Riesiges Custom Image ❌
3. Webhook auf Host → Simpel, funktioniert ✅

**Erkenntnis:** Nicht alles muss in Docker, nur weil es "modern" ist.

---

### 4. Hugo ist ein Single Binary

**Bedeutung:** Hugo hat NULL Runtime-Dependencies!

**Konsequenz:** Kein Grund für Docker-Container. Ein Binary auf Host ist simpler.

**Vergleich:**
- **Node:** Viele Dependencies → Container sinnvoll
- **Hugo:** Keine Dependencies → Host reicht

---

### 5. Absolute Pfade in hooks.json sind Pflicht

**Fehler:** `execute-command: "scripts/deploy.sh"` mit `command-working-directory: "/tmp"`

**Resultat:** Webhook sucht `/tmp/scripts/deploy.sh` ❌

**Fix:** `execute-command: "/home/user/webhook/scripts/deploy.sh"` ✅

---

### 6. NVM muss explizit geladen werden

**Problem:** In non-interactive Shell ist NVM nicht verfügbar.

**Fix:** `source ~/.nvm/nvm.sh` oder `source /usr/share/nvm/init-nvm.sh`

**Pfad variiert je nach OS:**
- Arch Linux: `/usr/share/nvm/init-nvm.sh`
- Ubuntu/Debian: `~/.nvm/nvm.sh`
- Installiert via curl: `~/.nvm/nvm.sh`

---

### 7. Nginx-Header MÜSSEN weitergeleitet werden

**Problem:** Webhook sagt "parameter node not found: X-Hub-Signature-256"

**Ursache:** Nginx Config vergisst Header weiterzuleiten.

**Fix:**
```nginx
proxy_set_header X-Hub-Signature-256 $http_x_hub_signature_256;
```

**Wichtig:** `$http_` Prefix + Lowercase + Underscores!

---

## Debugging-Strategien

### Ebene 1: Webhook empfängt nichts

**Symptom:** GitHub zeigt "504 Gateway Timeout" oder "Connection refused"

**Checks:**
```bash
# 1. Service läuft?
sudo systemctl status webhook

# 2. Port offen?
sudo ss -tlnp | grep 9000

# 3. Lokal erreichbar?
curl http://localhost:9000/hooks/deploy-kampagne

# 4. Nginx leitet weiter?
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# 5. GitHub Delivery Log
# GitHub → Settings → Webhooks → Recent Deliveries → Response
```

---

### Ebene 2: Webhook empfängt, aber triggert nicht

**Symptom:** Webhook-Log zeigt Request, aber kein Execution

**Checks:**
```bash
# Service-Log ansehen
sudo journalctl -u webhook -n 50

# Suche nach:
# - "parameter node not found" → Nginx Header fehlt
# - "trigger rules were not satisfied" → Secret falsch oder Branch != main
# - "hook triggered successfully" → Hat funktioniert!
```

**Debug-Pattern:**
```bash
# Temporär Secret-Regel deaktivieren (NUR ZUM TESTEN!)
# hooks.json: Entferne "and": [...] und behalte nur Branch-Check
# Dann: Funktioniert es ohne Secret? → Secret-Problem!
```

---

### Ebene 3: Script läuft nicht

**Symptom:** "error in exec: stat /path: no such file or directory"

**Checks:**
```bash
# 1. Datei existiert?
ls -la ~/webhook/scripts/deploy.sh

# 2. Ausführbar?
stat ~/webhook/scripts/deploy.sh  # Sollte x-Bit haben

# 3. Pfad absolut?
grep execute-command ~/webhook/config/hooks.json
# Sollte zeigen: "/home/user/..."

# 4. Manuell testen
~/webhook/scripts/deploy.sh
```

---

### Ebene 4: Script läuft, aber Build fehlschlägt

**Symptom:** Script startet, aber bricht ab

**Checks:**
```bash
# Deploy-Log ansehen
cat ~/webhook/config/deploy.log

# Häufige Fehler:
# - "nvm: command not found" → NVM-Pfad falsch
# - "hugo: command not found" → Hugo nicht installiert
# - "npm ERR!" → package.json Problem oder Node-Version
# - "fatal: unable to access" → Git-Permissions oder Netzwerk
# - "Error: EACCES" → Permissions-Problem bei npm

# Manuell durchgehen:
cd ~/repos/kampagnen-site
git pull origin main
source ~/.nvm/nvm.sh
nvm use 25
npm install
npm run build:css
hugo --minify -d /var/www/kampagnen-site
```

---

## Sicherheitsüberlegungen

### ✅ Gut implementiert

1. **HMAC SHA256 Secret-Validierung**
   - GitHub signiert Payload mit Secret
   - Webhook validiert Signatur
   - Verhindert unautorisierte Requests

2. **Branch-Whitelist**
   - Nur `refs/heads/main` triggert Deploy
   - Verhindert versehentliche Deploys von Feature-Branches

3. **Versteckter URL-Pfad**
   - `/webhook-a3f9e2c8d1b4f7e6...` schwer zu erraten
   - Zusätzlicher Schutz neben Secret

4. **Non-Root Execution**
   - Webhook-Service läuft als normaler User
   - deploy.sh läuft als normaler User
   - Begrenzt Schaden bei Kompromittierung

5. **Kein SSH-Key in GitHub**
   - VPS pullt Code, GitHub pusht nichts
   - Reduziert Attack Surface

### ⚠️ Potenzielle Risiken

1. **Secret-Leak**
   - Wer Secret kennt, kann Deploy triggern
   - **Mitigation:** Secret regelmäßig rotieren

2. **Webhook-URL-Leak**
   - URL könnte in Logs oder öffentlichen Docs landen
   - **Mitigation:** URL nicht committen, nur in README.local

3. **Deploy-Script hat volle User-Rechte**
   - Script kann alles löschen was User darf
   - **Mitigation:** Separater Deploy-User mit minimalen Rechten

4. **Kein Rollback-Mechanismus**
   - Fehlerhafte Commits gehen direkt live
   - **Mitigation:** Git-Tags + manueller Rollback möglich

### 🔐 Best Practices

```bash
# 1. Secret rotieren (alle 6 Monate)
openssl rand -hex 32  # Neues Secret
# → hooks.json updaten
# → GitHub Webhook updaten
# → Webhook-Service restart

# 2. Separater Deploy-User (optional, aber empfohlen)
sudo useradd -m -s /bin/bash deploy
sudo chown -R deploy:deploy /var/www/kampagnen-site
# → Webhook-Service User auf "deploy" setzen

# 3. Logs regelmäßig prüfen
sudo journalctl -u webhook --since "1 week ago" | grep -v "200 OK"

# 4. Rate-Limiting in Nginx (optional)
limit_req_zone