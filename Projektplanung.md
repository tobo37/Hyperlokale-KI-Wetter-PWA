# Projektplan: Hyperlokale KI-Wetter PWA

## Technologie-Stack
- **Frontend**: Flutter (für PWA)
- **Backend**: FastAPI (Python)
- **Datenbank**: PostgreSQL
- **KI-Modell**: XGBoost
- **Datenquelle (Start)**: Open-Meteo API

## Epic 1: MVP - Funktionierende PWA mit Live-Wetterdaten
Ziel: Ein Nutzer kann die PWA auf seinem Handy oder im Browser öffnen und sieht das aktuelle Wetter und eine Vorhersage für einen festen Standort (z.B. Köln), basierend auf den Daten von Open-Meteo.

| User Story ID | Titel | Beschreibung | Akzeptanzkriterien |
| --- | --- | --- | --- |
| **PWA-1** | Backend-Proxy einrichten | Als Entwickler möchte ich einen FastAPI-Endpunkt `/api/v1/weather/cologne` erstellen, damit das Frontend Wetterdaten abrufen kann, ohne direkt mit Open-Meteo zu kommunizieren. | 1. Der Endpunkt ruft die Open-Meteo API für den Standort Köln auf. <br>2. Der Endpunkt gibt die erhaltenen JSON-Daten 1:1 an den Client weiter. <br>3. Es ist noch keine Datenbank involviert. |
| **PWA-2** | Grundstruktur der Flutter PWA | Als Entwickler möchte ich ein neues Flutter-Projekt erstellen, das als PWA kompiliert und im Browser aufgerufen werden kann. | 1. Das Projekt kann mit `flutter build web` gebaut werden. <br>2. Die kompilierte Web-App kann über einen lokalen Webserver aufgerufen werden. <br>3. Ein leeres Grund-Layout mit einem Titel (z.B. "Meine Wetter-App") ist sichtbar. |
| **PWA-3** | Aktuelles Wetter anzeigen | Als Nutzer möchte ich auf dem Startbildschirm die aktuelle Temperatur, das Wettersymbol (z.B. Sonne, Wolken) und die gefühlte Temperatur sehen, damit ich schnell über das jetzige Wetter informiert bin. | 1. Die App ruft beim Start den Endpunkt `/api/v1/weather/cologne` auf. <br>2. Die aktuelle Temperatur wird groß angezeigt. <br>3. Ein passendes Icon (aus einer Icon-Bibliothek) und die gefühlte Temperatur werden angezeigt. |
| **PWA-4** | Tägliche Vorhersage anzeigen | Als Nutzer möchte ich eine Liste der nächsten 7 Tage mit der jeweiligen Höchst-/Tiefsttemperatur und einem Wettersymbol sehen, damit ich die kommende Woche planen kann. | 1. Die Daten aus dem API-Aufruf werden genutzt, um eine 7-Tage-Vorhersage zu rendern. <br>2. Jeder Listeneintrag zeigt Wochentag, Icon, Höchst- und Tiefsttemperatur. |
| **PWA-5** | Stündliche Vorhersage anzeigen | Als Nutzer möchte ich eine horizontale Liste der nächsten 24 Stunden mit Temperatur und Wettersymbol sehen, damit ich den Tagesverlauf einschätzen kann. | 1. Die stündlichen Daten aus dem API-Aufruf werden in einer scrollbaren, horizontalen Ansicht dargestellt. <br>2. Jeder Eintrag zeigt die Uhrzeit, das Icon und die Temperatur. |

## Epic 2: Backend-Persistenz & Datensammlung
Ziel: Das Backend speichert die abgerufenen Wetterdaten systematisch in einer PostgreSQL-Datenbank, um eine historische Datenbasis für das zukünftige KI-Modell aufzubauen.

| User Story ID | Titel | Beschreibung | Akzeptanzkriterien |
| --- | --- | --- | --- |
| **DB-1** | PostgreSQL-Datenbank aufsetzen | Als Entwickler möchte ich eine PostgreSQL-Datenbank (via Docker) aufsetzen und ein Datenbankschema für die Wetterdaten definieren, um die Daten strukturiert speichern zu können. | 1. Ein Docker-Compose-File startet einen PostgreSQL-Container. <br>2. Ein Schema mit Tabellen (z.B. `weather_logs`) ist definiert (z.B. mit SQLAlchemy Models). <br>3. Das FastAPI-Backend kann sich erfolgreich mit der Datenbank verbinden. |
| **DB-2** | Daten-Collector-Service | Als Entwickler möchte ich einen Hintergrundprozess einrichten, der regelmäßig (z.B. stündlich) die Open-Meteo API abfragt und die Daten in der PostgreSQL-Datenbank speichert. | 1. Ein Skript (z.B. mittels `apscheduler` in FastAPI) läuft periodisch. <br>2. Die abgerufenen Daten werden in die Tabelle `weather_logs` geschrieben. <br>3. Das Frontend ist von dieser Änderung unberührt und funktioniert weiterhin. |
| **DB-3** | API von Datenbank speisen | Als Entwickler möchte ich den Endpunkt `/api/v1/weather/cologne` so anpassen, dass er die Daten aus der Datenbank liest anstatt bei jeder Anfrage live von Open-Meteo zu laden. | 1. Der Endpunkt holt den neuesten Datensatz aus der `weather_logs` Tabelle. <br>2. Die API-Antwort an das Frontend bleibt strukturell identisch. <br>3. Dies reduziert die Latenz und die Abhängigkeit von der externen API. |

## Epic 3: Integration der lokalen Wetterstation
Ziel: Die Daten der persönlichen Wetterstation (z.B. Ecowitt) werden vom Backend empfangen und ebenfalls in der Datenbank gespeichert, um hyperlokale Messwerte zu erhalten.

| User Story ID | Titel | Beschreibung | Akzeptanzkriterien |
| --- | --- | --- | --- |
| **HW-1** | Empfangs-Endpunkt für Wetterstation | Als Entwickler möchte ich einen neuen FastAPI-Endpunkt `/api/ingress/weatherstation` erstellen, der Daten im Format meiner Wetterstation (z.B. Ecowitt HTTP Post) empfangen kann. | 1. Der Endpunkt kann die POST-Requests der Station entgegennehmen und validieren. <br>2. Die empfangenen Daten werden zur Fehlersuche geloggt. |
| **HW-2** | Speichern der lokalen Daten | Als Entwickler möchte ich die vom `/api/ingress/weatherstation` Endpunkt empfangenen Daten in einer separaten Tabelle (z.B. `local_weather_logs`) in PostgreSQL speichern. | 1. Ein neues Datenbankschema für die lokalen Messwerte wird erstellt. <br>2. Die validierten Daten werden erfolgreich in diese neue Tabelle geschrieben. |
| **HW-3** | Anzeige der lokalen Daten (Optional) | Als Nutzer möchte ich in der App einen kleinen Bereich sehen, der die "Live"-Daten meiner eigenen Station anzeigt, damit ich den Unterschied zu den Modelldaten sehen kann. | 1. Ein neuer API-Endpunkt gibt die neuesten lokalen Daten zurück. <br>2. Das Frontend zeigt diese Daten in einem separaten UI-Element an (z.B. "Live von deiner Station: 15.2°C"). |

## Epic 4: Entwicklung & Integration des KI-Modells
Ziel: Ein mit XGBoost trainiertes Modell nutzt die gesammelten historischen Daten (öffentlich + lokal), um eine verbesserte, hyperlokale Vorhersage zu erstellen, welche die Vorhersage von Open-Meteo ersetzt.

| User Story ID | Titel | Beschreibung | Akzeptanzkriterien |
| --- | --- | --- | --- |
| **ML-1** | Daten-Exploration & Feature Engineering | Als Data Scientist möchte ich die gesammelten Daten in einem Jupyter Notebook analysieren, bereinigen und aufbereiten, um eine saubere Datengrundlage für das Training zu schaffen. | 1. Ein Notebook kann auf die PostgreSQL-Daten zugreifen. <br>2. Daten sind visualisiert und Ausreißer/fehlende Werte behandelt. <br>3. Relevante Features (z.B. gleitende Durchschnitte, Zeitmerkmale) sind erstellt. |
| **ML-2** | XGBoost-Modell trainieren & evaluieren | Als Data Scientist möchte ich ein XGBoost-Modell trainieren, das z.B. die Temperatur der nächsten 6 Stunden vorhersagt, und seine Genauigkeit bewerten. | 1. Das Modell wird mit den aufbereiteten Daten trainiert. <br>2. Die Genauigkeit wird anhand einer Testmenge gemessen (z.B. mit RMSE, MAE). <br>3. Das finale, trainierte Modell wird als Datei (z.B. `xgboost_model.json`) gespeichert. |
| **ML-3** | Vorhersage-Endpunkt erstellen | Als Entwickler möchte ich einen neuen Endpunkt `/api/v1/predict/cologne` erstellen, der das gespeicherte KI-Modell lädt und eine Vorhersage generiert. | 1. Der Endpunkt lädt das Modell und die neuesten Daten aus der Datenbank. <br>2. Er führt eine Vorhersage aus und gibt diese in einem definierten JSON-Format zurück. |
| **ML-4** | PWA auf KI-Vorhersage umstellen | Als Nutzer möchte ich, dass die Wettervorhersage in der App auf den Daten des neuen KI-Modells basiert, damit ich eine genauere, hyperlokale Prognose erhalte. | 1. Das Frontend ruft nun den `/api/v1/predict/cologne` Endpunkt anstelle des alten Wetter-Endpunkts auf. <br>2. Die App zeigt die vom Modell generierte Vorhersage an. Die UI bleibt dabei unverändert. |

