# TravelTide
Find the right perks after customer segmentation

Natürlich! Hier ist der Text so angepasst, dass du ihn direkt in eine `README.md`-Datei bei GitHub einfügen kannst – mit Markdown-Formatierung, klarer Struktur und passenden Codeblöcken:

---

````markdown
# TravelTide: Customer Segmentation Query (PostgreSQL)

## 🎯 Ziel

Dieses SQL-Query dient der Nutzer-Segmentierung auf Basis von Session-, Flug- und Hoteldaten. Ziel ist es, aus dem Nutzungsverhalten individuelle **Customer Segments** wie „Business“, „Solo“, „Family“ usw. zu identifizieren. Die Datenbasis stammt aus dem eBooking-System von TravelTide.

---

## 📐 Query-Struktur

Die SQL-Abfrage nutzt mehrere **CTEs (Common Table Expressions)**, um die Daten schrittweise aufzubereiten und zu analysieren.

---

### 1. `sessions_2023`

Filtert alle Sessions nach dem 4. Januar 2023:

```sql
SELECT * FROM sessions
WHERE session_start > '2023-01-04'
````

---

### 2. `filtered_users`

Begrenzt die Analyse auf aktive Nutzer mit mehr als 7 Sessions:

```sql
SELECT user_id
FROM sessions_2023
GROUP BY user_id
HAVING COUNT(*) > 7
```

---

### 3. `session_base`

Verbindet Sessions mit Nutzerdaten, Flugdaten und Hoteldaten. Berechnet z. B.:

* Sessiondauer
* Buchungsdetails
* Nutzerdemografie
* Hotelkosten
* Fluginfos (Sitze, Airline etc.)

---

### 4. `canceled_trips`

Markiert alle stornierten Trips:

```sql
SELECT DISTINCT trip_id
FROM session_base
WHERE cancellation = TRUE
```

---

### 5. `not_canceled_trips`

Filtert gültige, nicht-stornierte Trips:

```sql
SELECT *
FROM session_base
WHERE trip_id IS NOT NULL
AND trip_id NOT IN (SELECT trip_id FROM canceled_trips)
```

---

### 6. `user_base_session`

Aggregiert pro Nutzer:

* Anzahl Sessions
* Durchschnittliche Klicks
* Durchschnittliche Sessiondauer (in Minuten)
* Durchschnittliches Gepäckvolumen

---

### 7. `user_base_trip`

Aggregiert pro Nutzer alle Buchungsaktivitäten:

* Anzahl Trips und Flüge
* Hotelbuchungsrate
* Ausgaben für Hotels
* Lead Time (Zeit zwischen Session und Reise)
* Rabattnutzung
* Rückflugquote
* Anteil an Wochenendreisen
* Durchschnittliche Anzahl Sitzplätze

---

## 🧠 Segmentierungslogik (Decision Tree)

Das finale Segment (`user_segment`) wird anhand verschachtelter `CASE WHEN`-Bedingungen vergeben. Kriterien sind u. a.:

* Anzahl gebuchter Sitzplätze (avg\_flight\_seats)
* Familienstand (married)
* Kinder vorhanden (has\_children)
* Sessiondauer
* Buchungsfrequenz
* Hotel-Ausgaben

Beispielhafte Segment-Zuordnung:

| Segment             | Beschreibung                               |
| ------------------- | ------------------------------------------ |
| `Business`          | Kurz unterwegs, wenige Trips, allein       |
| `Solo`              | Viel unterwegs, allein, hohe Hotelkosten   |
| `Couple`            | Zwei Personen, verheiratet, moderate Trips |
| `Family`            | Mit Kindern, 2–3 Sitzplätze                |
| `Groups`            | Viele Sitze, wenige Reisen                 |
| `Frequent Traveler` | Viel unterwegs, kein klares Muster         |
| `Other`             | Sonstige Kombinationen                     |

user_segment
│
├── avg_flight_seats ≤ 1
│   ├── has_children = 0
│   │   ├── married = 0
│   │   │   ├── avg_session_duration_min ≤ 3
│   │   │   │   ├── num_trips ≤ 4.9 → Business
│   │   │   │   └── num_trips > 4.9 → Frequent Traveler
│   │   │   └── avg_session_duration_min > 3
│   │   │       ├── money_spent_hotel ≤ 1500 → Other
│   │   │       └── money_spent_hotel > 1500 → Solo
│   │   └── married = 1
│   │       ├── num_trips ≤ 4.9
│   │       │   ├── avg_flight_seats = 2 → Couple
│   │       │   └── sonst → Frequent Traveler
│   │       └── num_trips > 4.9 → Frequent Traveler
│   └── has_children = 1
│       ├── num_trips ≤ 4.9
│       │   ├── avg_flight_seats ≤ 3 → Family
│       │   └── sonst → Frequent Traveler
│       └── num_trips > 4.9 → Frequent Traveler
│
└── avg_flight_seats > 1
    ├── num_trips ≤ 5 → Groups
    └── num_trips > 5 → Frequent Traveler


Hier ist eine **erweiterte Segmentbeschreibung** für jedes deiner sieben Segmente – inklusive:

* **Name (fiktiv, passend zum Profil)**
* **Herkunft (Land/Region)**
* **Beruf**
* **Gender**
* **Hobby**
* **Story-basierter Text** für spätere Marketing-Texte oder Personalisierung

---

### 🧑‍💼 1. **Business**

* **Name**: Jonas
* **Herkunft**: San Jose, USA
* **Beruf**: IT-Consultant
* **Gender**: Männlich
* **Hobby**: Podcasts hören, schnelle Hotelbuchung
* **Beschreibung**:

  > Jonas ist ein vielbeschäftigter IT-Consultant aus San Jose. Er bucht oft kurzfristig Flüge für Kundentermine. Komfort und Effizienz sind ihm wichtiger als der Preis – besonders schätzt er schnelle Buchung und reibungslose Stornierungen.

---

### 🧳 2. **Solo**

* **Name**: Lisa
* **Herkunft**: Seattle, USA
* **Beruf**: UX-Designerin
* **Gender**: Weiblich
* **Hobby**: Alleinreisen, Yoga-Retreats
* **Beschreibung**:

  > Lisa ist eine kreative Einzelreisende aus Seattle. Sie liebt es, neue Orte allein zu entdecken und bevorzugt ruhige Unterkünfte mit flexibler Stornierung. Ihre Buchungen sind gut geplant, aber manchmal spontan.

---

### 👩‍❤️‍👨 3. **Couple**

* **Name**: Matteo
* **Herkunft**: Vancouver, Kanada
* **Beruf**: Innenarchitekt
* **Gender**: Männlich
* **Hobby**: Boutique-Hotels entdecken mit seiner Frau
* **Beschreibung**:

  > Matteo reist gerne mit seiner Frau. Das Paar liebt stilvolle Hotels und verlängerte Wochenendtrips. Buchungen werden meist gemeinsam über das Tablet am Abend gemacht.

---

### 👨‍👩‍👧‍👦 4. **Family**

* **Name**: Anna
* **Herkunft**: Toronto, Kanada
* **Beruf**: Grundschullehrerin
* **Gender**: Weiblich
* **Hobby**: Familienurlaube planen
* **Beschreibung**:

  > Anna reist mit ihrer Familie – zwei Kinder, ein voller Kalender. Sie sucht nach kinderfreundlichen Hotels und günstigen Flugpaketen. Frühbucherrabatte und kostenfreie Stornierung sind ihr besonders wichtig.

---

### 🧑‍🤝‍🧑 5. **Groups**

* **Name**: Karim
* **Herkunft**: Chicago, USA
* **Beruf**: Eventmanager
* **Gender**: Männlich
* **Hobby**: Gruppenreisen organisieren
* **Beschreibung**:

  > Karim plant regelmäßig Gruppenreisen für Freundeskreise oder Firmen. Ihm ist wichtig, mehrere Buchungen effizient zu verwalten. Rabatte für Gruppen und übersichtliche Buchungsprozesse überzeugen ihn.

---

### ✈️ 6. **Frequent Traveler**

* **Name**: Emily
* **Herkunft**: New York City, USA
* **Beruf**: Digital Nomad
* **Gender**: Weiblich
* **Hobby**: Städtereisen & Co-Working weltweit
* **Beschreibung**:

  > Emily lebt zwischen Flugtickets und Hotel-Lounges. Als digitale Nomadin nutzt sie TravelTide intensiv – ob für Business oder Freizeit. Ihr Reisestil ist schnell, flexibel, global.

---

### 🌀 7. **Other**

* **Name**: Alex
* **Herkunft**: Portland, USA
* **Beruf**: Freelancer
* **Gender**: Divers
* **Hobby**: Spontane Trips & Hidden Gems entdecken
* **Beschreibung**:

  > Alex fällt in kein festes Muster. Mal ist es ein Luxushotel, mal ein günstiger Flug in letzter Minute. Das Entscheidende ist Flexibilität – Alex klickt, wenn das Bauchgefühl stimmt.

---

Möchtest du daraus auch ein Tableau-Dashboard-Tooltip-Template machen? Oder eine exportierbare CSV-Tabelle für den Mailmerge?

---

## 🧾 Ergebnis

Der finale `SELECT` liefert **eine Zeile pro Nutzer** mit:

* Demografischen Infos
* Session- und Buchungsverhalten
* Lieblingszielen
* Airline- und Hotelpräferenzen
* Segmentklassifizierung (`user_segment`)

---

## 📌 Hinweise

* Verwendet PostgreSQL-kompatible Syntax
* Performance-Optimierung über CTEs möglich
* Kann als Basis für Segment-basierte Marketing-Aktionen (z. B. personalisierte E-Mails) genutzt werden

---

WITH sessions_2023 AS (
    SELECT *
    FROM sessions
    WHERE session_start > '2023-01-04'
),

filtered_users AS (
    SELECT user_id
    FROM sessions_2023
    GROUP BY user_id
    HAVING COUNT(*) > 7
),

session_base AS (
    SELECT
        s.session_id, s.user_id, s.trip_id, s.session_start, s.session_end,
        EXTRACT(EPOCH FROM s.session_end - s.session_start) AS session_duration,
        s.page_clicks, s.flight_discount, s.flight_discount_amount,
        s.hotel_discount, s.hotel_discount_amount, s.flight_booked,
        s.hotel_booked, s.cancellation, u.birthdate, u.gender, u.married,
        u.has_children, u.home_country, u.home_city, u.home_airport,
        u.home_airport_lat, u.home_airport_lon, u.sign_up_date,
        f.origin_airport, f.destination, f.destination_airport, f.seats,
        f.return_flight_booked, f.departure_time, f.return_time,
        f.checked_bags, f.trip_airline, f.destination_airport_lat,
        f.destination_airport_lon, f.base_fare_usd,
        h.hotel_name,
        CASE WHEN h.nights < 0 THEN 1 ELSE h.nights END AS nights,
        h.rooms, h.check_in_time, h.check_out_time,
        h.hotel_per_room_usd AS hotel_price_per_room_night_usd
    FROM sessions_2023 s
    LEFT JOIN users u ON s.user_id = u.user_id
    LEFT JOIN flights f ON s.trip_id = f.trip_id
    LEFT JOIN hotels h ON s.trip_id = h.trip_id
    WHERE s.user_id IN (SELECT user_id FROM filtered_users)
),

canceled_trips AS (
    SELECT DISTINCT sb.trip_id
    FROM session_base sb
    WHERE sb.cancellation = TRUE
),

not_canceled_trips AS (
    SELECT *
    FROM session_base
    WHERE trip_id IS NOT NULL
      AND trip_id NOT IN (SELECT trip_id FROM canceled_trips)
),

user_base_session AS (
    SELECT
        user_id,
        MIN(EXTRACT(YEAR FROM AGE(birthdate))) AS age,
        COUNT(DISTINCT session_id) AS num_sessions,
        SUM(page_clicks) AS total_clicks,
        ROUND(AVG(page_clicks), 2) AS avg_clicks_per_session,
        ROUND(AVG(session_duration), 2) AS avg_session_duration_sec,
        ROUND(AVG(session_duration) / 60.0, 1) AS avg_session_duration_min,
        ROUND(AVG(checked_bags), 2) AS avg_checked_bags
    FROM session_base
    GROUP BY user_id
),

user_base_trip AS (
    SELECT
        user_id,
        COUNT(DISTINCT trip_id) AS num_trips,
        SUM(flight_booked::int + return_flight_booked::int) AS num_flights,
        ROUND(SUM(flight_booked::int)*1.0/NULLIF(COUNT(DISTINCT trip_id), 0), 2) AS flight_rate,
        ROUND(SUM(hotel_booked::int)*1.0/NULLIF(COUNT(DISTINCT trip_id), 0), 2) AS hotel_rate,
        ROUND(SUM((flight_booked OR hotel_booked)::int) * 1.0 / COUNT(*), 4) AS conversion_rate,
        COALESCE(SUM((hotel_price_per_room_night_usd * nights * rooms) * (1 - COALESCE(hotel_discount_amount, 0))), 0) AS money_spent_hotel,
        COALESCE(SUM((hotel_price_per_room_night_usd * nights * rooms) * (1 - COALESCE(hotel_discount_amount, 0))), 0)
            / NULLIF(SUM(hotel_booked::int), 0) AS avg_spending_hotel,
        ROUND(AVG(EXTRACT(EPOCH FROM (departure_time - session_end)) / 86400), 2) AS avg_lead_time_flight_days,
        ROUND(AVG(EXTRACT(EPOCH FROM (check_in_time - session_end)) / 86400), 2) AS avg_lead_time_hotel_days,
        ROUND(COALESCE(SUM(flight_discount::int) * 1.0 / NULLIF(COUNT(*), 0), 0), 2) AS flight_discount_proportion,
        ROUND(COALESCE(SUM(hotel_discount::int) * 1.0 / NULLIF(COUNT(*), 0), 0), 2) AS hotel_discount_proportion,
        ROUND(SUM(return_flight_booked::int) * 1.0 / NULLIF(SUM(flight_booked::int), 0), 2) AS round_trip_proportion,
        ROUND(COALESCE(COUNT(DISTINCT CASE
            WHEN EXTRACT(DOW FROM departure_time) IN (5, 6)
             AND return_flight_booked
             AND EXTRACT(DOW FROM return_time) IN (0, 1)
             AND return_time - departure_time < INTERVAL '3 days'
        THEN trip_id END) * 1.0 / NULLIF(COUNT(DISTINCT trip_id), 0), 0), 2) AS weekend_trip_proportion,
        ROUND(AVG(seats), 3) AS avg_flight_seats
    FROM not_canceled_trips
    GROUP BY user_id
)

SELECT
    u.*,
    b.*,
    t.*,
    sb.destination,
    sb.trip_airline,
    sb.hotel_name,
    sb.nights,
    sb.rooms,
    sb.hotel_price_per_room_night_usd,

    CASE
      WHEN t.avg_flight_seats <= 1 THEN
        CASE
          WHEN u.has_children::int = 0 THEN
            CASE
              WHEN u.married::int = 0 THEN
                CASE
                  WHEN b.avg_session_duration_min <= 3 THEN
                    CASE
                      WHEN t.num_trips <= 4.9 THEN 'Business'
                      ELSE 'Frequent Traveler'
                    END
                  ELSE
                    CASE
                      WHEN t.money_spent_hotel <= 1500 THEN 'Other'
                      ELSE 'Solo'
                    END
                END
              ELSE
                CASE
                  WHEN t.num_trips <= 4.9 THEN
                    CASE
                      WHEN t.avg_flight_seats = 2 THEN 'Couple'
                      ELSE 'Frequent Traveler'
                    END
                  ELSE 'Frequent Traveler'
                END
            END
          ELSE
            CASE
              WHEN t.num_trips <= 4.9 THEN
                CASE
                  WHEN t.avg_flight_seats <= 3 THEN 'Family'
                  ELSE 'Frequent Traveler'
                END
              ELSE 'Frequent Traveler'
            END
        END
      ELSE
        CASE
          WHEN t.num_trips <= 5 THEN 'Groups'
          ELSE 'Frequent Traveler'
        END
    END AS user_segment

FROM user_base_session b
LEFT JOIN users u ON b.user_id = u.user_id
LEFT JOIN user_base_trip t ON b.user_id = t.user_id
LEFT JOIN session_base sb ON sb.user_id = b.user_id
WHERE t.num_trips IS NOT NULL
ORDER BY b.user_id;

# TravelTide Rewards Program Project Plan and Tableau Workbook Guide

## 1. Projektübersicht

**Ziel:** Basierend auf Session-, Flug- und Hoteldaten Verhaltensmarker für fünf Rewards-Perks identifizieren und jedem Kunden seinen wahrscheinlich favorisierten Perk zuzuordnen. Die Ergebnisse werden in einer Präsentation und einem Tableau-Workbook visualisiert und dokumentiert.
**Zielpublikum:**

* **Primär:** Elena Tarrant (Head of Marketing) – nicht-technisch, optimistisch und zukunftsorientiert
* **Sekundär:** Technische Kolleg:innen – detaillierte Kommentierung in SQL- und Python-Scripts, Dokumentation für Reproduzierbarkeit

## 2. Projektstruktur und Deliverables

| Deliverable               | Inhalt                                                                 | Format               | Tool                          |
| ------------------------- | ---------------------------------------------------------------------- | -------------------- | ----------------------------- |
| Executive Summary         | Kurz & prägnant: Ziele, Methodik, Kernergebnisse, Empfehlungen         | Markdown (README.md) | VSCode/GitHub                 |
| Slide Deck                | 5-Minuten-Präsentation: max. 6 Folien (Intro, Key Results, Next Steps) | PPTX / Google Slides | PowerPoint bzw. Google Slides |
| Tableau Workbook          | Interaktive Dashboards für Segmente & Perks                            | .twbx                | Tableau                       |
| Jupyter Notebook          | Analytische Schritte, SQL-Abfragen, Visualisierungsvorbereitung        | .ipynb               | Colab                         |
| SQL Scripts               | Vollständige, kommentierte Abfragen                                    | .sql                 | Beekeeper                     |
| CSV mit Segment-Zuordnung | Kundenliste mit zugeordneten Favoriten-Perks                           | .csv                 | Spreadsheet                   |
| Vollständiges Repo        | Alle Dateien strukturiert im GitHub-Repository                         | GitHub Link          | GitHub                        |
| Video Präsentation        | 5-Minuten-Video mit Storytelling und Insights                          | Loom / Zoom Link     | Video-Tool                    |

## 3. Detaillierter Projektablauf

### 3.1 Datenvorbereitung & -exploration

1. **Daten laden** (TRAVELTIDE.csv) ins Spreadsheet & Colab
2. **Daten prüfen**: Fehlende Werte, Schema, Duplicates
3. **Datenbereinigung**: Outlier-Handling, Datentypkorrekturen, Cancellations entfernen
4. **Feature-Engineering**: Berechne Recency, Frequency, Monetary, Session-Metriken, Lead Times
5. **Dokumentation**: Kommentiere alle Schritte im Notebook und in SQL-Scripts

### 3.2 Segmentierung & Perk-Analyse

1. **RFM-Segmente und Decision Tree**: Übersetze den SQL-Case-Tree in Python-Pandas (optional)
2. **Perks definieren**: Für jeden Nutzer anreichern, z. B. „most-likely free cancellation“, „free checked bag“
3. **Validierung**: Statistische Tests oder Korrelationsanalysen, um Perk-Hypothesen zu prüfen
4. **Zuweisungs-CSV**: Exportiere Kunden-Perk-Zuordnung als `customer_perks.csv`

### 3.3 Dashboard-Design in Tableau

1. **Datenquelle verbinden**: `customer_perks.csv` + Originaldaten in Tableau
2. **Datenmodell**: Relationen (Customers, Sessions, Flights, Hotels)
3. **Blätter erstellen**:

   * **Segment-Übersicht**: Balkendiagramm Anzahl Users pro Segment
   * **Perk-Präferenzen**: Kreisdiagramm relative Anteile pro Perk
   * **Segment-Profile**: KPI-Karten (Avg. Trips, Avg. Spend, Lead Time)
   * **Beispiel-Kunden**: Tabelle mit Top 10 Users, Perk, Segment
4. **Dashboard zusammensetzen**: Single-Page Executive View + interaktives Detail-Dashboard
5. **Story**: 5 Slides in Tableau Story-Modus (optional)

### 3.4 Präsentation & Kommunikation

1. **Executive Summary**: README.md erstellen, Abschnitt mit Visualisierung aus Tableau (Screenshot)
2. **Slide Deck bauen**: 5 Folien, optimistisch und forward-looking:

   * Folie 1: Kontext & Objectives
   * Folie 2: Methodik (1–2 Sätze)
   * Folie 3: Key Findings (Grafik + Bulletpoints)
   * Folie 4: Empfehlung & Next Steps
   * Folie 5: Dank & Kontakt
3. **Video aufnehmen**: Max. 5 Minuten, klare Story, Fokus auf Business-Impact
4. **GitHub-Repo strukturieren**:

   ```
   /README.md
   /customer_perks.csv
   /src/
     analysis_notebook.ipynb
     segmentation.sql
   /tableau/
     rewards_dashboard.twbx
   /slides/
     rewards_presentation.pptx
   ```

### 3.5 Technische Dokumentation & Reproduzierbarkeit

* **SQL:** Kommentiere Entscheidungspunkte, CTEs im `segmentation.sql`
* **Python-Notebook:** Markdown-Zellen für Erläuterungen, Verlinkung auf SQL-Files
* **README.md:** Projektstruktur, Ausführungsanleitung (`pip install`, Pfade, typische Queries)

## 4. Next Steps & Timeline

* **Montag**: Datenexploration & -bereinigung abschließen
* **Dienstag**: Segmentierung & Perk-Zuweisung finalisieren, `customer_perks.csv` exportieren
* **Mittwoch**: Tableau-Dashboard entwickeln, Visuals gestalten
* **Donnerstag**: Slide Deck & README finalisieren
* **Freitag**: Video aufnehmen, alles in GitHub veröffentlichen, Abgabe bis 23:59h

---

*Let's create a compelling, optimistic, and data-driven presentation that helps Elena confidently launch our personalized rewards program!*


