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
- Farbliche Statusanzeige: Rot (kein AB) / Grün (AB) / Gelb (Rechnung offen) / Lila (Nachkalk. ausstehend) / Weiß (vollständig)
- Schnellzugriff auf AB, Rechnung, Geliefert, Nachkalkulation per Checkbox
- Volltextsuche über Projektnummer, Kunde, VK/EK-Positionen, Lieferanten, Aufgaben
- Jahresfilter + Status-Filter (Kein AB / AB / Rechnung / Nachkalk. ausstehend / Vollständig)
- VK/EK/DB-Summen pro Jahrgang
- **Projekt kopieren** — übernimmt VK/EK-Positionen und Gewerke ohne Stammdaten

### 📁 Projekteditierung
- Stammdaten: Projektnummer, Kunde, Projektbezeichnung, Auftragsnummer, Lieferdatum (mit KW-Anzeige)
- **Gewerke-Schnellauswahl** per Buttonbox + Freitexteingabe für Sonderfälle
- **Verkaufspositionen** (VK): Manuell oder aus Artikelstamm, Einzelpreis editierbar, Drag & Drop Sortierung
  - BOM-Dialog: wenn VK-Artikel eine Stückliste hat → automatischer Import als EK-Positionen
- **Einkaufspositionen** (EK): Manuell, aus Artikelstamm oder per BOM-Import, Einzelpreis editierbar, Drag & Drop Sortierung, mit:
  - Bestellstatus: Offen / Bestellt / Teilgeliefert / Geliefert / Aus Lager / Abgerechnet
  - Lieferantenartikelnummer pro Position
  - Zuständigkeitszuweisung (Franek, Hagen, Harry, Nicole)
  - Mehrere Positionen eines Lieferanten per Dialog
- **Aufgaben**: Projektbezogene Freitextaufgaben mit Zuständigkeit und Status
- **"Projekt abschließen"** — setzt alle EK-Positionen auf Abgerechnet / Nicht notwendig
- Gewerke-Tabelle mit VK/EK/DB-Aufschlüsselung

### 📋 Aufgabenliste (projektübergreifend)
- Drei Gruppen: **🛒 Einkauf** / **📌 Allgemein** / **📦 Lager** (Nachbestellbedarf)
- Filter nach Zuständigkeit und Jahr
- Checkbox "Erledigte ausblenden" — blendet Geliefert, Aus Lager, Abgerechnet und Erledigt aus

### 📦 Lagerverwaltung
- Lagerartikel mit Bestand, Mindestmenge, Einheit, optionaler Artikelnummer
- Übernahme aus Artikelstamm per Dropdown
- Farbliche Kennzeichnung: grün = OK, rot = unter Mindestmenge
- Kritische Artikel erscheinen automatisch in der Aufgabenliste

### 🗂️ Artikelstamm
- Artikel mit ID, Bezeichnung, EK-Preis, Marge, VK-Preis, Gewerk, Typ
- Artikel-ID beim Bearbeiten änderbar
- Stücklisten (BOM) mit Lieferant, Lieferantenartikelnummer, Komponente, Menge, Preis
- BOM-Import: Stückliste wird automatisch in EK-Positionen umgewandelt

### 📈 Statistiken
- Jahresbezogene Auswertungen
- Top-10 Kunden, Gewerke-Analyse, zeitliche Verteilung

### 💾 Export
- Excel-Export pro Projekt
- JSON-Export / -Import pro Projekt
- JSON-Backup / -Restore
- Druckbarer PDF-Report

---

## Nutzerrechte

Rollenbasiertes Berechtigungssystem auf Basis des Microsoft-Logins.

| Recht | Admin | Editor | Reader |
|---|---|---|---|
| Projekte lesen | ✅ | ✅ | ✅ |
| Projekte anlegen/bearbeiten | ✅ | ✅ | ❌ |
| Projekte löschen | ✅ | ❌ | ❌ |
| Projekt kopieren | ✅ | ✅ | ❌ |
| EK/VK Positionen bearbeiten | ✅ | ✅ | ❌ |
| Aufgaben bearbeiten | ✅ | ✅ | ❌ |
| Lager bearbeiten | ✅ | ✅ | ❌ |
| Artikelstamm nutzen | ✅ | ✅ | ✅ |
| Artikelstamm anlegen/bearbeiten | ✅ | ❌ | ❌ |
| Alles löschen | ✅ | ❌ | ❌ |

### Rollenzuweisung

```javascript
const PERMISSIONS = {
    'franek.dodek@dodek.de':  'admin',
    'hagen.dodek@dodek.de':   'editor',
    'nicole.merk@dodek.de':   'editor',
    'harry.dodek@dodek.de':   'reader'
};
```

Unbekannte E-Mail-Adressen erhalten automatisch `reader`. Die Rolle wird nach dem Login oben rechts angezeigt: 👑 Admin / ✏️ Editor / 👁️ Leser.

---

## Technischer Aufbau

- **Single-file HTML-Anwendung** — keine Installation, läuft direkt im Browser
- **Hosting**: GitHub Pages
- **Datenspeicherung**: SharePoint-Listen über Microsoft REST API
- **Authentifizierung**: Microsoft OAuth 2.0 (Azure AD / Entra ID), delegierte Berechtigung `AllSites.Write`
- **Abhängigkeiten**: MSAL Browser 2.38.3 (lokal im Repo), SheetJS (CDN)
- **Nutzer**: 4 Personen mit M365-Account

### SharePoint Konfiguration

| Parameter | Wert |
|---|---|
| SharePoint Site | `https://dodekgmbh.sharepoint.com/sites/DodekProjektverwaltung` |
| Liste Projekte | `DodekProjekte` |
| Liste Artikel | `DodekArtikel` |
| Liste Lager | `DodekLager` |
| Redirect URI | `https://franekdodek-tech.github.io/dodek-projektverwaltung/Projektuebersicht_APP.html` |

> Listen werden über GUID angesprochen — konfiguriert in `SP_CONFIG` im App-Code.

---

## Phasenstatus

### Phase 1 + 2 ✅ Abgeschlossen
- SharePoint Mehrbenutzer-Betrieb, Microsoft Login
- EK-Bestellstatus, Zuständigkeit, Aufgaben, Lager
- Gewerke-Schnellauswahl, BOM-Dialog, Projekt kopieren
- Rollenbasierte Nutzerrechte (Admin / Editor / Reader)
- Status-Filter im Dashboard, violette Zeilen für ausstehende Nachkalkulation

### Phase 2d ⏳ Vorkalkulation
- Kalkulation aus Artikelstamm mit zwei Faktoren (Wiederverkauf / Endkunde)
- Deckungsbeitragsanzeige
- Druckbares Ausgabeformat

### Phase 3 ⏳ Ausstehend
- Power Automate: E-Mail-Benachrichtigung bei Aufgabenzuweisung

---

## Konfiguration

### Standard-Gewerke
```javascript
const DEFAULT_TRADES = [
    'Filteranlage', 'Ventilator', 'WRG', 'Rohrleitung',
    'Elektrik', 'Steuerung', 'Montage', 'UVV',
    'Schallschutz', 'Erfassungselemente', 'Ersatzteile'
];
```

---

## Deployment

```
main  →  GitHub Pages  →  https://franekdodek-tech.github.io/dodek-projektverwaltung/
```
