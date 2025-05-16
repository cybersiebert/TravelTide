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

Hallo zusammen,
mein Name ist Matthias und ich arbeite als Datenanalyst bei TravelTide. Ich freue mich, Ihnen heute zu zeigen, wie wir mit Daten arbeiten und wie daraus unser neues Rewards-Programm entsteht. Dieses Programm soll genau das bieten, was unsere Kundinnen und Kunden sich wünschen.
In meiner Präsentation möchte ich Ihnen einfach erklären, wie wir das Programm geplant, analysiert und umgesetzt haben – und warum die Aufteilung unserer Kunden in verschiedene Gruppen dafür so wichtig ist.


## Folie 2: Welcome to TravelTide / Herausforderungen & Zielsetzung

TravelTide ist ein junges und modernes Online-Unternehmen für Reisen. Wir wurden im April 2021, nach der Corona-Zeit, gegründet. Unsere große Stärke ist das Sie bei uns die größte Auswahl an Reisen finden. Mit unserer schnellen Suchfunktion können unsere Kunden ganz einfach und schnell buchen. Das gefällt unseren Kunden und auch Reise-Experten sagen, dass wir sehr gut sind. Unsere Plattform wird immer größer, weil wir viele Daten sammeln und die Technik immer besser machen.

Bis jetzt haben wir uns vor allem darauf konzentriert, unser Angebot immer weiter zu vergrößern. Dadurch haben wir aber zu wenig darauf geachtet, dass unsere Kunden gerne zurückkommen. Viele buchen bei uns – aber viele kommen nicht wieder.
Unser CEO Kevin Talanick hat deshalb entschieden: Wir wollen, dass mehr Kunden bei uns bleiben. Dafür brauchen wir ein Marketing, das die Kunden und ihre Wünsche besser versteht – und das mit Hilfe von Daten. Unser Ziel ist es jetzt, nicht nur neue Kunden zu bekommen, sondern unsere bisherigen Kunden glücklich zu machen und an TravelTide zu binden.

## Folie 3: Neue Marketingchefin Elena Tarrant / Aufgabe der Analytics /  Hypothese & Mock-Ups / Vorgehen und Methodik 

Elena Tarrant ist unsere neue Marketing-Chefin. Sie soll ein besonderes Bonus-Programm für TravelTide entwickeln, das auf jeden Kunden passt. Das Ziel ist, dass unsere Kunden mehr Vorteile bekommen und öfter zu uns zurückkommen. Dafür schauen wir uns genau an, was unsere Kunden mögen und wie sie buchen.

Das Marketing-Team und das Daten-Team arbeiten dafür eng zusammen. Wir als Daten-Team schauen uns die Daten an und sagen, welche Vorteile für welche Kunden am wichtigsten sind. Dafür nutzen wir Programme wie SQL, Python und Tableau, um aus den Daten Gruppen von ähnlichen Kunden zu finden.

Bei der Datenaufbereitung haben wir Fehler und unnötige Werte entfernt, nur Buchungen ab dem 4. Januar 2023 genommen und negative Übernachtungen ausgeschlossen. Mit bestimmten Regeln in SQL teilen wir die Kunden in Gruppen ein, zum Beispiel: verheiratet oder nicht, mit oder ohne Kinder, wie oft sie buchen und wie viel sie für Hotels ausgeben.

Elena glaubt: Wenn wir jedem Kunden das anbieten, was ihm am wichtigsten ist, melden sich mehr Leute für das Bonus-Programm an. Unsere Daten zeigen: Eine persönliche Ansprache ist wirklich besser als eine allgemeine Nachricht an alle.

So kann unser Marketing-Team jeden Kunden ganz gezielt ansprechen. Die Ergebnisse zeigen wir in Berichten und Dashboards, damit sie einfach genutzt werden können.

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

Emily ist 32 Jahre alt und lebt als Digitaler Nomade. Ihr Lebensmittelpunkt ist New York City, aber eigentlich ist sie überall auf der Welt unterwegs. Ihre Arbeit im Bereich IT und Consulting erlaubt ihr, von überall zu arbeiten – heute Paris, nächste Woche Lissabon. Für Emily zählen Flexibilität, Geschwindigkeit und Belohnungen für Vielbucher. Sie bucht oft und erwartet besondere Benefits wie ein kostenloses Gepäckstück, exklusive Rabatte und Treuepunkte, die sie flexibel einlösen kann.

Das Rewards-Programm für Vielreisende wie Emily setzt auf schnelle Buchungsprozesse, spezielle Vielbucher-Angebote, Upgrades und Zugang zu besonderen Services wie Lounges oder Late-Check-Outs. Die Kommunikation ist international, dynamisch und auf ein urbanes, flexibles Publikum ausgerichtet.

---

### Folie 10: Other – Alex

Alex ist 27, Freelancer aus Portland und ein echter Freigeist. Sie oder er legt sich nicht fest, bucht mal spontan ein Luxushotel für ein Wochenende, dann wieder einen günstigen Last-Minute-Flug zu einem unbekannten Ziel. Für Alex zählt vor allem die Flexibilität und die Möglichkeit, immer wieder Neues zu entdecken, ohne an starre Konditionen gebunden zu sein.

Das Rewards-Programm spricht diese Zielgruppe mit ständig wechselnden Spezialdeals, maximal flexiblen Buchungs- und Stornierungsoptionen sowie Inspirationen für Hidden Gems und spontane Trips an. In der Kommunikation stehen Abenteuerlust, Spontanität und das Entdecken neuer Reiseziele im Mittelpunkt – das Rewards-Programm macht es möglich, immer wieder neue Highlights zu erleben.

---

### Folie 11: Struktur der Kundensegmente

Unsere Segmentierung ist die Grundlage für die Personalisierung im Rewards-Programm. Jeder Kunde wird automatisch einem Segment zugeordnet – zum Beispiel Business, Solo, Couple, Family, Groups, Frequent Traveler oder Other. Die Zuordnung basiert auf Buchungsverhalten, Vorlieben und speziellen Bedürfnissen.
So kann jedem Kunden genau der Vorteil angeboten werden, der für ihn am attraktivsten ist. Das erhöht die Motivation, am Rewards-Programm teilzunehmen, und steigert die Nutzung.

### Folie 12: Kundenanzahl pro Segment und in Prozent

Hier sehen wir, wie viele Kunden in den einzelnen Segmenten vertreten sind – sowohl als absolute Zahl als auch prozentual.
Besonders auffällig: Die meisten Kunden gehören zu den Segmenten „Groups“, „Frequent Traveler“ und „Family“.
Die kleineren Segmente „Other“ und „Solo“ spielen eine untergeordnete Rolle und werden in der weiteren Ansprache weniger stark berücksichtigt.

### Folie 13: Wichtige KPIs je Segment

Der Customer Lifetime Value (CLV) misst den erwarteten Gesamtumsatz pro Kunde anhand bereinigter Hotel-Ausgaben und Buchungs-häufigkeit seit Januar 2023. Frequent Traveler stehen mit über 160 USD Ausgaben an der Spitze, Geschäftsreisende profitieren von Upgrades und Frühstück, Gruppen und Familien von Kombi-Perks wie „1 Kind reist kostenlos“. Mit dem fortlaufend aktualisierten CLV-Dashboard in Tableau steuern wir unsere Marketingkampagnen dynamisch und bieten Vielreisenden flexible Stornierungsbedingungen sowie Gruppen und Familien maßgeschneiderte Paketangebote.

### Folie 14: Perks & A/B-Tests – So setzen wir Personalisierung um

Für jedes Segment haben wir die passenden Rewards-Perks identifiziert, wie zum Beispiel:

Kostenlose Hotelmahlzeit

Kostenloses Aufgabegepäck

Keine Stornogebühren

1 Nacht kostenloses Hotelzimmer bei Flugbuchung

Exklusive Rabatte



### Folie 15: Technische Umsetzung & Controlling

Die Auswahl des Perks wird individuell auf das jeweilige Segment zugeschnitten. Über segmentierte E-Mail-Kampagnen mit dynamischem Content sprechen wir jeden Kunden gezielt mit dem wichtigsten Vorteil an.
Um den besten Effekt zu erzielen, testen wir verschiedene Varianten – zum Beispiel „kostenlose Stornierung“ gegen „Zimmer-Upgrade“ – in regelmäßigen A/B-Tests. So finden wir heraus, welche Ansprache in welchem Segment am besten funktioniert.Die technische Umsetzung erfolgt automatisiert:

Die Segmentierungsdaten und Perk-Zuweisungen fließen direkt über die Datenbank in unser E-Mail-Marketing-System und in Tableau.

Alle Kampagnen werden kontinuierlich überwacht. Wichtige KPIs sind Anmelderaten, Conversion Rates und langfristige Kundenbindung.

Über Dashboards und regelmäßige Audits sorgen wir dafür, dass das Rewards-Programm stetig optimiert wird.

So stellen wir sicher, dass Personalisierung nicht nur ein Buzzword bleibt, sondern wirklich funktioniert und echten Mehrwert für unsere Kunden und TravelTide schafft.


## Folie 16: Q & A

Ich bedanke mich für Ihre Aufmerksamkeit. Gerne beantworte ich jetzt Ihre Fragen zu allen Themen rund um unser Rewards-Programm, die Segmentierung, Personalisierung oder die technische Umsetzung. Lassen Sie uns diskutieren, wie datengetriebene Personalisierung und Automatisierung echten Mehrwert für TravelTide und unsere Kund\:innen schaffen!

---

**Hinweis:** Das Skript dient als umfassender Leitfaden und ist bewusst detailliert gehalten, damit Sie als Sprecher flexibel zwischen den Themen springen und bei Nachfragen gezielt tiefer ins Detail gehen können. Die ausführlichen Stories zu den einzelnen Segmenten unterstützen Sie dabei, die Zielgruppen anschaulich und lebendig zu erklären und das Programm praxisnah zu präsentieren.



