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
| `Projektuebersicht_APP.html` | v1.6.81 |
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
    guidContacts: '3D2F7AE7-A6DE-4F84-A1EB-6FC7DDDDB70F',
    guidAnlagen:       'E7192882-A021-4BD4-86C5-F387B053D134',
    guidUVVPruefungen: '201B460E-3768-4401-A2F0-D1CD2A9BEB6E',
    guidUVVModule:     'DE98FE5E-7A10-4E5E-80A1-9BEC4F88D527'
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

### Startseite — `dashboardKpiView`, `renderHome()` (ab v1.6.81)
- Kopf (Logo, Nutzername aus MSAL, Online-Status), drei Kacheln (UVV offen, Projekte, Anlagen), „Handlungsbedarf“ (UVV offen, offene Aufgaben, Aktionsliste-Einträge), „Zuletzt geöffnet“. `renderHome()` läuft in `showView('dashboardKpi')`.
- **Zähler per eigenem SharePoint-Call** (`$select=Id&$top=500`, parallel, `Promise.allSettled`): UVV mit `$filter=Status eq 'Offen'`, Projekte und Anlagen ohne Filter. Bei mehr als 500 Einträgen ist der Zähler zu niedrig (`$top`-Grenze). „Projekte“ zählt alle Einträge, nicht nur aktive (Annahme: kein Aktiv-Flag in der Liste; „aktiv“ im Dashboard = `!hasInvoice`).
- `trackRecentItem(type, id, label)` (global, localStorage `dodek_recent_items`, max. 5): aufgerufen in `openProject()` (id = Projektnummer), `openAnlageDetail()` und `openPruefung()` (Label = Anlagen-Nr. — Kunde der Anlage). Die Kacheln UVV und Anlagen öffnen die Anlagenliste, Projekte die Projektübersicht.

### Dashboard (KPI) — `renderDashboardKpi()`, seit v1.6.81 im `statisticsView`
- **Seit v1.6.81 liegen `#kpiYearTitle`, `#kpiTiles`, `#kpiMyTasks` und `#kpiActions` im `statisticsView` (zwischen Jahresfilter und „Projekt-Status Übersicht“); die Startseite zeigt sie nicht mehr.** `renderDashboardKpi()` läuft in `showView('statistics')`, in `renderHome()` und in `loadFromSharePoint()` und legt `window._kpiSummary = { openTasks, actionCount }` für die Startseite ab (`actionCount` = Anzahl der Einträge der Aktionsliste, nicht der Projekte). Die folgenden Angaben gelten unverändert für diesen Code.
- Zeigt immer das laufende Kalenderjahr (`new Date().getFullYear()`), Jahr per `getYearFromProjectNumber()`, Daten aus `window.allProjects`. Aufruf in `showView('dashboardKpi')` und in `loadFromSharePoint()` direkt nach `renderDashboard()`.
- **Kacheln** (`#kpiTiles`, Flex mit Umbruch): VK Umsatz lfd. Jahr (mit Abgerechnet/Offen über `hasInvoice`), Deckungsbeitrag nur nachkalkulierter Projekte (`hasPostCalc`; grün nur bei DB > 0, sonst rot), Aktive Projekte (`!hasInvoice`).
- **Meine offenen Aufgaben** (`#kpiMyTasks`): Member-ID = `email.split('.')[0]` aus `msalInstance.getAllAccounts()[0].username`, Abgleich mit `TEAM_MEMBERS`. **Bewusste Entscheidung des Entwicklers: kein Mapping-Objekt einführen, auch nicht bei zukünftigen Änderungen.** Ohne Treffer in `TEAM_MEMBERS` wird die Zone ausgeblendet. Angezeigt werden `project.tasks` und `project.purchasePositions` mit `assignedTo === Member-ID` und Status `open`. Je Zeile steht neben der Projektnummer der Kunde (`project.customer`), bei Einkaufspositionen zusätzlich die Stückzahl (`quantity`, als „5× Beschreibung“); `project.tasks` haben kein Mengenfeld.
- **Aktionsliste** (`#kpiActions`): ohne Nachkalkulation (`!hasPostCalc`), geliefert aber nicht abgerechnet, ohne AB (`!hasAB && !hasInvoice`), offene Aufgaben je Mitglied (`project.tasks` mit Status ≠ `done`), Liefertermine der nächsten 14 Tage (ohne `hasDelivered`). Projektnummern öffnen das Projekt per `openProject()`.

### Kalkulationsvarianten — `quotesView` (ab v1.6.58)
- Kalkulationen („Vorkalkulation“) sind **nicht** Teil der Projekte, sondern eigene Einträge der SharePoint-Liste `DodekVorkalkulation` (`guidQuotes`, Felder `QuoteTitle`, `Customer`, `QuoteData`). Die Varianten liegen im bestehenden JSON-Feld `QuoteData`; es gibt kein neues Feld und keine neue Liste.
- **Speicherformat:** `{ id, title, customer, date, activeVariant, variants: [{ id, label, createdAt, positions, factorReseller, factorEndcustomer, discount }] }`. Positionen, beide Faktoren und Projektrabatt gelten je Variante; Bezeichnung, Kunde und Datum gemeinsam.
- **Arbeitskopie:** `window.currentQuote` enthält zusätzlich die Felder `positions`, `factorReseller`, `factorEndcustomer`, `discount` der aktiven Variante. Alle bestehenden Positions-Funktionen arbeiten weiter auf `currentQuote.positions`. Beim Speichern entfernt `buildQuoteForStorage()` diese Arbeitsfelder.
- **Migration:** `migrateQuoteToVariants()` packt Altdaten (flaches Objekt) im Speicher als `v1` („V1 – Erstkalkulation“) ein; `createdAt` von v1 = Kalkulationsdatum (Annahme). Das passiert in `loadQuotesFromSharePoint()` und in `openQuoteEditor()`; zurückgeschrieben wird erst beim nächsten Klick auf „Speichern“.
- **Variantenwechsel** (`switchQuoteVariant()`) speichert zuerst die gesamte Kalkulation nach SharePoint (`saveQuote({ silent: true, activateVariant: id })`, die neue Variante wird als `activeVariant` mitgeschrieben) und lädt dann die gewählte Variante. Schlägt das Speichern fehl (Netzwerk, fehlende Bezeichnung), bleibt die Ansicht unverändert und die Fehlermeldung erscheint. `saveQuote()` liefert `true`/`false` zurück.
- **Löschen** (`deleteQuoteVariant()`): das × erscheint nur bei mindestens zwei Varianten und nur auf nicht aktiven Tabs; die aktive Variante bleibt aktiv. Gelöscht wird nur im Speicher, geschrieben wird beim nächsten Speichern oder Variantenwechsel.
- **„+ Variante“** kopiert im Speicher und macht die Kopie aktiv; es speichert nicht.
- `renderQuoteList()` zeigt EK/Rabatt/VK der aktiven Variante (`quoteActiveVariant()`).

### Kalkulation: Titel-Positionen und Variantenvergleich (ab v1.6.59)
- **Titel-Positionen (Gewerke):** Eintrag in `positions` mit `type: 'title'` (`{ id, type, description, totalEK: 0 }`); alle anderen Positionen sind implizit `type: 'item'`. `totalEK: 0` sichert Summenschleifen ab. `isQuoteTitle(p)` prüft den Typ. Titel liegen in der Variante und werden beim Kopieren mitgenommen.
- **Abschnitt** = Titel bis zum nächsten Titel bzw. Listenende. Positionen vor dem ersten Titel gehören zu keinem Abschnitt. `renderQuotePositions()` zeigt unter der letzten Position eines Abschnitts eine Zwischensumme (EK, WV, End; VK je Position gerundet wie in `renderQuoteSummary()`). Titel verschieben und löschen wie Positionen (`moveQuotePosition()`, `deleteQuotePosition()`); „+ Titel“ → `addQuoteTitle()`, Text ändern → `updateQuoteTitle()`.
- **Titel-sicher:** Summen (`renderQuoteSummary()`, Summen in `printQuote()`) und Umwandlung in ein Projekt (`convertQuoteToProject()`) ignorieren Titel; bei der Umwandlung werden Titel bisher nicht übernommen (offen). Neue Schleifen über `positions` müssen Titel überspringen.
- **Druck (`printQuote()`):** Titel als hervorgehobene Zeile (`.pos-title`), nach der letzten Position eines Abschnitts eine Zwischensumme-Zeile (`.pos-subtotal`) mit EK, VK Wiederverkauf und VK Endkunde, analog zur Bildschirmdarstellung. Positionen vor dem ersten Titel und Titel ohne Positionen erhalten keine Zwischensumme.
- **Variantenvergleich:** `renderQuoteCompareBar()` (Leiste unter den Varianten-Tabs, wird am Ende von `renderQuoteSummary()` und `renderQuoteVariantBar()` aktualisiert) und `openQuoteCompare()` (Modal `#quoteCompareModal`, schließt per ×, Klick außerhalb und Escape). Datenquelle: alle Varianten in `window.currentQuote`, die aktive mit dem Live-Stand des Editors (`quoteCompareVariants()`); Summen aus `quoteVariantTotals()`. Es wird kein SharePoint-Call ausgeführt.
- **Alternativpositionen (ab v1.6.61):** Feld `included` an Positionen (nur `item`); fehlt es oder ist es nicht `false`, gilt die Position als einbezogen (`isQuoteIncluded(p)`). `included: false` schließt die Position aus EK/VK/DB, Abschnitts-Zwischensummen, Vergleichsleiste/-modal und Druck-Summen aus; sie bleibt sichtbar, editierbar (gelber Zeilenhintergrund) und im Druck mit „Alt."-Label und Gelbton. Checkbox ganz links (`toggleQuotePositionIncluded()`), Zustand je Variante. `convertQuoteToProject()` wurde nicht angepasst (übernimmt weiterhin alle Positionen, auch nicht einbezogene — offen). Pfeil-Buttons ↑/↓ je Position und Titel (`moveQuotePosition()`); an Listenanfang/-ende entfällt der jeweilige Pfeil.
- Anzeige je Variante: EK, Rabatt, VK Wiederverkauf und VK Endkunde mit DB € und DB %, bei Titeln zusätzlich VK-Zwischensummen je Abschnitt (Zuordnung über den Titeltext). Differenz zur vorherigen Variante absolut und prozentual; Farbe aus Kundensicht: VK höher = rot, VK niedriger = grün, Rabatt höher = grün; EK und DB bewusst neutral grau.


### Anlagen — `anlagenView` (ab v1.6.63)
- Daten aus SharePoint-Liste `DodekAnlagen` (`guidAnlagen`); `guidUVVPruefungen` und `guidUVVModule` werden ab v1.6.67 für das UVV-Prüfformular verwendet (siehe unten). `window.anlagenDatabase` (Schlüssel = SharePoint-Id), geladen von `loadAnlagenFromSharePoint()` in `initAppAfterLogin()`.
- **Annahme (nicht gegen die Liste geprüft):** interne Spaltennamen `AnlagenNr`, `Bezeichnung`, `Kunde`, `Standort`, `Ursprungsprojekt`, `Inbetriebnahme`, `AktiveModule`, `Notizen`, `VerknuepfteProjekte`. Gibt es keine Spalte `AnlagenNr`, wird die Standardspalte `Title` verwendet; ist `Title` zusätzlich zu `AnlagenNr` vorhanden, wird sie mit derselben Nummer befüllt. Fehlt eine andere Spalte, bricht das Speichern mit einer Fehlermeldung ab (`buildAnlagePayload()`, liest die Spalten einmalig über `/fields`). Ist `Inbetriebnahme` vom Typ DateTime, wird `YYYY-MM-DDT00:00:00Z` geschrieben.
- `AktiveModule` und `VerknuepfteProjekte` liegen als JSON-Text in der Liste: `[0,1,2,14]` (ab v1.6.79 keine feste Vorbelegung von 0 und 14 mehr; gespeichert wird, was im Modal angehakt ist) bzw. `[{ "projektnr": "…", "typ": "Erstverkauf|UVV-Prüfung|Ersatzteile|Sonstiges" }]`. Modulnamen in `ANLAGEN_MODULE_NAMES` (0 = „Deckblatt / Prüfzertifikat“, 14 = „Allgemeine Angaben“).
- **Anlagentyp:** Werte aus `DodekEinstellungen` (`SettingKey: "anlagenTypen"`, JSON-Array; `getAnlagenTypen()`), Fallback `DTL, DTZ, DTS, DTM`. Verwaltung in `settingsView` (Abschnitt „Anlagentypen“, `addAnlagenTyp()`/`removeAnlagenTyp()`, speichert sofort über `saveSetting()`; Typen dürfen keinen Bindestrich enthalten; eine leere Liste fällt auf den Fallback zurück). Der Typ wird zusätzlich in der Spalte `AnlagenTyp` gespeichert. **Diese Spalte muss in `DodekAnlagen` angelegt werden (einzeilig Text); die App legt sie nicht an.** Fehlt sie, wird ohne sie gespeichert (Hinweismeldung), und der Typ wird beim Laden aus der Nummer (`DT-…-TYP-n`) abgeleitet (`anlagenTypFromNr()`). Die Spaltenliste wird pro Sitzung einmal gelesen — nach dem Anlegen der Spalte Seite neu laden.
- Easter Egg: In den Jahres-Subnavs (Projektübersicht, Anlagen) steht vor jeder Jahreszahl das chinesische Tierkreiszeichen (`getZodiacEmoji(year)`, 12-Jahres-Zyklus, 2020 = 🐭). Neue Subnavs mit Jahreszahlen sollen es ebenfalls verwenden.
- Sidebar: „🏭 Anlagen“ mit Subnav „📋 Alle“ + Inbetriebnahmejahre (`updateAnlagenSubnav()`, `navToAnlagenYear()`, Filter in `window._anlagenYearFilter`). `showView('anlagen')` zeigt immer die Liste; Detail ist eine Unteransicht innerhalb der View (`#anlagenDetailPane`).
- Liste: AnlagenNr, Bezeichnung, Kunde, Standort, Ursprungsprojekt, Inbetriebnahme, Prüfungen (Anzahl aus `window.uvvPruefungen`). Modal (`openAnlageModal(originProject, editId)`): Pflichtfeld Anlagentyp (Dropdown) über der Anlagen-Nr.; Nummernvorschlag `DT-[Ursprungsprojekt]-[Typ]-[lfd. Nr.]`, ohne Projekt `DT-[Typ]-[lfd. Nr.]` (lfd. Nr. = max(Anzahl vorhandener Anlagen mit gleicher Projekt/Typ-Kombination, größte Nummer mit gleichem Präfix) + 1; wird nach Typ-/Projektwechsel neu vorgeschlagen, solange die Nummer nicht von Hand geändert wurde). Doppelte Nummern werden abgelehnt. Mit `editId` (Button „✏️ Bearbeiten“ im Detail) wird die Anlage per PATCH aktualisiert; die Anlagen-Nr. bleibt änderbar (Tippfehler), die Eindeutigkeit gegenüber allen anderen Anlagen wird geprüft, Verknüpfungen bleiben unberührt. Button „Anlage erfassen“ im `editorView` (nur admin/editor) öffnet dasselbe Modal mit vorbefülltem Ursprungsprojekt (und Kunde, falls das Projekt geladen ist); beim Speichern einer so gestarteten Neuanlage fragt `confirm` „Projekt … als Erstverkauf verknüpfen?“, bei Ja steht das Projekt als `Erstverkauf` sofort in `VerknuepfteProjekte` (`window._anlageOriginProject`). Button „Mit Anlage verknüpfen“ (ab v1.6.80, `anlageLinkBtn`, nur admin/editor, `openProjektAnlageLinkModal()`/`saveProjektAnlageLink()`) verknüpft das geöffnete Projekt mit einer bestehenden Anlage (Typ Erstverkauf, UVV-Prüfung, Ersatzteile, Sonstiges).
- Detail: Stammdaten, aktive Module, „Verknüpfte Projekte“ (Projektnummer öffnet per `openProject()`, nur wenn das Projekt geladen ist), „Prüfungen“ (Liste, „＋ Neue Prüfung“). Verknüpfungen lassen sich per × (nach Bestätigung, PATCH) entfernen. „🗑️ Anlage löschen“ (nur admin, `deleteAnlage()`): Bestätigung durch Eintippen der Anlagen-Nr., DELETE auf den SharePoint-Eintrag, Rückkehr zur Liste; Prüfungen in `DodekUVVPruefungen` werden nicht mitgelöscht (Hinweis im Dialog).

### UVV-Prüfformular (ab v1.6.67)
- **Annahmen (nicht gegen die Listen geprüft):** `DodekUVVModule` (Einzelspalten ab v1.6.69, keine Spalte `ModulData`): `Title` (= ModulNr als String), `ModulNr` (Zahl), `ModulName`, `Aktiv` (Ja/Nein), `Pflicht` (Ja/Nein), `Pruefpunkte` (Mehrzeilentext, JSON-Array `[{ nr, bezeichnung, typ, optionen, einheit }]`, ab v1.6.70; ältere Einträge `{ id, text, intervall, position }` bleiben lesbar). `DodekUVVPruefungen` (Spaltennamen ab v1.6.68 an die Liste angepasst): `Title`, `AnlagenNr`, `PruefDatum`, `Pruefer`, `Status` (Choice `Offen` / `In Bearbeitung` / `Abgeschlossen`), `Ergebnisse` (Mehrzeilentext). `AnlagenNr` enthält weiterhin die SharePoint-Id der Anlage (Text oder Zahl, Lookup nicht unterstützt), nicht die Anlagen-Nr. selbst — Entscheidung offen. Fehlt eine Spalte, bricht das Speichern mit Fehlermeldung ab (`buildUvvPayload()`, liest die Spalten einmalig über `/fields`).
- **Module:** `getUVVModule()` lädt einmal pro Sitzung (`uvvModuleCache`). Standarddaten in `UVV_MODULE_DEFAULTS` (Module 0–14 mit Prüfpunkt-Eingabetypen, aus `korrektur_pruefpunkt_typen.md`). `settingsView` → „📝 UVV-Module“: Button „DodekUVVModule initialisieren“ (`initUVVModule()`, nur admin; bei vorhandenen Modulen Bestätigung; fehlende Module werden angelegt, bei vorhandenen (per ModulNr) werden nur die `Pruefpunkte` überschrieben, Name/Aktiv/Pflicht bleiben; nicht enthaltene Module wie 15 bleiben unberührt) und Modulliste mit Aktiv-Toggle (`toggleUVVModulAktiv()`, PATCH auf die Spalte `Aktiv`); Pflichtmodule (`Pflicht: true`, 0 und 14) sind nicht abschaltbar.
- **Prüfungen:** `window.uvvPruefungen` (Schlüssel = SharePoint-Id), geladen von `loadUVVPruefungenFromSharePoint()` in `initAppAfterLogin()`. Im Anlagendetail „Prüfungen“ mit „＋ Neue Prüfung“ (Datum, Prüfer; legt Status `Offen` an). Öffnen: `Offen` → Modulauswahl (Pflichtmodule fest, ausschließlich nach `Pflicht` in `DodekUVVModule`, keine Hardcodierung von 0 und 14 mehr ab v1.6.78; `pfStart()` bricht mit Fehlermeldung ab, wenn Modul 0 nicht in der Liste ist (ab v1.6.79); Vorauswahl = aktive Module der Anlage), `In Bearbeitung` → Formular mit gespeichertem Stand, `Abgeschlossen` → Read-only. Leser sehen immer Read-only.
- **Formular:** je Modul ein `<details>`-Abschnitt, je Prüfpunkt nach `typ`: `freitext` (Textfeld, bei „Bemerkung…“ Textarea), `auswahl` / `ja_nein` (Schaltflächen aus `optionen` plus „entfällt“, erneuter Klick hebt auf; ohne `typ` Fallback i.O./n.i.O.) und `messwert` (Zahlenfeld mit `einheit`); Bemerkung zu Auswahl- und Messwert-Punkten (bei „n.i.O.“ automatisch sichtbar). Der Wert steht als Text im bisherigen Feld `bewertung` (alte Codes `io`/`nio`/`entfaellt` werden beim Laden in „i.O.“/„n.i.O.“/„entfällt“ umgesetzt), je Modul freie Bemerkung. State in `window._pf`. `Ergebnisse`: `{ module, ergebnisse:[{ modulNr, pruefpunktId, bewertung, bemerkung }], modulBemerkungen:{ "<ModulNr>": text }, abgeschlossenAm, pruefer }` (`modulBemerkungen` ist eine Ergänzung gegenüber der Vorgabe, `ergebnisse` enthält nur bearbeitete Prüfpunkte). „Zwischenspeichern“ → `In Bearbeitung`; „Prüfung abschließen“ verlangt je Pflichtmodul mindestens eine Bewertung → `Abgeschlossen` (+ `abgeschlossenAm`). „Prüfung wieder öffnen“ nur für admin (zurück auf `In Bearbeitung`).
- **Prüfung löschen (ab v1.6.71):** Papierkorb-Button je Zeile der Prüfungsliste im Anlagendetail (`deletePruefung()`), Bestätigungsdialog, DELETE auf den SharePoint-Eintrag, danach Neuladen der Prüfungen. Bewusst ohne Rollenprüfung (jeder angemeldete Nutzer, auch Leser).
- **Vorjahreswerte (ab v1.6.71):** `pfFindVorjahr()` nimmt die zuletzt abgeschlossene, frühere Prüfung derselben Anlage (nach `PruefDatum` absteigend; die aktuelle und nicht abgeschlossene Prüfungen zählen nicht). Ohne Treffer gibt es keinen Vorjahresbereich und keine Buttons. Je Prüfpunkt zeigt „↩ Vorjahr: Wert“ (nur für Module, die in der Vorjahresprüfung enthalten waren). „✓ übernommen“ (grüner linker Rand) wird ausschließlich durch „Aus Vorjahr übernehmen“ gesetzt (nur im bearbeitbaren Formular sichtbar, in abgeschlossenen/Read-only-Prüfungen nie) und verschwindet für einen Prüfpunkt, sobald der Nutzer ihn manuell anfasst (Auswahl-Klick, Text- oder Messwert-Eingabe), unabhängig davon, ob der Wert dem Vorjahreswert entspricht. Je Modul (nur bearbeitbar): „↩ Aus Vorjahr übernehmen“ (setzt alle Prüfpunkte mit Vorjahreswert, nur Werte, keine Bemerkungen) und „Markierungen entfernen“ (Werte bleiben). Die Markierung wird als optionales Feld `uebernommen: true` im Eintrag von `Ergebnisse` gespeichert.
- **Unterschriften + PDF-Export (ab v1.6.73, Layout ab v1.6.74, Kopfbalken und Prüfsiegel ab v1.6.75, Siegel ab v1.6.76, Kopfbalken ab v1.6.77):** In Modul 0 zwei Canvas-Felder (Auftraggeber / Prüfer), `pfSignaturesHtml()`, `pfInitSignatures()` (Pointer-Events, Finger/Stift), `pfClearSig()`; Base64-PNG in `window._pf.sigKunde` / `sigPruefer`, gespeichert in den Spalten `Unterschrift_Kunde` / `Unterschrift_Pruefer` von `DodekUVVPruefungen` (**Annahme: Spalten existieren, Mehrzeilentext; nicht gegen die Liste geprüft**; geschrieben werden sie nur, wenn eine Unterschrift vorhanden ist oder zuvor gespeichert war). `pfSave(true)` verlangt zusätzlich beide Unterschriften. Button „PDF exportieren“ nur bei Status `Abgeschlossen` (`pfExportPdf()` → `buildPruefbuchPdf()`): A4 hoch im Dodek-Corporate-Design (roter Kopfbalken über volle Seitenbreite, Höhe am senkrechten Strich des „D“ im Logo ausgerichtet (Bildzeilen 173–513 von 1328, ausgemessen), keine Trennlinie darunter; roter Fußbalken #CC0000, Grautöne), Prüfsiegel als schwarzer Stempelkreis (gebogener Text „DODEK TECHNIK“ / „PRÜFSIEGEL“, graue Schraffur, Unterschrift Prüfer zwischen zwei Linien, Datum darunter), Seite 1 Prüfzertifikat mit Schreiblinien-Feldern (Felder aus Modul 0 per Bezeichnungs-Präfix „Auftraggeber“, „Angaben zur Anlage“/„Interne Angaben“, „Nächster…“), danach je Modul eine Seite: von Hand gezeichnetes Zeilenraster (Nr, Bezeichnung, Ergebnis/Wert, Bemerkung), auswahl/ja_nein als Checkboxen, freitext/messwert als Schreiblinie mit Wert darüber; Modul 14 als Formular (Gruppen nach Präfix vor „ – “, Folgeprüfungen dreispaltig), Fußzeile = erstes Wort des Modulnamens in Großbuchstaben (Annahme, kein eigenes Feld). jsPDF 2.5.1 wird erst beim Export von cdnjs geladen (mit SRI-Hash); `DT-Logo.png` wird relativ zur App geladen und auf 800 px verkleinert. Kein Mailversand, keine Unterschrift-Validierung über „nicht leer“ hinaus.
- Nicht enthalten (Phase 3 Rest): Prüfpunkt-Bearbeitung im UI, Statistiken, E-Mail.
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
| `dashboardKpiView` | Startseite (ab v1.6.81: Schnellzugriff-Kacheln, Handlungsbedarf, Zuletzt geöffnet; Startansicht) |
| `dashboardView` | Projektübersicht (Projektliste nach Jahren) |
| `tasksView` | Aufgabenverwaltung nach Mitglied |
| `stockView` | Lagerverwaltung |
| `quotesView` | Kalkulation (BOM, VK/EK-Positionen) |
| `contactsView` | Kunden & Lieferanten |
| `articlesView` | Artikelstamm |
| `anlagenView` | Anlagen (Liste, Detail, Neuanlage; ab v1.6.63) |
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
