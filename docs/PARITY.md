# JJBars 2 – Paritäts-Checkliste

Stand: 25. Nov. 2026. Die Fragen F1 bis F15 sind beantwortet; die Antworten stehen in der letzten Tabelle ganz unten und gelten vor der Spalte "Soll (Vorschlag)" in den Tabellen darüber. Zeilen ohne Verweis auf eine Frage (zum Beispiel H1 bis H3, E4 bis E8, E10) sind Vorschläge, die noch nicht ausdrücklich abgenommen sind.

Diese Liste hält fest, was die alte App heute tut (Ist, auch Fehler) und was die neue tun soll (Soll). Das Soll ist ein Vorschlag von mir; du nimmst es ab oder änderst es, bevor Code entsteht. Gelesen ist der gesamte Code der alten App (Stand: Repository `jjbars_2`, ein Commit), geprüft auf einem Gerät ist nichts: Aussagen zum Verhalten sind aus dem Code abgeleitet.

## Start und Benutzername

Die Startseite zeigt beim ersten Start die Namenseingabe, danach drei Aktionen und eine Datenbank-Statuskarte. Phase 3 setzt das um; der Katalog-Teil gehört zu Phase 2.

| Nr. | Funktion | Ist (auch Fehler) | Soll (Vorschlag) |
|---|---|---|---|
| H1 | Benutzername beim ersten Start | Eingabe "Dein Benutzername..." mit Button "Speichern", gespeichert in AsyncStorage unter `userName`. Keine Prüfung: ein leerer Name ("") wird gespeichert und ergibt "Willkommen zurück, !". Ohne Eingabe passiert nichts, der Fehler wird still geschluckt. | Name ist Pflicht und wird getrimmt; leere Eingabe zeigt eine Meldung. Der Name bleibt in den Einstellungen (AsyncStorage). |
| H2 | Begrüßung | Logo "J&J BARS", Karte "Willkommen zurück, {Name}! 👋 / Bereit für dein nächstes Training?". Beim Scrollen schrumpft das Logo und der Name erscheint in einer Kopfzeile. | Inhalt gleich. Die Scroll-Animation ist Optik und kommt, wenn überhaupt, in Phase 5. |
| H3 | Drei Aktionen | Buttons "Workout erstellen", "Online Workouts", "Gespeicherte Workouts" navigieren zu den Screens. | Gleich. |
| H4 | Benutzername löschen | Langdruck auf die Statuskarte löscht den Namen ohne Rückfrage (Warnung "You deleted your Username!!"). Die Anzeige bleibt unverändert, die Eingabemaske erscheint erst nach einem Neustart. | Namen ändern statt nur löschen, mit sichtbarer Wirkung. Frage F1. |
| H5 | Katalog aktualisieren | Tippen auf die Statuskarte löscht den SQLite-Ordner und lädt `jjbars_2.db` neu von GitHub. Schlägt der Download fehl (z. B. offline), bleibt keine Datenbank; Meldung "Konnte Datenbank nicht laden. Bitte überprüfe deine Internetverbindung.". Karte zeigt "Download DB!" oder "DB geladen (n Tabellen) - Klick zum Update". | Katalog ist gebündelt und funktioniert offline. Ein optionales Update lädt in eine Temp-Datei, prüft ein Manifest und tauscht erst dann; bei Fehler bleibt die alte Datenbank. |
| H6 | Firebase-Start | Wird im Konstruktor von Home mit `ApiKey` initialisiert. Die Datei fehlt im Repo, ein frischer Clone startet nicht. | Firebase-Init in `lib/`, Konfiguration über EAS-Umgebungsvariablen. |

## Workout erstellen und bearbeiten

Der Editor ist der größte Bildschirm (981 Zeilen) und der mit den meisten Fehlern. Links steht der Übungskatalog, rechts das Workout mit seinen Runden, unten das Suchfeld und der Button "Add Round". Spec-Tiefe: ausführlich (Phase 3).

| Nr. | Funktion | Ist (auch Fehler) | Soll (Vorschlag) |
|---|---|---|---|
| E1 | Katalogliste | Lädt alle Übungen per Join über drei Tabellen. Ungefiltert gibt es eine Zeile pro Übung und Muskelgruppe: 188 Zeilen für 131 Übungen, 50 Übungen erscheinen mehrfach (57 zusätzliche Zeilen). Gefiltert (Suche) gibt es eine Zeile pro Übung. | Eine Zeile pro Übung, Muskelgruppen zusammengefasst. Frage F2. |
| E2 | Suche | Filtert nach Name, Schwierigkeit und Muskelgruppe mit LIKE, bei jedem Tastendruck. Der Suchtext wird in den SQL-Text eingesetzt: ein Apostroph im Suchtext löst den Fehler "Konnte Übungen nicht filtern" aus. | Parameter-Binding statt String-Interpolation; kurze Verzögerung beim Tippen. |
| E3 | Übung hinzufügen | Tippen auf eine Übung öffnet "Reps eingeben" mit Zahlenfeld (Start 10, danach der zuletzt eingegebene Wert) und Umschalter Reps/Secs. Speichern hängt die Übung ans Ende der aktiven Runde. Beim Hinzufügen wird nicht auf ≥ 1 geprüft (0 geht durch), beim Bearbeiten schon. | Gleicher Ablauf; Prüfung ≥ 1 auch beim Hinzufügen. Frage F3. |
| E4 | Runde hinzufügen | "Add Round" legt "Round n" mit 3 Wiederholungen an und macht sie aktiv. Aktiv wird eine Runde auch durch Tippen auf den Kreis links (Haken = aktiv). Neue Übungen landen immer in der aktiven Runde. | Gleich. |
| E5 | Rundenzahl ändern | Tippen auf Runde oder "3x" öffnet "Rundenanzahl bearbeiten". Ungültig oder kleiner 1: Meldung "Ungültig". | Gleich. |
| E6 | Runde löschen | Mülleimer mit Rückfrage "Runde löschen". Danach werden alle Runden neu nummeriert, die letzte ist aktiv. Es dürfen alle Runden gelöscht werden. | Gleich. |
| E7 | Übung bearbeiten | Tippen auf eine Übung öffnet "Übung bearbeiten" (Zahl und Reps/Secs), Prüfung ≥ 1. | Gleich. |
| E8 | Übung löschen | Langdruck mit Rückfrage "Übung löschen". Intern wird der Zustand direkt verändert (`splice`). | Gleich, ohne direkte Zustandsänderung. |
| E9 | Reihenfolge ändern | Langdruck (500 ms) auf eine Runde schaltet den Ziehmodus für alle Runden und Übungen ein; Ziehen um mehr als 50 px nach oben oder unten verschiebt um eine Position, Langdruck beendet den Modus. Übungen bleiben in ihrer Runde. Nach dem Verschieben von Runden werden die Titel neu nummeriert, die Runde "Round 2" ist also immer die zweite. Auch hier wird der Zustand direkt verändert. | Frage F4. |
| E10 | Speichern | Name Pflicht ("No Title"); gleicher Name wie ein vorhandener Eintrag: Warnung "... Already Exists" (nur beim Neuanlegen). Gespeichert wird in AsyncStorage unter dem Namen, danach geht es zur Startseite. Das Speichern wird nicht abgewartet, ein Fehler dabei wird still geschluckt. | Fehler werden angezeigt; weg geht es erst nach erfolgreichem Speichern. |
| E11 | Bearbeiten-Modus | Öffnen über Langdruck in "Gespeicherte Workouts"; der Name ist gesperrt, die letzte Runde ist aktiv. Der ID-Zähler startet bei 0, neue Übungen bekommen also IDs, die schon vergeben sein können. Doppelte IDs ergeben doppelte Listen-Schlüssel; fehlt die ID ganz, nimmt der Editor `Math.random()` als Schlüssel. | Stabile IDs (UUID), Name ändern erlaubt oder bewusst gesperrt. Frage F5. |
| E12 | Ungespeicherte Änderungen | Zurück ohne Rückfrage verwirft alles; im Code gibt es keinen Schutz davor. | Rückfrage beim Verlassen mit Änderungen. Frage F6. |
| E13 | Katalog nicht ladbar | Beim ersten Start ist Internet nötig. Schlägt der Download fehl, erscheint die Meldung "DB Fehler", der Bildschirm bleibt aber dauerhaft bei "Lade Datenbank...". | Gebündelter Katalog, Editor funktioniert offline. |
| E14 | Zeilenhöhe | Die Konstante `ITEM_HEIGHT` wird vor ihrer Definition benutzt (Zeile 136 gegen 169). Die Höhe der Zeilen kommt dadurch faktisch aus `minHeight: 68` des Textes (aus dem Code abgeleitet, nicht auf dem Gerät gesehen). | Zeilenhöhe 68 als Token im Theme. |

## Gespeicherte und Online-Workouts

Beide Listen sind gleich aufgebaut. Lokal ist der Name zugleich der Schlüssel in AsyncStorage, online sucht die App per Name statt per ID. Spec-Tiefe: mittel für Online (Phase 2), kurz für die lokale Liste (Phase 3).

| Nr. | Funktion | Ist (auch Fehler) | Soll (Vorschlag) |
|---|---|---|---|
| S1 | Lokale Liste | Zeigt alle AsyncStorage-Schlüssel außer `userName`. Leer: "No Workouts Saved Yet!". Antippen öffnet die Workout-Ansicht, Langdruck (500 ms) das Menü mit Upload, Edit, Delete, X. | Liste aus dem Repository; in der Migration werden nur die Workout-Schlüssel übernommen (siehe D3). |
| S2 | Lokal löschen | Sofort, ohne Rückfrage. Das Löschen wird nicht abgewartet, die Liste kann den gelöschten Eintrag noch kurz zeigen. | Rückfrage vor dem Löschen. Frage F7. |
| S3 | Upload | Prüft, ob online schon ein Workout mit gleichem Namen existiert (von irgendwem). Wenn nicht: neuer Eintrag mit Datum als Text "dd.mm.yyyy", Benutzername, Name und Runden; Meldung "Workout Uploaded". | Eintrag mit `ownerUid`, Zeitstempel statt Datumstext. Verhalten bei gleichem Namen: Frage F8. |
| S4 | Upload überschreiben | Bei gleichem Namen fragt "This Workout Already Exists.. Do You Want It To Be Overwritten?" (Yes/No). Bei Yes wird der Eintrag ersetzt, auch wenn er von jemand anderem stammt; Datum und Benutzername werden dabei überschrieben. | Nur der Besitzer darf überschreiben (Rules). Frage F8. |
| S5 | Online-Liste | Zeigt alle Workouts, neueste zuerst, mit "Uploaded by: {Name} on {Datum}". Das Datum wird aus dem Text zurückgerechnet; bei einem Eintrag ohne Datum löst die Sortierung einen Fehler aus, und die Liste bleibt leer. Leer oder beim Laden: "No Workouts Saved Yet!". | Sortierung nach Zeitstempel; defekte Einträge werden beim Einlesen übersprungen. Anzeige ohne Netz: Frage F9. |
| S6 | Online-Workout ansehen | Antippen öffnet die Workout-Ansicht und lädt per Name aus Firebase. Bei gleichen Namen gewinnt der letzte Treffer. Der Listener wird nie abgemeldet. | Laden per ID, einmaliges Lesen oder Abmelden beim Verlassen. |
| S7 | Download | Langdruck, "Download": Existiert lokal schon ein Workout mit gleichem Namen, Warnung "... Already Exists". Sonst Speichern unter dem Namen und "Workout Downloaded". Der Listener bleibt aktiv: ändert jemand das Online-Workout später, überschreibt er die lokale Kopie still (auch nach eigenen Änderungen) und zeigt die Erfolgsmeldung erneut. | Einmaliges Lesen, keine Listener. Namenskonflikt: Frage F10. |
| S8 | Runden ohne Übungen | Firebase speichert leere Listen nicht, ein Download legt solche Runden ohne `data` an. Die Workout-Ansicht fängt das beim Online-Anzeigen ab (`data` wird ergänzt), Bearbeiten und lokale Anzeige nicht (aus dem Code abgeleitet). | Beim Einlesen wird jedes Workout validiert und normalisiert (zod); fehlende Listen werden zu leeren. |
| S9 | Seitenköpfe | Die statischen Kopfzeilen mit Home-Symbol und dem Titel "Create Shit" in beiden Screens greifen in React Navigation 7 nicht (toter Code); sichtbar ist der Titel aus `route.params.title`. | Einheitliche Titel; Sprache: Frage F11. |

## Workout-Ansicht und Timer

Die Ansicht zeigt ein Workout und dient zugleich als Training: Nur Übungen in Sekunden haben einen Timer. Spec-Tiefe: ausführlich für den Timer (Phase 4), kurz für die Anzeige (Phase 3).

| Nr. | Funktion | Ist (auch Fehler) | Soll (Vorschlag) |
|---|---|---|---|
| W1 | Anzeige | Runden als Abschnitte "Round n" mit "x3 Times", die Kopfzeilen bleiben beim Scrollen oben. Übungen als Karten mit Name und "10 Reps". Der Titel der Kopfzeile wird erst nach dem Laden gesetzt: "{Name} - (2 S / 5 Ü)". | Gleich; Titel und Sprache: Frage F11. |
| W2 | Quelle und Leerzustand | Lokal aus AsyncStorage, online (Parameter `source`) per Name aus Firebase. Fehlendes oder ungültiges JSON ergibt dieselbe leere Seite wie ein leeres Workout: "Keine Übungen gefunden! ... Dieses Workout enthält noch keine Übungen." | Leer, nicht gefunden und Ladefehler werden unterschieden. |
| W3 | Übungen in Wiederholungen | Nur Anzeige "n Reps". Kein Abhaken, kein Fortschritt über Übungen oder Runden; "x3 Times" ist nur eine Information. | Frage F12. |
| W4 | Übungen in Sekunden | Jede Zeile mit "Secs" hat einen Play-Knopf und die Restzeit ("n Secs", bei 0 "Time's Up!"). Jede Zeile hat ihren eigenen Zustand: mehrere Timer können gleichzeitig laufen, der Stand geht beim Verlassen der Seite verloren. | Frage F13. |
| W5 | Ablauf eines Timers | Play startet einen 3-Sekunden-Vorlauf ("Gooo in 3"), der Knopf ist dabei gesperrt. Danach Startsignal (ein Piepton, 250 ms Vibration) und Countdown im Sekundentakt. Bei 1 Sekunde Restzeit kommt das Endsignal (zwei Pieptöne mit Vibration, 500 ms Abstand); bei 0 steht "Time's Up!", das Refresh-Symbol erscheint eine Sekunde später und setzt zurück. | Gleiches Signalmuster, aber als Zustandsautomat: Bereit, Vorlauf, Läuft, Pause, Fertig. |
| W6 | Pause | Tippen im Countdown stoppt, die Restzeit bleibt. Play startet danach wieder mit dem vollen 3-Sekunden-Vorlauf. Im Vorlauf selbst ist Pause nicht möglich, weil der Knopf gesperrt ist. | Fortsetzen mit oder ohne neuen Vorlauf: Frage F14. |
| W7 | Genauigkeit | Die Sekunden werden mit `setInterval` gezählt, nicht gemessen. Bremst Android die Ticks oder wird die App pausiert, geht der Timer nach. Der Bildschirm bleibt nicht an. | Restzeit aus Zeitstempeln, `expo-keep-awake` im Training (siehe Plan, Phase 4). |
| W8 | Ton | Nur `Beep.mp3` über `expo-av`, ohne eigenen Audio-Modus. | `expo-audio`, gleiches Signalmuster; die ungenutzten `beer.mp3` und `timer.mp3` entfallen. |

## Daten, Katalog und Aufräumen

Der Katalog der Übungen ist geklärt: `assets/testdb.txt` ist keine alte Kopie, sondern byte-identisch mit der Datei, die die App heute von GitHub lädt. Sie kann deshalb unverändert gebündelt werden.

| Nr. | Datenquelle | Ist | Soll (Vorschlag) |
|---|---|---|---|
| D1 | Lokale Workouts | AsyncStorage, Schlüssel ist der Name. Wert ist eine JSON-Liste von Runden: `title` ("Round n"), `times`, `data` mit `id`, `title`, `reps` und `repText` ("Reps" oder "Secs"). | `user.db` in SQLite mit UUID, siehe Plan (Datenmodell). |
| D2 | Benutzername | AsyncStorage, Schlüssel `userName`. | Bleibt in AsyncStorage. |
| D3 | Migration | Alle AsyncStorage-Schlüssel außer `userName` gelten als Workouts. Ab Phase 2 legt Firebase Auth ebenfalls Einträge in AsyncStorage ab. | Die Migration nimmt nur Einträge, die das Workout-Schema erfüllen, und läuft vor dem Firebase-Start; fremde Schlüssel landen nicht in der Fehlerliste. |
| D4 | Online-Workouts | `workouts/<Push-Key>` mit `date` (Text "dd.mm.yyyy"), `user`, `workoutname`, `sections`. Zuordnung über den Namen. | `ownerUid`, `updatedAt` als Zeitstempel, Zuordnung über ID, siehe Plan. |

**Katalog**

- **Quelle:** `jjbars_2.db` im öffentlichen Repository `Joelmarugg/dbtest` (Branch master, letzter Commit vom 26. Juni 2025, "neue db"). Daneben liegt `jjbars.db` mit denselben Übungen und Zuordnungen.
- **Inhalt:** 131 Übungen (Beginner, Intermediate, Advanced), 7 Muskelgruppen (Abs, Back, Bizeps, Chest, Legs, Shoulders, Trizeps), 188 Zuordnungen. Schlüssel ist der Übungsname, es gibt keine ID.
- **Gepflegt wird er heute** durch einen Commit in `dbtest`; die Apps holen ihn, wenn jemand auf die Statuskarte tippt. Wer ihn künftig pflegt und wie neue Übungen hineinkommen, ist Frage F15.

**Ungenutzt, entfällt im Neuaufbau**

- `app/data/exercises.js`: 122 Namen, nirgends importiert. 120 davon stehen auch in der Datenbank, "One Arm Australian Chin Ups" und "Pull Ups Shoulder" nicht.
- Bilder `iconbig.png`, `iconround.png`, `iconshape.png`, `splash.png`: von Code und `app.json` nicht verwendet (benutzt werden `icon.png`, `adaptive-icon.png`, `splash-icon.png`, `favicon.png`).
- Töne `beer.mp3` und `timer.mp3`; benutzt wird nur `Beep.mp3`.
- Komponenten ohne Verwendung: `CWContainer`, `ListContainer`, `SearchContainer`, `WorkoutContainer`, `NumberInput`, `ListEnd`. `Container` wird in Home importiert, aber nicht gerendert.
- Pakete: `@expo/vector-icons`, `color` und `hoist-non-react-statics` werden importiert, stehen aber nicht in `package.json` (der Plan nennt bisher nur zwei davon). Deklariert, aber nie importiert sind `expo-status-bar`, `react-native-reanimated`, `react-native-screens` und `react-native-gesture-handler`; die letzten beiden braucht `@react-navigation/stack` als Peer-Abhängigkeit.
- `Platform.OS` steht an 11 Stellen im App-Code (der Plan nennt acht); die beiden Home-Kopfzeilen `HomeHeader` und `IosHomeHeader` hängen nur am wirkungslosen `navigationOptions`.
- `config/M3Theme.js` wird nur von `PopUpModal` benutzt; die meisten Farben und Schriften stehen als Rohwerte in den Screens, `M3Typography` sogar doppelt (Home, Workouts). Das ist die Quelle für die Tokens in Phase 1.

## Entscheidungen zu F1 bis F15

Alle 15 Fragen wurden am 25. Nov. 2026 beantwortet. Wo die Antwort vom Vorschlag abweicht, steht der Grund dabei. Diese Spalte "Deine Antwort" ist die verbindliche Entscheidung.

| Nr. | Frage | Mein Vorschlag | Entscheidung (Deine Antwort) |
|---|---|---|---|
| F1 | Benutzername löschen oder ändern (H4) | Wie heute per Langdruck, aber die Eingabemaske erscheint sofort statt erst nach einem Neustart. Ein eigener Ändern-Dialog ist eine neue Funktion und kommt später. | Passt (Vorschlag übernommen, 25. Nov.) |
| F2 | Katalogliste (E1) | Eine Zeile pro Übung, mehrere Muskelgruppen in der Untertitelzeile. | Passt, mit Ergänzung: Das Level (difficulty) steht wie bisher weiterhin in der Untertitelzeile, zusammen mit den Muskelgruppen (25. Nov.). |
| F3 | Zahl beim Hinzufügen (E3) | Auch hier mindestens 1, wie beim Bearbeiten. | Passt (Vorschlag übernommen, 25. Nov.) |
| F4 | Reihenfolge ändern (E9) | Hoch- und Runter-Buttons statt Langdruck und Ziehen; kein Gesten-Paket nötig, gut testbar. Echtes Drag-and-drop kann nach Phase 7 folgen. | Vorschlag abgelehnt: Drag-and-drop bleibt (Langdruck und Ziehen, wie heute). Grund aus der Erfahrung mit der alten App: Der Bildschirm hat wenig Platz, Hoch-/Runter-Buttons wären zu klein, um gut bedienbar zu sein. Folge: Das Gesten-Paket und die Bibliothek für das Sortieren müssen in Phase 3 mit SDK 58 geprüft werden (25. Nov.). |
| F5 | Name im Bearbeiten-Modus (E11) | Weiter gesperrt; Umbenennen ist eine neue Funktion. | Variante 2: Der Name bleibt im Bearbeiten-Modus gesperrt. Neu (bewusste Abweichung von der Parität): Das Langdruck-Menü der gespeicherten Workouts bekommt den Eintrag "Kopieren". Er fragt nach einem neuen Namen und legt eine Kopie an; zum Umbenennen wird kopiert und das alte Workout gelöscht. Die alte App hat keine Kopierfunktion (im Code geprüft). Der neue Name wird auf Duplikate geprüft (25. Nov.). |
| F6 | Verlassen mit ungespeicherten Änderungen (E12) | Rückfrage, aber nur wenn etwas geändert wurde. | Passt (Vorschlag übernommen, 25. Nov.) |
| F7 | Lokal löschen (S2) | Rückfrage vor dem Löschen eines Workouts. | Passt (Vorschlag übernommen, 25. Nov.) |
| F8 | Upload mit gleichem Namen (S3, S4) | Eigener Eintrag: nachfragen und überschreiben. Fremder Eintrag: als zusätzlichen Eintrag hochladen, ohne ihn anzutasten. | Vorschlag geändert: Geteilte Workouts sollen von allen bearbeitet werden können. Soll: Gleicher Name online → nachfragen und überschreiben, unabhängig vom Besitzer (wie heute). Schutzmaßnahmen in den Firebase-Regeln (Phase 7): (1) nur gültig aufgebaute Workouts mit begrenzter Größe werden akzeptiert; (2) Überschreiben darf jeder, Löschen nur der Ersteller (`ownerUid`). Restrisiko: Jemand kann einen Eintrag mit sinnlosen, aber gültigen Daten überschreiben; das Backup ist vorhanden (25. Nov.). |
| F9 | Online-Liste ohne Netz (S5) | Hinweis "Keine Verbindung" statt "No Workouts Saved Yet!"; kein Zwischenspeicher. | Passt (Vorschlag übernommen, 25. Nov.) |
| F10 | Download bei gleichem Namen lokal (S7) | Nachfragen: überschreiben oder als Kopie "Name (2)" speichern. | Passt (Vorschlag übernommen, 25. Nov.) |
| F11 | Sprache und Titel (S9, W1) | Alles Deutsch (heute gemischt): "Workout erstellen", "Gespeicherte Workouts", "Online-Workouts". | Vorschlag geändert: Alle Texte der App auf Englisch (nicht Deutsch), direkt von Anfang an. Das schließt Titel, Meldungen und Platzhalter ein, z. B. "Create Workout", "Saved Workouts", "Online Workouts", "No connection". "Create Shit" und die gemischten Texte fallen weg. Umsetzung: alle Texte an einer zentralen Stelle, damit eine zweite Sprache später ohne Umbau möglich wäre (25. Nov.). |
| F12 | Trainingsablauf (W3) | Wie heute: Übung für Übung von Hand, ohne Rundenzähler. Ein automatischer Ablauf ist eine neue Funktion nach Phase 7. | Passt: Wie heute, von Hand und ohne Rundenzähler. Zukünftige Funktion (nach Phase 7): ein Trainingsmodus, der die aktuelle Übung hervorgehoben zeigt und automatisch zur nächsten wechselt, sobald die Übung abgehakt oder die Zeit abgelaufen ist (25. Nov.). |
| F13 | Mehrere Timer gleichzeitig (W4) | Nur ein Timer läuft; ein neuer Start stoppt den laufenden. Das verhindert überlappende Signale. | Passt: Es läuft immer nur ein Timer, ein neuer Start stoppt den laufenden. Zusätzlich Fehler beheben (aus der alten App bekannt): Nach Pause und Fortsetzen läuft die Zeit teilweise in den Minusbereich. Soll: Nach Pause läuft die Uhr korrekt weiter, nie unter 0, und endet genau bei 0. Dazu ein Test in Phase 4 (25. Nov.). |
| F14 | Fortsetzen nach Pause (W6) | Ohne neuen 3-Sekunden-Vorlauf; der Vorlauf gilt nur beim ersten Start. | Vorschlag abgelehnt: Der 3-Sekunden-Vorlauf beim Fortsetzen nach einer Pause ist gewollt, damit man sich wieder in die Übungsstellung bringen kann. Soll wie heute: Fortsetzen startet den vollen Vorlauf ("Gooo in 3"), danach läuft die Restzeit weiter. Während des Vorlaufs bleibt der Button gesperrt (25. Nov.). |
| F15 | Katalog pflegen (Katalog) | Der Katalog liegt als Datei mit Versionsnummer im neuen Repository, du pflegst ihn per Commit, die App holt Updates über das Manifest. | Vorschlag überarbeitet und bestätigt: (1) Die aktuelle Datenbank steckt als Sicherung in der App, damit sie auch bei der Erstinstallation ohne Internet funktioniert (heute wird `testdb.txt` im Code nirgends benutzt). (2) Updates kommen von GitHub über ein Manifest (Version, URL, Prüfsumme); die neue Datei wird erst heruntergeladen und geprüft, dann ausgetauscht, nie vorher gelöscht. (3) Du pflegst den Katalog wie bisher durch Ersetzen der Datei; das Manifest erzeugt ein Skript beim Commit. Ob der Katalog in `dbtest` bleibt oder ins neue Repo zieht, wird später entschieden (25. Nov.). |
