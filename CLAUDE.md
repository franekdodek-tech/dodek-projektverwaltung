# Dodek Hub — CLAUDE.md

Dieses Repo enthält das interne ERP-System der Dodek Technik GmbH & Co. KG.
Lies diese Datei vollständig, bevor du irgendeine Änderung machst.

---

## Architektur

- **Single-file HTML-App** — die gesamte Anwendung steckt in `Projektuebersicht_APP.html` (inkl. CSS, JS, HTML)
- **Hosting:** GitHub Pages (`https://franekdodek-tech.github.io/dodek-projektverwaltung/`)
- **Backend:** SharePoint REST API (Dodek Microsoft 365 Tenant)
- **Auth:** MSAL Browser 2.38.3 (`msal-browser.min.js` liegt im Repo, wird lokal eingebunden)
- **Companion-Tools:** `spedition.html` und `bestellung.html` laufen als iframes im Hub und kommunizieren via `postMessage`-Bridge (Message-Types: `SPED_LOAD/SAVE/DELETE`, `BEST_LOAD/SAVE/DELETE`, `ADDR_LOAD/SAVE/DELETE`)
- **Kein Build-Prozess** — keine npm, kein Bundler, kein Transpiler

## Dateistruktur

```
/
├── Projektuebersicht_APP.html   ← Haupt-App (diese Datei wird am häufigsten geändert)
├── spedition.html               ← Companion-Tool Speditionsaufträge
├── bestellung.html              ← Companion-Tool Filter/Ventilatoren-Bestellung
├── index.html                   ← Weiterleitung (meta refresh) auf Projektuebersicht_APP.html für GitHub Pages
├── msal-browser.min.js          ← MSAL-Bibliothek (NICHT ANFASSEN)
├── README.md                    ← Projektbeschreibung, Funktionsumfang, SharePoint-Listen, Phasenstatus
└── CLAUDE.md                    ← diese Datei
```

## Aktuelle Versionen

| Datei | Version |
|---|---|
| `Projektuebersicht_APP.html` | v1.6.56 |
| `spedition.html` | v0.6.0 |
| `bestellung.html` | v0.6.0 |

- `Projektuebersicht_APP.html`: Version steht als `const APP_VERSION = 'v...'` im JS-Block.
- `spedition.html` / `bestellung.html`: kein `APP_VERSION`; die Version ist fest im `<title>` und in einem `<span>` im Header eingetragen (jeweils beide Stellen ändern).
- `README.md` enthält eine Versionstabelle, die bei Versionsänderungen mitgepflegt werden muss.

---

## SharePoint-Konfiguration (SP_CONFIG)

Diese Werte sind technisch notwendig und werden VOR dem Login gebraucht.
**Niemals entfernen, umbenennen oder verschieben.**

```javascript
const SP_CONFIG = {
    tenantId:     '1100d6c4-c332-4483-a651-db30fc47132e',
    clientId:     'b1546e68-d699-43ac-8a09-277fd5f18b57',
    siteUrl:      'https://dodekgmbh.sharepoint.com/sites/DodekProjektverwaltung',
    redirectUri:  'https://franekdodek-tech.github.io/dodek-projektverwaltung/Projektuebersicht_APP.html',
    guidProjects: '1e1a835c-20b9-41ea-920f-f8b391981c71',
    guidArticles: '26a0bbca-31bf-4a0d-8bd2-abe07d614725',
    guidStock:    'df491bc5-f23b-493a-a8d0-37a15024cf29',
    guidSettings: '89b9fdeb-6907-4cfc-a1f5-2bc8bc0bdbe2',
    guidContacts: '3D2F7AE7-A6DE-4F84-A1EB-6FC7DDDDB70F'
};
```

---

## Hard Rules — IMMER einhalten

### Was nie geändert wird
- `msal-browser.min.js` — nicht anfassen
- `SP_CONFIG` (s.o.) — keine Umbenennung, keine Verschiebung
- SharePoint-Feldnamen und JSON-Schlüssel in gespeicherten Daten — Abwärtskompatibilität zwingend
- Git-Historie — nicht umschreiben (kein `--force`, kein Rebase auf gepushten Commits)
- Firmenname "Dodek" — bleibt, ist öffentlich bekannt

### Was bei jeder Änderung gilt
- **APP_VERSION immer hochzählen** bei jeder commit-würdigen Änderung (patch: letzte Stelle, z.B. v1.6.47 → v1.6.48)
- **Commit-Messages immer auf Deutsch**
- **Keine Refactorings**, keine reinen Formatierungsänderungen, keine neuen externen Abhängigkeiten
- **Keine Funktionsänderungen** ohne explizite Anforderung
- Annahmen, die nicht im Code belegt sind, **explizit kennzeichnen**

### Commit-Message-Format
```
<Was wurde geändert, auf Deutsch>

- Bullet 1
- Bullet 2
- APP_VERSION: vX.X.X → vX.X.Y

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_0112q721THq4Xa5MtZn82Y5R
```

---

## Code-Architektur (Projektuebersicht_APP.html)

### Ausführungsreihenfolge
1. `SP_CONFIG` — global, ganz oben, vor allem anderen
2. Sidebar-Funktionen (global scope, vor `initApp`) — müssen global sein, weil onclick-Attribute im HTML sie direkt aufrufen
3. `initApp()` — großer Wrapper, startet nach Page-Load
4. `initAppAfterLogin()` — läuft nach erfolgreichem MSAL-Login; hier werden alle Daten geladen

### Wichtige Muster
- Funktionen die von `onclick`-Attributen aufgerufen werden **müssen** global sein (entweder vor `initApp()` definiert, oder `window.xxx = xxx` innerhalb von `initApp()`)
- `showView()`, `renderDashboard()` etc. sind über `window.xxx` exponiert
- `spFetch()` ist die zentrale Funktion für alle SharePoint REST-Calls
- `applyPermissions()` muss als **hoisted function declaration** vor `initApp()` stehen (kein `const` / `let`)

### Sidebar-System (eingeführt ab v1.6.44)
- Pinned-State: `localStorage.getItem('sidebarPinned') !== 'false'` → Standard ist pinned
- Body-Klasse `sidebar-unpinned` steuert Overlay-Modus
- Sidebar-Drawer im unpinned-Modus: `position: fixed; top: 60px` (unterhalb des App-Headers)
- Subnav-Accordions: `.subnav` / `.subnav.subnav-open` per `toggleSubnav(id)`
- Globale Sidebar-Funktionen: `togglePin()`, `toggleSidebar()`, `closeSidebar()`, `toggleSubnav()`, `openSubnav()`, `navToContactType()`, `navToTasksMember()`, `navToTool()`, `navToDashboardYear()`

---

## Syntax-Diagnose-Workflow

Bei JS-Syntaxfehlern (passiert durch unbalancierte geschweifte Klammern):

```bash
# 1. Script-Block extrahieren
python3 -c "
content = open('Projektuebersicht_APP.html').read()
start = content.find('<script>')
end = content.rfind('</script>')
open('/tmp/check.js', 'w').write(content[start+8:end])
"

# 2. Syntax prüfen
node --check /tmp/check.js

# 3. Klammer-Tiefe zählen (bei Fehler ohne Zeilennummer)
python3 -c "
content = open('/tmp/check.js').read()
depth = 0
for i, c in enumerate(content):
    if c == '{': depth += 1
    elif c == '}': depth -= 1
    if depth < 0:
        print(f'Unerwartete }} an Position {i}')
        print(repr(content[max(0,i-50):i+50]))
        break
print(f'Endtiefe: {depth}')
"
```

---

## Module / Views

| View-ID | Inhalt |
|---|---|
| `dashboardKpiView` | Dashboard (KPI-Kacheln + Aktionsliste, nur lfd. Kalenderjahr; Startansicht nach Login) |
| `dashboardView` | Projektübersicht (Projektliste nach Jahren) |
| `tasksView` | Aufgabenverwaltung nach Mitglied |
| `stockView` | Lagerverwaltung |
| `quotesView` | Kalkulation (BOM, VK/EK-Positionen) |
| `contactsView` | Kunden & Lieferanten |
| `articlesView` | Artikelstamm |
| `toolsView` | iframe-Einbettung spedition.html / bestellung.html |
| `editorView` | Projektbearbeitung (admin/editor) |
| `settingsView` | Einstellungen (admin only) |
| `statisticsView` | Statistiken |

---

## Rollenmodell

Drei Rollen: `admin`, `editor`, `reader`
Fallback-Berechtigungen sind im Code hardcodiert (`PERMISSIONS`-Objekt).
Produktiv-Berechtigungen kommen aus SharePoint-Liste `DodekEinstellungen` (SettingKey: `"permissions"`).

---

## Was der Entwickler (Franek) erwartet

- Klare, nüchterne Kommunikation — keine Wohlfühlsprache
- Risiken und Unsicherheiten offen benennen
- Nichts annehmen, was nicht im Code belegt ist — Unsicherheiten markieren
- Keine Verschönerungen, die nicht explizit angefordert wurden
- Ergebnisse sollen direkt deploybar sein (committen + fertig)
