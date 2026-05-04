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
- **Linienbedingung**: Drei verwendete Feldziffern liegen horizontal, vertikal oder diagonal auf einer Geraden.
- **Runde**: Chip verteilen, Startsignal, Lösungsphase, Prüfung, Wertung/Fortsetzung.
- **Penalty**: 5-sekündige Meldesperre nach falscher Meldung.

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
Das Chipdeck MUSS exakt die Menge `{1..49}` enthalten, jeden Wert genau einmal, ohne Wiederholungen.

**Testspezifikation**
- **TRIO-TEST-006A**: Deck enthält genau 49 Einträge.
- **TRIO-TEST-006B**: Sortiertes Deck entspricht exakt `[1,2,...,49]`.
- **TRIO-TEST-006C**: Doppelte oder fehlende Werte MUSS System ablehnen.

## 3.2 Rundenprotokoll und Synchronisation

### TRIO-SPEC-010 – Chipauswahl pro Runde
Pro Runde MUSS der Master genau einen noch nicht verwendeten Chipwert aus dem Deck zufällig wählen.

**Testspezifikation**
- **TRIO-TEST-010A**: Gewählter Chip ist im Restdeck vorhanden.
- **TRIO-TEST-010B**: Bereits verwendeter Chipwert darf nicht erneut gezogen werden.

### TRIO-SPEC-011 – Chipverteilung vor Start
Der gewählte Chipwert MUSS zunächst an alle Clients verteilt werden.

**Testspezifikation**
- **TRIO-TEST-011A**: Jeder Client erhält ein `chipDistributed`-Event mit identischer Chipzahl.

### TRIO-SPEC-012 – Startsignal nach ACK
Der Master DARF das Startsignal erst senden, nachdem alle Clients den Chipempfang bestätigt haben.

**Testspezifikation**
- **TRIO-TEST-012A**: Ohne vollständige ACK-Menge wird kein Startsignal gesendet.
- **TRIO-TEST-012B**: Mit vollständiger ACK-Menge wird genau ein Startsignal gesendet.

### TRIO-SPEC-013 – Sichtbarkeit nach Startsignal
Erst nach Startsignal DARF die Chipzahl den Spielenden sichtbar angezeigt werden.

**Testspezifikation**
- **TRIO-TEST-013A**: Vor Startsignal bleibt Chipanzeige verborgen.
- **TRIO-TEST-013B**: Nach Startsignal ist Chipanzeige auf allen Clients sichtbar.

### TRIO-SPEC-014 – Rundenzeit
Pro Runde MUSS ein Timer von 60.000 Millisekunden laufen.

**Testspezifikation**
- **TRIO-TEST-014A**: Timer startet bei 60.000 ms.
- **TRIO-TEST-014B**: Timeout-Event feuert bei Ablauf der 60.000 ms.

### TRIO-SPEC-015 – Timeout-Button
Nach Timeout MUSS ein Button mit exakt folgendem Text erscheinen:
`Wenn Sie keine Lösung gefunden haben, können Sie hier die nächste Runde starten`

**Testspezifikation**
- **TRIO-TEST-015A**: Button wird erst nach Timeout sichtbar.
- **TRIO-TEST-015B**: Buttontext entspricht exakt dem Solltext.

### TRIO-SPEC-016 – Timeout-Fortsetzung
Bei Klick auf den Timeout-Button wird der aktuelle Chip niemandem zugeordnet und die nächste Runde mit dem nächsten Chip gestartet.

**Testspezifikation**
- **TRIO-TEST-016A**: Nach Timeout-Fortsetzung bleibt Score aller Spielenden unverändert.
- **TRIO-TEST-016B**: Folgerunde nutzt neuen, ungenutzten Chipwert.

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
Die drei gewählten Positionen MÜSSEN horizontal, vertikal oder diagonal auf einer Geraden liegen.

**Testspezifikation**
- **TRIO-TEST-022A**: Zeile/Spalte/Diagonale werden akzeptiert.
- **TRIO-TEST-022B**: V- oder L-Form wird abgewiesen.

### TRIO-SPEC-023 – Erlaubte Rechenformen
Aus den drei Ziffern MUSS mindestens eine gültige Rechnung `A*B+C` oder `A*B-C` ableitbar sein.

**Testspezifikation**
- **TRIO-TEST-023A**: Beispielhafte gültige Kombination wird erkannt.
- **TRIO-TEST-023B**: Keine passende Form -> Meldung ungültig.

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
Falls keine gültige Rechnung für die gewählten Ziffern existiert, MUSS auf allen Geräten ein Buzzersignal ertönen und die falsch gewählten Ziffern MÜSSEN markiert werden.

**Testspezifikation**
- **TRIO-TEST-030A**: Invalid-Meldung löst globales Buzzer-Event aus.
- **TRIO-TEST-030B**: Die drei gewählten Felder erhalten einen Fehler-Markierungszustand.

### TRIO-SPEC-031 – Falsche Meldung: Penalty
Bei falscher Meldung MUSS die meldende Person für 5.000 Millisekunden für neue Meldungen gesperrt werden.

**Testspezifikation**
- **TRIO-TEST-031A**: Während 5.000 ms werden Meldungen dieser Person abgewiesen.
- **TRIO-TEST-031B**: Nach Ablauf der 5.000 ms sind Meldungen wieder zulässig.

### TRIO-SPEC-032 – Penalty-Visualisierung
Die Penalty MUSS über einen ablaufenden Fortschrittsbalken über dem Namen der gesperrten Person angezeigt werden.

**Testspezifikation**
- **TRIO-TEST-032A**: Bei Penalty-Start erscheint Fortschrittsbalken am betroffenen Namen.
- **TRIO-TEST-032B**: Balken läuft bis 0 innerhalb von 5.000 ms.

### TRIO-SPEC-033 – Korrekte Meldung: Jingle + Chipanimation
Bei korrekter Meldung MUSS ein Belohnungsjingle abgespielt werden und der Chip MUSS visuell in den Zähler unter dem Spielernamen „fliegen“.

**Testspezifikation**
- **TRIO-TEST-033A**: Korrekte Meldung erzeugt globales Jingle-Event.
- **TRIO-TEST-033B**: UI-Animation verschiebt Chip in den korrekten Player-Scorebereich.

### TRIO-SPEC-034 – Simultane korrekte Meldungen
Wenn technisch gleichzeitig (gleiche Millisekunde) mehrere korrekte Meldungen eintreffen, MÜSSEN alle diese Spielenden den Punkt erhalten.

**Testspezifikation**
- **TRIO-TEST-034A**: Zwei korrekte Meldungen mit identischem Millisekunden-Zeitstempel -> beide +1.
- **TRIO-TEST-034B**: Unterschiedliche Millisekunde -> normale Reihenfolgeverarbeitung (kein simultan-Bonus).

## 3.5 Wertung und Spielende

### TRIO-SPEC-040 – Punktevergabe
Bei korrekter Meldung erhält die meldende Person den aktiven Chip als Punkt.

**Testspezifikation**
- **TRIO-TEST-040A**: Korrekte Meldung erhöht den Chipzähler der Person um 1.

### TRIO-SPEC-041 – Spielende
Das Spiel endet, sobald alle 49 Chips verwendet wurden.

**Testspezifikation**
- **TRIO-TEST-041A**: Nach letzter Runde Zustand `gameFinished=true`.

### TRIO-SPEC-042 – Siegerbestimmung
Alle Spielenden mit maximalem Chipstand MÜSSEN als Sieger ausgewiesen werden (Mehrfachsieg bei Gleichstand).

**Testspezifikation**
- **TRIO-TEST-042A**: Eindeutiges Maximum -> genau ein Sieger.
- **TRIO-TEST-042B**: Geteiltes Maximum -> alle Top-Spielenden Sieger.

## 3.6 Multiplayer-/Transport-Spezifikation

### TRIO-SPEC-050 – Session-ID
Eine Partie MUSS über einen Session-Code identifiziert sein.

**Testspezifikation**
- **TRIO-TEST-050A**: Join ohne gültigen Session-Code schlägt fehl.

### TRIO-SPEC-051 – Konsistenz
Alle Clients MÜSSEN denselben autoritativen Spielzustand (Board, aktiver Chip, Scores, Timerstatus) führen.

**Testspezifikation**
- **TRIO-TEST-051A**: Nach jedem Event sind State-Hashes auf allen Clients identisch.

### TRIO-SPEC-052 – Millisekundentimer als Zeitbasis
Für Runde und Gleichzeitigkeit MUSS ein Millisekundentimer verwendet werden.

**Testspezifikation**
- **TRIO-TEST-052A**: Ereignisse tragen Millisekunden-Zeitstempel.
- **TRIO-TEST-052B**: Gleichzeitigkeit wird ausschließlich über Gleichheit des Millisekundenwertes bestimmt.

## 4. Verbleibende offene Punkte

Aktuell keine offenen inhaltlichen Regelfragen.

## 5. Testmatrix (Schnellübersicht)

- Setup/Board/Deck: `TRIO-TEST-001A..006C`
- Rundenprotokoll/Timer: `TRIO-TEST-010A..016B`
- Eingabe/Validierung: `TRIO-TEST-020A..025B`
- Feedback/Penalty/Simultanität: `TRIO-TEST-030A..034B`
- Wertung/Spielende: `TRIO-TEST-040A..042B`
- Multiplayer/Timingbasis: `TRIO-TEST-050A..052B`
