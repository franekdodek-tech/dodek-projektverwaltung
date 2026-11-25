# Dodek Hub

Operatives Steuerungssystem für Dodek GmbH & Co. KG — Projekte, Einkauf, Kalkulation, Aufgaben, Artikelstamm und Lager in einer Anwendung.

🌐 **App:** https://franekdodek-tech.github.io/dodek-projektverwaltung/

---

## Aktuelle Versionen

| Datei | Version |
|---|---|
| `Projektuebersicht_APP.html` | v1.6.90 |
| `spedition.html` | v0.6.0 |
| `bestellung.html` | v0.6.0 |

---

## Funktionsumfang

### 🏠 Startseite (Startansicht, ab v1.6.81)
- Kopf: DT-Logo, „DODEK TECHNIK“, Nutzername, Online-Status (`navigator.onLine`)
- Schnellzugriff: Kacheln UVV-Prüfungen (offen), Projekte, Anlagen mit Zählern aus eigenen SharePoint-Calls (`$select=Id&$top=500`, bei Fehler „—“)
- Handlungsbedarf (ab v1.6.82): fünf ausklappbare Blöcke (Meine offenen Aufgaben, ohne Nachkalkulation, geliefert aber nicht abgerechnet, ohne AB, Liefertermine 14 Tage) mit den Listen der Statistik-Ansicht; Klick auf Eintrag öffnet das Projekt
- Zuletzt geöffnet: letzte 5 Einträge (UVV, Projekt, Anlage, Kalkulation, Stundenlohn-Nachweis) aus localStorage

### 📊 Dashboard-Kennzahlen (in Statistiken, laufendes Kalenderjahr)
- KPI-Kacheln: VK Umsatz (abgerechnet / offen), Deckungsbeitrag nachkalkulierter Projekte, aktive Projekte
- „Meine offenen Aufgaben“ des eingeloggten Nutzers (allgemeine Aufgaben und Einkaufspositionen mit Status „Offen“)
- Aktionsliste: ohne Nachkalkulation, geliefert aber nicht abgerechnet, ohne AB, offene Aufgaben je Mitglied, Liefertermine der nächsten 14 Tage
- Projektnummern öffnen das Projekt direkt

### 📋 Projektübersicht
- Projekte nach Jahrgängen gruppiert und kollabierbar
- Farbliche Statusanzeige (kein AB / AB / Rechnung / Nachkalkulation)
- Volltextsuche, Jahresfilter, VK/EK/DB-Summen pro Jahrgang
- Projekt kopieren

### 📁 Projekteditierung
- Stammdaten, Gewerke-Schnellauswahl, Lieferdatum mit KW
- **Verkaufspositionen (VK):** aus Artikelstamm oder manuell, Drag & Drop
- **Einkaufspositionen (EK):** Bestellstatus, Zuständigkeit, BOM-Import
- Aufgaben mit Status und Zuständigkeit
- Projekt abschließen

### 🧮 Kalkulation
- Positionen aus Artikelstamm oder manuell, BOM klappbar
- Faktoren: Wiederverkauf (WV) und Endkunde (End)
- EP und GP pro Position, Projektrabatt
- Position kopieren, als Artikel speichern
- Kalkulationsvarianten: Tab-Leiste mit beliebig vielen Varianten je Kalkulation (Positionen, Faktoren, Rabatt), Variante kopieren, umbenennen, löschen; Wechsel speichert automatisch
- Titel-Positionen (Gewerke) mit Zwischensummen je Abschnitt, auch im Druck
- Alternativpositionen: Checkbox „In Kalkulation einbeziehen“ je Position (nicht angehakt = gelb, nicht in Summen; im Druck als „Alt.“); Positionen und Titel per Pfeil verschiebbar
- Variantenvergleich: Kurzübersicht unter den Varianten-Tabs und Modal mit Gegenüberstellung (EK, Rabatt, VK, DB, Abschnitte, Differenz zur vorherigen Variante)
- PDF-Druck mit Rabatt und EP/GP

### 📋 Aufgabenliste (projektübergreifend)
- Gruppen: Einkauf / Allgemein / Lager
- Filter nach Zuständigkeit und Jahr

### 📦 Lagerverwaltung
- Bestand, Mindestmenge, kritische Artikel in Aufgabenliste

### 🏭 Anlagen
- Anlagenliste mit Filter nach Inbetriebnahmejahr, Neuanlage (auch aus dem Projekt heraus) und Bearbeiten, Anlagentyp mit Nummernvorschlag (DT-Projekt-Typ-lfd. Nr.), Detailansicht mit verknüpften Projekten und Prüfungen
- Anlagentypen sind in den Einstellungen pflegbar
- UVV-Prüfformular: Modulauswahl, Eingabe je Prüfpunkt nach Typ (Freitext, Auswahl, Messwert), Zwischenspeichern, Abschluss, Read-only-Ansicht; UVV-Module in den Einstellungen verwaltbar; Prüfungen löschbar, Vorjahreswerte der letzten abgeschlossenen Prüfung mit Übernahme-Funktion; Unterschriften (Auftraggeber/Prüfer) und PDF-Export (A4 hoch) abgeschlossener Prüfungen
- Stundenlohn-Nachweise je Anlage: Erfassung (Kopfdaten, Personaleinsatz, Materialverbrauch), Unterschriften auf dem Tablet, Detailansicht, Bearbeiten und PDF-Export; Listen `DodekStundennachweise` und `DodekStundenzeilen`

### 🗂️ Artikelstamm
- ID, Bezeichnung, EK/VK-Preis, Marge, Gewerk, Typ
- Stücklisten (BOM) mit Lieferant und Lieferantenartikelnummer

### 🛠️ Tools (iframe-Apps)
- **spedition.html** — Speditionsaufträge mit Adressbuch
- **bestellung.html** — Filter- und Ventilator-Bestellformulare
- Beide über postMessage-Bridge mit Hub verbunden, Daten in SharePoint

---

## Technischer Aufbau

- Single-file HTML, gehostet auf GitHub Pages
- Auth: Microsoft OAuth 2.0 (MSAL Browser 2.38.3, lokal im Repo)
- Daten: SharePoint REST API über GUID
- Kein Build-Prozess, kein Server

### SharePoint Listen

| Liste | Zweck |
|---|---|
| DodekProjekte | Projekte |
| DodekArtikel | Artikelstamm |
| DodekLager | Lagerbestand |
| DodekVorkalkulation | Kalkulationen |
| DodekEinstellungen | App-Einstellungen |
| DodekSpedition | Speditionsaufträge |
| DodekBestellungen | Filter/Ventilator-Bestellungen |
| DodekAdressen | Adressbuch |

> Listen werden über GUID angesprochen — konfiguriert in `SP_CONFIG` im App-Code.

---

## Phasenstatus

### ✅ Phase 1 + 2 — Abgeschlossen
- SharePoint Mehrbenutzer-Betrieb, Microsoft Login
- EK-Bestellstatus, Zuständigkeit, Aufgaben, Lagerverwaltung
- Kalkulation mit EP/GP, Rabatt, Kopieren, Als Artikel speichern
- Rollenbasierte Nutzerrechte (Admin / Editor / Reader)
- Tools-Tab mit Spedition und Bestellung

### ⏳ Phase 3 — Ausstehend
- Power Automate: E-Mail-Benachrichtigung bei Aufgabenzuweisung

---

## Deployment

Änderungen im `main`-Branch werden automatisch über GitHub Pages deployed.

```
main → GitHub Pages → https://franekdodek-tech.github.io/dodek-projektverwaltung/
```
