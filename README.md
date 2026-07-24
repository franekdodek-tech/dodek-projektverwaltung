# Dodek Hub

Operatives Steuerungssystem für Dodek GmbH & Co. KG — Projekte, Einkauf, Aufgaben, Artikelstamm und Lager in einer Anwendung.

🌐 **App aufrufen:** [https://franekdodek-tech.github.io/dodek-projektverwaltung/](https://franekdodek-tech.github.io/dodek-projektverwaltung/)

---

## Zweck

Dodek Hub ist das zentrale operative System für ca. 150 Aufträge pro Jahr. Es deckt ab: Projektsteuerung, Angebots- und Auftragskalkulation, Einkaufsverwaltung, Aufgaben- und Teamkoordination, Artikelstamm mit Stücklisten sowie Lagerverwaltung.

---

## Funktionsumfang

### 📊 Dashboard
- Alle Projekte gelistet, nach Jahrgängen gruppiert und kollabierbar
- Farbliche Statusanzeige je Projekt (kein AB / AB gesetzt / Rechnung / Nachkalkulation)
- Schnellzugriff auf AB, Rechnung, Geliefert, Nachkalkulation per Checkbox
- Volltextsuche über Projektnummer, Kunde, Bezeichnung und Artikelpositionen
- Jahresfilter
- VK/EK/DB-Summen pro Jahrgang
- **Projekt kopieren** — übernimmt VK/EK-Positionen und Gewerke ohne Stammdaten (Gedächtnisstütze für Standardkomponenten)

### 📁 Projekteditierung
- Stammdaten: Projektnummer, Kunde, Projektbezeichnung, Auftragsnummer, Lieferdatum (mit KW-Anzeige)
- **Gewerke-Schnellauswahl** per Buttonbox (Filteranlage, Ventilator, WRG, Rohrleitung, Elektrik, Steuerung, Montage, Service, Schallschutz, Erfassungselemente) + Freitexteingabe für Sonderfälle
- **Verkaufspositionen** (VK): Manuell oder aus Artikelstamm, Einzelpreis editierbar, Drag & Drop Sortierung
  - BOM-Dialog: wenn VK-Artikel eine Stückliste hat → automatischer Import als EK-Positionen
- **Einkaufspositionen** (EK): Manuell, aus Artikelstamm oder per BOM-Import, Einzelpreis editierbar, Drag & Drop Sortierung, mit:
  - Bestellstatus: Offen / Bestellt / Teilgeliefert / Geliefert / Aus Lager / Abgerechnet
  - Lieferantenartikelnummer pro Position
  - Zuständigkeitszuweisung (Franek, Hagen, Harry, Nicole)
  - Mehrere Positionen eines Lieferanten per Dialog
- **Aufgaben**: Projektbezogene Freitextaufgaben mit Zuständigkeit und Status (Offen / In Bearbeitung / Erledigt)
- **"Projekt abschließen"** — setzt alle EK-Positionen auf Abgerechnet / Nicht notwendig
- Gewerke-Tabelle mit VK/EK/DB-Aufschlüsselung
- Zusammenfassungskacheln (VK, EK, DB absolut und %)

### 📋 Aufgabenliste (projektübergreifend)
- Drei Gruppen: **🛒 Einkauf** (EK-Positionen) / **📌 Allgemein** (Projektaufgaben) / **📦 Lager** (Nachbestellbedarf)
- Filter nach Zuständigkeit und Jahr
- Checkbox "Erledigte ausblenden" — blendet Geliefert, Aus Lager, Abgerechnet und Erledigt aus
- Direkter Sprung ins Projekt bzw. ins Lager per Button

### 📦 Lagerverwaltung
- Lagerartikel mit Bestand, Mindestmenge, Einheit, optionaler Artikelnummer
- Übernahme aus Artikelstamm per Dropdown
- Farbliche Kennzeichnung: grün = OK, rot = unter Mindestmenge
- Kritische Artikel werden zuerst angezeigt
- Daten in SharePoint `DodekLager`
- Kritische Artikel erscheinen automatisch in der Aufgabenliste

### 🗂️ Artikelstamm
- Artikel mit ID, Bezeichnung, EK-Preis, Marge, VK-Preis, Gewerk, Typ
- Artikel-ID beim Bearbeiten änderbar
- Stücklisten (BOM) pro Artikel mit Lieferant, Lieferantenartikelnummer, Komponente, Menge, Preis
- Artikelauswahl beim Anlegen von VK- und EK-Positionen
- BOM-Import: Stückliste wird automatisch in EK-Positionen umgewandelt (inkl. Lieferantenartikelnummern)

### 📈 Statistiken
- Jahresbezogene Auswertungen
- Top-10 Kunden nach Umsatz, Projektanzahl und Deckungsbeitrag
- Gewerke-Analyse
- Zeitliche Verteilung

### 💾 Export
- Excel-Export pro Projekt (Verkauf, Einkauf, Gewerke auf separaten Sheets)
- JSON-Export / -Import pro Projekt
- JSON-Backup / -Restore für Gesamtdaten und Artikelstamm
- Druckbarer PDF-Report

---

## Technischer Aufbau

- **Single-file HTML-Anwendung** — keine Installation, läuft direkt im Browser
- **Hosting**: GitHub Pages ([franekdodek-tech.github.io](https://franekdodek-tech.github.io/dodek-projektverwaltung/))
- **Datenspeicherung**: SharePoint-Listen über Microsoft REST API
- **Authentifizierung**: Microsoft OAuth 2.0 (Azure AD / Entra ID), delegierte Berechtigung `AllSites.Write`
- **Abhängigkeiten**: MSAL Browser 2.38.3 (lokal im Repo), SheetJS (CDN, Excel-Export)
- **Zielplattform**: Desktop-Browser (Chrome, Edge, Firefox)
- **Nutzer**: 4 Personen mit M365-Account (Dodek GmbH & Co. KG)

### SharePoint Konfiguration

| Parameter | Wert |
|---|---|
| SharePoint Site | `https://dodekgmbh.sharepoint.com/sites/DodekProjektverwaltung` |
| Liste Projekte | `DodekProjekte` (GUID in `SP_CONFIG`) |
| Liste Artikel | `DodekArtikel` (GUID in `SP_CONFIG`) |
| Liste Lager | `DodekLager` (GUID in `SP_CONFIG`) |
| Tenant-ID | in `SP_CONFIG` im App-Code |
| Client-ID | in `SP_CONFIG` im App-Code |
| Redirect URI | `https://franekdodek-tech.github.io/dodek-projektverwaltung/Projektuebersicht_APP.html` |

> **Hinweis:** SharePoint-Listen werden über GUID angesprochen (nicht über Namen) da der interne Titel in dieser Umgebung leer bleibt.

---

## Phasenstatus

### Phase 1 ✅ Abgeschlossen
- EK-Bestellstatus und Zuständigkeit
- "Projekt abschließen" Button
- Aufgaben-Sektion im Projekt
- Aufgabenliste projektübergreifend (Einkauf / Allgemein / Lager)
- Einzelpreis editierbar in VK und EK
- Drag & Drop Sortierung
- Gewerke-Schnellauswahl per Buttonbox
- BOM-Import Dialog beim VK-Hinzufügen
- Lieferantenartikelnummer in BOM und EK
- Projekt kopieren
- Lagerverwaltung mit SharePoint-Anbindung
- "Aus Lager" als EK-Bestellstatus

### Phase 2 ✅ Abgeschlossen
- SharePoint als zentrale Datenbasis (Mehrbenutzer-Betrieb für 4 Nutzer)
- Microsoft Login (Azure AD OAuth über GitHub Pages)
- Datenmigration aus localStorage-Backups
- SharePoint-Felder HasAB / HasInvoice / HasDelivered / HasPostCalc

### Phase 3 ⏳ Ausstehend
- Power Automate: E-Mail-Benachrichtigung bei Aufgabenzuweisung
- Voraussetzung: Phase 2 stabil im Produktivbetrieb

---

## Konfiguration

### Team-Mitglieder
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

### Standard-Gewerke
```javascript
const DEFAULT_TRADES = [
    'Filteranlage', 'Ventilator', 'WRG', 'Rohrleitung',
    'Elektrik', 'Steuerung', 'Montage', 'Service',
    'Schallschutz', 'Erfassungselemente'
];
```

---

## Datensicherung

Projektdaten liegen zentral in SharePoint — SharePoint versioniert automatisch. JSON-Backup / -Restore bleibt als Notfalloption in der App erhalten.

---

## Deployment

Änderungen an `Projektuebersicht_APP.html` im `main`-Branch werden automatisch über GitHub Pages deployed.

```
main  →  GitHub Pages  →  https://franekdodek-tech.github.io/dodek-projektverwaltung/
```
