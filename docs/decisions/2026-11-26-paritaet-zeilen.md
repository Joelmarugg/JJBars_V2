# Entscheidungen zu den Zeilen der Paritäts-Checkliste (26. Nov. 2026)

Quelle: `docs/PARITY.md`. Hier stehen nur die Entscheidungen, die über "gleich wie heute" hinausgehen. Ergänzt `2026-11-25-phase-0.md`.

## Daten und Firebase

- **Lokale Workouts:** SQLite (`user.db`) bleibt, mit ID statt Name als Schlüssel. Die alten AsyncStorage-Daten werden bei der Migration nur kopiert und erst in einer Folgeversion aufgeräumt (D1, D3).
- **Migration:** läuft beim ersten Start der neuen App, vor dem Firebase-Start. Sie nimmt nur Einträge im Workout-Format und überspringt fremde Schlüssel. Defekte Workouts landen in einer Fehlerliste (D3).
- **Firebase-Felder (D4):** Die Feldnamen der alten App bleiben bestehen (`workoutname`, `sections`, `date`, `user`), damit die alte App-Version die Online-Liste weiter lesen kann. Die neue App ordnet sie intern `title` und `rounds` zu. `date` ist "erstellt am", `user` der Name des ursprünglichen Erstellers; beide ändern sich beim Überschreiben durch andere nicht. Neu: `ownerUid`, `updatedAt`, `schemaVersion`. Alte Einträge werden mit Prüfung und Standardwerten gelesen, nicht umgeschrieben. Die Rules erzwingen nur, dass `ownerUid` unverändert bleibt.
- **Online-Workout laden (S6, S8):** per ID, einmaliges Lesen ohne Listener, jeder Eintrag wird beim Einlesen geprüft (zod); fehlende Listen werden zu leeren Listen, defekte Einträge übersprungen.

## Bedienung

- **Startseite (H1 bis H6):** Name ist Pflicht und wird getrimmt. Das Scroll-Verhalten bleibt wie heute (Phase 3). Neu: bei jedem Start ein anderer englischer Willkommenssatz (Liste von etwa 15, nie derselbe zweimal hintereinander). Die Statuskarte bleibt, mit Katalogversion; beim Start wird nur das Manifest geprüft, geladen wird erst nach dem Tipp, kein automatisches Update. Firebase-Konfiguration über EAS-Umgebungsvariablen, lokal über eine nicht eingecheckte `.env`.
- **Editor (E2, E10, E14):** Suche mit Parameter-Binding, maskierten `%` und `_`, etwa 300 ms Verzögerung und "No exercises found" bei keinem Treffer. Speichern: Name Pflicht und getrimmt, mindestens eine Übung, Fehler zeigen die Meldung und der Editor bleibt offen. Das Layout des Editors (Zeilenhöhe, Scrollen, Tastatur, Abstände zu den Systemleisten, Schriftgröße, Zieh-Modus) bekommt vor dem Code eine eigene Layout-Spec und einen Prototyp. Bekannte Auffälligkeiten der alten App: Tastatur verdeckt das Suchfeld, Zeilen werden abgeschnitten, manchmal großer Abstand oben.
- **Workout-Ansicht (W2):** drei Zustände mit englischen Texten: leer, nicht gefunden, Ladefehler mit "Try again".
- **Timer (W5, W7, W8):** Signalmuster wie heute (Vorlauf, ein Signal am Start, zwei am Ende); das Zurücksetzen-Symbol erscheint zusammen mit "Time's Up!". Der Bildschirm bleibt an, solange die Workout-Ansicht offen ist. Töne und Vibration nur im Vordergrund; Signale bei gesperrtem Bildschirm wären eine spätere Funktion mit geplanten Benachrichtigungen.

## Testgerät

- Samsung Galaxy S25 mit der neuesten verfügbaren Android-Version. Kleine Bildschirme und große Systemschrift werden zusätzlich im Emulator geprüft.
