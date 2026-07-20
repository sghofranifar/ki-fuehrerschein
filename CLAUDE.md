# KI-Führerschein · Projektkontext für Claude Code

> Diese Datei liegt im Projekt-Root und wird von Claude Code bei jedem Sitzungsstart
> automatisch gelesen. Sie ersetzt langes Wiedererklären. Bitte auf Deutsch antworten.

## Was das ist
Bilinguale (DE/EN), gamifizierte **Single-File-Web-App** zur KI-Kompetenz für die
Klassen **K05–K09** (German International Stream, DIA/Abitur-Pfad) an der
**German Swiss International School (GSIS), Hongkong**. Vorbild-Mechanik: basiswissen-ki.de
(Punkte, Streak, Ränge, Fehlerspeicher, PNG-Zertifikate).

## Wichtigste Datei
- `ki-fuehrerschein.html` — die komplette App (HTML + CSS + JS in **einer** Datei).
- `gsis-logo.png` — muss **neben** der HTML liegen (relativ verlinkt, blendet sich sonst aus).

## Technik
- Reines HTML/CSS/JS, **keine Build-Tools, keine Frameworks**.
- Fortschritt in `localStorage` (Key `gsis_ki_v1`), mit In-Memory-Fallback.
- **Offline-first**; Deployment als statische Seite (Netlify).
- Export/Import des Fortschritts per Base64-Code (Lehrkraft-Panel).
- PNG-Export von Zertifikat und Dashboard via `html2canvas` (CDN).
- **Hongkong-Hinweis:** Anthropic-API / Claude.ai sind in HK teils gesperrt. Die App
  selbst braucht **keine Live-KI** und läuft überall. Falls je Live-KI gewünscht:
  Google Gemini über einen Serverless-Proxy, nicht die Anthropic-API.

## Design / Marke
- **GSIS-Grün `#008445`** (Pantone 348C) ist die zentrale Markenfarbe — alle Farben
  liegen als CSS-Variablen in `:root`. Tiefes Grün: `#006B37`.
- Jahrgangs-Akzente: K05 `#1FA37A`, K06 `#2F7DC2`, K07 `#7A5CC0`, K08 `#D98324`, K09 `#C0563E`.
- Schriften: Space Grotesk (Display) + Inter (Body).

## Aufbau der App
- Navigation **nach Jahrgang** (K05–K09), nicht nach Kompetenzbereich.
- Freischalt-Logik: nur K05 offen; höhere Stufen öffnen sich mit dem Führerschein der
  Vorstufe. Lehrkraft-Code öffnet alles.
- Aufgabentypen: `info | quiz | multi | match | sort | cloze | reflect | classify`.
  `classify` = visuelle „Ist das KI?"-Aufgabe mit eingebetteten SVG-Icons.
- Pro Jahrgang ein eigener **Abschlusstest** (`EXAM[grade]`) — bündelnde Aufgaben,
  **keine** Modul-Kopien. Bestehensschwelle `PASS = 0.7`.
- Zertifikat + Dashboard zeigen **Name und Klasse**.
- Inhalts-Backbone: KI-Kompetenzen (Verstehen/Anwenden/Reflektieren/Mitgestalten) +
  AILit-Framework (OECD/EU): Engage → Create/Manage → Shape.

## Konventionen (bitte einhalten)
- Alles **zweisprachig** pflegen: Textobjekte `{de:'…', en:'…'}`.
- Code-Kommentare **auf Deutsch** (ich lerne gerade programmieren — bitte erklären,
  was der Code tut, wenn du etwas änderst).
- **Keine externen Bilder** für Inhalte — stattdessen die vorhandenen Inline-SVG-Icons.
- **Keine IB-/MYP-Begriffe** in der Schüler-Ansicht (reiner deutscher Stream).
- Distraktoren = **plausible Fehlvorstellungen**, keine Witz-Antworten.

## Wichtige Konstanten (im JS, oben)
- `TEACHER_CODE = 'gsis-ki-2026'` — **noch auf echten Code ändern**.
- `PASS = 0.7`.

## Arbeitsweise mit mir
- Erst kurz sagen, was du vorhast, dann ändern — nicht alles auf einmal.
- Nach einer Änderung: kurz zusammenfassen, was du getan hast, und wie ich es teste.
