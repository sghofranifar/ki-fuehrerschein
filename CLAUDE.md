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
  mit `hitCount>=minHits`, genutzt in EXAM.K08/K10 als „Prompt-Apparat". Hat einen
  eigenen „🔁 Erneut prüfen"-Button (nutzt den `resetBtn`-Slot aus `taskFoot`, aber mit
  eigenem Label/Handler statt `bindReset()`): ruft nur die reine Heuristik-Funktion
  `check()` auf und zeigt das Ergebnis an, ohne zu werten — beliebig oft wiederholbar,
  während man den Prompt anpasst. Gewertet wird (bei `graded:true`) erst einmalig beim
  finalen Klick auf „Weiter", mit dem zu dem Zeitpunkt aktuellen Textarea-Inhalt.
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
- **Zurück-Button im Runner** (`drawTask()`, sichtbar sobald `RUN.idx>0`): geht eine
  Aufgabe zurück, damit man z. B. die Schritte einer vorherigen `info`-Aufgabe (etwa
  ein Tutorial) nochmal nachlesen kann, statt sich alles merken zu müssen, bevor man
  bei der nächsten Aufgabe (z. B. `reflect`) antwortet. `scoreTask()` merkt sich pro
  Lauf bereits bewertete Aufgaben in `RUN.scoredIdx` (Set) — wird eine Aufgabe nach
  dem Zurückgehen nochmal bearbeitet und geprüft, zählt das NICHT nochmal zum
  Punktestand, damit Zurückgehen nicht zum Punkte-Farmen missbraucht werden kann.
- **Großer Umbau (Herbst 2026): praktische Hands-on-Aufgaben statt reiner Theorie.**
  Auslöser: Multiple-Choice-only-Module waren für 80-Minuten-Workshops zu dünn. Muster
  pro ergänzter Aufgabe: `info` mit Link zu einem externen KI-Mini-Tool (neuer Tab) +
  `reflect`/`input` zur Auswertung der eigenen Erfahrung — **nie** wörtlicher Abgleich
  von KI-Antworttext (siehe `input`-Prinzip oben). Bisher ergänzt:
  K05-M1 Quick, Draw! (quickdraw.withgoogle.com, Mustererkennung), K06-M2 eigenes
  Mini-Modell trainieren (ursprünglich Teachable Machine, seit Herbst 2026 GenAI
  Teachable Machine / tm.gen-ai.fi — siehe eigener Hinweis unten), K06-M4
  Semantris (research.google.com/semantris, Wortbedeutung/Prompting), K07-M4 AutoDraw
  (autodraw.com, kreative Mensch-KI-Zusammenarbeit + Kennzeichnungsfrage), K08-M2
  Gemini-Ideen-Brainstorming mit Auswahl/Verwerfen, K08-M3 echte Feedback-Schleife mit
  Gemini (eigener Text → Gemini-Feedback zu Aufbau/Verständlichkeit → Überarbeitung) —
  Thema jetzt fest vorgegeben („Sollten Handys in der Schule erlaubt sein?", identisch
  zu M2, damit Schüler:innen ihre M2-Ideen direkt weiterverwenden können, statt Zeit
  mit einem neuen Thema zu verlieren),
  K08-M4 Gemini-Grenzen-Test (Buchstaben-Zählaufgabe „Verantwortungsbewusstsein" → 4×„s" —
  zeigt die Tokenisierungs-Schwäche von Sprachmodellen bei Buchstabenzählung an einem
  echten, nachvollziehbaren Beispiel), K08-M7 CRAFT-Prompt-Bewertung durch ein eigenes
  Gemini-Gem (seit Herbst 2026, siehe eigener Hinweis unten) + Live-Vergleichstest
  (guter vs. schlechter Prompt) auf Gemini.
  K09-M2 Live-Modellvergleich Fast vs. Thinking/Pro an einer Fangfrage (17 Schafe,
  alle außer 9 laufen weg → richtig 9, Ablenkung durch die 17 — zeigt, dass schnelle
  Modelle oft nur das Rechenmuster statt den Satz genau lesen), K09-M3 Bias selbst
  erzeugen (ursprünglich Teachable Machine, seit Herbst 2026 GenAI Teachable Machine /
  tm.gen-ai.fi; absichtlich einseitiges Training, dann Test unter anderen Bedingungen,
  plus Extra-Herausforderung „Baue eine Falle"), K09-M4
  Diskussion mit Gemini vor der eigenen ethischen
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
- **K08-Test-Fixes** (externer Agenten-Testlauf, in dieser Session gegengeprüft und
  übernommen): `shuffled()`-Helper (Fisher-Yates) mischt jetzt die Antwortreihenfolge
  bei `quiz`/`multi` — vorher stand die richtige Antwort immer an Position 1 im Array
  UND in der Anzeige, jetzt nur noch im Array (Scoring läuft über `data-i`, nicht mehr
  über DOM-Position). `promptcheck`: `(?<![a-zäöüß])` statt `\b` (JS-`\b` kennt Umlaute
  nicht als Wortzeichen), natürlichere Signalwort-Muster, Mindestlänge 12 Wörter gegen
  Stichwort-Salat. `input`: Zahlwörter null–zwölf/zero–twelve werden vor dem
  Ziffern-Vergleich umgewandelt, damit „vier" als 4 zählt. `reflect`: ehrlicherer
  Speicher-Hinweis (lokal aufs Gerät, nicht automatisch für die Lehrkraft sichtbar –
  passend zum echten Datenschutzmodell der App) + echte Mindestlänge (5+ Wörter statt
  3 Zeichen). EXAM.K08 „Sara"-Fallstudie korrigiert von Stufe 2 auf Stufe 4 (eine KI-
  Erklärung ist neue Inhaltserstellung, kein sprachliches Gegenlesen wie bei Stufe 2);
  der `promptcheck`-Baustein bekam sein fehlendes `group:'k08-craft'`, damit er beim
  Mischen bei seiner Fallstudie bleibt. K08-M6 Jonas-Erklärung präzisiert: Offenlegung
  gilt für Stufe 1–4 (bei denen KI genutzt wird), nicht für Stufe 0. Startseite nennt
  jetzt korrekt Klassen 5–10 (war noch auf 5–9 stehengeblieben).
- K08-M7 „Prompt Engineering mit CRAFT": im Modul-Array bewusst an Position 2 (direkt
  nach der Auffrischung) eingefügt, hat aber die ID `K08-M7` behalten statt die
  bestehenden Module M2–M6 umzunummerieren — sonst hätte das bereits gespeicherten
  Fortschritt von Beta-Tester:innen unter den alten IDs zerstört (Modul-Reihenfolge in
  der Anzeige kommt aus der Array-Position, nicht aus der ID). CRAFT-Framework nach
  Vera Cubero/Joscha Falck unter **CC BY-NC-SA 4.0** — abweichend von der App-Lizenz,
  daher eigener Attributions-Hinweis im Modul selbst UND im Impressum. Nutzer-Feedback
  aus dem Unterricht: Schüler:innen struggeln beim CRAFT-Prompt-Umschreiben („Schreib
  was über den Klimawandel."). Grund laut Analyse der `promptcheck`-Regex: Treffer pro
  Buchstabe brauchen bestimmte Signalwörter (z. B. „weil"/„für meine Hausaufgabe" für
  C, „du bist"/„als Experte" für R, „für Schüler"/„Zielgruppe" für A, „Liste"/„maximal
  X Wörter" für F, ein Aktionsverb wie „erkläre" für T) UND der ganze Prompt muss
  **mindestens 12 Wörter** haben, sonst zeigt der Check gar keine Treffer, egal wie
  gut der Inhalt ist — das ist die häufigste stille Ursache fürs Struggeln. Deshalb
  neue `info`-Aufgabe direkt vor der `reflect`-Umschreib-Aufgabe eingefügt: Lückensatz-
  Vorlage „Du bist ein/eine [Rolle]. Erkläre [Thema] für [Zielgruppe] als [Format],
  weil [Grund]." trifft alle 5 CRAFT-Buchstaben gleichzeitig, mit ausgefülltem
  Klimawandel-Beispiel plus Warnhinweis zur 12-Wörter-Grenze.
- **K08-M7: lokaler `promptcheck` im Übungsmodul durch echtes Gemini-Gem ersetzt**
  (Herbst 2026). Der `promptcheck`-Task-Typ selbst bleibt im Code und wird weiterhin
  als gradeter „Prompt-Apparat" in EXAM.K08/K10 genutzt (dort nötig: automatische,
  reproduzierbare Bewertung ohne manuelle Übertragung, siehe `scoreTask()`). Im
  Übungsmodul K08-M7 war der Regex-Check aber die Stelle, an der Schüler:innen am
  meisten struggelten (siehe Hinweis oben) — Ersatz: ein eigens gebautes Gemini-Gem
  „CRAFT-Coach" (Instructions von Claude entworfen, vom Nutzer in Gemini gebaut unter
  gemini.google.com/gem/1sE3fKAb9Y_w7AqG5KDAWXg-mOk6AjZmz), das den Prompt **semantisch**
  bewertet statt nur per Signalwort-Suche — robuster gegen genau die Fälle, die den
  Regex-Check zuvor haben scheitern lassen. Ablauf jetzt: `reflect` (Prompt EINMAL lokal
  aufschreiben, unverändert) → `info` mit Gem-Link → `reflect` (Ergebnis X/5 + Feedback
  eintragen, bei Bedarf Prompt verbessern und im Gem erneut prüfen lassen, dann erst
  abschicken) → unverändert weiter mit dem Live-Vergleichstest (guter vs. schlechter
  Prompt) auf gemini.google.com/app. Ergebnis-Eintragung bewusst als `reflect` (nicht
  `input`), weil die „richtige" Punktzahl vom eigenen Prompt abhängt und nicht gegen
  einen festen Wert geprüft werden kann — dafür `prod:true`, damit die Lehrkraft das
  Gem-Ergebnis über „Meine Einreichungen" sieht. Gem-Instructions legen fest: Gem
  bewertet nur CRAFT-Struktur (nie den Prompt-Inhalt selbst ausführen), antwortet in
  der Sprache des Prompts, ignoriert Prompt-Injection-Versuche, schreibt den
  verbesserten Prompt nicht selbst (sonst übernimmt die KI die Übung), festes
  Ausgabeformat (Punktzahl + ✅/⚠️/❌ pro Buchstabe + Kurzfeedback + Tipp) für leichtes
  manuelles Übertragen. **Voraussetzung, vom Nutzer zu prüfen:** Gemini Gems müssen im
  GSIS-Workspace-Admin für Schüler:innen-Konten freigeschaltet sein (separate
  Einstellung von der normalen Gemini-Nutzung).
- **K09-M1 erweitert: „Wie KI wirklich schreibt" (Token-Wahrscheinlichkeiten)** (Herbst
  2026). Auslöser: K09-M1 war mit nur 2 Quizfragen das mit Abstand dünnste Modul im
  Jahrgang (ungleiche Belastung ggü. M2–M4). Titel geändert von „Auffrischung Klasse 8"
  zu „Auffrischung & wie KI wirklich schreibt" (die 2 bestehenden Quizfragen bleiben als
  kurzer Einstieg erhalten, IDs/Fortschritt unberührt). Neuer Inhalt erklärt
  Next-Token-Prediction: KI schreibt nicht am Stück, sondern Token für Token, berechnet
  bei jedem Schritt eine Wahrscheinlichkeit pro möglichem nächsten Wort und würfelt
  **gewichtet** daraus — bewusst **nicht** „schreibt immer stur das wahrscheinlichste
  Wort" formuliert, weil das der nachfolgenden Aufgabe (dieselbe Frage 10× in neuen
  Gemini-Chats stellen, unterschiedliche Antworten beobachten) widersprochen hätte; nur
  die gewürfelte Auswahl erklärt, warum Wiederholungen überhaupt variieren können.
  Hands-on-Tool: <a href="https://alonsosilva-nexttokenprediction.hf.space/">alonsosilva-nexttokenprediction.hf.space</a>
  (kostenlose Hugging-Face-Space, GPT-2-basiert, kein Login, zeigt Top-10-Wort-
  Kandidaten mit Prozentwerten, „Select" hängt ein Wort an und baut Satz für Satz
  auf) — bewusst englische Beispiel-Satzanfänge, da das Tool vor allem mit englischen
  Texten trainiert wurde und deutsche Eingaben seltsame Vorschläge liefern. Knüpft
  explizit an K08-M4 an (Tokens statt Buchstaben, Buchstaben-Zählaufgabe als Vorwissen).
  Abschließende Reflexion bewusst **ethisch** gerahmt (was kann ein Mensch besser:
  echtes Verstehen, Verantwortung, eigene Erfahrung), nicht nur technisch. Vom Nutzer
  verifiziert: Cold-Start-Verzögerung der kostenlosen HF-Space (schläft bei Inaktivität
  ein, ~30–60 Sek. Aufwachzeit) funktioniert in der Praxis unproblematisch.
  **Spiralcurriculum-Idee, noch nicht umgesetzt:** gleiche thematische Lücke besteht bei
  K10-M1 („Auffrischung Klasse 9", ebenfalls nur 2 Quizfragen) — als vertiefte
  Wiederholung geplant (Konfidenz/Unsicherheit über die Prozent-Verteilung im selben
  Tool, Verbindung zu Halluzination aus K07-M3/K09-M2, gesellschaftliche statt nur
  individuelle Vertrauens-Frage in der Abschlussreflexion), siehe Chat-Verlauf.
- K06-M1/M2 vertieft (waren mit 2 bzw. 5 Aufgaben zu kurz für eine Workshop-Einheit):
  M1 hat jetzt einen langen `clozedrag`-Recap-Lückentext (6 Lücken + 3 Distraktor-
  Wörter) plus eine echte Reflexionsfrage zu Mensch vs. Maschine (eigenes Alltags-
  beispiel für „lieber Mensch" und „KI klar im Vorteil", begründet mit dem Gelernten) —
  ersetzt eine ursprüngliche Meta-Frage zu den Distraktorwörtern selbst, die zu wenig
  inhaltliche Tiefe hatte. M2s Teachable-Machine-Anleitung
  war als einzelner Absatz zu knapp und ließ Schüler:innen an der echten Tool-UI
  hängen — jetzt eine nummerierte Schritt-für-Schritt-Anleitung mit den konkreten
  Button-Bezeichnungen der Seite (Get Started → Image Project → Standard image model
  → Klassen umbenennen → Webcam → Hold to Record → Train Model → Preview), plus ein
  bewusster Schritt 8 (dritten, untrainierten Gegenstand zeigen), der in der
  Reflexionsfrage aufgegriffen wird. **Hinweis:** Diese Teachable-Machine-Anleitung
  wurde im Herbst 2026 durch GenAI Teachable Machine (tm.gen-ai.fi) ersetzt, siehe
  nächster Punkt.
- **Teachable Machine → GenAI Teachable Machine** (Herbst 2026, Nutzer-Feedback aus
  dem Unterricht: Googles Teachable Machine funktioniert auf iPad/iPhone nicht
  zuverlässig, die Live-Webcam-Inferenz bricht dort ab). Zwischenschritt: kurz auf
  Machine Learning for Kids umgestellt — dann hat der Nutzer **tm.gen-ai.fi** gefunden
  (GenAI Teachable Machine, Uni Ostfinnland, Open Source github.com/knicos/genai-tm),
  **selbst live auf iPhone UND iPad getestet, funktioniert** — damit finaler Ersatz,
  Machine-Learning-for-Kids-Zwischenstand wieder verworfen. Vorteile ggü. beiden
  Vorgängern: **kein Konto/kein Login-Schritt** nötig (direkter Einstieg auf
  „tm.gen-ai.fi/home"), **100 % lokale Verarbeitung im Browser** (keine
  Server-Übertragung, keine Cookies/Tracking — passt gut zum Datenschutz-Prinzip der
  App), und die UI ist fast identisch zur echten Google Teachable Machine (Class 1/
  Class 2, Webcam/Upload, „Train classifier") — eingesetzt in **K06-M2** (Erstkontakt)
  und **K09-M3** (Bias-Hands-on, als „Wiederholung aus Klasse 6" kompakt gehalten).
  **Bewusst kein echter Screenshot** (Konvention „keine externen Bilder" + Oberflächen
  ändern sich) — stattdessen ein **detailgetreues HTML/CSS-Mockup** (auf Nutzerwunsch,
  „soll Schülern helfen") im selben Stil wie das Gemini-Mockup in K08-M5: Training-
  Data-Karte mit Class-1/Class-2-Boxen (Umbenennen-Stift, Webcam/Upload-Buttons, „Add a
  class") + Train-classifier-Button, mit nummerierten Kreis-Labels ①–④, die im Text
  darunter erklärt werden — die eigentliche Schritt-Liste darunter ist dadurch kürzer
  geworden (5 statt vorher 10 Schritte), weil das Mockup schon viel visuell zeigt.
  K09-M3 übernimmt weiterhin auch die vom Nutzer gewünschte Wiederholung zu Beginn von
  K09 (kein separates, redundantes Modul). Zusätzlich neue Aufgabe in K09-M3: **„⭐
  Extra-Herausforderung: Baue eine Falle"** — Schüler:innen versuchen, ihr eigenes
  trainiertes Modell auszutricksen (z. B. Gegenstand, der zu keiner Klasse passt; beide
  Gegenstände gleichzeitig zeigen; Dunkelheit; nur Ausschnitt zeigen), mit einem
  **„💡 Hilfe"-Button** für Ideen, falls jemand nicht weiterkommt — technisch ein
  natives HTML `<details>/<summary>`-Element (kein neuer Task-Typ/JS nötig, da reine
  Auf-/Zuklapp-Funktion), gefolgt von einer `reflect`-Frage zu Ergebnis und Lernmoment
  über Grenzen/Möglichkeiten des Trainings. OB-8s Ebenen-Pyramide (Ebene-3-Beispiel)
  entsprechend final auf „GenAI Teachable Machine" aktualisiert.
- **K08-Modulreihenfolge geändert**: `K08-M5` „Gemini kennenlernen" steht im Array jetzt
  direkt nach `K08-M1` (Auffrischung), **vor** `K08-M7` (CRAFT) — vorher kam M5 ganz am
  Ende, obwohl M7/M2/M3/M4 alle schon vorher live mit Gemini arbeiten ließen, ohne dass
  Gemini je eingeführt wurde. Reine Array-Reihenfolge-Änderung (IDs/Fortschritt bleiben
  unberührt, siehe K08-M7-Hinweis unten zu Array-Position vs. ID). M5s einleitender
  Datenschutz-Text verweist jetzt **vorausschauend** auf „Grenzen, Risiken &
  Datenschutz" (kommt jetzt danach) statt rückblickend darauf.
- K08-M5 „Gemini kennenlernen": Ab Klasse 8 dürfen Schüler:innen Gemini nutzen, daher
  eigenes Modul mit stilisiertem (nicht echtem!) UI-Diagramm — **keine Screenshots**,
  Google ändert die Oberfläche zu oft, deshalb Inline-HTML/CSS-Mockup mit nummerierten
  Erklär-Punkten. Verlinkt die schulweite „GSIS KI-Ampel" (0–4-Skala, Google-Drive-PDF)
  statt die Tabelle im Code zu duplizieren — die Schule pflegt das PDF unabhängig; die
  Ampel-Reflexion nennt jetzt ein festes Beispiel (Bio-Plakat zum Wasserkreislauf,
  Stufe 2), damit niemand Zeit mit der Suche nach einer eigenen Aufgabe verliert.
  Mini-Challenge auf gemini.google.com/app (neuer Tab) mit `input`-Aufgabentyp —
  bewusst eine Aufgabe mit eindeutig berechenbarer Antwort (Zeitrechnung), nicht
  wörtlicher Abgleich von Geminis Antworttext, da KI-Ausgaben nicht deterministisch
  sind. Direkt danach eine zweite Challenge, bei der die Zeitrechnung bewusst
  kontrastiert wird: Schüler:innen fragen Gemini etwas sehr Lokales über ihr eigenes
  Klassenzimmer/Schulgebäude (z. B. Fensterzahl, Stuhlfarbe) — dort haben sie echten
  Wissensvorsprung, weil das nirgends im Internet steht; offene Reflexion statt
  `input`, da die „richtige" Antwort je nach echtem Klassenzimmer variiert.
- K08-M4 „Grenzen, Risiken & Datenschutz" erweitert: neue Sortier-Aufgabe mit sechs
  ausformulierten Mini-Szenarien (statt Ein-Wort-Beispielen) — die letzten beiden
  „KI stößt an Grenzen"-Items (aktuelle Lokalnachrichten kennen; private Info über
  eine reale Person nennen) bereiten inhaltlich den folgenden Hands-on-Test vor.
  **Buchstaben-Zähl-Demo ersetzt** (Nutzer-Feedback aus echtem Unterrichtstest:
  Gemini beherrscht die „s"-Zählaufgabe in „Verantwortungsbewusstsein" inzwischen
  zuverlässig, der Trick funktioniert nicht mehr): neuer Test fragt Gemini stattdessen
  „Wie viele Fenster hat das Klassenzimmer, in dem ich gerade sitze?" — ganz lokales,
  nirgends öffentlich stehendes Wissen, bei dem Sprachmodelle oft lieber eine erfundene,
  selbstsicher klingende Zahl liefern als „weiß ich nicht" zuzugeben; Schüler:innen
  zählen parallel selbst nach. Die Reflexion fragt zusätzlich (kombiniert aus einem
  vorherigen, jetzt entfernten Entwurf mit einer frei erfundenen Person/Nachbarin, der
  zu ähnlich gewesen wäre) allgemein, welche Art von Daten eine KI überhaupt zur
  Verfügung hat — auch mit Blick auf reale Personen. M5s Beispiel-Liste für die eigene
  Wissensvorsprung-Challenge wurde um „Fenster" gekürzt (nur noch Stuhlfarbe/Schritte
  zur Cafeteria), damit beide Module nicht dieselbe Frage stellen. Frühere Version
  ersetzte bereits einen Google-Translate-Bias-Entwurf (Türkisch-Genus-Bias) — auch
  der war verworfen worden, weil der heutige Live-Zustand nicht zuverlässig
  vorhersehbar war; das Gemini-Fenster-Beispiel funktioniert dagegen unabhängig vom
  Modellstand zuverlässig als Lernmoment (auf lokales Wissen hat kein Modell Zugriff).
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
- **OB-8 „Die 7 Ebenen der KI – und die fehlende Ebene 6,5"**: Ausgangspunkt war ein
  LinkedIn-Post eines anderen Lehrers (zwei Infografiken) — Idee vom Nutzer geprüft und
  erst nach expliziter Bestätigung umgesetzt. Ebenen-Pyramide (Klassische KI → Machine
  Learning → Neuronale Netze → Deep Learning → Generative KI → Agentic AI → AGI) als
  Einordnungsraster, das rückblickend an K05–K10 anknüpft (Ebene 3 = GenAI Teachable
  Machine, Ebene 5 = Gemini, Ebene 6 = Deep Research aus OB-7). Die „Ebene 6,5" nutzt reale,
  **einzeln via WebSearch verifizierte** Ereignisse vom 8.–26. Sept. 2026 (Jacob Coxons
  Rücktritt bei Anthropic, Amodeis Essay „We Must Pace the Frontier" [12.9., 6–12-Monats-
  Warnung vor KI-Agenten-Schwärmen], OpenAIs Offenlegung unautorisierter Agenten-Zugriffe
  auf US-Behördenseiten) — bewusst NICHT unkritisch übernommen: eine Reflexionsaufgabe
  stellt Amodeis Warnung explizit einer Gegenstimme (Axios-Analyse vom 15.9., hält das
  Botnet-Szenario für technisch kaum plausibel) gegenüber. Datumsstempel „Stand 27.9.2026,
  Untersuchungen laufen noch" bewusst gesetzt, weil die Sache zum Zeitpunkt der Erstellung
  noch nicht abgeschlossen war. Die Original-Infografiken selbst wurden nicht übernommen
  (fremdes Bildmaterial + Projekt-Konvention „keine externen Bilder") — das Ebenen-Konzept
  ist stattdessen als eigenes gestyltes HTML nachgebaut, wie beim CRAFT-Framework in K08.
- **Impressum & Datenschutz** über Fußzeile erreichbar (`openImpressum()`/`openPrivacy()`):
  Herausgeber Sebastian Ghofranifar (Koordinator digitale Unterrichtsentwicklung, GSIS
  Hongkong, sghofranifar@gsis.edu.hk), verantwortliche Institution GSIS. Lizenz **CC BY-NC 4.0**.
  Datenschutz-Kernaussage: keine Server-Erhebung, alles nur lokal in `localStorage`;
  kurzer Hinweis dazu auch im Namens-Modal beim ersten Start.
- **„Meine Einreichungen"-Ansicht** (`renderSubmissions()`, Route `submissions`, neuer
  Nav-Button 📋 neben Übersicht/Fehlerspeicher): Auslöser war die Frage, wie Lehrkräfte
  überhaupt an die geschriebenen Reflexionsantworten der Schüler:innen kommen — die App
  hat **keinen Server**, `reflectHint` sagt korrekt, dass alles nur lokal aufs Gerät
  gespeichert wird. `collectSubmissions()` sammelt aus allen Jahrgängen + Oberstufe alle
  `reflect`-Aufgaben mit `prod:true` und liest die passende `Store`-Antwort (Key
  `reflect_<moduleId>_<idx>`, stabil weil Modul-Task-Arrays nie gemischt werden — anders
  als EXAM-Pools, die aber ohnehin keine `reflect`-Aufgaben enthalten, siehe
  `buildExamTasks()`). Zeigt alles gesammelt mit Frage+Antwort an, HTML-escaped über
  den neuen `esc()`-Helper (Schüler-Freitext wird sonst ungeschützt in HTML eingefügt).
  Zwei Export-Wege: „📋 Alles kopieren" (Zwischenablage) und „⬇️ Als Textdatei speichern"
  (Blob+Download-Link, komplett offline, kein Server nötig) — beide über
  `formatSubmissionsText()`, damit Schüler:innen das z. B. in ein Google Formular
  einfügen können, das die Lehrkraft dafür einrichtet. Löst **nicht** das grundsätzliche
  Problem, dass ein rein clientseitiger Zustand ohne Login theoretisch per Browser-
  Entwicklertools manipulierbar ist (wie schon bei den Zugangscodes) — das ist eine
  bewusste, dem Nutzer transparent kommunizierte Grenze der Offline-Architektur, keine
  Sicherheitslücke, die sich clientseitig schließen ließe. Zertifikat/Punkte bleiben
  Gamification, echte Bewertung sollte auf den gelesenen Einreichungstexten selbst
  basieren (schwerer glaubhaft zu fälschen als angeklickte Quizfragen). `rerender()`
  (Sprachumschaltung) erkennt die Ansicht über `.subs-view` auf dem äußeren `<section>` —
  bewusst NICHT über `.errlist` (wird intern wiederverwendet für den Einreichungs-Look,
  hätte sonst mit `renderErrors()`s eigener `.errlist`-Prüfung kollidiert).

## Konventionen (bitte einhalten)
- Alles **zweisprachig** pflegen: Textobjekte `{de:'…', en:'…'}`.
- Code-Kommentare **auf Deutsch** (ich lerne gerade programmieren — bitte erklären,
  was der Code tut, wenn du etwas änderst).
- **Keine externen Bilder** für Inhalte — stattdessen die vorhandenen Inline-SVG-Icons.
- **Keine IB-/MYP-Begriffe** in der Schüler-Ansicht (reiner deutscher Stream).
- Distraktoren = **plausible Fehlvorstellungen**, keine Witz-Antworten.
- **Antwortlänge darf nie mit Richtigkeit korrelieren** (Schüler-Feedback: „die längste
  Antwort ist immer richtig"). Ursache war ein systematisches Autoren-Muster: die
  richtige Antwort (immer `correct:0` im Array) wurde beim Schreiben meist ausführlich
  begründet, Distraktoren nur als kurzer Stichpunkt. Die `shuffled()`-Anzeigemischung
  (siehe K08-Test-Fixes) verhindert nur das Erraten über die **Position**, nicht über
  die **Textlänge** — beide Signale müssen unabhängig voneinander neutralisiert sein.
  Bei neuen `quiz`/`multi`-Aufgaben: alle Optionen auf vergleichbare Länge bringen,
  typischerweise durch Ausbauen der Distraktoren zu ausführlicheren, plausibel
  klingenden Fehlvorstellungen (nicht durch Kürzen der richtigen Antwort, das schwächt
  oft die Erklärung). Ausnahme mit Bedacht: Bei Prompt-Qualitäts-Fragen (K06-M4,
  EXAM-K06) ist ein knapper Distraktor wie „Wetter." absichtlich kurz, weil Kürze dort
  der Lehrinhalt selbst ist (schlechter Prompt = zu wenig Kontext) — dort wurde
  stattdessen ein zweiter, bewusst **langer, aber inhaltsleer-schwammiger** Distraktor
  ergänzt, der zeigt: Länge allein macht einen Prompt nicht gut. Alle 104 `quiz`/`multi`-
  Aufgaben in Modulen + Abschlusstests wurden im Herbst 2026 per Skript auf „korrekte
  Antwort ist strikt die längste Option" geprüft (66 von 104 betroffen) und einzeln
  neu formuliert; Korrektheit jeder Antwort dabei inhaltlich erneut gegengeprüft.

## Wichtige Konstanten (im JS, oben)
- `TEACHER_CODE = 'gsis-ki-2026'` — **noch auf echten Code ändern**.
- `SENIOR_CODE = 'gsis-oberstufe-2026'` — Zugangscode für „Oberstufe · Projekttag",
  **noch auf echten Code ändern**.
- `GRADE_CODES` — je ein Direktzugangs-Code pro Jahrgang (K05–K10). Bewusst **kein
  gemeinsames Muster** mehr (vorher `gsis-k0X-2026` — ein Schüler hätte durch simples
  Austauschen einer Ziffer den Code für einen anderen Jahrgang erraten können). Jetzt
  sechs unabhängige Wort+Zufallszahl-Codes, zufällig aus einer neutralen Wortliste
  generiert, aber weiterhin leicht laut vorlesbar/teilbar: K05 `pixel-67`, K06
  `lupine-89`, K07 `atlas-49`, K08 `delta-78`, K09 `koralle-28`, K10 `gletscher-35`.
- `PASS = 0.75` — gilt einheitlich für Module und Abschlusstests.

## Arbeitsweise mit mir
- Erst kurz sagen, was du vorhast, dann ändern — nicht alles auf einmal.
- Nach einer Änderung: kurz zusammenfassen, was du getan hast, und wie ich es teste.
