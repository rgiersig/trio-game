# Trio (Tribulation/Trimatics) – Regel- und Testspezifikation

## 1. Ziel

Dieses Dokument definiert die verbindliche, nachverfolgbare Spezifikation für die digitale Trio-Umsetzung.

- Anforderungen: `TRIO-SPEC-*`
- Testspezifikationen: `TRIO-TEST-*`

Alle zuletzt bereitgestellten Klarstellungen sind eingearbeitet.

## 2. Begriffe

- **Zahlenkärtchen**: Quadratische Feldkärtchen mit Ziffern 1–9.
- **Zahlenfeld**: 7×7-Auslage aus 49 Zahlenkärtchen.
- **Zahlenchip**: Runder Chip mit Zielwert.
- **Kombination**: Ausdruck mit genau drei Feldziffern in der Form `a*b+c` oder `a*b-c`.
- **Linienbedingung**: Drei verwendete Feldziffern liegen horizontal, vertikal oder diagonal unmittelbar aufeinanderfolgend auf einer Geraden.
- **Aktive Spielende**: Spielende, die in der aktuellen Runde teilnehmen, meldungsberechtigt sind und für Bereitschafts-ACKs berücksichtigt werden.
- **Deaktivierte Spielende**: Spielende, die nur für die aktuelle Runde wegen AFK gemeinschaftlich ausgesetzt wurden.
- **Runde**: Chip verdeckt vorbereiten/verteilen, Bereitschaft für nächstes Zahlenplättchen, Startsignal/Aufdecken, Lösungsphase, Prüfung, Wertung/Fortsetzung.
- **Penalty**: 5-sekündige Meldesperre nach falscher Meldung.
- **Digitalisierungsregel**: Regel, die nicht aus dem physischen Grundspiel stammt, sondern Fairness, Synchronisation oder Bedienbarkeit der digitalen Umsetzung regelt.

## 2.1 Regelherkunft

Nicht gesondert markierte Spielregeln bilden das fachliche Trio-Grundspiel ab. Regeln mit „Digitalisierungsregel“ sind verbindliche Ergänzungen der digitalen Umsetzung, insbesondere für Synchronisation, Bedienbarkeit, AFK-Verhalten, Timer, Feedback und faire Wertung bei Netzwerklatenz.

## 3. Funktionale Spezifikation inkl. Tests

## 3.1 Setup, Board und Chipdeck

### TRIO-SPEC-001 – Feldgröße
Das Spiel MUSS ein Zahlenfeld von exakt 7×7 verwenden.

**Testspezifikation**
- **TRIO-TEST-001A**: Setup erzeugt Feld mit 7 Zeilen und 7 Spalten.
- **TRIO-TEST-001B**: Setup mit abweichender Dimension MUSS fehlschlagen.

### TRIO-SPEC-002 – Feldkartenanzahl
Das Zahlenfeld MUSS aus genau 49 Zahlenkärtchen bestehen.

**Testspezifikation**
- **TRIO-TEST-002A**: Nach Setup sind genau 49 Feldpositionen belegt.
- **TRIO-TEST-002B**: Setup mit 48/50 Karten MUSS abgewiesen werden.

### TRIO-SPEC-003 – Ziffernbereich
Jedes Zahlenkärtchen MUSS eine Ziffer zwischen 1 und 9 enthalten.

**Testspezifikation**
- **TRIO-TEST-003A**: Prüfe alle Feldwerte auf Bereich [1,9].
- **TRIO-TEST-003B**: Werte 0 oder 10 MÜSSEN invalid sein.

### TRIO-SPEC-004 – Verbindliche Feldverteilung
Pro Partie MUSS dieselbe Ziffern-Multimenge verwendet werden; nur Positionen sind zufällig:
- `1×5`, `2×6`, `3×6`, `4×6`, `5×6`, `6×6`, `7×5`, `8×5`, `9×4`.

**Testspezifikation**
- **TRIO-TEST-004A**: Feld-Histogramm entspricht exakt der Sollverteilung.
- **TRIO-TEST-004B**: Über mehrere Partien bleibt Histogramm konstant, Positionen variieren.

### TRIO-SPEC-005 – Board-Verteilung an Clients
Das zufällig erzeugte 7×7-Board MUSS an alle Spielenden verteilt werden.

**Testspezifikation**
- **TRIO-TEST-005A**: Alle Clients erhalten dasselbe Board (identischer Hash).
- **TRIO-TEST-005B**: Runde darf ohne abgeschlossenes Board-Sync nicht starten.

### TRIO-SPEC-006 – Chipdeck-Zusammensetzung
Das Chipdeck MUSS exakt die Menge `{1..50}` enthalten, jeden Wert genau einmal, ohne Wiederholungen.

**Testspezifikation**
- **TRIO-TEST-006A**: Deck enthält genau 50 Einträge.
- **TRIO-TEST-006B**: Sortiertes Deck entspricht exakt `[1,2,...,50]`.
- **TRIO-TEST-006C**: Doppelte oder fehlende Werte MUSS System ablehnen.

### TRIO-SPEC-007 – Spieleranzahl
Eine Partie MUSS 2 bis 6 Spielende unterstützen. Maximal 6 Spielende dürfen gleichzeitig an einer Partie teilnehmen.

**Testspezifikation**
- **TRIO-TEST-007A**: Join der Spielenden 2 bis 6 wird akzeptiert.
- **TRIO-TEST-007B**: Join eines siebten Spielenden wird abgewiesen.
- **TRIO-TEST-007C**: Start einer Mehrspielerpartie mit weniger als 2 Spielenden wird abgewiesen.

### TRIO-SPEC-008 – Späterer 1-Spieler-Modus
Ein 1-Spieler-Modus ist fachlich vorgesehen, gehört aber NICHT zum Funktionsumfang der ersten Version.

**Testspezifikation**
- **TRIO-TEST-008A**: In Version 1 ist der 1-Spieler-Modus nicht auswählbar.

## 3.2 Rundenprotokoll und Synchronisation

### TRIO-SPEC-010 – Chipauswahl pro Runde
Pro Runde MUSS der Master genau einen Chipwert aus der aktuellen Chip-Warteschlange wählen. Zu Partiebeginn ist diese Warteschlange eine zufällige Permutation des Decks `{1..50}`. Ein nach hinten angestellter Chip MUSS am Ende der Warteschlange erneut erscheinen.

**Testspezifikation**
- **TRIO-TEST-010A**: Gewählter Chip ist der vorderste Wert der aktuellen Warteschlange.
- **TRIO-TEST-010B**: Verwendete oder dauerhaft abgelegte Chipwerte erscheinen nicht erneut.
- **TRIO-TEST-010C**: Nach hinten angestellter Chipwert erscheint am Ende der Warteschlange.

### TRIO-SPEC-009 – Digitalisierungsregel: Bereitschaft vor nächster Runde
Vor jeder Runde DARF der nächste Chip bereits verdeckt vorbereitet und an die Clients verteilt werden. Das gilt zu Partiebeginn und nach Abschluss jeder vorherigen Runde, solange die Chip-Warteschlange nicht leer ist. Das Startsignal für die neue Runde DARF erst gesendet werden, wenn alle aktiven Spielenden auf „nächstes Zahlenplättchen“ geklickt und damit ihre Bereitschaft bestätigt haben. Deaktivierte Spielende werden für diese Runde bei der Bereitschaft nicht mitgezählt.

**Testspezifikation**
- **TRIO-TEST-009A**: Ohne Bereitschaft aller aktiven Spielenden wird kein Startsignal gesendet.
- **TRIO-TEST-009B**: Mit Bereitschaft aller aktiven Spielenden wird nach verdeckter Chipverteilung das Startsignal gesendet.
- **TRIO-TEST-009C**: Deaktivierte Spielende blockieren die Bereitschaftsprüfung nicht.
- **TRIO-TEST-009D**: Verdeckte Chipauswahl und -verteilung dürfen vor Abschluss der Bereitschaftsprüfung stattfinden.

### TRIO-SPEC-011 – Chipverteilung vor Start
Der gewählte Chipwert MUSS zu Partiebeginn oder direkt nach Abschluss der vorherigen Runde verdeckt an alle verbundenen Clients verteilt werden, bevor die Bereitschaftsprüfung für die neue Runde abgeschlossen ist. Er darf erst nach Startsignal sichtbar werden.

**Testspezifikation**
- **TRIO-TEST-011A**: Jeder verbundene Client erhält ein `chipDistributed`-Event mit identischer Chipzahl.
- **TRIO-TEST-011B**: Vor Startsignal bleibt die Chipzahl für Spielende verborgen.
- **TRIO-TEST-011C**: `chipDistributed` wird vor Abschluss der Bereitschaftsprüfung gesendet.

### TRIO-SPEC-012 – Startsignal nach ACK
Der Master DARF das Startsignal erst senden, nachdem alle für die nächste Runde berücksichtigten Clients den verdeckten Chipempfang bestätigt haben und alle aktiven Spielenden bereit sind.

**Testspezifikation**
- **TRIO-TEST-012A**: Ohne vollständige ACK-Menge wird kein Startsignal gesendet.
- **TRIO-TEST-012B**: Mit vollständiger ACK-Menge und vollständiger Bereitschaft wird genau ein Startsignal gesendet.
- **TRIO-TEST-012C**: Mit vollständiger ACK-Menge, aber fehlender Bereitschaft wird kein Startsignal gesendet.

### TRIO-SPEC-013 – Sichtbarkeit nach Startsignal
Erst nach Startsignal DARF die Chipzahl den aktiven Spielenden sichtbar angezeigt werden. Jeder Client MUSS lokal den Empfangszeitstempel dieses Startsignals speichern.

**Testspezifikation**
- **TRIO-TEST-013A**: Vor Startsignal bleibt Chipanzeige verborgen.
- **TRIO-TEST-013B**: Nach Startsignal ist Chipanzeige auf allen Clients sichtbar.
- **TRIO-TEST-013C**: Jeder aktive Client speichert einen Startsignal-Empfangszeitstempel.

### TRIO-SPEC-014 – Rundenzeit
Pro Runde MUSS ein konfigurierbarer Timer laufen. Zulässige Werte sind 60.000, 90.000, 120.000 und 180.000 Millisekunden. Die Werte MÜSSEN in der Oberfläche als „eine Minute“, „eineinhalb Minuten“, „zwei Minuten“ und „drei Minuten“ dargestellt werden. Die Timer-Konfiguration DARF während einer laufenden Partie geändert werden; eine Änderung wird für die nächste noch nicht gestartete Runde wirksam. Diese Zeitregel ist eine Digitalisierungsregel.

**Testspezifikation**
- **TRIO-TEST-014A**: Timer akzeptiert ausschließlich 60.000, 90.000, 120.000 und 180.000 ms.
- **TRIO-TEST-014B**: Timeroptionen werden als „eine Minute“, „eineinhalb Minuten“, „zwei Minuten“ und „drei Minuten“ angezeigt.
- **TRIO-TEST-014C**: Timeout-Event feuert bei Ablauf der für die Runde aktiven Timer-Konfiguration.
- **TRIO-TEST-014D**: Änderung während einer laufenden Partie wird für die nächste noch nicht gestartete Runde übernommen.
- **TRIO-TEST-014E**: Änderung während einer bereits gestarteten Runde verändert den laufenden Rundentimer nicht.

### TRIO-SPEC-015 – Digitalisierungsregel: Timeout-Fortsetzung
Nach Timeout MUSS die Runde ohne Punktevergabe beendet werden, falls bis dahin keine korrekte Meldung gewertet wurde. Danach wird gemäß `TRIO-SPEC-011` der nächste Chip verdeckt vorbereitet und verteilt; das Startsignal bleibt bis zur Bereitschaft aller aktiven Spielenden gesperrt.

**Testspezifikation**
- **TRIO-TEST-015A**: Timeout beendet die Lösungsphase ohne Punktevergabe.
- **TRIO-TEST-015B**: Nach Timeout wird der nächste Chip verdeckt vorbereitet, bevor alle aktiven Spielenden ihre Bereitschaft bestätigt haben.
- **TRIO-TEST-015C**: Nach Timeout erscheint für aktive Spielende die Bereitschaftsaktion für das nächste Zahlenplättchen.

### TRIO-SPEC-016 – Digitalisierungsregel: Ungelöste Chips
Nach Timeout MUSS der Master prüfen, ob der aktive Chip auf dem aktuellen Board grundsätzlich lösbar ist. Wenn keine Lösung existiert, wird der Chip dauerhaft abgelegt und niemandem zugeordnet. Wenn mindestens eine Lösung existiert, wird der Chip beim ersten ungelösten Auftreten ans Ende der Chip-Warteschlange gestellt, sodass es zum Schluss noch einen Versuch gibt. Bleibt dieser letzte Versuch ebenfalls ungelöst, wird der Chip dauerhaft abgelegt und niemandem zugeordnet.

**Testspezifikation**
- **TRIO-TEST-016A**: Unlösbarer Chip wird dauerhaft abgelegt und verändert keinen Score.
- **TRIO-TEST-016B**: Lösbarer, erstmals ungelöster Chip wird ans Ende der Warteschlange gestellt.
- **TRIO-TEST-016C**: Lösbarer Chip wird nach einem zweiten ungelösten Versuch dauerhaft abgelegt.
- **TRIO-TEST-016D**: Nach Ablegen oder Nach-hinten-Stellen beginnt keine Folgerunde ohne erneute Bereitschaft aller aktiven Spielenden.

### TRIO-SPEC-017 – Digitalisierungsregel: AFK-Deaktivierung für eine Runde
In Partien mit mehr als 2 Spielenden können die übrigen aktiven Spielenden eine AFK-Person gemeinschaftlich für genau die nächste Runde deaktivieren. Dafür MÜSSEN alle nicht betroffenen aktiven Spielenden zustimmen. Deaktivierte Spielende sehen die Runde, sind aber nicht meldungsberechtigt und werden für Bereitschafts- und Chip-ACKs dieser Runde nicht berücksichtigt. Nach der Runde werden sie automatisch wieder aktiv.

**Testspezifikation**
- **TRIO-TEST-017A**: In einer Partie mit 3 oder mehr Spielenden deaktivieren einstimmige Stimmen aller übrigen aktiven Spielenden die betroffene Person für die nächste Runde.
- **TRIO-TEST-017B**: Ohne Einstimmigkeit bleibt die betroffene Person aktiv.
- **TRIO-TEST-017C**: In einer 2-Spieler-Partie ist AFK-Deaktivierung nicht verfügbar.
- **TRIO-TEST-017D**: Deaktivierte Spielende können in dieser Runde keine Meldung abgeben.
- **TRIO-TEST-017E**: Nach der Runde wird die deaktivierte Person automatisch wieder aktiv.

## 3.3 Eingabe und Lösungsvalidierung

### TRIO-SPEC-020 – Eingabemethoden
Spielende MÜSSEN drei Ziffern in gerader Linie per Dragover oder durch separates Anklicken auswählen können.

**Testspezifikation**
- **TRIO-TEST-020A**: Dragover-Auswahl von 3 Feldern erzeugt gültigen Selektions-Event.
- **TRIO-TEST-020B**: Klick-Klick-Klick-Auswahl von 3 Feldern erzeugt gleichwertigen Selektions-Event.

### TRIO-SPEC-021 – Genau drei Positionen
Eine Meldung MUSS genau drei Feldpositionen enthalten.

**Testspezifikation**
- **TRIO-TEST-021A**: 3 Positionen -> validierbar.
- **TRIO-TEST-021B**: <3 oder >3 Positionen -> sofort invalid.

### TRIO-SPEC-022 – Linienbedingung
Die drei gewählten Positionen MÜSSEN horizontal, vertikal oder diagonal unmittelbar aufeinanderfolgend auf einer Geraden liegen.

**Testspezifikation**
- **TRIO-TEST-022A**: Drei unmittelbar benachbarte Felder in Zeile/Spalte/Diagonale werden akzeptiert.
- **TRIO-TEST-022B**: V- oder L-Form wird abgewiesen.
- **TRIO-TEST-022C**: Drei Felder auf derselben Geraden mit Lücken werden abgewiesen.

### TRIO-SPEC-023 – Erlaubte Rechenformen
Aus den drei Ziffern MUSS mindestens eine gültige Rechnung `A*B+C` oder `A*B-C` ableitbar sein. Jede der drei gewählten Ziffern darf in der Addition/Subtraktion stehen; die beiden übrigen Ziffern bilden den Multiplikationsteil.

**Testspezifikation**
- **TRIO-TEST-023A**: Beispielhafte gültige Kombination wird erkannt.
- **TRIO-TEST-023B**: Keine passende Form -> Meldung ungültig.
- **TRIO-TEST-023C**: Alle Permutationen der drei gewählten Ziffern werden für `A*B+C` und `A*B-C` geprüft.

### TRIO-SPEC-024 – Priorität Punkt vor Strich
Bei der Auswertung MUSS Standardpriorität gelten (Multiplikation vor Addition/Subtraktion).

**Testspezifikation**
- **TRIO-TEST-024A**: `2*3+4` ergibt 10.
- **TRIO-TEST-024B**: `2*3-4` ergibt 2.

### TRIO-SPEC-025 – Zielwerttreffer
Eine Meldung ist nur dann korrekt, wenn mindestens eine erlaubte Rechnung exakt den aktiven Chipwert ergibt.

**Testspezifikation**
- **TRIO-TEST-025A**: Exakter Treffer -> valid.
- **TRIO-TEST-025B**: Kein exakter Treffer -> invalid.

## 3.4 Feedback, Penalty und Meldelogik

### TRIO-SPEC-030 – Falsche Meldung: Buzzer + Markierung
Falls keine gültige Rechnung für die gewählten Ziffern existiert, MUSS auf allen Geräten ein Buzzersignal ertönen und die falsch gewählten Ziffern MÜSSEN markiert werden. Akustisches und visuelles Feedback sind Digitalisierungsregeln.

**Testspezifikation**
- **TRIO-TEST-030A**: Invalid-Meldung löst globales Buzzer-Event aus.
- **TRIO-TEST-030B**: Die drei gewählten Felder erhalten einen Fehler-Markierungszustand.

### TRIO-SPEC-031 – Falsche Meldung: Penalty
Bei falscher Meldung MUSS die meldende Person für 5.000 Millisekunden für neue Meldungen gesperrt werden. Diese Penalty ist eine Digitalisierungsregel.

**Testspezifikation**
- **TRIO-TEST-031A**: Während 5.000 ms werden Meldungen dieser Person abgewiesen.
- **TRIO-TEST-031B**: Nach Ablauf der 5.000 ms sind Meldungen wieder zulässig.

### TRIO-SPEC-032 – Penalty-Visualisierung
Die Penalty MUSS über einen ablaufenden Fortschrittsbalken über dem Namen der gesperrten Person angezeigt werden.

**Testspezifikation**
- **TRIO-TEST-032A**: Bei Penalty-Start erscheint Fortschrittsbalken am betroffenen Namen.
- **TRIO-TEST-032B**: Balken läuft bis 0 innerhalb von 5.000 ms.

### TRIO-SPEC-033 – Korrekte Meldung: Jingle + Chipanimation
Bei korrekter Meldung MUSS ein Belohnungsjingle abgespielt werden und der Chip MUSS visuell in den Zähler unter dem Spielernamen „fliegen“. Jingle und Animation sind Digitalisierungsregeln.

**Testspezifikation**
- **TRIO-TEST-033A**: Korrekte Meldung erzeugt globales Jingle-Event.
- **TRIO-TEST-033B**: UI-Animation verschiebt Chip in den korrekten Player-Scorebereich.

### TRIO-SPEC-034 – Digitalisierungsregel: Zeitbasis und Gleichstand
Jeder Client MUSS bei korrekter Meldung die lokal gemessene Lösungsdauer melden: Empfangszeitstempel des Startsignals bis Zeitstempel des korrekten Lösens. Der Chip geht an die schnellste gemeldete korrekte Lösung. Die Granularität für die Wertung beträgt eine Zehntelsekunde; haben mehrere Spielende dieselbe Lösungsdauer auf Zehntelsekunden gerundet, erhalten alle diese Spielenden den Punkt.

**Testspezifikation**
- **TRIO-TEST-034A**: Client meldet Startsignal-Empfangszeitstempel und Lösungszeitstempel.
- **TRIO-TEST-034B**: Schnellere Lösungsdauer gewinnt auch dann, wenn die Nachricht später beim Master eingeht.
- **TRIO-TEST-034C**: Zwei korrekte Meldungen mit gleicher Lösungsdauer auf 0,1 Sekunden gerundet -> beide +1.
- **TRIO-TEST-034D**: Unterschiedliche Lösungsdauer auf 0,1 Sekunden gerundet -> nur die schnellste Person bzw. Gruppe erhält den Punkt.

### TRIO-SPEC-035 – Digitalisierungsregel: Wertungsfenster nach erster korrekter Meldung
Sobald der Master die erste korrekte Meldung für eine Runde erhält, MUSS er ein `roundClosing`-Event senden und ein technisches Wertungsfenster von 1.000 Millisekunden öffnen. Clients dürfen nach Empfang von `roundClosing` keine neuen Auswahlen mehr beginnen. Korrekte Meldungen, die innerhalb des Wertungsfensters beim Master eintreffen und deren lokaler Lösungszeitstempel vor dem lokalen `roundClosing`-Empfang liegt, MÜSSEN in die Wertung nach `TRIO-SPEC-034` einbezogen werden. Nach Ablauf des Wertungsfensters wird die Runde final gewertet.

**Testspezifikation**
- **TRIO-TEST-035A**: Erste korrekte Meldung löst `roundClosing` aus.
- **TRIO-TEST-035B**: Korrekte Meldung innerhalb des 1.000-ms-Wertungsfensters wird berücksichtigt.
- **TRIO-TEST-035C**: Korrekte Meldung nach Ablauf des Wertungsfensters wird abgewiesen.
- **TRIO-TEST-035D**: Client blockiert neue Auswahlen nach lokalem Empfang von `roundClosing`.

## 3.5 Wertung und Spielende

### TRIO-SPEC-040 – Punktevergabe
Bei korrekter Meldung erhalten alle nach `TRIO-SPEC-034` und `TRIO-SPEC-035` siegreichen Spielenden den aktiven Chip als Punkt.

**Testspezifikation**
- **TRIO-TEST-040A**: Siegerische korrekte Meldung erhöht den Chipzähler der Person um 1.
- **TRIO-TEST-040B**: Bei Zehntelsekunden-Gleichstand erhöht sich der Chipzähler aller gleich schnellen siegreichen Personen um 1.

### TRIO-SPEC-041 – Spielende
Das Spiel endet, sobald die Chip-Warteschlange leer ist und alle 50 Chips entweder vergeben oder dauerhaft abgelegt wurden.

**Testspezifikation**
- **TRIO-TEST-041A**: Nach Vergabe oder dauerhaftem Ablegen des letzten Chips ist `gameFinished=true`.
- **TRIO-TEST-041B**: Ein nach hinten angestellter Chip verhindert das Spielende bis zu seinem letzten Versuch.

### TRIO-SPEC-042 – Siegerbestimmung
Alle Spielenden mit maximalem Chipstand MÜSSEN als Sieger ausgewiesen werden (Mehrfachsieg bei Gleichstand).

**Testspezifikation**
- **TRIO-TEST-042A**: Eindeutiges Maximum -> genau ein Sieger.
- **TRIO-TEST-042B**: Geteiltes Maximum -> alle Top-Spielenden Sieger.

### TRIO-SPEC-043 – Digitalisierungsregel: Ergebnisanzeige
Nach der letzten Runde MUSS eine Ergebnisliste angezeigt werden. Die Liste MUSS alle Spielenden mit der jeweils erreichten Chipanzahl enthalten und absteigend nach Chipanzahl sortiert sein. Die Gewinnerin, der Gewinner oder bei Gleichstand alle gemeinschaftlich Gewinnenden MÜSSEN grafisch hervorgehoben werden.

**Testspezifikation**
- **TRIO-TEST-043A**: Nach Spielende wird eine Ergebnisliste angezeigt.
- **TRIO-TEST-043B**: Ergebnisliste enthält alle Spielenden mit Chipanzahl.
- **TRIO-TEST-043C**: Ergebnisliste ist absteigend nach Chipanzahl sortiert.
- **TRIO-TEST-043D**: Eindeutige Gewinnerin oder eindeutiger Gewinner wird grafisch hervorgehoben.
- **TRIO-TEST-043E**: Bei Gleichstand werden alle gemeinschaftlich Gewinnenden grafisch hervorgehoben.

## 3.6 Multiplayer-/Transport-Spezifikation

### TRIO-SPEC-050 – Session-ID
Eine Partie MUSS über einen Session-Code identifiziert sein.

**Testspezifikation**
- **TRIO-TEST-050A**: Join ohne gültigen Session-Code schlägt fehl.

### TRIO-SPEC-051 – Konsistenz
Alle Clients MÜSSEN denselben autoritativen Spielzustand (Board, aktiver Chip, Scores, Timerstatus, Timer-Konfiguration) führen.

**Testspezifikation**
- **TRIO-TEST-051A**: Nach jedem Event sind State-Hashes auf allen Clients identisch.

### TRIO-SPEC-052 – Digitalisierungsregel: Zeitstempel und Wertungsgranularität
Für Runde und Lösungsmessung MÜSSEN Client-Zeitstempel mit Millisekundenauflösung verwendet werden. Für die Punktewertung wird die gemessene Lösungsdauer auf Zehntelsekunden granular verglichen.

**Testspezifikation**
- **TRIO-TEST-052A**: Ereignisse tragen Millisekunden-Zeitstempel.
- **TRIO-TEST-052B**: Wertung vergleicht Lösungsdauern auf 0,1 Sekunden.
- **TRIO-TEST-052C**: Bei gleicher Zehntelsekunden-Dauer erhalten alle gleich schnellen Spielenden den Punkt.

## 4. Verbleibende offene Punkte

Aktuell keine offenen inhaltlichen Regelfragen.

## 5. Testmatrix (Schnellübersicht)

- Setup/Board/Deck/Spielende: `TRIO-TEST-001A..008A`
- Rundenprotokoll/Timer/AFK: `TRIO-TEST-009A..017E`
- Eingabe/Validierung: `TRIO-TEST-020A..025B` inkl. `TRIO-TEST-022C` und `TRIO-TEST-023C`
- Feedback/Penalty/Wertungsgleichstand: `TRIO-TEST-030A..035D`
- Wertung/Spielende: `TRIO-TEST-040A..043E`
- Multiplayer/Timingbasis: `TRIO-TEST-050A..052C`
