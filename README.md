# Dodek Hub

Operatives Steuerungssystem für Dodek GmbH & Co. KG — Projekte, Einkauf, Kalkulation, Aufgaben, Artikelstamm und Lager in einer Anwendung.

🌐 **App:** https://franekdodek-tech.github.io/dodek-projektverwaltung/

---

## Aktuelle Versionen

| Datei | Version |
|---|---|
| `Projektuebersicht_APP.html` | v1.6.6 |
| `spedition.html` | v0.6.0 |
| `bestellung.html` | v0.8.0 |

---

## Funktionsumfang

### 📊 Dashboard
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
- PDF-Druck mit Rabatt und EP/GP

### 📋 Aufgabenliste (projektübergreifend)
- Gruppen: Einkauf / Allgemein / Lager
- Filter nach Zuständigkeit und Jahr

### 📦 Lagerverwaltung
- Bestand, Mindestmenge, kritische Artikel in Aufgabenliste

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

### Nutzerrollen

| Nutzer | Rolle |
|---|---|
| franek.dodek@dodek.de | Admin |
| hagen.dodek@dodek.de | Editor |
| nicole.merk@dodek.de | Editor |
| harry.dodek@dodek.de | Reader |

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
