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
| **Planjahr** | Wählbares Kalenderjahr (Default: 2026) für Soll-Ist-Berechnung |
| **Wochenkurse** | Wöchentliche Kurse mit automatischer Feiertags-/Ferien-Aussetzung |
| **Workshops** | Mehrtägige Blocktermine (von Sperrzeiten ausgenommen) |
| **Sperrzeiten** | Editierbare Ferien/Schließzeiten (Wien vorbefüllt), ein-/ausklappbar |
| **Eingetragen-Status** | Häkchen pro Kurs/Workshop zur Überführung in den Live-Betrieb |
| **Soll-Ist-Abgleich** | Jahresgenau: nur Termine/Tage im Planjahr zählen |
| **Konfliktprüfung** | Raum- und Dozenten-Doppelbelegungen auf Datumsebene |
| **Kopieren** | Kurs-/Workshop-Details als Textblock in die Zwischenablage |
| **JSON Export/Import** | Plan als Datei sichern/laden (`zf-programm-<Planjahr>.json`) |
| **Szenarien** | Pläne im Browser speichern/laden (localStorage) |

---

## Ausführen & Szenarien

Datei im Browser öffnen. Werte über Regler und Felder ändern — alles rechnet live. Über die Leiste oben lassen sich benannte **Szenarien speichern, laden und löschen** (lokal im Browser, `localStorage`). Die drei Tools nutzen getrennte Schlüssel (`zf_wien_scenarios_v1`, `zf_graz_scenarios_v1`, `zf_wien_programm_v1`), kollidieren also nicht. „Zurücksetzen" stellt die Startwerte wieder her.

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
  Ergebnis        = Σ Deckungsbeiträge − Standortleitung(Fixum + Beteiligung)
                    − Werbung/Verwaltung − Betriebskosten − Miete − …
```

**Alle Werte sind netto** (USt 20 % herausgerechnet).

---

## Code-Struktur (Single-File)

Eine Datei, drei IIFE-Blöcke (je Tool), ein gemeinsamer `<style>`-Block und eine gemeinsame Tab-Navigation.

- **Element-IDs** mit Präfixen: `wien_`, `graz_`, `programm_` — verhindert Kollisionen.
- **`PERSIST_IDS`** pro IIFE — steuert Live-Update (Event-Listener), Szenarien (speichern/laden) und Zurücksetzen. Jedes neue Eingabefeld muss dort eingetragen werden.
- **Wien:** `computeAll(overrides)` ist die einzige Rechenfunktion. `render()` zeigt sie an. Die Hebel-Analyse ruft `computeAll` mehrfach mit kleinen `overrides` auf — sauber erweiterbar um weitere Hebel.
- **Graz:** `calc()` rendert das Einzeljahr und ruft `updateProjection()` auf. Die Projektion nutzt `pCourse / pAfter / pTeam / projYear` mit je eigenem Auslastungs-/Fixum-/Miet-Override.
- **Programmplanung:** IIFE mit globalem `state`-Objekt (kurse, workshops, sperr, planjahr, sollWk, sollWs). `renderAll()` → `renderKurse()`, `renderWs()`, `renderSperr()`, `recompute()`.

## Erweitern

**Neue Schicht (z. B. weiteres Standbein):**
1. Bestehende Schicht-Card im `<body>` kopieren, eindeutige IDs mit Präfix vergeben.
2. In der Rechenfunktion (`calc()`/`computeAll()`) die Schicht ergänzen.
3. Die neuen IDs in `PERSIST_IDS` eintragen.

**Wien: Mehrjahres-Projektion** — Das Muster aus der Graz-IIFE übernehmen. `computeAll` akzeptiert bereits `overrides` — eine Jahresschleife, die je Jahr `wien_wkR/weR/feR`, `wien_fx` und `wien_miete` überschreibt, genügt.

**Standortleitung ins echte Leitungsmodell heben** (beide Rechner): Fixum erhöhen (KV-Einstufung) und Beteiligung% auf 8 setzen — am besten als zwei Szenarien gespeichert („Ist heute" / „Leitung neu").

---

## Versionshistorie

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
