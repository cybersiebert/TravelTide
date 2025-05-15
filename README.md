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

# Präsentationsskript – TravelTide Rewards Programm

## Folie 1: Titel & Begrüßung

Guten Tag zusammen, mein Name ist Matthias und ich bin Datenanalyst bei TravelTide. Es freut mich sehr, Ihnen heute einen tiefen Einblick in unsere datengetriebene Arbeitsweise zu geben und zu zeigen, wie wir ein neues, personalisiertes Rewards-Programm entwickeln, das direkt auf die Bedürfnisse und Wünsche unserer Kundinnen und Kunden zugeschnitten ist. Ziel meiner Präsentation ist es, die Entstehung, Analyse und Umsetzung dieses Programms verständlich und praxisnah darzustellen und Ihnen dabei auch die Bedeutung unserer Segmentierungsarbeit näherzubringen.

## Folie 2: Welcome to TravelTide / Herausforderungen & Zielsetzung

TravelTide ist ein dynamisches, innovatives E-Booking-Startup, das am Ende der Corona-Pandemie gegründet wurde – genauer gesagt im April 2021. Unsere große Stärke liegt darin, das größte und vielfältigste Reiseinventar im Markt zu bieten. Wir ermöglichen mit unserer hochmodernen Suchtechnologie eine besonders einfache und schnelle Buchungserfahrung. Das schätzen unsere Kunden und das bestätigen auch unabhängige Analysten der Reisebranche. Unsere Plattform wächst stetig und wir sind stolz, durch konsequente Datenaggregation und eine moderne technische Infrastruktur unseren Nutzern einen echten Mehrwert zu bieten.

Unsere starke Fokussierung auf die Erweiterung unseres Reiseinventars hatte aber auch einen Preis: Die Kundenbindung und das gezielte Kundenerlebnis blieben bislang teilweise auf der Strecke. Die Folge: Unsere Kundinnen und Kunden nutzen das breite Angebot, kommen aber oft nicht dauerhaft zurück. Unser CEO Kevin Talanick hat daher eine klare Entscheidung getroffen. Er möchte die Kundenbindung strategisch stärken – mit einer Marketingstrategie, die sich auf datenbasierte Einblicke und tiefes Kundenverständnis stützt. Das Ziel ist es, nicht nur Neukunden zu gewinnen, sondern die bestehenden Kunden gezielt und nachhaltig an TravelTide zu binden.

## Folie 3: Neue Marketingchefin Elena Tarrant / Aufgabe der Analytics /  Hypothese & Mock-Ups / Vorgehen und Methodik 

Dafür wurde Elena Tarrant als neue Head of Marketing an Bord geholt. Elena gilt in der Branche als ausgewiesene Expertin für Kundenbindungsprogramme und innovative Rewards-Konzepte. Ihre Aufgabe ist es, ein modernes, personalisiertes Rewards-Programm zu entwerfen und umzusetzen, das einen echten Mehrwert für unsere Kunden bietet und sie dazu motiviert, regelmäßig auf TravelTide zurückzukommen. Damit dies gelingt, arbeitet Elena von Anfang an eng mit uns aus der Datenanalyse zusammen – denn ohne belastbare Insights lässt sich keine echte Personalisierung schaffen.

Das Projekt ist ein Paradebeispiel für die enge Zusammenarbeit von Marketing und Data Team. Unsere Aufgabe als Analytics-Team ist es, auf Basis großer Datenmengen das Verhalten, die Vorlieben und die Wünsche unserer Nutzer zu analysieren und Elena so fundierte Empfehlungen für die Ansprache und die Struktur des Rewards-Programms zu liefern. Wir nutzen dazu fortschrittliche Tools wie SQL für die Datenbankabfragen, Python in Google Colab für explorative Analysen und Machine Learning, sowie Tableau für die visuelle Aufbereitung und das Dashboarding. Unser Ziel: Personalisierung, die auf Fakten basiert und wirklich Wirkung zeigt. 

Bei dieser Analyse haben wir die Daten zunächst intensiv bereinigt und vorbereitet: Mit SQL wurden alle Null-Werte entfernt, sodass wir eine saubere Datenbasis hatten. Wir haben zudem nur Sessions berücksichtigt, deren session\_start nach dem 04.01.2023 lag, und sämtliche negativen Nächte (negative Übernachtungswerte) ausgeschlossen. Zusätzlich wurden stornierte Buchungen systematisch entfernt, um die Analyse nicht zu verzerren. Für die Segmentierung kam eine verschachtelte CASE WHEN-Expression zum Einsatz, mit der wir mehrere entscheidende Merkmale direkt im SQL-Code auswerten konnten: die durchschnittliche Anzahl gebuchter Sitzplätze (avg\_flight\_seats), den Familienstand (married), ob Kinder vorhanden sind (has\_children), die durchschnittliche Sessiondauer, die Buchungsfrequenz und die Höhe der Hotel-Ausgaben. Auf Basis dieser Dimensionen haben wir verschiedene Segmente gebildet und daraus Personas abgeleitet, die typische Kundengruppen und ihr Reiseverhalten repräsentieren.

Elena hat die Hypothese aufgestellt, dass es die Conversion-Raten massiv erhöht, wenn wir bei der Einladung zum Rewards-Programm genau den Vorteil (Perk) in den Vordergrund stellen, der dem einzelnen Kunden am wichtigsten erscheint. Dazu hat sie zwei E-Mail-Entwürfe erstellt: Einer ist generisch und listet alle Vorteile gleichwertig auf, der andere hebt einen spezifischen Haupt-Perk, zum Beispiel „kostenlose Stornierung“, gezielt hervor. Unsere Analysen belegen: Die gezielte, personalisierte Ansprache kommt deutlich besser an und erhöht die Wahrscheinlichkeit einer Anmeldung erheblich. Genau darauf richten wir unsere Segmentierung und die weitere Personalisierung aus.

Unser Analyseprozess besteht aus zwei wesentlichen Aufgaben:

1. Zunächst prüfen wir, ob es in unseren Daten tatsächlich klar unterscheidbare Kundensegmente mit unterschiedlichen Präferenzen für bestimmte Rewards gibt. Das bedeutet, wir identifizieren mithilfe von Data Mining und Clustering verschiedene Nutzertypen.
2. Im nächsten Schritt ordnen wir jedem Kunden individuell den wahrscheinlich beliebtesten Perk zu – auf Basis von Verhaltensdaten, Buchungshistorie und Vorlieben. Diese Insights werden dann an das Marketing weitergegeben, das damit die Personalisierung der Ansprache umsetzt.

Für die Analyse kommen SQL, Python, Machine-Learning-Methoden und anschauliche Tableau-Dashboards zum Einsatz. Damit sorgen wir dafür, dass sowohl die Geschäftsführung als auch das Marketingteam die Ergebnisse intuitiv erfassen und direkt nutzen können.

## Folie 4: Segmentierung – Die 7 Personas

Um die Kommunikation und die Personalisierung des Rewards-Programms maximal relevant zu machen, haben wir sieben verschiedene Kundensegmente – sogenannte Personas – identifiziert. Jede dieser Personas steht exemplarisch für eine typische Reisegruppe mit ganz eigenen Bedürfnissen, Reisegewohnheiten und Motivationen. Sie helfen uns, komplexe Nutzerdaten in anschauliche, greifbare Profile zu übersetzen und das Marketing zielgerichtet auf individuelle Wünsche zuzuschneiden.

---

### Folie 4.1: Business – Jonas

Stellen wir uns Jonas vor: Er ist 38 Jahre alt, arbeitet als IT-Consultant in San Jose und hat einen vollen Terminkalender. Für Jonas ist Reisen Teil des Berufsalltags. Oft wird er kurzfristig zu Kundenterminen geschickt, muss Flüge und Hotels in letzter Minute buchen und ist dabei stets auf Flexibilität angewiesen. Jonas schätzt Komfort, digitale Services und vor allem reibungslose Abläufe – der Preis ist für ihn zweitrangig, wenn alles effizient und zuverlässig funktioniert.

Im Arbeitsalltag ist für Jonas besonders wichtig, dass Umbuchungen oder Stornierungen unkompliziert möglich sind. Er nutzt TravelTide gerne, weil er weiß, dass auf der Plattform schnelle Buchungen und flexible Konditionen garantiert sind. Deshalb stellen wir für Business-Kunden wie Jonas im Rewards-Programm Vorteile wie „kostenlose Stornierung“, „schnelle Abwicklung bei Umbuchungen“ und exklusive Business-Services ganz nach vorne. Die Marketingkommunikation wird für diese Gruppe besonders klar und direkt auf den Zeitvorteil und die Zuverlässigkeit zugeschnitten.

---

### Folie 5: Solo – Lisa

Unsere nächste Persona ist Lisa, 29 Jahre alt, UX-Designerin aus Seattle. Sie liebt es, alleine auf Reisen zu gehen und nutzt ihre Freizeit, um neue Städte und Länder zu entdecken. Besonders zieht es sie zu Yoga-Retreats, inspirierenden Unterkünften und Orten abseits des Mainstreams. Lisa plant gerne, bleibt aber auch spontan und bucht manchmal Last Minute. Für sie ist das Wichtigste, dass sie flexibel stornieren kann und dass es spezielle Angebote für Alleinreisende gibt, zum Beispiel Rabatte auf Einzelzimmer oder besondere Retreat-Pakete.

Das Rewards-Programm von TravelTide spricht Solo-Reisende wie Lisa gezielt mit flexiblen Buchungsoptionen, exklusiven Rabatten auf besondere Unterkünfte und inspirierenden Insider-Tipps an. Für diese Zielgruppe sind auch persönliche Empfehlungen und Storytelling in der Kommunikation besonders wirksam – zum Beispiel Reiseberichte anderer Solo-Traveler oder individuelle Reisevorschläge auf Basis ihres bisherigen Buchungsverhaltens.

---

### Folie 6: Couple – Matteo

Matteo ist 41 Jahre alt, Innenarchitekt aus Vancouver. Gemeinsam mit seiner Frau liebt er es, regelmäßig Kurztrips zu unternehmen und neue Boutique-Hotels zu entdecken. Für Matteo und seine Frau stehen die Qualität des Aufenthalts, einzigartiges Design und besondere Atmosphäre im Vordergrund. Sie nehmen sich gerne Zeit, stöbern abends zusammen auf der Couch nach dem nächsten Reiseziel und legen Wert auf stilvolle Unterkünfte, die ihnen ein besonderes Erlebnis bieten.

Das Rewards-Programm für Paare wie Matteo und seine Frau bietet Vorteile wie Gratisnächte, kostenfreie Upgrades, exklusive Wochenendangebote und Willkommensgeschenke im Hotelzimmer. Die Kommunikation legt den Fokus auf gemeinsame Erlebnisse, Romantik und das gewisse Extra, das den Aufenthalt besonders macht. Zudem werden Empfehlungen für Paarziele und romantische Hideaways gezielt ausgespielt.

---

### Folie 7: Family – Anna

Anna ist 36, lebt mit ihrem Mann und zwei Kindern in Toronto und arbeitet als Grundschullehrerin. Familienurlaub ist für sie immer ein kleines Projekt: Das Reiseziel muss kinderfreundlich, bezahlbar und unkompliziert sein. Anna bucht oft im Voraus, achtet auf Frühbucherrabatte und sucht nach Hotels mit Familienzimmern, Spielmöglichkeiten und Services wie kostenfreiem Frühstück für Kinder. Für sie ist besonders wichtig, dass sie im Notfall stornieren kann, ohne hohe Kosten zu haben, da sich mit kleinen Kindern manchmal kurzfristig alles ändern kann.

Für Familien wie Anna setzt TravelTide im Rewards-Programm gezielt Vorteile wie Frühbucherrabatte, kostenfreie Stornierungen, Kinderübernachtungen gratis oder exklusive Familienpakete ein. Die Marketingkommunikation stellt die Entlastung im Familienalltag, Planungssicherheit und Sicherheit in den Mittelpunkt – und liefert praktische Tipps für stressfreies Reisen mit Kindern.

---

### Folie 8: Groups – Karim

Karim ist 34 Jahre alt, Eventmanager aus Chicago und organisiert regelmäßig Reisen für Freundesgruppen, Vereine oder Unternehmen. Für ihn zählt vor allem: Die Buchung mehrerer Zimmer und Flüge muss effizient und übersichtlich sein, und er möchte von Gruppenrabatten profitieren. Karim plant im Vorfeld, gleicht Wünsche ab und schätzt Buchungsplattformen, die ihm Tools für die Verwaltung, Bezahlung und Kommunikation mit der Gruppe bieten.

Das Rewards-Programm von TravelTide bietet Gruppenreisenden wie Karim spezielle Rabatte, einfache Tools zur Koordination mehrerer Buchungen und exklusive Deals für Gruppenunterkünfte. In der Kommunikation steht die einfache Handhabung und der Mehrwert für Gruppen im Fokus. Karim erhält außerdem regelmäßige Inspirationen und Tipps für besondere Gruppenreisen und kann die Vorteile des Programms gezielt für seine Gruppen einsetzen.

---

### Folie 9: Frequent Traveler – Emily

Emily ist 32 Jahre alt und lebt als Digital Nomad. Ihr Lebensmittelpunkt ist New York City, aber eigentlich ist sie überall auf der Welt unterwegs. Ihre Arbeit im Bereich IT und Consulting erlaubt ihr, von überall zu arbeiten – heute Paris, nächste Woche Lissabon. Für Emily zählen Flexibilität, Geschwindigkeit und Belohnungen für Vielbucher. Sie bucht oft und erwartet besondere Benefits wie ein kostenloses Gepäckstück, exklusive Rabatte und Treuepunkte, die sie flexibel einlösen kann.

Das Rewards-Programm für Vielreisende wie Emily setzt auf schnelle Buchungsprozesse, spezielle Vielbucher-Angebote, Upgrades und Zugang zu besonderen Services wie Lounges oder Late-Check-Outs. Die Kommunikation ist international, dynamisch und auf ein urbanes, flexibles Publikum ausgerichtet.

---

### Folie 10: Other – Alex

Alex ist 27, Freelancer aus Portland und ein echter Freigeist. Sie oder er legt sich nicht fest, bucht mal spontan ein Luxushotel für ein Wochenende, dann wieder einen günstigen Last-Minute-Flug zu einem unbekannten Ziel. Für Alex zählt vor allem die Flexibilität und die Möglichkeit, immer wieder Neues zu entdecken, ohne an starre Konditionen gebunden zu sein.

Das Rewards-Programm spricht diese Zielgruppe mit ständig wechselnden Spezialdeals, maximal flexiblen Buchungs- und Stornierungsoptionen sowie Inspirationen für Hidden Gems und spontane Trips an. In der Kommunikation stehen Abenteuerlust, Spontanität und das Entdecken neuer Reiseziele im Mittelpunkt – das Rewards-Programm macht es möglich, immer wieder neue Highlights zu erleben.

---

## Folie 10: Datengetriebene Personalisierung

Unsere detaillierte Segmentierung ist die Grundlage für die datengetriebene Personalisierung im gesamten Rewards-Programm. Die Zuweisung des wahrscheinlich beliebtesten Rewards-Perks pro Kunde erfolgt vollautomatisch, indem wir das bisherige Buchungsverhalten, Präferenzen und besondere Anlässe berücksichtigen. Dadurch bekommt jede und jeder genau die Belohnung, die am attraktivsten ist – und das erhöht die Motivation zur Teilnahme und Nutzung des Programms erheblich.

## Folie 11: Struktur der Kundensegmente

Die Struktur unserer Kundensegmente ist ein zentraler Bestandteil der Strategie für das TravelTide Rewards-Programm. Prozentual setzen sich die Segmente wie folgt zusammen (Details können als Diagramm visualisiert werden):

* Jedes Segment unterscheidet sich hinsichtlich des durchschnittlichen Buchungsverhaltens, des Gesamtumsatzes und der Vorbuchungszeit (Lead Time) bei Hotelbuchungen. Besonders interessant sind hierbei folgende KPIs:

  * **Durchschnittliche Lead Time Hotel (in Tagen):** Wie früh vor der Reise buchen die Kund\:innen ihr Hotel?
  * **Gesamtausgaben Hotel (total money spent hotel):** Wie hoch ist der durchschnittliche Umsatz pro Segment?
  * **Gesamtanzahl der Reisen (total number of trips):** Wie reisefreudig ist das jeweilige Segment?

Diese Kennzahlen ermöglichen es uns, gezielt die Segmente mit den jeweils passenden Perks und Kampagnen anzusprechen. Die Daten zeigen beispielsweise, dass Segmente mit hoher Buchungsfrequenz andere Vorteile bevorzugen als Gelegenheitsbucher mit langer Vorlaufzeit.

Die Segmentgröße und das Verhalten werden fortlaufend in Tableau visualisiert und analysiert, sodass wir Trends frühzeitig erkennen und darauf reagieren können.

## Folie 15: Umsetzung & Ausblick

Zur Realisierung der personalisierten Rewards-Kommunikation setzen wir auf die Integration moderner Marketing-Automation-Systeme. Die Kunden-Segmente und Perk-Zuweisungen werden direkt in unsere E-Mail-Marketing-Plattform überführt, sodass dynamischer Content – also der jeweils passende Haupt-Perk – für jede Kund\:in individuell ausgespielt werden kann. Über eine direkte Tableau-Integration stellen wir sicher, dass die Ergebnisse der Analysen laufend überwacht und in Dashboards für Marketing und Management visualisiert werden.

Die Ausspielung erfolgt segmentiert und wird über verschiedene Varianten getestet, z. B. mit Fokus auf „kostenlose Stornierung“, „Zimmer-Upgrade“ oder Treuepunkte. Hierfür werden regelmäßig A/B-Tests durchgeführt, um herauszufinden, welcher Perk in welchem Segment die höchste Conversion erzielt.

Für jedes Kundensegment werden eigene, individualisierte E-Mail-Kampagnen erstellt. Der Haupt-Perk (z. B. kostenlose Hotelmahlzeit, kein Stornogebühr, exklusive Rabatte etc.) steht im Mittelpunkt der Botschaft. So sprechen wir gezielt die Bedürfnisse und Wünsche der verschiedenen Kundengruppen an und erhöhen die Wahrscheinlichkeit einer Rewards-Programm-Anmeldung deutlich.

Mittels dynamischer E-Mail-Inhalte wird der für die Zielperson am attraktivsten wirkende Perk automatisch in der Betreffzeile und im E-Mail-Body hervorgehoben.

Die Wirkung unterschiedlicher Perks und Kommunikationsstrategien wird laufend getestet. Varianten wie „kostenlose Stornierung“, „Zimmer-Upgrade“ oder Treuepunkte werden je nach Segment gegeneinander geprüft und die beste Option fortlaufend ausgespielt.

## Folie 16: Technische Umsetzung & Controlling

Die technische Umsetzung erfolgt über ein automatisiertes Marketing-System, das direkt mit unseren Segmentierungsdaten und der Analyse aus Tableau verbunden ist. Dadurch können neue Segmentierungen und Insights in Echtzeit in laufende Kampagnen einfließen.

Die Kampagnen werden durch Dashboards permanent überwacht. Zu den wichtigsten KPIs zählen Anmelderaten, Conversion Rates, Segment-Performance und langfristige Retention. Über regelmäßige Reviews und Audits stellen wir sicher, dass das Programm nachhaltig wirkt und weiter optimiert wird.

Die Ausspielung erfolgt segmentiert und wird über verschiedene Varianten getestet, z. B. mit Fokus auf „kostenlose Stornierung“, „Zimmer-Upgrade“ oder Treuepunkte. Hierfür werden regelmäßig A/B-Tests durchgeführt, um herauszufinden, welcher Perk in welchem Segment die höchste Conversion erzielt.

## Folie 17: Q & A

Ich bedanke mich für Ihre Aufmerksamkeit. Gerne beantworte ich jetzt Ihre Fragen zu allen Themen rund um unser Rewards-Programm, die Segmentierung, Personalisierung oder die technische Umsetzung. Lassen Sie uns diskutieren, wie datengetriebene Personalisierung und Automatisierung echten Mehrwert für TravelTide und unsere Kund\:innen schaffen!

---

**Hinweis:** Das Skript dient als umfassender Leitfaden und ist bewusst detailliert gehalten, damit Sie als Sprecher flexibel zwischen den Themen springen und bei Nachfragen gezielt tiefer ins Detail gehen können. Die ausführlichen Stories zu den einzelnen Segmenten unterstützen Sie dabei, die Zielgruppen anschaulich und lebendig zu erklären und das Programm praxisnah zu präsentieren.



