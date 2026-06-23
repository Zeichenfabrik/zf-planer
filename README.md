# Zeichenfabrik – Standortrechner & Programmplanung

Zwei eigenständige Tools für die Standorte **Wien** und **Graz**:

## 1. Standortrechner

Zwei Kalkulations-Apps (Wien & Graz). Jede ist eine einzelne HTML-Datei – kein Server, keine Abhängigkeiten, kein Build.
Im Browser öffnen (Doppelklick), läuft offline.

```
zeichenfabrik-rechner/
├── Zeichenfabrik_Wien_Standortrechner.html
├── Zeichenfabrik_Graz_Standortrechner.html
└── README.md

zeichenfabrik-planung/
└── Zeichenfabrik_Wien_Programmplanung.html
```

---

## 2. Programmplanung (Wien)

Ein **Stundenplan-Entwurfstool** für den Standort Wien. Ermöglicht das Erstellen und Prüfen von Kurs- und Workshop-Plänen **vor** der CMS-Eintragung.

- **Datei:** `Zeichenfabrik_Wien_Programmplanung.html`
- **Zweck:** Stundenplan-Entwurf · Soll-Ist-Abgleich zum Plansoll · feiertags- und ferienbewusst
- **Läuft offline** (einzelne HTML-Datei, keine Abhängigkeiten)

### Funktionen

| Funktion | Beschreibung |
|----------|--------------|
| **Planjahr** | Wählbares Kalenderjahr (Default: 2026) für Soll-Ist-Berechnung |
| **Wochenkurse** | Wöchentliche Kurse mit automatischer Feiertags-/Ferien-Aussetzung (verschieben sich nach hinten) |
| **Workshops** | Mehrtägige Blocktermine (von Sperrzeiten ausgenommen) |
| **Sperrzeiten** | Editierbare Ferien/Schließzeiten (Wien vorbefüllt) |
| **Eingetragen-Status** | Häkchen pro Kurs/Workshop, um den Live-Betrieb zu tracken |
| **Soll-Ist-Abgleich** | Jahresgenaue Berechnung: Nur Termine/Tage im Planjahr zählen |
| **Konfliktprüfung** | Prüft Raum- und Dozenten-Doppelbelegungen auf Datumsebene |
| **Kopieren** | Kopiert Kurs-/Workshop-Details als Textblock in die Zwischenablage |
| **JSON Export/Import** | Sichert den Plan als Datei (z. B. `zf-programm-2026.json`) |
| **Szenarien** | Speichern/Laden von Plänen im Browser (localStorage) |

### Bedienung

1. **Plansoll** eintragen (aus dem Standortrechner)
2. **Sperrzeiten** anpassen (falls nötig)
3. **Kurse/Workshops** hinzufügen (über „+ Kurs hinzufügen“ / „+ Workshop hinzufügen“)
4. **Detailformular** nutzen, um Einträge zu bearbeiten
5. **Soll-Ist** prüfen (automatisch aktualisiert)
6. **Konflikte** auflösen (falls vorhanden)
7. **Exportieren** (JSON) für Datensicherheit

### Technische Hinweise

- **Feiertage:** Automatisch berechnet (Oster-Algorithmus für Österreich)
- **Ferien:** Voreingestellt für Wien (Schuljahre 2025/26 & 2026/27), frei editierbar
- **Workshops:** Bleiben von Sperrzeiten ausgenommen (feste Blocktermine)
- **Jahresgenaue Soll-Ist:** Nur Termine/Tage im gewählten Planjahr werden gezählt
- **Konfliktprüfung:** Prüft alle Kurs-Sitzungen und Workshop-Tage auf Überlappungen (Raum/Dozent)

### Datenfluss & Speicherung

- **localStorage:** Szenarien werden lokal im Browser gespeichert (`zf_wien_programm_v1`)
- **JSON Export:** Erstellt eine Datei (z. B. `zf-programm-2026.json`) für Datensicherheit
- **JSON Import:** Lädt eine vorher exportierte Datei und ersetzt den aktuellen Zustand

---

## Standortrechner – Grundkonvention

**Alle Werte sind netto** (USt 20 % herausgerechnet). Es gibt bewusst keine
Brutto-Eingabefelder, um Missverständnisse zu vermeiden. Kurspreis 15 €/Std/TN
netto entspricht dem gewohnten Bruttopreis – die Umrechnung passiert nicht in
der App, sondern bleibt außerhalb.

## Ausführen & Szenarien

Datei im Browser öffnen. Werte über Regler und Felder ändern – alles rechnet
live. Über die Leiste oben lassen sich benannte **Szenarien speichern, laden und
löschen** (lokal im Browser, `localStorage`). Die Apps nutzen getrennte Speicher-
schlüssel (`zf_wien_scenarios_v1`, `zf_graz_scenarios_v1`), kollidieren also nicht.
„Zurücksetzen" stellt die Startwerte wieder her.

## Das Modell (beide Apps gemeinsam)

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

## Unterschiede Wien vs. Graz

**Graz** ist ein Aufbau-Standort: enthält die Schicht *Nachmittag (Walk-in-Event-
betrieb)* und eine **Mehrjahres-Projektion (J1→J3)** mit Anlaufkurve und Miet-
staffel; die Break-even-Miete steht im Fokus (umsatzbasierte Miete als Verhand-
lungsthema).

**Wien** ist ein etablierter Standort ohne Pilotjahr. Statt der Mehrjahres-
Projektion gibt es eine **Hebel-Analyse**, die zeigt, wie stark eine einzelne
kleine Änderung (z. B. +1 Ø-TN, +1 Kurs/Abend) aufs Jahresergebnis wirkt. Die
Miete ist ein frei einstellbarer Transfer (Eigentümer = Privatperson), daher wird
das **Ergebnis vor und nach Miete** ausgewiesen, plus ein Feld für
*Unternehmerlohn/Entnahme* und eine *DB-Quoten*-Anzeige als Gesundheitskennzahl.

## Code-Struktur

Jede Datei hat drei Teile: `<style>` (Design-Tokens als CSS-Variablen, Hell/Dunkel
automatisch), `<body>` (eine Card je Schicht + Standort-/Ergebnis-Block) und ein
`<script>` (ein gekapseltes IIFE).

Zentrale Stellen:

- **`PERSIST_IDS`** – Array aller Eingabe-IDs. Steuert drei Dinge zugleich: Live-
  Update (Event-Listener), Szenarien (speichern/laden) und Zurücksetzen. Jedes neue
  Eingabefeld muss hier eingetragen werden.
- **Graz:** `calc()` rendert das Einzeljahr und ruft am Ende `updateProjection()`
  auf. Die Projektion nutzt die Helfer `pCourse / pAfter / pTeam / projYear`, die
  je Jahr eigene Auslastung, Fixum und Miete anwenden; Struktur und Preise kommen
  aus den Karten.
- **Wien:** `computeAll(overrides)` ist die einzige Rechenfunktion. `render()`
  zeigt sie an. Die Hebel-Analyse ruft `computeAll` mehrfach mit kleinen
  `overrides` auf und bildet die Differenz – sauber erweiterbar um weitere Hebel.

## Erweitern

**Neue Schicht hinzufügen** (z. B. ein weiteres Standbein):
1. Eine bestehende Schicht-Card im `<body>` kopieren, eindeutige IDs vergeben.
2. In der Rechenfunktion (`calc()`/`computeAll()`) die Schicht analog ergänzen und
   ihren Deckungsbeitrag in `sumDB` sowie ihren Umsatz in `totalNet` aufnehmen.
3. Die neuen IDs in `PERSIST_IDS` eintragen.

**Wien um eine Mehrjahres-Projektion erweitern:** Das Muster aus der Graz-Datei
übernehmen. `computeAll` akzeptiert bereits `overrides` – eine Jahresschleife, die
je Jahr `wkR/weR/feR` (Auslastung), `fx` (Fixum) und `miete` überschreibt, genügt.

**Graz: Werbung/Betriebskosten über die Jahre variabel machen** – derzeit konstant.
In `projYear` statt `num('wv')` / `num('betrieb')` je Jahr eigene Felder lesen.

**Standortleitung ins echte Leitungsmodell heben** (beide Apps): Fixum erhöhen
(KV-Einstufung) und Beteiligung% auf 8 setzen. In Wien lässt sich so in einem
Schritt vom heutigen geringfügigen Stand auf das künftige Modell umstellen –
am besten als zwei gespeicherte Szenarien („Ist heute" / „Leitung neu").

## Arbeiten mit Claude Code

Diesen Ordner in Claude Code öffnen und in natürlicher Sprache an den Dateien
arbeiten lassen („ergänze im Wien-Rechner die Teamevents-Sensitivität", „bau die
Mehrjahres-Projektion ein"). Claude Code ändert die Datei direkt, zeigt das Diff,
und du bestätigst.

Empfehlenswert: den Ordner unter Versionsverwaltung stellen, damit jeder Stand
nachvollziehbar bleibt.

```
git init
git add .
git commit -m "Standortrechner Wien & Graz, Ausgangsstand"
```

Danach genügt nach jeder Änderung ein `git add -A && git commit -m "…"`.
