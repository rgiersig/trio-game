# Architektur

## Festgelegte Zielarchitektur (Trio, anonym, Web + Android)

Dieses Kapitel beschreibt die verbindlichen Architekturentscheidungen für die erste Version.

### 1) Verbindliche Festlegungen

- Es wird **MQTT + Session-Service** verwendet.
- **MQTT-Broker und Session-Service laufen zentral auf dem Server**.
- Der Server übernimmt darüber hinaus nur das Ausliefern einer **statischen Webseite** (Browserversion).
- Keine Benutzeraccounts, keine klassische Lobby; Zusammenfinden erfolgt über **Session-Code**.

### 2) Komponenten

- **Statischer Webserver**
  - Liefert HTML/CSS/JS der Browserversion aus.
  - Keine serverseitige Spiel-UI-Logik.
- **Session-Service (leichtgewichtig)**
  - Erzeugt Session-Codes.
  - Verwaltet Join/Close/TTL.
  - Liefert Session-Metadaten für MQTT-Zugriff.
- **MQTT-Broker**
  - Realtime-Kommunikation für Spielzustände/Ereignisse.
  - Topic-Namespace je Session, z. B. `trio/<code>/...`.

### 3) Session- und Spielfluss

1. Host erzeugt über den Session-Service eine Session und erhält einen Code.
2. Spieler teilen den Code extern (z. B. Chat) und joinen mit diesem Code.
3. Alle Clients verbinden sich mit dem MQTT-Broker im jeweiligen Session-Topic.
4. Der Host (Master) initialisiert Setup und veröffentlicht autoritative Zustandsupdates.
5. Clients senden Aktionen/Meldungen; Master bewertet und publisht den gültigen Zustand.

### 4) Sicherheit der MQTT-Kommunikation (Festlegung)

Die MQTT-Verbindung wird durch ein Zertifikat abgesichert, das in den Clients mitgeführt wird:

- **Android-App:** Zertifikat wird in der App ausgeliefert und zur Verbindungsprüfung genutzt.
- **Webversion:** Zertifikatsmaterial/Trust-Anker wird mit dem Web-Bundle ausgeliefert.
- Der Broker akzeptiert nur Verbindungen/Nachrichten, die zur erwarteten Zertifikatsprüfung passen.

Ziel dieser Maßnahme ist, das Einspeisen falscher Nachrichten durch nicht autorisierte Clients zu erschweren und damit triviale DoS-/Spam-Angriffe zu reduzieren.

> Hinweis: In der Webversion sind eingebettete Secrets grundsätzlich extrahierbar. Die Maßnahme erhöht die Hürde, ersetzt aber kein vollständiges Missbrauchsschutz-Konzept.

### 5) Ergänzende Schutzmaßnahmen (empfohlen)

- TLS/WSS durchgehend aktivieren.
- Strikte Topic-ACLs pro Session (`trio/<code>/...`).
- Rate-Limits auf Broker/Ingress-Ebene.
- Kurze Session-TTL und Aufräumlogik.
- Kollisionarme Codes (mindestens 8 Zeichen, serverseitiges generate-and-check).

### 6) Implementierung der Spieleapp mit maximalem Code-Share

Für maximalen Wiederverwendungsgrad zwischen Browserversion und Android-App wird ein **Flutter-Monorepo** festgelegt.

#### 6.1 Ziel

- Eine gemeinsame Codebasis für UI-Logik, State-Management, Session-Flow und MQTT-Kommunikation.
- Plattformabweichungen nur in kleinen Adapter-Schichten.

#### 6.2 Architekturprinzip für den Client

- **Shared Domain-Layer** (spielnahe Modelle und Regeln)
- **Shared Application-Layer** (Use Cases wie Session erstellen/joinen, Aktionen senden)
- **Shared Infrastructure-Layer** (Session-API + MQTT-Adapter)
- **Plattform-Adapter** nur für Android/Web-spezifische TLS-/Runtime-Themen

#### 6.3 Vorgesehene Repository-Struktur

```text
trio-game/
├─ architektur.md
├─ apps/
│  └─ trio_app/                    # Flutter App (Web + Android Targets)
├─ packages/
│  ├─ app_core/                    # Domain + Application Use Cases
│  ├─ mqtt_adapter/                # MQTT Topic/Publish/Subscribe/Reconnect
│  ├─ session_api_adapter/         # HTTP API für Session-Service
│  ├─ shared_models/               # Gemeinsame DTOs/Serialisierung
│  └─ platform_security/           # Plattform-spezifische Security-Adapter
└─ docs/
   └─ client-architecture.md       # Detaildoku zur Client-Architektur
```

#### 6.4 Verantwortlichkeiten der Pakete

- `packages/shared_models`
  - JSON-serialisierbare Modelle für Session, Events und Snapshots.
- `packages/app_core`
  - Spielfluss-Use-Cases und State-Machine.
- `packages/session_api_adapter`
  - Aufrufe an `POST /session`, `/join`, `/close`.
- `packages/mqtt_adapter`
  - Verbindung, Subscription, Publish, QoS-Strategie, Reconnect.
- `packages/platform_security`
  - Android/Web-spezifische Einbindung von Zertifikats-/Trust-Konfiguration.
- `apps/trio_app`
  - Präsentationsschicht (Screens, Navigation, ViewModels/State).

#### 6.5 Plattformgrenzen bewusst halten

- Android und Web teilen denselben fachlichen Code.
- Unterschiede (z. B. Web-WSS-Umgebung vs. Android-Stack) werden nur über Adapter gekapselt.
- Zertifikatsbasierte Absicherung bleibt als Infrastrukturkonfiguration zentral dokumentiert.

### 7) Abgrenzung

Diese Architektur ist auf **anonyme, private Runden** mit geringem Serveraufwand optimiert. Für kompetitive Szenarien mit strikter Fairness wäre später ein serverautoritatives Regel-Backend erforderlich.


### 8) Regelwerk-Spezifikation

Die nachverfolgbare Regel-Spezifikation mit eindeutigen IDs liegt in `docs/trio-rules-spec.md`. Offene bzw. mehrdeutige Punkte sind dort explizit als `TRIO-OQ-*` markiert.
