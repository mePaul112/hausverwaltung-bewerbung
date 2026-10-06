# CLAUDE.md – hausverwaltung (Mietersuche-Funnel; GitHub-Name bleibt hausverwaltung-bewerbung)

Einordnung: **Hauptpfad 05 Immobilien › Toolentwicklung › Mietersuche-Funnel**
Leitfaden: `C:\Users\steph\OneDrive - prodAIx\xx_Lebensraum\LEITFADEN-Projektstruktur.md`

## Zweck
- Öffentliche Bewerber-Front-Door für Mietinteressenten der Lebensraum Körnerstr. 35 GmbH
  (aktuell Whg BHV-KS35-04, 3 Zimmer, Bremerhaven-Leh).
- Ersetzt die Papier-Mieterselbstauskunft und meldet zugleich zur Besichtigung an.
- Aufruf je Wohnung mit `?unit=<UNIT>`, Sprache per `?lang=de|en|einfach`.

## Aufbau
- `index.html` – volles Bewerbungsformular + Selbstauskunft (DE/EN/Einfache Sprache),
  Terminwahl, drei Upload-Felder (SCHUFA, Vermieter, Einkommen).
- `termin.html` – schlankes Folgetermin-Formular für bereits Registrierte (nur Name/E-Mail/
  Telefon/Slot/Notiz, fester Notiz-Marker „Folgetermin-Anmeldung (bereits registriert)").
  Ohne aktiven Termin blendet es das Formular aus und verweist auf `index.html`.
- `aussen.jpg` – Banner. Kein Build-Step, reines statisches HTML/JS.
- Lokale Vorschau: `.claude/launch.json` → `python -m http.server 8532`.

## Backend (Supabase, Projekt `vermietung-ks35`, `qdkfzzfnntfyvutztckl.supabase.co`)
- Tabelle `applicants` (anon schreibt, darf aber nicht lesen), Tabelle `viewing_slots` (aktive Termine, Pflege im
  Table Editor), View `slot_auslastung` (Belegung je Fenster, max. 3 Parteien).
- Storage-Bucket `nachweise/<ref>/<schufa|vermieter|einkommen>/<datei>`.
- Im HTML steht nur der öffentliche, RLS-geschützte anon-Key. Service-Key liegt NUR auf dem
  Server (`/etc/llm-wiki/supabase.env`, per SSH auf 100.112.57.115) – nie lokal suchen/ablegen.
- Weiterverarbeitung serverseitig im Engine-Repo (`mietersuche_import.py` führt Dubletten über
  E-Mail/Telefon/Name zusammen, `server/triage_mietersuche.py` baut Antwortentwürfe).
- Keep-Alive gegen Free-Tier-Pause: Scheduled Task `fourstands-feedback-check` (Aufgabe 4,
  Tabelle `keepalive`, bewusst nicht `applicants`).

## Hosting / Deploy
- Remote: `https://github.com/mepaul112/hausverwaltung-bewerbung` (Branch `main`).
- Live über GitHub Pages: `https://mepaul112.github.io/hausverwaltung-bewerbung/?unit=<UNIT>`
  bzw. `.../termin.html?unit=<UNIT>`. Kein Workflow im Repo; Pages-Einstellung nicht hier belegt.
- **Zwei Kopien synchron halten:** dieselben Dateien liegen im Engine-Repo unter
  `C:\Users\steph\Projekte\llm-wiki-engine\web\bewerber-form\` – jede Änderung in beiden.
- Commit/Push nur auf ausdrücklichen Auftrag von StS.

## Harte Regeln
- **Nur Belegtes:** Keine Angaben zu Wohnung, Kosten oder Terminen erfinden; Quelle ist das
  Wiki (ks35) bzw. `viewing_slots`.
- **Adresse erst nach dem Absenden** und nur bei gewähltem Termin zeigen (sonst unangemeldete
  Besucher der Sammelbesichtigung).
- **Terminwahl nur Pflicht, wenn ein Slot wählbar ist** – sonst blockiert das ganze Formular
  (Befund 27.09.2026). Bei jedem Aktivieren/Deaktivieren von `viewing_slots` Formular-Verhalten
  UND Antworttext (`VORGANG["besichtigung"]` im Engine-Repo) gegenprüfen.
- **Datenschutz:** Bewerberdaten und Nachweise nie in Code, Commits, Logs oder URLs; keine
  Testbewerbungen in die Produktiv-Tabelle `applicants`. Abgelehnte Daten werden gelöscht
  (Einwilligungstext) – Löschen der Nachweis-Ordner ist offener Punkt.
- **Antwort-Regeln Interessenten** (Triage/Entwürfe): Formularlink immer mit + Hinweis, dass
  Kleinanzeigen Links „verschluckt" + Frage nach privater E-Mail; **niemals WhatsApp anbieten**.
  KdU-fragliche Jobcenter-Fälle → Förderrahmen-Vorlage; Jobcenter-Kosten-Handout ist Default-
  Anhang. Mails nur als Entwurf, Senden macht StS.
- Drei Sprachvarianten (DE/EN/Einfach) bei jeder Textänderung mitpflegen.
