# KI-Führerschein · Projektkontext für Claude Code

> Diese Datei liegt im Projekt-Root und wird von Claude Code bei jedem Sitzungsstart
> automatisch gelesen. Sie ersetzt langes Wiedererklären. Bitte auf Deutsch antworten.

## Was das ist
Bilinguale (DE/EN), gamifizierte **Single-File-Web-App** zur KI-Kompetenz für die
Klassen **K05–K10** (German International Stream, DIA/Abitur-Pfad) an der
**German Swiss International School (GSIS), Hongkong**. Vorbild-Mechanik: basiswissen-ki.de
(Punkte, Streak, Ränge, Fehlerspeicher, PNG-Zertifikate).

## Wichtigste Datei
- `index.html` — die komplette App (HTML + CSS + JS in **einer** Datei). Heißt bewusst
  `index.html` (nicht mehr `ki-fuehrerschein.html`), damit GitHub Pages sie automatisch
  unter der Root-URL ausliefert.
- `gsis-logo.png` — muss **neben** der HTML liegen (relativ verlinkt, blendet sich sonst aus).
- `gsis-icon.png` — aus `gsis-logo.png` freigestelltes „GSIS"-Icon (ohne Schriftzug/chinesische
  Zeichen), wird im Header (Nav) verwendet, da dort nur wenig Platz ist.

## Technik
- Reines HTML/CSS/JS, **keine Build-Tools, keine Frameworks**.
- Fortschritt in `localStorage` (Key `gsis_ki_v1`), mit In-Memory-Fallback.
- **Offline-first**; Deployment als statische Seite über **GitHub Pages** (Root braucht `index.html`).
- Export/Import des Fortschritts per Base64-Code (Lehrkraft-Panel).
- PNG-Export von Zertifikat und Dashboard via `html2canvas` (CDN).
- **Hongkong-Hinweis:** Anthropic-API / Claude.ai sind in HK teils gesperrt. Die App
  selbst braucht **keine Live-KI** und läuft überall. Falls je Live-KI gewünscht:
  Google Gemini über einen Serverless-Proxy, nicht die Anthropic-API.

## Design / Marke
- **GSIS-Grün `#008445`** (Pantone 348C) ist die zentrale Markenfarbe — alle Farben
  liegen als CSS-Variablen in `:root`. Tiefes Grün: `#006B37`.
- Jahrgangs-Akzente: K05 `#1FA37A`, K06 `#2F7DC2`, K07 `#7A5CC0`, K08 `#D98324`, K09 `#C0563E`, K10 `#A67C27`.
- Schriften: Space Grotesk (Display) + Inter (Body).

## Aufbau der App
- Navigation **nach Jahrgang** (K05–K10), nicht nach Kompetenzbereich. K10 ist das
  Abschlussmodul der Mittelstufe (Level III): Auffrischung + KI & Zukunft der Arbeit,
  KI-Workflows gestalten, KI-gestützt vs. klassisch vergleichen, über KI kommunizieren.
- Freischalt-Logik: nur K05 offen; höhere Stufen öffnen sich mit dem Führerschein der
  Vorstufe. Lehrkraft-Code öffnet alles. Zusätzlich hat jeder Jahrgang einen eigenen
  Direktzugangs-Code (`GRADE_CODES`) — Klick auf eine gesperrte Jahrgangskarte öffnet
  ein Code-Modal (`openGradeCode()`), das genau diesen einen Jahrgang freischaltet,
  ohne die Kette/andere Jahrgänge zu berühren. **Kein echter Zugriffsschutz** — die
  App hat keinen Server, jeder Code steht im Klartext im Seitenquelltext.
- Aufgabentypen: `info | quiz | multi | match | sort | cloze | reflect | classify`.
  `classify` = visuelle „Ist das KI?"-Aufgabe mit eingebetteten SVG-Icons.
- Pro Jahrgang ein eigener **Abschlusstest** (`EXAM[grade]`) — bündelnde Aufgaben,
  **keine** Modul-Kopien. Bestehensschwelle `PASS = 0.7`.
- Zertifikat + Dashboard zeigen **Name und Klasse**.
- Inhalts-Backbone: KI-Kompetenzen (Verstehen/Anwenden/Reflektieren/Mitgestalten) +
  AILit-Framework (OECD/EU): Engage → Create/Manage → Shape.
- Zusatzbereich **„Oberstufe · Projekttag"** (`CONTENT.OB`): immer sichtbare Kachel,
  unabhängig von der K05–K10-Freischalt-Kette, eigener Zugangscode (`SENIOR_CODE`),
  kein Abschlusstest/Zertifikat. 5 Blöcke zu wissenschaftlichem Arbeiten, Prüfungsregeln,
  Eigenständigkeit und Quellenkritik/Plagiat für die Qualifikationsphase.
- **Impressum & Datenschutz** über Fußzeile erreichbar (`openImpressum()`/`openPrivacy()`):
  Herausgeber Sebastian Ghofranifar (Koordinator digitale Unterrichtsentwicklung, GSIS
  Hongkong, sghofranifar@gsis.edu.hk), verantwortliche Institution GSIS. Lizenz **CC BY-NC 4.0**.
  Datenschutz-Kernaussage: keine Server-Erhebung, alles nur lokal in `localStorage`;
  kurzer Hinweis dazu auch im Namens-Modal beim ersten Start.

## Konventionen (bitte einhalten)
- Alles **zweisprachig** pflegen: Textobjekte `{de:'…', en:'…'}`.
- Code-Kommentare **auf Deutsch** (ich lerne gerade programmieren — bitte erklären,
  was der Code tut, wenn du etwas änderst).
- **Keine externen Bilder** für Inhalte — stattdessen die vorhandenen Inline-SVG-Icons.
- **Keine IB-/MYP-Begriffe** in der Schüler-Ansicht (reiner deutscher Stream).
- Distraktoren = **plausible Fehlvorstellungen**, keine Witz-Antworten.

## Wichtige Konstanten (im JS, oben)
- `TEACHER_CODE = 'gsis-ki-2026'` — **noch auf echten Code ändern**.
- `SENIOR_CODE = 'gsis-oberstufe-2026'` — Zugangscode für „Oberstufe · Projekttag",
  **noch auf echten Code ändern**.
- `GRADE_CODES` — je ein Direktzugangs-Code pro Jahrgang (K05–K10), **noch auf echte
  Codes ändern**: K05 `gsis-k05-2026`, K06 `gsis-k06-2026`, K07 `gsis-k07-2026`,
  K08 `gsis-k08-2026`, K09 `gsis-k09-2026`, K10 `gsis-k10-2026`.
- `PASS = 0.7`.

## Arbeitsweise mit mir
- Erst kurz sagen, was du vorhast, dann ändern — nicht alles auf einmal.
- Nach einer Änderung: kurz zusammenfassen, was du getan hast, und wie ich es teste.
