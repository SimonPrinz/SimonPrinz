# Dependabot Konfiguration für Enterprise – Vollständige Anleitung (Deutsch)

## Inhaltsverzeichnis
1. [Übersicht](#übersicht)
2. [Enterprise-Level Einstellungen](#enterprise-level-einstellungen)
3. [Organisation-Level Einstellungen](#organisation-level-einstellungen)
4. [Repository-Level Einstellungen](#repository-level-einstellungen)
5. [`.github/dependabot.yml` Konfiguration](#githubdependabotyml-konfiguration)
6. [Auto-Merge Einrichtung](#auto-merge-einrichtung)
7. [Häufig gestellte Fragen](#häufig-gestellte-fragen)

---

## Übersicht

Dependabot wird auf drei Ebenen konfiguriert:

| Ebene | Fokus | Beispiel |
|-------|-------|---------|
| **Enterprise** | Sicherheitsrichtlinien und Feature-Aktivierung | Dependency Graph aktivieren für alle Repos |
| **Organisation** | Gemeinsame Ressourcen und Defaults | Private Registry-Zugänge, gruppierte Updates |
| **Repository** | Stack-spezifisches Verhalten | npm/Python-spezifische Zeitpläne |

**Rollout-Reihenfolge:** Enterprise-Baseline → Org-Grouping & Zugriff → Repo-Templates → Auto-Merge

---

## Enterprise-Level Einstellungen

### Wo finde ich diese Einstellungen?

**Pfad:** `github.com/enterprises/{enterprise-name}/settings/security`

### Was kann ich auf Enterprise-Level konfigurieren?

Unter **Enterprise → Security configurations** kannst du eine Baseline erstellen und auf alle existierenden und neuen Repositories anwenden.

### Empfohlene Enterprise-Baseline

```
✓ Dependency Graph: aktiviert
✓ Dependabot Alerts: aktiviert
✓ Dependabot Security Updates: aktiviert
✓ Malware Alerts: aktiviert
✓ Automatic Dependency Submission: (evaluieren nach Ökosystem)
✓ Delegated Alert Dismissal: (nur wenn Governance erforderlich)
```

### Wichtig zu verstehen

- **Enterprise-Konfigurationen verwalten Feature-Aktivierung**, nicht Zeitpläne oder Dependency-Gruppen
- **Sicherheitsupdates benötigen keine `dependabot.yml`** – sie werden automatisch erstellt, wenn aktiviert
- **Scheduled Version-Updates benötigen `dependabot.yml`** in jedem Repository
- Die Konfiguration muss **explizit auf existierende und neue Repositories angewendet werden**

### Schritte zum Einrichten

1. Gehe zu Enterprise Settings → Advanced Security → Security configurations
2. Klicke "Create new configuration"
3. Aktiviere die oben empfohlenen Features
4. Klicke "Apply to existing repositories"
5. Wähle die gewünschten Repositories aus

---

## Organisation-Level Einstellungen

### Wo finde ich diese Einstellungen?

**Pfad:** `github.com/orgs/{org-name}/settings/security_analysis`

### 1. Gruppierte Sicherheitsupdates

**Pfad:** `github.com/orgs/{org-name}/settings/security_analysis` → "Global settings"

**Warum:** Reduziert PR-Volumen durch Zusammenfassung von Updates pro Ökosystem

**So aktivierst du es:**
1. Navigiere zu Org Settings → Code security and analysis
2. Scrolla zu "Global settings"
3. Aktiviere "Grouped security updates"

### 2. Dependabot-Secrets (Private Registry-Zugriff)

**Pfad:** `github.com/orgs/{org-name}/settings/secrets/dependabot`

**Warum:** Centralisiert Zugangsdaten für private Registries

**So richtest du es ein:**
1. Navigiere zu Org Settings → Secrets → Dependabot secrets
2. Klicke "New organization secret"
3. Erstelle Secrets für deine Private Registries:
   - `NPM_TOKEN` für npm
   - `PIP_INDEX_URL` für PyPI
   - `MAVEN_TOKEN` für Maven
   - etc.
4. Wähle die Repositories aus, die Zugriff haben sollen

### 3. Auto-Triage-Regeln (Optional)

**Pfad:** `github.com/orgs/{org-name}/settings/security_analysis` → "Auto-triage rules"

**Warum:** Automatisiert Alert-Handling (nicht Auto-Merge!)

**Beispiel-Szenarien:**
- Alerts für bestimmte Severity automatisch snoozed
- Sicherheits-PRs automatisch für kritische Vulnerabilities erstellen
- Alerts für interne Dependencies automatisch dismissieren

**Wichtig:** Dies ist für Alert-Management, **nicht für Auto-Merge**

### 4. Private Repository-Zugriff (Vorsicht!)

**Einstellung:** "Grant Dependabot access to private repositories"

⚠️ **Warnung:** Dies kann den Inhalt privater Repositories durch Dependency Updates anderen Org-Mitgliedern offenbaren. Nur aktivieren, wenn notwendig.

---

## Repository-Level Einstellungen

### Wo finde ich diese Einstellungen?

**Pfad:** `github.com/{owner}/{repo}/settings`

### 1. Auto-Merge aktivieren

**Pfad:** `github.com/{owner}/{repo}/settings/general` → "Pull Requests"

**Schritte:**
1. Navigiere zu Repository Settings → General
2. Scrolla zu "Pull Requests"
3. Aktiviere "Allow auto-merge"

**Wichtig:** Dies allein führt nicht zum automatischen Mergen! Du brauchst zusätzlich:
- Eine GitHub Action, die Auto-Merge für Dependabot-PRs aktiviert
- Alle erforderlichen Branch-Protection-Rules müssen erfüllt sein

### 2. Branch Protection Rules

**Pfad:** `github.com/{owner}/{repo}/settings/branches`

**Erforderliche Einstellungen für Auto-Merge mit Dependabot:**

```
✓ Require a pull request before merging
✓ Require status checks to pass before merging
  - Status checks: wähle deine CI-Checks aus (z.B. "build", "test", "lint")
  - Require branches to be up to date before merging (optional, aber empfohlen)
✓ Restrict who can push to matching branches (optional)
```

### 3. Dependabot Configuration File

**Pfad:** `.github/dependabot.yml` (muss auf dem Default-Branch sein)

→ Siehe Abschnitt [`.github/dependabot.yml` Konfiguration](#githubdependabotyml-konfiguration)

---

## `.github/dependabot.yml` Konfiguration

### Erforderliche Einstellungen (Pflichtfelder)

| Einstellung | Zweck | Beispiel |
|---------|---------|---------|
| `version` | Konfigurationssyntax-Version | `version: 2` (immer `2`) |
| `updates` | Container für alle Paketmanager-Konfigurationen | Parent-Key |
| `package-ecosystem` | Welcher Paketmanager überwacht wird | `npm`, `pip`, `docker`, `maven`, `gradle` |
| `directory` | Pfad zu Manifest-Dateien | `/`, `/app`, `/services/api` |
| `schedule.interval` | Wie oft auf Updates geprüft wird | `daily`, `weekly`, `monthly` |

### Minimales Beispiel

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
```

### Vollständiges Beispiel (Empfohlen)

```yaml
version: 2

updates:
  # NPM Package Updates
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "Europe/Berlin"
    
    # PR-Limits
    open-pull-requests-limit: 5
    
    # Gruppierung für weniger PRs
    groups:
      # Routine Minor/Patch Updates zusammenfassen
      routine-updates:
        applies-to: "version-updates"
        patterns:
          - "*"
        update-types:
          - "minor"
          - "patch"
      
      # Major Updates einzeln
      major-updates:
        applies-to: "version-updates"
        patterns:
          - "*"
        update-types:
          - "major"
      
      # Security Updates separat
      security-fixes:
        applies-to: "security-updates"
        patterns:
          - "*"
    
    # Abkühlungszeit (nur für version-updates)
    cooldown:
      default-days: 7
    
    # Commit- und PR-Anpassungen
    commit-message:
      prefix: "chore"
      include: "scope"
    
    labels:
      - "dependencies"
      - "npm"
    
    assignees:
      - "@frontend-team"
    
    reviewers:
      - "@frontend-team"

  # Docker Updates
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"
    
    labels:
      - "dependencies"
      - "docker"

  # GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "monthly"
    
    labels:
      - "dependencies"
      - "ci"
```

### Optionale Einstellungen

#### Zeitplansteuerung

```yaml
schedule:
  interval: "weekly"           # daily, weekly, monthly, quarterly, semiannually, yearly
  day: "monday"                # Für weekly/monthly: monday-sunday (englisch)
  time: "09:00"                # HH:MM in UTC
  timezone: "Europe/Berlin"    # IANA Timezone (Standard: UTC)
```

#### Pull-Request-Gruppierung (Reduziert PR-Volumen)

```yaml
groups:
  # Minor & Patch zusammenfassen
  routine:
    applies-to: "version-updates"
    patterns: ["*"]
    update-types: ["minor", "patch"]
  
  # Major einzeln lassen
  major:
    applies-to: "version-updates"
    patterns: ["*"]
    update-types: ["major"]
  
  # Sicherheitsupdates eigene Gruppe
  security:
    applies-to: "security-updates"
    patterns: ["*"]
```

#### PR-Limits & Abkühlungszeit

```yaml
open-pull-requests-limit: 10    # Max. gleichzeitige PRs (Standard: unbegrenzt)

cooldown:
  default-days: 7               # Warten vor Wiedereröffnung (nur version-updates)

pull-request-branch-name:
  separator: "/"                # Trennzeichen in Branch-Namen
```

#### PR-Anpassung

```yaml
commit-message:
  prefix: "chore"               # Präfix für Commit-Nachricht
  prefix-development: "chore"   # Anderes Präfix für Dev-Dependencies
  include: "scope"              # Abhängigkeitsnamen einschließen

labels:
  - "dependencies"
  - "npm"

assignees:
  - "@user1"
  - "@user2"

reviewers:
  - "@team-name"
```

#### Abhängigkeitskontrolle

```yaml
# Nur bestimmte Typen erlauben
allow:
  - dependency-type: "direct"   # Nur direkte Abhängigkeiten

# Spezifische Abhängigkeiten ignorieren
ignore:
  - dependency-name: "lodash"
    versions:
      - ">= 4.0, < 5.0"         # 4.x-Versionen ignorieren
  
  - dependency-name: "express"
    update-types:
      - "version-update:semver-major"  # Ignoriere Major-Versionen
```

#### Private Registry-Zugriff

```yaml
registries:
  npm-company:
    type: "npm-registry"
    url: "https://npm.company.com"
    username: "bot-username"
    password: "${{ secrets.NPM_COMPANY_TOKEN }}"
    replaces-base: true
```

#### Weitere Optionen

```yaml
target-branch: "develop"        # Anderen Branch aktualisieren (nicht Default)

rebase-strategy: "auto"         # auto, disabled oder always

vendor: true                    # Für Go/vendor-Verzeichnis

security-advisories-only: true  # Nur Sicherheitsupdates (ignoriert Versions-Updates)
```

### Häufige Konfigurationen nach Anwendungsfall

#### 1. Monorepo (Mehrere Manifeste)

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
  
  - package-ecosystem: "npm"
    directory: "/services/api"
    schedule:
      interval: "weekly"
  
  - package-ecosystem: "npm"
    directory: "/services/web"
    schedule:
      interval: "weekly"
```

#### 2. Instabile Versionen Ignorieren

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    
    ignore:
      - dependency-name: "typescript"
        update-types: ["version-update:semver-major"]
      
      - dependency-name: "react"
        versions: ["6.0.0-alpha*"]
```

#### 3. Private Registry + Custom PR-Einstellungen

```yaml
version: 2

registries:
  artifactory:
    type: "npm-registry"
    url: "https://artifactory.company.com/npm"
    username: "bot"
    password: "${{ secrets.ARTIFACTORY_TOKEN }}"
    replaces-base: true

updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
    
    labels: ["dependencies", "internal-registry"]
    assignees: ["@platform-team"]
    reviewers: ["@platform-team"]
    
    commit-message:
      prefix: "chore(deps)"
      include: "scope"
```

#### 4. Mehrere Ökosysteme

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
  
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"
  
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "monthly"
  
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"
```

---

## Auto-Merge Einrichtung

### Warum ist Auto-Merge sinnvoll?

Auto-Merge für Dependabot reduziert manuelles Mergen von stabilen Updates (Patches, Minor-Versionen) und beschleunigt den Release-Zyklus.

### Voraussetzungen

1. ✓ Auto-Merge in Repository Settings aktiviert
2. ✓ Branch Protection Rules mit Status Checks konfiguriert
3. ✓ Zuverlässige CI/CD-Tests
4. ✓ GitHub Action Workflow für Dependabot-Auto-Merge

### Schritt 1: Auto-Merge aktivieren

**Pfad:** `github.com/{owner}/{repo}/settings/general`

1. Navigiere zu Repository Settings
2. Scrolla zu "Pull Requests"
3. Aktiviere "Allow auto-merge"

### Schritt 2: Branch Protection Rules einrichten

**Pfad:** `github.com/{owner}/{repo}/settings/branches`

**Erforderliche Einstellungen:**

1. Wähle deinen Protected Branch (z.B. `main`)
2. Aktiviere "Require a pull request before merging"
3. Aktiviere "Require status checks to pass before merging"
   - Wähle deine CI-Checks aus (z.B. `build`, `test`, `lint`)
4. (Optional) Aktiviere "Require branches to be up to date before merging"

### Schritt 3: GitHub Action für Auto-Merge erstellen

**Datei:** `.github/workflows/dependabot-auto-merge.yml`

```yaml
name: Dependabot Auto-Merge

on: pull_request_target

permissions:
  contents: write
  pull-requests: write

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    
    steps:
      - name: Dependabot metadata
        id: dependabot-metadata
        uses: dependabot/fetch-metadata@v1

      - name: Enable auto-merge for Dependabot PRs (patch + minor)
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        if: |
          steps.dependabot-metadata.outputs.update-type == 'version-update:semver-patch' ||
          steps.dependabot-metadata.outputs.update-type == 'version-update:semver-minor'

      - name: Comment on major version PRs
        run: |
          gh pr comment "$PR_URL" --body "⚠️ **Major Version Update** – Requires manual review. Please review before merging."
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        if: steps.dependabot-metadata.outputs.update-type == 'version-update:semver-major'
```

### Schritt 4: Testen

1. Erstelle eine Test-Dependency Update manuell oder warte auf die nächste Schedule
2. Überprüfe, dass die Action läuft
3. Verifiziere, dass Auto-Merge für Patch/Minor aktiviert wird
4. Verifiziere, dass Major Updates einen Kommentar bekommen

### Wichtige Hinweise zu Auto-Merge

- **Dependabot PR = read-only `GITHUB_TOKEN`** – Die Standard-Action hat begrenzte Berechtigungen
- **Merge Queue:** Wenn du eine Merge Queue verwendest, kann der Standard-Token nicht hinzufügen. Du brauchst einen GitHub App Token oder PAT.
- **Abhängigkeiten respektieren** – Auto-Merge wartet auf alle Branch Protection Rules, bevor es merged
- **Sicherheit first:** Starte mit Patch/Minor-Updates, erweitere später auf Major-Updates

---

## Häufig gestellte Fragen

### F: Kann ich mehrere Paketmanager in einer Datei haben?

**A:** Ja! Füge mehrere Einträge unter `updates:` hinzu.

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
  
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"
```

### F: Warum funktioniert meine Konfiguration nicht?

**A:** Überprüfe:
- ✓ Dateiname ist `.github/dependabot.yml` (nicht `.yaml`)
- ✓ Datei befindet sich auf dem Default-Branch
- ✓ YAML-Syntax ist korrekt (richtige Einrückung mit Spaces, nicht Tabs)
- ✓ `package-ecosystem` ist unterstützt
- ✓ `directory` existiert im Repository

### F: Wie kann ich Auto-Merge aktivieren?

**A:** Das kann nicht in `dependabot.yml` gemacht werden. Du brauchst:
1. Auto-Merge in Repository Settings aktiviert
2. Branch Protection Rules mit Status Checks
3. Ein GitHub Action Workflow (siehe Abschnitt [Auto-Merge Einrichtung](#auto-merge-einrichtung))

### F: Kann ich Sicherheitsupdates ohne `dependabot.yml` bekommen?

**A:** Ja! Sicherheitsupdates werden automatisch erstellt, wenn sie in der Enterprise Security Configuration aktiviert sind. `dependabot.yml` ist optional und nur für Anpassungen erforderlich.

### F: Wie reduziere ich das PR-Volumen?

**A:** Nutze Gruppierung in `dependabot.yml`:

```yaml
groups:
  routine:
    applies-to: "version-updates"
    patterns: ["*"]
    update-types: ["minor", "patch"]
  
  major:
    applies-to: "version-updates"
    patterns: ["*"]
    update-types: ["major"]
```

Dies erstellt eine PR für alle Minor/Patch-Updates zusammengefasst und eine separate für Major-Updates.

### F: Wie bekomme ich Private Registry-Zugriff?

**A:** Auf Org-Level:
1. Gehe zu Org Settings → Secrets → Dependabot secrets
2. Erstelle Secrets (z.B. `NPM_TOKEN`)
3. Gebe Repositories Zugriff

In `dependabot.yml`:

```yaml
registries:
  npm-company:
    type: "npm-registry"
    url: "https://npm.company.com"
    username: "bot"
    password: "${{ secrets.NPM_TOKEN }}"
```

### F: Kann ich den Zeitplan mit Cron anpassen?

**A:** Ja, aber nur wenige Ökosysteme unterstützen Cron. Nutze standardmäßig `interval`:

```yaml
schedule:
  interval: "weekly"
  day: "monday"
  time: "09:00"
```

Für Cron siehe die [offizielle Dokumentation](https://docs.github.com/en/code-security/reference/supply-chain-security/dependabot-options-reference).

### F: Wie ignoriere ich bestimmte Versionen?

**A:** Nutze `ignore`:

```yaml
ignore:
  - dependency-name: "lodash"
    versions:
      - ">= 4.0, < 5.0"
  
  - dependency-name: "react"
    update-types: ["version-update:semver-major"]
```

### F: Was ist der Unterschied zwischen `security-updates` und `version-updates`?

| Typ | Beschreibung | Beispiel |
|-----|-------------|---------|
| **security-updates** | Nur Sicherheitskritische Updates | `1.0.0` → `1.0.1` (CVE Fix) |
| **version-updates** | Alle neuen Versionen | `1.0.0` → `1.1.0` oder `2.0.0` |

---

## Zusammenfassung: Enterprise Rollout-Plan

### Phase 1: Enterprise-Baseline (Woche 1)
- [ ] Gehe zu Enterprise Settings → Security configurations
- [ ] Erstelle neue Konfiguration mit empfohlenen Features
- [ ] Wende auf existierende und neue Repositories an

### Phase 2: Organisation-Einstellungen (Woche 1-2)
- [ ] Aktiviere "Grouped security updates"
- [ ] Erstelle Dependabot Secrets für Private Registries
- [ ] Konfiguriere Auto-Triage-Regeln (falls erforderlich)

### Phase 3: Repository-Templates (Woche 2-3)
- [ ] Erstelle `.github/dependabot.yml` Template für jedes Ökosystem
- [ ] Rollout über Rollout-PRs oder Dokumentation
- [ ] Beispiel-Repositories konfigurieren

### Phase 4: Auto-Merge (Woche 3-4)
- [ ] Teste in ausgewählten Repositories
- [ ] Aktiviere Auto-Merge in Settings
- [ ] Erstelle GitHub Action Workflows
- [ ] Rollout schrittweise erweitern

### Phase 5: Monitoring & Optimierung (Laufend)
- [ ] Überwache Dependabot Alerts-Abdeckung
- [ ] Sammle Feedback von Teams
- [ ] Optimiere Zeitpläne und Gruppen basierend auf PR-Volumen

---

**Letzte Aktualisierung:** 2026-10-09
**Version:** 1.0