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
| `Projektuebersicht_APP.html` | v1.6.57 |
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
- Einträge: „📊 Dashboard“ (`data-view="dashboardKpi"`, kein Subnav), „📋 Projektübersicht“ (`data-view="dashboard"`, Subnav mit Jahren), Statistiken, Aufgaben (Subnav), Lager, Kalkulation, Kontakte (Subnav), Artikelstamm, Tools (Subnav)
- **`showView()` darf Subnav-Accordions nicht öffnen** (kein `openSubnav()` in `showView()`). Geöffnet wird nur per `toggleSubnav()` (Klick auf den Hauptpunkt) und in den `navTo…()`-Funktionen. Sonst lässt sich die Unterstruktur nicht mehr zuklappen.

### Navigation / Views (Stand v1.6.57)
- **View-Namen:** `showView()` bekommt den Namen ohne Suffix „View“, z. B. `showView('dashboardKpi')` → `#dashboardKpiView`, `showView('dashboard')` → `#dashboardView`. `showView('dashboardKpiView')` würde keinen View anzeigen.
- **Startansicht = Dashboard (KPI).** Muss an drei Stellen übereinstimmen: `class="view active"` auf `#dashboardKpiView` im HTML, der Start-Block am Ende des Scripts (`(async () => { … msalInstance.initialize() … })()`, setzt `dashboardKpiView` aktiv) und `showView('dashboardKpi')` am Ende von `initAppAfterLogin()`. Wird eine Stelle geändert, die anderen mitändern, sonst Flackern und falscher Zurück-Stack.
- **History:** `_navHistory` (max. 20), oberster Eintrag = aktuelle Ansicht, keine direkt aufeinanderfolgenden Duplikate. `goBack()` entfernt nur die aktuelle Ansicht und ruft `showView(prev)` mit `isNavigatingBack = true` auf; dann schreibt `showView()` nicht in den Stack. `isNavigatingBack` steht vor `initApp()`.
- **`renderDashboard()`** baut die Projektliste komplett neu auf und wendet am Ende `applyYearFilter()` (Jahres- und Status-Filter) wieder an. Das muss erhalten bleiben, sonst gehen Filter beim Neuaufbau verloren (z. B. Sidebar → Jahr). Der Suchbegriff wird beim Neuaufbau nicht wiederhergestellt (offen).

### Dashboard (KPI) — `dashboardKpiView`, `renderDashboardKpi()`
- Zeigt immer das laufende Kalenderjahr (`new Date().getFullYear()`), Jahr per `getYearFromProjectNumber()`, Daten aus `window.allProjects`. Aufruf in `showView('dashboardKpi')` und in `loadFromSharePoint()` direkt nach `renderDashboard()`.
- **Kacheln** (`#kpiTiles`, Flex mit Umbruch): VK Umsatz lfd. Jahr (mit Abgerechnet/Offen über `hasInvoice`), Deckungsbeitrag nur nachkalkulierter Projekte (`hasPostCalc`; grün nur bei DB > 0, sonst rot), Aktive Projekte (`!hasInvoice`).
- **Meine offenen Aufgaben** (`#kpiMyTasks`): Member-ID = `email.split('.')[0]` aus `msalInstance.getAllAccounts()[0].username`, Abgleich mit `TEAM_MEMBERS`. **Bewusste Entscheidung des Entwicklers: kein Mapping-Objekt einführen, auch nicht bei zukünftigen Änderungen.** Ohne Treffer in `TEAM_MEMBERS` wird die Zone ausgeblendet. Angezeigt werden `project.tasks` und `project.purchasePositions` mit `assignedTo === Member-ID` und Status `open`.
- **Aktionsliste** (`#kpiActions`): ohne Nachkalkulation (`!hasPostCalc`), geliefert aber nicht abgerechnet, ohne AB (`!hasAB && !hasInvoice`), offene Aufgaben je Mitglied (`project.tasks` mit Status ≠ `done`), Liefertermine der nächsten 14 Tage (ohne `hasDelivered`). Projektnummern öffnen das Projekt per `openProject()`.

---

## Bekannte offene Punkte (Stand 2026-11-05)

- Aktionsliste „Offene Aufgaben“ zählt nur `project.tasks` (Status ≠ `done`); die Zone „Meine offenen Aufgaben“ zeigt `project.tasks` und Einkaufspositionen mit Status `open`. Die beiden Zählungen sind nicht angeglichen — Entscheidung offen.
- Suchbegriff geht beim Neuaufbau der Projektliste (`renderDashboard()`) verloren.
- Unter 900 px Breite gibt es Seitenscroll, weil `#appLayout` und `body` `min-width: 900px` haben (Tablet im Hochformat — nicht auf echtem Gerät geprüft).
- README: Die SharePoint-Listen-Tabelle enthält die Kontakte-Liste (`guidContacts`) nicht; ihre übrigen Listennamen wurden nicht gegen den Code geprüft.
- Auf dem Entwicklungsrechner sind weder `python3` noch `node` installiert; der Syntax-Diagnose-Workflow unten lief in diesen Sitzungen nicht. Änderungen wurden im Browser getestet.

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
| `dashboardKpiView` | Dashboard (KPI-Kacheln, „Meine offenen Aufgaben“, Aktionsliste; nur lfd. Kalenderjahr; Startansicht) |
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
