# EV Charging Dataset Analysis Overview  
**Author:** Vinayak Sankaranarayanan  
**Date:** 04/21/2025  

---

## Datasets

> **Citation:**  
> Stephen Makonin, Isla Ziyat, Rylen Sampson, Siyu An, Fred Popowich, Patrick Palmer, David Agosti. (2023). SFU‑EVP: SFU EV Parking Dataset. IEEE Dataport. https://dx.doi.org/10.21227/ya1w‑m583

### 1. `Sessions.csv` (66,009 rows)  
Main log of every charging session: when EVs plugged in and out, how long they ran, and how much energy was delivered.  
**Columns:**
```text
session_id, user_id, credential_id, station_id, 
port_no, start_ts, end_ts, start_dt, end_dt,
energy, total_charging_duration, 
total_session_duration, address
```

### 2. `Stations.csv` (38 rows)
Master list of all physical charging ports and their metadata (location, power specs, model, etc.).  
**Columns:**
```text
station_id, org_id, station_group, model,
activation_dt, timezone_offset, address,
manufacturer, station_name, description,
port_no, reservable, status, level,
time_stamp, mode, connector,
voltage, current, power, estimated_cost,
location_lat, location_long
```

### 3. `Alarms.csv` (17,820 rows)
Time‑stamped alarm events raised by stations/ports during charging.  
**Columns:**
```text
station_id, station_name, model, org_id,
port_no, alarm_type, alarm_ts, alarm_dt,
session_id
```

### 4. `Anomalies.csv` (247 rows)
Unusual error or anomaly records logged within charging sessions.  
**Columns:**
```text
session_id, anomaly_description, value, unit
```

---

## Dataset Structure & Integrity Analysis

1. **Compound Primary Key**  
   In `Stations.csv`, **`(station_id, port_no)`** together uniquely identify each record.
   - Each physical location has exactly **2 ports** (1 & 2) → 19 sites × 2 = 38 rows.

2. **Referential Integrity Issue**
   - **56 %** of `Sessions.csv` rows (36,974 / 66,009) reference a `station_id` **not present** in `Stations.csv`.
   - These orphaned sessions involve **13 distinct** missing station IDs.
   - Their port‑utilization patterns match valid stations, and each orphan ID appears with two ports (1 & 2).

3. **Remediation Steps**
   - **Created 26 new rows** in the stations master file (13 missing IDs × 2 ports), complete with address and geolocation.
   - Amended `Stations.csv` now contains **64 rows** → **32 physical stations**, restoring full referential integrity.
   - The enriched master file supports comprehensive analysis across **all 66,009** charging sessions and underpins the dead‑zone identification methodology.

---

## Objective
Identify reliable “deadzones” — periods when EV charging stations are consistently under‑utilised — so car‑share vehicles can be scheduled to charge without competing with public demand.

---

## Analysis Approach

1. **Utilisation‑rate calculation**  
   Utilisation Rate = (Chargers in use / Total chargers) × 100
   - Compute hourly rates for every station.
   - Aggregate by hour‑of‑day, day‑of‑week, and station — only over days when that station was active.
   - Mark any hour < X % utilisation as a candidate “deadzone.”

2. **Deadzone identification**
   - Require ≥ Y consecutive sub‑X % hours to count as a meaningful window.
   - Map these windows network‑wide (weekday vs. weekend).

3. **Reliability assessment (future)**  
   Reliability Index = (Days deadzone occurs / Total days observed) × 100
   - Rate each deadzone; prioritise those ≥ Z %.
   - Produce a ranked list of high‑reliability charging opportunities.

---

## Results

The analysis produced three sets of station‑level visuals for all 32 stations (linked in the [Slide Deck](https://github.com/vinaySankar2004/ev-charging-deadzone-analysis/blob/main/docs/Deadzone%20Analysis%20Results.pdf)):

1. **Station Heatmaps**
   - A 7×24 heatmap per station (hours × days), colored by average utilization rate.
   - Reveals the full daily & weekly usage pattern.

2. **Dot‑Calendar (Threshold = 6 %)**
   - Applied a 6 % cutoff (≈ 20th percentile of all 3 hr windows over 2019–2024) and extracted the five lowest‑utilization 3 hr blocks for weekdays vs. weekends.
   - Red circles = weekday, blue squares = weekend.
   - Each dot is annotated with its utilization rate.

3. **Dot‑Calendar (No Threshold)**
   - Selected the five absolute lowest‑utilization 3 hr windows per station (weekdays vs. weekends), regardless of threshold.
   - Guarantees exactly five records per station in each category.

🔗 **Git Repository**  
Git repo containing source code as well as a full slide deck of results with one page per station (32 slides) is available here: https://github.com/vinaySankar2004/ev-charging-deadzone-analysis

---

## Future Work

- **Reliability assessment:** implement step 3 to compute and rank deadzone persistence over time.
- **Year‑by‑year analysis:** evaluate how deadzone patterns evolve annually to spot seasonal or operational shifts.

---

## Station Address Reference

| station_id   | address                                                                                   |
|--------------|-------------------------------------------------------------------------------------------|
| 4573a9ca18   | 8888 University Drive                                                                     |
| 047effd886   | 8888 University Drive                                                                     |
| bddc428b79   | 8888 University Drive                                                                     |
| 877bdb5fe8   | 8888 University Drive                                                                     |
| 0342d90fcf   | 8888 University Drive                                                                     |
| 3488aacebc   | 8888 University Drive                                                                     |
| 7543409772   | 8888 University Drive                                                                     |
| 18173fd803   | 8888 University Drive                                                                     |
| a9eb1fa250   | 8888 University Drive                                                                     |
| 0b22b7c266   | 8888 University Drive                                                                     |
| bc1fdb94c5   | 8888 University Drive                                                                     |
| 1ff7182ea0   | 8888 University Drive                                                                     |
| 4a34bc74b0   | 8888 University Drive                                                                     |
| 8d245803d0   | 8999 Nelson Way                                                                           |
| 44a62bb258   | 8911 Cornerstone Mews                                                                     |
| adcfa1983d   | 8888 University Drive                                                                     |
| f605e53f24   | 8999 Nelson Way                                                                           |
| 2f3eaeac89   | 13419 103 Ave                                                                             |
| 2f753a1a41   | 13409 102a Ave                                                                            |
| 0dc8ed1cc1   | 8915 Cornerstone Mews, Burnaby, British Columbia, V5A 4Y6, Canada                         |
| 162185ed79   | 8999 Nelson Way, Burnaby, British Columbia, V5G 4N2, Canada                               |
| 23ad8d0095   | 8888 University Drive, Burnaby, British Columbia, V5A1S6, Canada                          |
| 615db15a48   | 13419 103 Ave, Surrey, British Columbia, V3T 1S6, Canada                                  |
| 87adf1b6d0   | 13419 103 Ave, Floor P1, Surrey, British Columbia, V3T 1R7, Canada                        |
| 00c4ca50ea   | 8888 University Dr W, Floor P7000, Burnaby, British Columbia, Canada                     |
| 01335ad401   | 8888 University Dr W, Burnaby, British Columbia, Canada                                  |
| 18db03c780   | 8888 University Dr W, Burnaby, British Columbia, Canada                                  |
| 4ce863e612   | 8888 University Dr W, Burnaby, British Columbia, Canada                                  |
| 956028aa4c   | 8888 University Dr W, Floor P7000, Burnaby, British Columbia, Canada                     |
| a9571abfa4   | 8888 University Dr W, Floor P7000, Burnaby, British Columbia, Canada                     |
| a96c3b900c   | 8888 University Dr W, Floor P7000, Burnaby, British Columbia, Canada                     |
| bba10a9daf   | 8999 Nelson Way, Burnaby, British Columbia, V5A 4W9, Canada                               |


