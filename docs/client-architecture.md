# Client-Architektur (Flutter, Web + Android)

Diese Doku konkretisiert die in `architektur.md` definierte Zielstruktur für maximale Code-Wiederverwendung.

## Grundsätze

- Shared-first: Fachlogik und Kommunikationslogik liegen in `packages/`.
- Thin App-Shell: `apps/trio_app` enthält primär UI und Navigation.
- Plattformspezifika nur in `platform_security` und kleinen Runtime-Adaptern.

## Paketübersicht

- `shared_models`: DTOs und Serialisierung.
- `app_core`: Use Cases und State-Management-naher Kern.
- `session_api_adapter`: HTTP-Anbindung Session-Service.
- `mqtt_adapter`: MQTT-Verbindung, Topics, QoS, Reconnect.
- `platform_security`: Zertifikats-/Trust-Konfiguration je Plattform.

## Nächste Schritte

1. Flutter workspace konfigurieren.
2. Gemeinsame Interfaces für Session API und MQTT Client definieren.
3. E2E-Flow: Session erstellen -> joinen -> Event senden -> Snapshot empfangen.
