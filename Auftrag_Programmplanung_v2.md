# Umsetzungs-Auftrag — Zeichenfabrik Wien · Programmplanung (v2)

**Zieldatei:** `Zeichenfabrik_Wien_Programmplanung.html`

Erweitere das bestehende Planungstool. Es bleibt eine **einzelne, eigenständige
HTML-Datei** (offline, keine Abhängigkeiten, kein Build). Behalte die bestehende
Logik bei: den Oster-/Feiertags-Algorithmus, die feiertags- und ferienbewusste
Terminberechnung (ausgesetzte Wochen verschieben den Kurs nach hinten), und
**Workshops bleiben von Sperrzeiten ausgenommen**. Design-Sprache (CSS-Variablen,
Georgia-Überschriften, ruhige Optik) beibehalten.

Bitte Punkt für Punkt umsetzen und **nach jedem Block committen** und im Browser
testen. Reihenfolge wie unten.

---

## 1 · Datenmodell vorbereiten

- Ergänze bei **Kursen** und **Workshops** je ein Feld `eingetragen` (Boolean,
  Default `false`) — Status „schon im CMS/online".
- **Workshops** werden mehrtägig: Modell je Workshop = `titel`, `dozent`,
  `start` (Datum), `tage` (Anzahl, ≥1), `zeiten` (Array je Tag: `{von, bis}`),
  `raum`, `eingetragen`. Die Tagesdaten ergeben sich aus `start` + fortlaufende
  Tage (aufeinanderfolgend). Gesamtstunden = Summe der Tages­dauern (bis − von).
- Ergänze ein globales **Planjahr** (z. B. Auswahl/__Number-Feld, Default 2026).

---

## 2 · Workshops mehrtägig (Eingabeformular)

- Im Workshop-Formular: `Startdatum`, `Dauer (Tage)`, und daraus generiert pro Tag
  eine Zeile mit `von`/`bis` (Tag 1, Tag 2 …). Datum je Tag = Start + (n−1).
- Gesamtstunden live aus den Tageszeiten berechnen und anzeigen.
- Workshops weiterhin **nicht** an Feiertagen/Ferien aussetzen (feste Blocktermine).

---

## 3 · „Eingetragen"-Status

- Pro Kurs/Workshop ein Häkchen „Eingetragen" (online auf der Website).
- In der schlanken Liste (Punkt 4) als gut sichtbarer Status anzeigen
  (z. B. ✓-Spalte oder farbiger Punkt), damit man die Überführung in den
  Live-Betrieb nachverfolgen kann.

---

## 4 · Schlanke, sortierbare Liste + Detail-Edit (Kurse UND Workshops)

Das ausführliche Eingabeformular soll **nicht mehr dauerhaft** sichtbar sein,
sondern nur beim Hinzufügen oder Editieren erscheinen.

- Button **„+ Kurs hinzufügen"** / **„+ Workshop hinzufügen"** öffnet ein leeres
  Detailformular (das bisherige ausführliche Formular, als eine Einheit). Mit
  „Übernehmen" wandert der Eintrag in die Liste und das Formular schließt sich;
  „Abbrechen" verwirft.
- **Schlanke Liste** je Eintrag mit den Spalten:
  Kurse: Titel · Dozent · Tag · Uhrzeit · Start · Termine · Std/Termin · Raum ·
  Stunden ges. · Ende · Status · Aktionen.
  Workshops: Titel · Dozent · Start · Tage · Uhrzeiten · Raum · Stunden ges. ·
  Status · Aktionen.
- **Sortierbar** durch Klick auf die Spaltenüberschriften (auf-/absteigend
  umschaltbar).
- **Klick auf eine Zeile** (oder einen „Bearbeiten"-Button) öffnet das
  Detailformular dieses Eintrags zum Editieren (Titel ändern, Start verlegen
  usw.). Umsetzung als aufklappbares Formular unter der Zeile **oder** als
  Overlay — wähle die schlankere Variante.
- Aktionen je Zeile: Bearbeiten, **Kopieren** (Punkt 7), Entfernen.

---

## 5 · Sperrzeiten ein-/ausklappbar

- Die Sperrzeiten-Karte standardmäßig **eingeklappt** (sie nimmt sonst viel Platz).
  Aufklappen per Klick auf die Überschrift; Zustand merken (localStorage genügt).
- Inhalt und Editierbarkeit der Ferien/Schließzeiten bleiben unverändert.

---

## 6 · Plansoll mit Planjahr + jahresgenaue Soll-Ist

- Das Plansoll bezieht sich auf **ein Kalenderjahr** (das gewählte Planjahr).
- Soll-Ist **jahresgenau** rechnen: Für jeden Kurs zählen nur die Sitzungen, deren
  konkretes Datum **im Planjahr** liegt (die Sitzungsdaten werden ohnehin schon
  berechnet). Für Workshops zählen nur Tage im Planjahr. Ein Kurs, der über den
  Jahreswechsel läuft, steuert also nur seinen Im-Jahr-Anteil bei.
- In der schlanken Liste „Stunden ges." = alle Termine des Kurses; im
  **Soll-Ist-Panel** dagegen der Im-Planjahr-Anteil. (Beides ist gewollt — bitte
  nicht verwechseln.)
- Soll-Ist-Anzeige und die fixe Ergebnisleiste auf das Planjahr beziehen.

---

## 7 · „Kopieren"-Button (für Mail an die Kursleitung)

- Pro Eintrag ein Button, der die relevanten Details als **sauberen Textblock** in
  die Zwischenablage legt. Format z. B.:

  ```
  Aktzeichnen Grundkurs — Dozent: [Name]
  Mittwochs 18:00–21:00 · Raum 1
  7 Termine: 04.03., 11.03., 18.03., 25.03., 08.04., 15.04., 22.04.2026
  21 Stunden gesamt
  ```

  Für Workshops analog mit den Tagesdaten und Tageszeiten.
- **Technischer Haken:** `navigator.clipboard.writeText` funktioniert auf `file://`
  nicht in jedem Browser. Mit `try/catch` absichern und einen **Fallback** bauen
  (temporäres `<textarea>`, selektieren, `document.execCommand('copy')`).
  Kurze Rückmeldung „kopiert ✓".

---

## 8 · Konfliktprüfung auf Datumsebene

- Erweitere die Prüfung von „gleicher Wochentag" auf **konkrete Termine**: Bilde
  aus allen Kurs-Sitzungen **und** Workshop-Tagen eine Liste belegter Slots
  (`{datum, von, bis, raum, dozent, titel}`).
- Konflikt, wenn am **selben Datum** die Zeiten überlappen **und** derselbe Raum
  **oder** dieselbe Dozent:in doppelt belegt ist. Mit Datum auflisten.
- Das dient dem Ziel: konfliktfreie, überschneidungsfreie Planung als Vorlage für
  die CMS-Eintragung.

---

## 9 · JSON-Export / -Import (Datensicherheit)

- **Export**-Button: aktuellen Stand als JSON-Datei herunterladen (Blob +
  temporärer Download-Link, Dateiname z. B. `zf-programm-<Planjahr>.json`).
- **Import**-Button: JSON-Datei einlesen, Zustand ersetzen, neu rendern.
- Die bestehenden localStorage-Szenarien bleiben zusätzlich erhalten.
- Grund: localStorage kann beim Cache-Löschen verloren gehen und ist auf `file://`
  unzuverlässig — der JSON-Export macht Pläne zu echten, sicherbaren Dateien.

---

## 10 · Dokumentation

- `README.md` um diese App und ihre Funktionen ergänzen (bzw. aktualisieren).
- Falls eine `CLAUDE.md` existiert: dort die stehende Regel „README aktuell halten"
  belassen/ergänzen.

---

### Hinweise zum Vorgehen
- Reihenfolge einhalten, nach jedem Block committen, im Browser testen
  (besonders die Terminberechnung und die jahresgenaue Soll-Ist-Zählung).
- Nichts an der bestehenden Feiertags-/Ferienlogik kaputt machen.
- Datei muss eigenständig und offline lauffähig bleiben.
