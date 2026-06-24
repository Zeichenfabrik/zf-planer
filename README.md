# Zeichenfabrik – Standortrechner & Programmplanung

Alle drei Tools in einer einzigen, offline-fähigen HTML-Datei:

**`Zeichenfabrik-All-in-One-Single.html`** — im Browser öffnen (Doppelklick), kein Server, keine Abhängigkeiten.

---

## Enthaltene Tools

### Wien Standortrechner
Jahresplanung für den etablierten Standort Wien. Schichten: Wochenkurse, Wochenend-Workshops, Ferienworkshops, Teamevents (B2B), sonstige Umsätze. Ergebnis vor und nach Miete; Hebel-Analyse zeigt, wie stark eine kleine Änderung (+1 Ø-TN, +5 % Realisierung …) aufs Jahresergebnis wirkt.

### Graz Standortrechner
Aufbau-Standort mit zusätzlicher Schicht *Nachmittag (Walk-in-Eventbetrieb)* und einer **Mehrjahres-Projektion (J1 → J3)** mit Anlaufkurve und Mietstaffel. Break-even-Miete als zentrales Verhandlungsthema.

### Wien Programmplanung
Stundenplan-Entwurfstool vor der CMS-Eintragung.

| Funktion | Beschreibung |
|----------|--------------|
| **Planjahr (pro Jahr)** | Wählbares Kalenderjahr (Default: 2027). Kurse, Workshops und Soll werden **je Jahr getrennt** gehalten; beim Wechsel auf ein leeres Jahr lässt sich das jüngste Vorjahr als Vorlage kopieren |
| **Auto-Speichern** | Jede Änderung wird sofort lokal gesichert (`localStorage`-Schlüssel `zf_programm_v2`); Anzeige „zuletzt gesichert HH:MM" |
| **Wochenkurse** | Wöchentliche Kurse mit automatischer Feiertags-/Ferien-Aussetzung |
| **Unterrichtsfreie Termine** | Übersprungene Wochen (Feiertag/Sperrzeit) werden mit Grund angezeigt — in der Liste als Hinweis und im Kopier-Text als eigene Zeile |
| **Workshops** | Mehrtägige Blocktermine (von Sperrzeiten ausgenommen) |
| **Autovervollständigung** | Bereits verwendete Dozent:innen und Titel werden über alle Jahre als Vorschläge angeboten (`<datalist>`) |
| **Sperrzeiten** | Editierbare Ferien/Schließzeiten (Wien vorbefüllt, global über alle Jahre), ein-/ausklappbar |
| **Status-Spalte** | Eingetragen-Status als gut sichtbarer Punkt direkt nach dem Titel (grün = eingetragen, grau = offen) |
| **Soll-Ist-Abgleich** | Jahresgenau: nur Termine/Tage im Planjahr zählen |
| **Soll aus Wien-Rechner** | Button überträgt die Planstunden des Wien-Standortrechners (Wochenkurse → Soll Kurse, Wochenend- + Ferien-Workshops → Soll Workshops) |
| **Konfliktprüfung** | Raum- und Dozenten-Doppelbelegungen auf Datumsebene |
| **Kopieren** | Kurs-/Workshop-Details als Textblock in die Zwischenablage |
| **JSON Export/Import** | Gesamten Plan (alle Jahre) als Datei sichern/laden |
| **Szenarien** | Benannte Pläne im Browser speichern/laden (localStorage) |

---

## Ausführen & Szenarien

Datei im Browser öffnen. Werte über Regler und Felder ändern — alles rechnet live. Über die Leiste oben lassen sich benannte **Szenarien speichern, laden und löschen** (lokal im Browser, `localStorage`). Die Tools nutzen getrennte Schlüssel und kollidieren nicht: die Standortrechner speichern Szenarien unter `zf_wien_scenarios_v1` / `zf_graz_scenarios_v1`; die Programmplanung sichert ihren Arbeitsstand **automatisch** unter `zf_programm_v2` und benannte Szenarien unter `zf_wien_programm_v1`. „Zurücksetzen" stellt die Startwerte wieder her.

---

## Das Modell (Standortrechner)

Nettobeträge, Jahreswerte. Pro Schicht:

```
Kurse/Workshops:
  realisierte Std = Planstunden × Realisierung
  Umsatz netto    = realisierte Std × Ø-TN × Kurspreis
  Fremdleistung   = realisierte Std × (Honorar + Material/Modell €/Std)
  Deckungsbeitrag = Umsatz − Fremdleistung

Teamevents (B2B):
  Umsatz netto    = Events × Ø-TN × Preis/Kopf(netto)
  Fremdleistung   = Events × (Honorar/Event + Ø-TN × Material/Kopf)

Standortweit:
  Beteiligung     = Beteiligung% × Gesamt-Nettoumsatz
  Verwaltung      = Σ (Steuer, Versicherung, Software, Telekom, Bank,
                       Kammer, Büro, Betriebskosten, AfA)
  Ergebnis        = Σ Deckungsbeiträge − Standortleitung(Fixum + Beteiligung)
                    − Werbung/Marketing − Verwaltung & Betrieb − Miete − …
```

Werbung und die übrigen Verwaltungs-/Betriebsposten sind **einzeln editierbar** (kompakte Number-Inputs statt eines Sammel-Reglers); die AfA ist nicht zahlungswirksam und standardmäßig 0. In Graz fließt dieselbe Verwaltungs-Summe als Konstante in die Mehrjahres-Projektion (J1 → J3).

**Alle Werte sind netto** (USt 20 % herausgerechnet).

---

## Code-Struktur (Single-File)

Eine Datei, drei IIFE-Blöcke (je Tool), ein gemeinsamer `<style>`-Block und eine gemeinsame Tab-Navigation.

- **Element-IDs** mit Präfixen: `wien_`, `graz_`, `programm_` — verhindert Kollisionen.
- **`PERSIST_IDS`** pro IIFE — steuert Live-Update (Event-Listener), Szenarien (speichern/laden) und Zurücksetzen. Jedes neue Eingabefeld muss dort eingetragen werden.
- **Wien:** `computeAll(overrides)` ist die einzige Rechenfunktion. `render()` zeigt sie an. Die Hebel-Analyse ruft `computeAll` mehrfach mit kleinen `overrides` auf — sauber erweiterbar um weitere Hebel.
- **Graz:** `calc()` rendert das Einzeljahr und ruft `updateProjection()` auf. Die Projektion nutzt `pCourse / pAfter / pTeam / projYear` mit je eigenem Auslastungs-/Fixum-/Miet-Override.
- **Programmplanung:** IIFE mit `state`-Objekt in **Jahres-Struktur**: `{ planjahr, sperr:[…], jahre:{ "2027":{ sollWk, sollWs, kurse:[…], workshops:[…] } } }`. `sperr` und `planjahr` sind global, alles andere liegt pro Jahr. Helfer `cur()` liefert das aktuelle Jahr, `ensureYear(y)` legt fehlende Jahre an, `normalize(s)` hebt altes flaches Format an. `persist()` sichert den ganzen `state` nach jeder Mutation (`zf_programm_v2`). `renderAll()` → `renderKurse()`, `renderWs()`, `renderSperr()`, `updateDataLists()`, `recompute()`.

## Erweitern

**Neue Schicht (z. B. weiteres Standbein):**
1. Bestehende Schicht-Card im `<body>` kopieren, eindeutige IDs mit Präfix vergeben.
2. In der Rechenfunktion (`calc()`/`computeAll()`) die Schicht ergänzen.
3. Die neuen IDs in `PERSIST_IDS` eintragen.

**Wien: Mehrjahres-Projektion** — Das Muster aus der Graz-IIFE übernehmen. `computeAll` akzeptiert bereits `overrides` — eine Jahresschleife, die je Jahr `wien_wkR/weR/feR`, `wien_fx` und `wien_miete` überschreibt, genügt.

**Standortleitung ins echte Leitungsmodell heben** (beide Rechner): Fixum erhöhen (KV-Einstufung) und Beteiligung% auf 8 setzen — am besten als zwei Szenarien gespeichert („Ist heute" / „Leitung neu").

---

## Versionshistorie

### Single-File-App (v1.2) – 2026-06-24
- Programmplanung: **Pro-Planjahr-Datenmodell** (`state.jahre`) — Kurse, Workshops und Soll je Jahr getrennt; Vorjahr beim Wechsel optional als Vorlage kopierbar
- Programmplanung: **Auto-Speichern** nach jeder Änderung (`zf_programm_v2`) mit „zuletzt gesichert"-Anzeige; Migration des alten flachen Formats
- Standortkosten (Wien + Graz): Sammel-Regler „Werbung/Verwaltung" in **einzeln editierbare Posten** aufgeteilt (Werbung getrennt ausgewiesen, inkl. AfA); Graz-Projektion nutzt die neue Verwaltungs-Summe
- Programmplanung: **unterrichtsfreie Termine** (Feiertag/Sperrzeit) in Liste und Kopier-Text sichtbar
- Programmplanung: **Autovervollständigung** für Dozent:innen und Titel über alle Jahre (`<datalist>`)
- Programmplanung: **„Soll aus Wien-Rechner übernehmen"** überträgt die Planstunden ins Jahres-Soll
- Programmplanung: breitere Listen, **Status als führende Spalte** (Symbol mit Tooltip)
- Planjahr-Default auf 2027
- Bugfix: Sperrzeiten-Kollaps-Pfeil bleibt im Header (Flexbox statt absoluter Positionierung)

### Single-File-App (v1.1) – 2026-06-24
- Bugfix: Python f-string-Artefakte (`{{`/`}}`) aus Programmplanung-IIFE entfernt (Syntax-Fehler)
- Bugfix: alle `PERSIST_IDS` und `course()`/`courseLayer()`-Argumente auf `wien_`/`graz_`-Präfixe umgestellt
- Bugfix: `updateProjection()` auf `graz_`-präfixierte Element-IDs umgestellt
- Bugfix: Sortier-Selektoren auf `#programm_kursTable`/`#programm_wsTable` korrigiert
- Bugfix: `programm_toggleCard` und `initCollapsibleCards` auf `programm_sperrCard` korrigiert
- Bugfix: `copyToClipboard` Race-Condition (doppelter Fallback) behoben
- Bugfix: dreifach falsche localStorage-Schlüssel korrigiert (`zf_wien_scenarios_v1` etc.)

### Single-File-App (v1.0) – 2026-06-24
- Alle drei Tools nativ in einer Datei (ohne IFrames)
- Tab-Navigation, sticky Fußzeilen, JSON Export/Import, Szenarien

### Programmplanung Wien (v1.0) – 2026-06-23
- Mehrtägige Workshops, Eingetragen-Status, sortierbare Listen, Sperrzeiten ein-/ausklappbar
- Jahresgenaue Soll-Ist-Berechnung, Konfliktprüfung, Kopieren-Button, JSON Export/Import

### Standortrechner Wien & Graz (v1.0) – 2026-06-23
- Wien: Hebel-Analyse, Ergebnis vor/nach Miete, DB-Quoten
- Graz: Mehrjahres-Projektion (J1→J3), Break-even-Miete
- Szenarien-Verwaltung (localStorage), JSON Export/Import

---

## Arbeiten mit Claude Code

Diesen Ordner in Claude Code öffnen und in natürlicher Sprache Änderungen beschreiben — Claude Code ändert die Datei direkt. Danach committen:

```
git add Zeichenfabrik-All-in-One-Single.html
git commit -m "…"
git push
```
