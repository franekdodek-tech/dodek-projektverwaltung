# Dodek Projektverwaltung

Browserbasierte Projektcontrolling- und Nachkalkulations-App für Dodek Technik GmbH & Co. KG.

## Zweck

Die App dient als digitales Projektbuch für ca. 150 Aufträge pro Jahr. Sie erfasst alle laufenden Projekte, verwaltet Verkaufs- und Einkaufspositionen und ermöglicht eine vollständige Nachkalkulation pro Auftrag sowie projektübergreifende Auswertungen.

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
- **Verkaufspositionen** (VK): Manuell oder aus Artikelstamm, mit Gewerk- und Typ-Zuordnung
- **Einkaufspositionen** (EK): Manuell, aus Artikelstamm oder per BOM-Import, mit:
  - Bestellstatus: Offen / Bestellt / Teilgeliefert / Geliefert / Abgerechnet
  - Zuständigkeitszuweisung an Teammitglied
- Gewerke-Tabelle mit VK/EK/DB-Aufschlüsselung
- Zusammenfassungskacheln (VK, EK, DB absolut und %)

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

## Technischer Aufbau

- **Single-file HTML-Anwendung** — keine Installation, kein Server, läuft direkt im Browser
- **Datenspeicherung**: `localStorage` des Browsers (je Gerät lokal)
- **Abhängigkeiten**: SheetJS (Excel-Export, CDN)
- **Zielplattform**: Desktop-Browser (Chrome, Edge, Firefox)

## Geplante Erweiterungen

### Phase 1 (in Arbeit)
- [x] Bestellstatus pro EK-Position (Offen / Bestellt / Teilgeliefert / Geliefert / Abgerechnet)
- [x] Zuständigkeitszuweisung pro EK-Position an Teammitglied
- [ ] Projektübergreifende Aufgabenliste (analog Statistiken-View)

### Phase 2
- SharePoint als zentrale Datenbasis (Mehrbenutzer-Betrieb für 4 Nutzer, M365)
- Microsoft Login (Azure AD OAuth)

### Phase 3
- Power Automate: automatische E-Mail-Benachrichtigung bei Aufgabenzuweisung

## Konfiguration

Vor dem Einsatz sind zwei Konstanten im `<script>`-Block anzupassen:

```javascript
// Team-Mitglieder (Namen und IDs anpassen)
const TEAM_MEMBERS = [
    { id: 'unassigned', name: '-- Nicht zugewiesen --' },
    { id: 'member1',    name: 'Mitarbeiter 1' },
    ...
];
```

## Datensicherung

Da die Daten im `localStorage` des Browsers liegen, **regelmäßige JSON-Backups über die Backup-Verwaltung in der App anlegen**. Backups können jahresweise oder als Gesamtpaket exportiert und wiederhergestellt werden.

## Branching-Strategie

```
main          — stabile, getestete Version
feature/...   — aktive Entwicklungsblöcke
```

Commits folgen dem Schema: `Block N: Kurzbeschreibung der Änderung`
