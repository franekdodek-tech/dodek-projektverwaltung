# Dodek Projektverwaltung

Browserbasierte Projektcontrolling- und Nachkalkulations-App für Dodek GmbH & Co. KG.

🌐 **App aufrufen:** [https://franekdodek-tech.github.io/dodek-projektverwaltung/](https://franekdodek-tech.github.io/dodek-projektverwaltung/)

---

## Zweck

Die App dient als digitales Projektbuch für ca. 150 Aufträge pro Jahr. Sie erfasst alle laufenden Projekte, verwaltet Verkaufs- und Einkaufspositionen, ermöglicht eine vollständige Nachkalkulation pro Auftrag sowie projektübergreifende Auswertungen und Aufgabenverwaltung.

---

## Funktionsumfang

### Projektübersicht (Dashboard)
- Alle Projekte gelistet, nach Jahrgängen gruppiert und kollabierbar
- Farbliche Statusanzeige je Projekt (kein AB / AB gesetzt / Rechnung / Nachkalkulation)
- Schnellzugriff auf AB, Rechnung, Geliefert, Nachkalkulation per Checkbox
- Volltextsuche über Projektnummer, Kunde, Bezeichnung und Artikelpositionen
- Jahresfilter
- VK/EK/DB-Summen pro Jahrgang

### Projekteditierung
- Stammdaten: Projektnummer, Kunde, Projektbezeichnung, Auftragsnummer, Lieferdatum (mit KW-Anzeige)
- Gewerke-Verwaltung pro Projekt
- **Verkaufspositionen** (VK): Manuell oder aus Artikelstamm, mit Gewerk- und Typ-Zuordnung, Einzelpreis editierbar, Drag & Drop Sortierung
- **Einkaufspositionen** (EK): Manuell, aus Artikelstamm oder per BOM-Import, Einzelpreis editierbar, Drag & Drop Sortierung, mit:
  - Bestellstatus: Offen / Bestellt / Teilgeliefert / Geliefert / Abgerechnet
  - Zuständigkeitszuweisung an Teammitglied (Franek, Hagen, Harry, Nicole)
- **Aufgaben**: Projektbezogene Freitextaufgaben mit Zuständigkeit und Status (Offen / In Bearbeitung / Erledigt)
- **"Projekt abschließen"** Button — setzt alle EK-Positionen auf Abgerechnet / Nicht notwendig
- Gewerke-Tabelle mit VK/EK/DB-Aufschlüsselung
- Zusammenfassungskacheln (VK, EK, DB absolut und %)

### Aufgabenliste (projektübergreifend)
- Zwei Gruppen: **🛒 Einkauf** (EK-Positionen) und **📌 Allgemein** (Projektaufgaben)
- Filter nach Zuständigkeit und Jahr
- Checkbox "Erledigte ausblenden" (Standard: aktiv)
- Direkter Sprung ins Projekt per Button

### Artikelstamm
- Artikel mit ID, Bezeichnung, EK-Preis, Marge, VK-Preis, Gewerk, Typ
- Stücklisten (BOM) pro Artikel mit Lieferant, Komponente, Menge, Preis
- Artikelauswahl beim Anlegen von VK- und EK-Positionen
- BOM-Import: Stückliste eines Artikels wird automatisch in EK-Positionen umgewandelt

### Statistiken
- Jahresbezogene Auswertungen
- Top-10 Kunden nach Umsatz, Projektanzahl und Deckungsbeitrag
- Gewerke-Analyse
- Zeitliche Verteilung

### Export
- Excel-Export pro Projekt (Verkauf, Einkauf, Gewerke auf separaten Sheets)
- JSON-Export / -Import pro Projekt
- JSON-Backup / -Restore für Gesamtdaten und Artikelstamm
- Druckbarer PDF-Report

---

## Technischer Aufbau

- **Single-file HTML-Anwendung** — keine Installation, läuft im Browser
- **Hosting**: GitHub Pages ([franekdodek-tech.github.io](https://franekdodek-tech.github.io/dodek-projektverwaltung/))
- **Datenspeicherung**: SharePoint-Listen (`DodekProjekte`, `DodekArtikel`) über Microsoft REST API
- **Authentifizierung**: Microsoft OAuth 2.0 (Azure AD / Entra ID), delegierte Berechtigung `AllSites.Write`
- **Abhängigkeiten**: MSAL Browser 2.38.3 (lokal im Repo), SheetJS (CDN, Excel-Export)
- **Zielplattform**: Desktop-Browser (Chrome, Edge, Firefox)
- **Nutzer**: 4 Personen mit M365-Account (Dodek GmbH & Co. KG)

### Azure / SharePoint Konfiguration
| Parameter | Wert |
|---|---|
| SharePoint Site | `https://dodekgmbh.sharepoint.com/sites/DodekProjektverwaltung` |
| Liste Projekte | `DodekProjekte` |
| Liste Artikel | `DodekArtikel` |
| Tenant-ID | in `SP_CONFIG` im App-Code |
| Client-ID | in `SP_CONFIG` im App-Code |
| Redirect URI | `https://franekdodek-tech.github.io/dodek-projektverwaltung/Projektuebersicht_APP.html` |

---

## Phasenstatus

### Phase 1 ✅ Abgeschlossen
- Bestellstatus pro EK-Position (Offen / Bestellt / Teilgeliefert / Geliefert / Abgerechnet)
- Zuständigkeitszuweisung pro EK-Position an Teammitglied
- "Projekt abschließen" Button
- Projektübergreifende Aufgabenliste
- Einzelpreis editierbar in VK und EK
- Drag & Drop Sortierung für VK und EK Positionen
- Aufgaben-Sektion im Projekt (allgemeine Projektaufgaben)
- Aufgabenliste in zwei Gruppen (Einkauf / Allgemein) mit Filter

### Phase 2 ✅ Abgeschlossen
- SharePoint als zentrale Datenbasis (Mehrbenutzer-Betrieb)
- Microsoft Login (Azure AD OAuth über GitHub Pages)
- Datenmigration aus localStorage-Backups
- SharePoint-Felder HasAB / HasInvoice / HasDelivered / HasPostCalc für spätere Power Automate Nutzung

### Phase 3 ⏳ Ausstehend
- Power Automate: automatische E-Mail-Benachrichtigung bei Aufgabenzuweisung
- Voraussetzung: Phase 2 stabil im Produktivbetrieb

---

## Konfiguration

Team-Mitglieder werden in der Konstante `TEAM_MEMBERS` im `<script>`-Block gepflegt:

```javascript
const TEAM_MEMBERS = [
    { id: 'unassigned',   name: '-- Nicht zugewiesen --' },
    { id: 'not_required', name: '-- Nicht notwendig --' },
    { id: 'franek',       name: 'Franek' },
    { id: 'hagen',        name: 'Hagen' },
    { id: 'harry',        name: 'Harry' },
    { id: 'nicole',       name: 'Nicole' }
];
```

---

## Datensicherung

Projektdaten liegen zentral in SharePoint — kein manuelles Backup mehr erforderlich. SharePoint versioniert automatisch. JSON-Backup / -Restore bleibt als Notfalloption in der App erhalten.

---

## Deployment

Änderungen an `Projektuebersicht_APP.html` im `main`-Branch werden automatisch über GitHub Pages deployed. Kein Build-Prozess, keine Pipeline.

```
main  →  GitHub Pages  →  https://franekdodek-tech.github.io/dodek-projektverwaltung/
```
