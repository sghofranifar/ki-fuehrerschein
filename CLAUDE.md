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
- Aufgabentypen: `info | quiz | multi | match | sort | cloze | clozedrag | reflect | classify | input | promptcheck | order`.
  `classify` = visuelle „Ist das KI?"-Aufgabe mit eingebetteten SVG-Icons. `input` =
  freies Textfeld, per `task.accept`-Liste geprüft; `norm()` entfernt beim Vergleich
  alles außer Ziffern, damit unterschiedliche Schreibweisen (z. B. „17:02"/„17.02 Uhr")
  als richtig erkannt werden — wichtig, weil KI-Antworten nie exakt gleich formuliert sind.
  `promptcheck` = **lokaler, regelbasierter** CRAFT-Prompt-Check (Signalwort-Heuristik
  pro Buchstabe C/R/A/F/T, reines `RegExp.test()` im Browser) — bewusst **keine**
  externe API/kein Aufruf eines KI-Dienstes zur Bewertung, wegen Datenschutz und weil
  Anthropic/teilweise auch andere KI-Dienste aus Hongkong nicht zuverlässig erreichbar
  sind. Standardmäßig zählt er nicht zu `RUN.gradeable` (wie `info`/`reflect`) —
  reines Feedback-Tool, kein Auf-Bestehen-Gate. Mit `task.graded:true` (+ optional
  `task.minHits`, Standard 4) wird er zum Prüfungsbaustein: `scoreTask()` läuft dann
  mit `hitCount>=minHits`, genutzt in EXAM.K08/K10 als „Prompt-Apparat".
  `order` = Reihenfolge-Aufgabe: `task.items` liegt in der RICHTIGEN Reihenfolge vor,
  wird aber gemischt angezeigt; richtig, wenn Klick-Reihenfolge = Ursprungsindex.
  Für „Prozess-Aufgaben" in den Abschlusstests (z. B. Workflow-Phasen, CRAFT-Merkwort).
  `clozedrag` = Lückentext mit **klick-basierter Wortbank** statt Dropdown (bewusst
  kein echtes HTML5-Drag&Drop, da das auf iPads/Touch unzuverlässig ist): Wort in der
  Wortbank anklicken (wird aktiv), dann eine Lücke anklicken zum Platzieren; eine
  bereits gefüllte Lücke erneut anklicken gibt das Wort zurück in die Wortbank.
  `task.segments`/`task.words`/`task.answers` (Array, Index = Lücken-Nummer).
- **Zurücksetzen-Button** (`taskFoot(label,showStreak,reset=true)` + `bindReset()`):
  bei `match`/`sort`/`order`/`classify`/`clozedrag` kann man eigene Fehlplatzierungen
  vor dem Prüfen per Klick verwerfen und neu anfangen, statt das ganze Modul neu
  starten zu müssen. Technisch ruft der Button einfach `drawTask()` erneut auf (baut
  die Aufgabe mit leerem Zustand + neuer Zufalls-Reihenfolge neu auf); er ist nur vor
  dem Prüfen aktiv (`bindReset(()=>locked)` sperrt ihn danach), damit nach dem Prüfen
  bereits vergebene Punkte (`RUN.gradeable`/`RUN.score`) nicht doppelt zählen können.
- **Großer Umbau (Herbst 2026): praktische Hands-on-Aufgaben statt reiner Theorie.**
  Auslöser: Multiple-Choice-only-Module waren für 80-Minuten-Workshops zu dünn. Muster
  pro ergänzter Aufgabe: `info` mit Link zu einem externen KI-Mini-Tool (neuer Tab) +
  `reflect`/`input` zur Auswertung der eigenen Erfahrung — **nie** wörtlicher Abgleich
  von KI-Antworttext (siehe `input`-Prinzip oben). Bisher ergänzt:
  K05-M1 Quick, Draw! (quickdraw.withgoogle.com, Mustererkennung), K06-M2 Teachable
  Machine (teachablemachine.withgoogle.com, eigenes Mini-Modell trainieren), K06-M4
  Semantris (research.google.com/semantris, Wortbedeutung/Prompting), K07-M4 AutoDraw
  (autodraw.com, kreative Mensch-KI-Zusammenarbeit + Kennzeichnungsfrage), K08-M2
  Gemini-Ideen-Brainstorming mit Auswahl/Verwerfen, K08-M3 echte Feedback-Schleife mit
  Gemini (eigener Text → Gemini-Feedback zu Aufbau/Verständlichkeit → Überarbeitung),
  K08-M4 Gemini-Grenzen-Test (Buchstaben-Zählaufgabe „Verantwortungsbewusstsein" → 4×„s" —
  zeigt die Tokenisierungs-Schwäche von Sprachmodellen bei Buchstabenzählung an einem
  echten, nachvollziehbaren Beispiel), K08-M7 `promptcheck`-Aufgabe für den eigenen
  CRAFT-Prompt + Live-Vergleichstest (guter vs. schlechter Prompt) auf Gemini.
  K09-M2 Live-Modellvergleich Fast vs. Thinking/Pro an einer Fangfrage (17 Schafe,
  alle außer 9 laufen weg → richtig 9, Ablenkung durch die 17 — zeigt, dass schnelle
  Modelle oft nur das Rechenmuster statt den Satz genau lesen), K09-M3 Bias selbst
  erzeugen mit Teachable Machine (absichtlich einseitiges Training, dann Test unter
  anderen Bedingungen), K09-M4 Diskussion mit Gemini vor der eigenen ethischen
  Stellungnahme (bewusst zuerst eigene Meinung bilden, dann stärkstes Gegenargument
  einholen), K10-M3 Workflow-Ausführung: die vorhandene Planungs-Aufgabe wird um
  echte Ausführung von mind. 2 Workflow-Schritten mit Gemini + Soll/Ist-Vergleich
  ergänzt (Planen war bisher nur Theorie).
- **Vertiefungsrunde (Herbst 2026, Teil 2): noch mehr Tiefe statt mehr Wiederholung.**
  K07-M5 „KI erkennen: Google-Suche & KI-Übersicht" (neues, 5. K07-Modul): erklärt AI
  Overviews (Gemini-generiert, kein einzelner Autor) mit den echten, dokumentierten
  Fehlerfällen „Klebstoff auf Pizza" / „Steine essen" (Mai 2024), Hands-on-Suche zur
  Frage nach dem bevölkerungsreichsten Land (Indien überholte China 2023 laut UN –
  bewusst ein Beispiel, bei dem veraltete KI-Trainingsdaten noch die alte Antwort
  liefern könnten), plus Pflicht zur unabhängigen Gegenquelle. K09-M2: Schaf-Rätsel um
  eine zweite Falle erweitert (17 Schafe, alle außer 9 laufen weg, Hälfte der
  Weggelaufenen kommt zurück → richtig 13, nicht 9 oder 8 – testet zweistufiges
  Lesen statt Mustererkennung), plus „erfinde eigene Fallen"-Aufgabe (Dokumentation
  eigener Fast-vs-Thinking-Tests) und eine Halluzinations-Vertiefung mit echten,
  recherchierten Fällen (Mata v. Avianca: Anwälte reichten von ChatGPT erfundene
  Gerichtsurteile ein, 2023 sanktioniert; Google-AI-Overview-Fehlantworten 2024) plus
  eigener Versuch, bei Gemini eine Halluzination zu provozieren. K10-M6 „Deep Research
  kennenlernen" (neues, 6. K10-Modul): erklärt Geminis Deep-Research-Agent (recherchiert
  selbstständig über mehrere Minuten, liefert zitierten Bericht), Hands-on mit echter
  Unterrichtsfrage, Reflexion zu sinnvoller/unsinniger Schulnutzung. OB-7 erweitert:
  Deep Research als konkretes Agenten-Beispiel + kritische Quellenbewertung an einer
  echten Facharbeits-/Seminarkursfrage. Alle neuen Fakten via WebSearch verifiziert
  (Mata v. Avianca, Google-AI-Overview-Fehler 2024, UN-Bevölkerungsdaten 2023,
  Gemini-Deep-Research-Funktionsweise). Damit ist der große Umbau (K05–K10)
  abgeschlossen.
- K08-M7 „Prompt Engineering mit CRAFT": im Modul-Array bewusst an Position 2 (direkt
  nach der Auffrischung) eingefügt, hat aber die ID `K08-M7` behalten statt die
  bestehenden Module M2–M6 umzunummerieren — sonst hätte das bereits gespeicherten
  Fortschritt von Beta-Tester:innen unter den alten IDs zerstört (Modul-Reihenfolge in
  der Anzeige kommt aus der Array-Position, nicht aus der ID). CRAFT-Framework nach
  Vera Cubero/Joscha Falck unter **CC BY-NC-SA 4.0** — abweichend von der App-Lizenz,
  daher eigener Attributions-Hinweis im Modul selbst UND im Impressum.
- K06-M1/M2 vertieft (waren mit 2 bzw. 5 Aufgaben zu kurz für eine Workshop-Einheit):
  M1 hat jetzt einen langen `clozedrag`-Recap-Lückentext (6 Lücken + 3 Distraktor-
  Wörter) plus Anschlussfrage zu den Distraktoren. M2s Teachable-Machine-Anleitung
  war als einzelner Absatz zu knapp und ließ Schüler:innen an der echten Tool-UI
  hängen — jetzt eine nummerierte Schritt-für-Schritt-Anleitung mit den konkreten
  Button-Bezeichnungen der Seite (Get Started → Image Project → Standard image model
  → Klassen umbenennen → Webcam → Hold to Record → Train Model → Preview), plus ein
  bewusster Schritt 8 (dritten, untrainierten Gegenstand zeigen), der in der
  Reflexionsfrage aufgegriffen wird.
- K08-M5 „Gemini kennenlernen": Ab Klasse 8 dürfen Schüler:innen Gemini nutzen, daher
  eigenes Modul mit stilisiertem (nicht echtem!) UI-Diagramm — **keine Screenshots**,
  Google ändert die Oberfläche zu oft, deshalb Inline-HTML/CSS-Mockup mit nummerierten
  Erklär-Punkten. Verlinkt die schulweite „GSIS KI-Ampel" (0–4-Skala, Google-Drive-PDF)
  statt die Tabelle im Code zu duplizieren — die Schule pflegt das PDF unabhängig.
  Letzte Aufgabe: echte Mini-Challenge auf gemini.google.com/app (neuer Tab) mit
  `input`-Aufgabentyp — bewusst eine Aufgabe mit eindeutig berechenbarer Antwort
  (Zeitrechnung), nicht wörtlicher Abgleich von Geminis Antworttext, da KI-Ausgaben
  nicht deterministisch sind.
- K08-M6 „KI im Unterricht: Zulässig oder nicht?": fünf Alltagsszenarien als Quiz —
  Entscheidung + Begründung stecken direkt in den Antwortoptionen (nicht Freitext),
  damit automatisch auswertbar; zwei Szenarien sind bewusst „knifflig": (1) Gegenintuitiv
  richtig erlaubt: KI-Nutzung beim unbewerteten Üben; (2) Erlaubnis zur KI-Nutzung und
  Offenlegungspflicht sind getrennte Dinge — Verstoß trotz eigentlich erlaubter Nutzung,
  weil die Kennzeichnung fehlt (spiegelt die echten Offenlegungspflichten der GSIS-Skala).
  Abschließend eine offene Reflexion, in der Schüler:innen ihre eigene Begründung zu
  einer Situation aufschreiben.
- Pro Jahrgang ein eigener **Abschlusstest** (`EXAM[grade]`) — bündelnde Aufgaben,
  **keine** Modul-Kopien. Bestehensschwelle `PASS = 0.75`, gilt **auch für einzelne
  Module** (nicht nur den Abschlusstest): Unter 75% richtig wird das Modul nicht als
  „done" gespeichert, die Aufgaben müssen wiederholt werden (siehe `finishRun()`).
  `buildExamTasks()` mischt die Reihenfolge des Pools zufällig, respektiert dabei aber
  `task.group`: Aufgaben mit derselben group-Kennung bleiben zusammen und in ihrer
  Original-Reihenfolge (wichtig für mehrteilige Szenario-Fallstudien). Seit Herbst
  2026 sind alle sechs Abschlusstests (K05–K10) über reine Multiple-Choice hinaus um
  differenzierte Formate ergänzt: **Szenario-Fallstudien** (z. B. „Mias Foto-App",
  „Bens Google-Recherche", „Toms Facharbeit" — durchgehende Geschichte aus mehreren
  gruppierten Teilaufgaben), **Fehlersuche/Error-Analysis** (ein fehlerhaftes Vorgehen
  wird gezeigt, `multi` identifiziert die Fehler), **Prozess-Aufgaben** (`order`-Typ,
  z. B. Workflow-Phasen oder CRAFT-Reihenfolge) und ab K08 ein **Prompt-Apparat**
  (gradeter `promptcheck`). Diese Formate ergänzen die bestehenden Einzelfragen im
  selben Pool, ersetzen sie nicht.
- Zertifikat + Dashboard zeigen **Name und Klasse**.
- **Zwischenzertifikat „Gemini-Nutzung"** (`renderGeminiCert()`, Route `geminicert`):
  erscheint als zusätzliche Kachel auf der K08-Jahrgangsübersicht, sobald
  `geminiEligible()` true ist — prüft **explizit alle vier** Führerscheine
  K05–K08 (nicht nur K08!), weil die Direktzugangs-Codes K05–K07 übersprungen
  haben könnten. Muss von Schüler:innen per Browser-Druckfunktion
  (`window.print()`, `@media print`-Regel blendet Nav/Footer/Buttons aus) als
  echtes PDF gespeichert und an `GEMINI_CERT_EMAIL` gemailt werden (bewusst
  keine PDF-Bibliothek — der Zweck ist ja gerade, dass Schüler:innen den
  PDF-Export+Versand selbst können). Der Druck-Button erscheint aus Konsistenz
  auch beim normalen Zertifikat (`renderCert()`).
- Inhalts-Backbone: KI-Kompetenzen (Verstehen/Anwenden/Reflektieren/Mitgestalten) +
  AILit-Framework (OECD/EU): Engage → Create/Manage → Shape.
- Zusatzbereich **„Oberstufe · Projekttag"** (`CONTENT.OB`): immer sichtbare Kachel,
  unabhängig von der K05–K10-Freischalt-Kette, eigener Zugangscode (`SENIOR_CODE`),
  kein Abschlusstest/Zertifikat. 7 Blöcke zu wissenschaftlichem Arbeiten, Prüfungsregeln,
  Eigenständigkeit und Quellenkritik/Plagiat für die Qualifikationsphase, plus OB-6
  „KI und die Arbeitswelt von morgen" (echte PwC-2026-Zahlen zum Jobmarkt, kritische
  Reflexion zu Dario Amodeis KI-Tempo-Warnung von Sept. 2026 + Recherche einer
  Gegenposition, Uni-Workflow-Übung) und OB-7 „AI Agents – Nutzen und Grenzen"
  (Mehrschritt-Fehlerfortpflanzung, wofür Agenten heute schon taugen vs. riskant sind).
  Alle Fakten/Zitate recherchiert (WebSearch), nicht erfunden — externe Original-Tabellen
  (PwC, Elements of AI) werden verlinkt statt kopiert.
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
- `PASS = 0.75` — gilt einheitlich für Module und Abschlusstests.

## Arbeitsweise mit mir
- Erst kurz sagen, was du vorhast, dann ändern — nicht alles auf einmal.
- Nach einer Änderung: kurz zusammenfassen, was du getan hast, und wie ich es teste.
