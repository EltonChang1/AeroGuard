# AeroGuard Build Resource Specification for Coding Agent

**Document purpose:**  
This file converts the AeroGuard startup resource research into a coding-agent-friendly implementation specification. It is designed to help a coding agent build the MVP without needing to reinterpret long strategy prose.

**Recommended product direction:**  
AeroGuard should be a **software-first pre-flight risk intelligence and compliance binder platform for commercial drone operators**, especially infrastructure inspection operators.

**Core MVP promise:**  

> Upload drone logs, create a mission, select drone/battery/pilot, check weather and airspace, calculate mission risk, explain the risk, recommend safer actions, and generate a professional compliance binder.

---

## 0. Coding Agent Instructions

Build this as a modular SaaS MVP.

Prioritize:

1. Data ingestion
2. Normalized schema
3. Rule-based risk engine
4. Weather and airspace checks
5. Fleet readiness dashboard
6. Mission risk report
7. PDF compliance binder

Do **not** build:

- Drone autopilot
- Real-time flight control
- Detect-and-avoid system
- Full UTM platform
- Custom hardware
- Insurance underwriting engine
- Full BVLOS certification workflow

Use advisory language only. Do not use claims like:

- “Safe”
- “Approved”
- “FAA compliant”
- “Crash prevention guaranteed”

Use language like:

- “Risk advisory”
- “Recommended review”
- “Risk factors detected”
- “Compliance documentation support”
- “Human approval required”

---

## 1. Product Definition

### 1.1 Product Name

Working name: **AeroGuard**

### 1.2 Product Category

Commercial drone operations safety, mission-risk intelligence, and compliance documentation platform.

### 1.3 Target Customer

Initial customer segment:

- Small-to-mid-sized drone service providers
- 5–50 drones
- Infrastructure inspection focus

Priority verticals:

- Utility / power-line inspection
- Solar farm inspection
- Telecom tower inspection
- Pipeline inspection
- Railway inspection
- Construction monitoring
- Industrial facility inspection

### 1.4 Core User Roles

| Role | Description | Permissions |
|---|---|---|
| Admin | Company owner / operations lead | Manage org, users, assets, all missions |
| Operations Manager | Plans missions and assigns assets | Create missions, assign drones/batteries/pilots, review risk |
| Pilot | Executes missions | View assigned missions, complete checklist, upload logs |
| Safety/Compliance Manager | Reviews safety and records | Review binders, approve/flag missions, export reports |
| Viewer/Client | Optional external viewer | Read-only access to reports |

---

## 2. MVP User Flow

### 2.1 Main Demo Flow

1. User logs into AeroGuard.
2. User uploads historical flight logs.
3. System parses logs and detects anomalies.
4. User sees fleet readiness dashboard:
   - Drones
   - Batteries
   - Pilots
   - Missions
5. User creates a new mission:
   - Location
   - Route
   - Planned date/time
   - Mission type
   - Altitude
   - Drone
   - Battery
   - Pilot
6. System retrieves weather data.
7. System checks airspace data.
8. System calculates mission risk.
9. System explains risk factors.
10. System recommends safer actions.
11. User completes pre-flight checklist.
12. System generates compliance binder PDF.
13. User uploads post-flight log.
14. System generates post-flight anomaly report.

---

## 3. MVP Modules

## 3.1 Authentication and Organizations

### Features

- User signup/login
- Organization creation
- Role-based permissions
- Multi-user team support

### Suggested Implementation

- Use Clerk, Auth.js, Supabase Auth, or custom JWT auth.
- Each user belongs to one or more organizations.
- Every mission, drone, battery, pilot, and report is scoped to an organization.

### Acceptance Criteria

- Users can log in.
- Users can create or join an organization.
- Users can only see their organization’s data.
- Admins can invite users.

---

## 3.2 Fleet Asset Management

### Entities

- Drones
- Batteries
- Pilots
- Payloads
- Maintenance records

### Drone Fields

```text
id
organization_id
internal_name
manufacturer
model
serial_number
remote_id_status
remote_id_serial
registration_number
max_flight_time_min
max_hover_time_min
max_wind_resistance_mps
max_gust_resistance_mps
operating_temp_min_c
operating_temp_max_c
ip_rating
max_payload_kg
battery_model
rtk_supported
obstacle_sensing
transmission_range_km
max_takeoff_altitude_m
status
created_at
updated_at
```

### Battery Fields

```text
id
organization_id
internal_name
manufacturer
model
serial_number
compatible_drone_models
cycle_count
state_of_health_percent
full_charge_capacity_mah
nominal_capacity_mah
last_voltage_v
min_voltage_recent_v
max_temperature_recent_c
cell_imbalance_mv
last_used_at
status
created_at
updated_at
```

### Pilot Fields

```text
id
organization_id
name
email
phone
certificate_id
part_107_current
certificate_expiration_date
night_ops_allowed
bvlos_training_status
last_flight_at
total_flight_hours
status
created_at
updated_at
```

### Maintenance Record Fields

```text
id
organization_id
asset_type
asset_id
issue_title
issue_description
severity
status
opened_at
resolved_at
assigned_to
notes
created_at
updated_at
```

### Asset Status Values

```text
READY
WARNING
GROUNDED
IN_MAINTENANCE
RETIRED
UNKNOWN
```

### Acceptance Criteria

- Admin can create/edit/archive drones.
- Admin can create/edit/archive batteries.
- Admin can create/edit/archive pilots.
- Users can add maintenance records.
- Assets with unresolved high-severity maintenance are marked not ready.

---

## 3.3 Flight Log Upload and Parsing

### MVP Supported Inputs

Prioritize in this order:

1. CSV telemetry upload
2. PX4 ULog upload
3. ArduPilot DataFlash log upload
4. Manual sample/demo dataset import

### Useful Parsing Libraries

| Format | Library / Tool |
|---|---|
| PX4 ULog | `pyulog` |
| MAVLink / ArduPilot | `pymavlink`, `mavlogdump.py` |
| CSV | pandas / polars |
| Geo | GeoPandas / Shapely |

### Normalized Telemetry Table

All parsers should convert logs into a shared format.

```text
id
flight_log_id
timestamp
relative_time_sec
latitude
longitude
altitude_m
agl_altitude_m
ground_speed_mps
vertical_speed_mps
heading_deg
roll_deg
pitch_deg
yaw_deg
accel_x
accel_y
accel_z
gyro_x
gyro_y
gyro_z
gps_fix_type
gps_satellites
gps_hdop
gps_vdop
battery_voltage_v
battery_current_a
battery_percent
battery_temperature_c
battery_cell_min_v
battery_cell_max_v
motor_output_1
motor_output_2
motor_output_3
motor_output_4
flight_mode
warning_code
warning_text
created_at
```

### Flight Log Metadata Table

```text
id
organization_id
mission_id
drone_id
battery_id
pilot_id
source_format
original_filename
storage_url
parsed_status
start_time
end_time
duration_sec
max_altitude_m
max_speed_mps
min_battery_voltage_v
max_battery_temperature_c
gps_warning_count
battery_warning_count
anomaly_count
created_at
updated_at
```

### Parser Behavior

For every upload:

1. Store raw file in object storage.
2. Create `flight_logs` row.
3. Run parsing job.
4. Normalize data into `telemetry_points`.
5. Generate summary metrics.
6. Run anomaly detection.
7. Update parsed status.

### Parser Status Values

```text
PENDING
PROCESSING
PARSED
FAILED
PARTIALLY_PARSED
```

### Acceptance Criteria

- User can upload CSV telemetry.
- System stores original file.
- System normalizes telemetry into database.
- System shows flight path on map.
- System calculates summary metrics.
- System detects at least basic anomalies.

---

## 3.4 Public Demo Dataset Loader

Create seed/demo import commands for:

### Public / Open Sources to Use

| Dataset | Use |
|---|---|
| PX4 public flight logs | Normal flight telemetry |
| RflyMAD | Multicopter faults and anomaly examples |
| ALFA UAV fault dataset | Labeled fault/anomaly examples |
| UAV waypoint/trajectory datasets | Mission replay and route deviation demo |
| NASA battery aging dataset | Battery degradation modeling reference |
| CALCE battery dataset | Battery SOH/SOC modeling reference |

### Seed Command

Implement:

```bash
python scripts/seed_demo_data.py
```

This should create:

```text
demo organization
demo users
demo drones
demo batteries
demo pilots
demo missions
demo routes
demo flight logs
demo maintenance records
demo risk scores
demo reports
```

### Acceptance Criteria

- Fresh database can be populated with demo data.
- Demo data includes at least:
  - 10 drones
  - 30 batteries
  - 5 pilots
  - 20 missions
  - 50 flight logs
  - 10 risky/anomalous flights
  - 10 generated risk reports

---

## 3.5 Mission Planner

### Mission Fields

```text
id
organization_id
mission_name
customer_name
site_name
mission_type
planned_start_time
planned_end_time
timezone
route_id
planned_altitude_m
max_altitude_m
expected_duration_min
assigned_drone_id
assigned_battery_id
assigned_pilot_id
payload_id
bvlos_flag
night_operation_flag
controlled_airspace_flag
operations_over_people_flag
remote_id_required
laanc_required
laanc_status
status
created_by
created_at
updated_at
```

### Mission Types

```text
POWER_LINE_INSPECTION
SOLAR_FARM_INSPECTION
TELECOM_TOWER_INSPECTION
PIPELINE_INSPECTION
RAILWAY_INSPECTION
CONSTRUCTION_MONITORING
INDUSTRIAL_SITE_INSPECTION
MAPPING_SURVEY
PUBLIC_SAFETY_SUPPORT
OTHER
```

### Mission Status Values

```text
DRAFT
RISK_REVIEW_REQUIRED
READY_FOR_HUMAN_REVIEW
APPROVED_BY_OPERATOR
BLOCKED
IN_PROGRESS
COMPLETED
CANCELLED
ARCHIVED
```

### Mission Planner UI

Required fields:

- Mission name
- Site/location
- Planned time
- Mission type
- Route or point location
- Altitude
- Expected duration
- Drone
- Battery
- Pilot

Optional fields:

- Customer
- Payload
- BVLOS flag
- Night operation flag
- Operations over people flag
- Notes

### Acceptance Criteria

- User can create mission.
- User can assign drone, battery, pilot.
- User can draw or upload route.
- System checks basic completeness.
- System triggers risk calculation.

---

## 3.6 Route and Geospatial Engine

### Route Fields

```text
id
organization_id
name
geometry_geojson
start_latitude
start_longitude
end_latitude
end_longitude
length_km
max_planned_altitude_m
min_planned_altitude_m
site_boundary_geojson
created_at
updated_at
```

### Suggested Tools

| Need | Tool |
|---|---|
| Spatial DB | PostgreSQL + PostGIS |
| Server-side geometry | Shapely / GeoPandas |
| Frontend map | Mapbox GL JS or Leaflet |
| Open map data | OpenStreetMap |
| Route operations | Turf.js |

### Geospatial Checks

Implement checks:

```text
route_intersects_controlled_airspace
route_near_airport
route_exceeds_uas_facility_map_altitude
route_exceeds_drone_max_takeoff_altitude
route_crosses_site_boundary
route_length_too_long_for_battery
route_has_high_wind_segment
```

### Acceptance Criteria

- Mission route can be stored as GeoJSON.
- Route can be displayed on map.
- System can intersect route with airspace polygons.
- System can calculate approximate route length.
- System can flag route/altitude conflicts.

---

## 3.7 Weather Integration

### MVP Weather Sources

Use these first:

1. National Weather Service API
2. NOAA Aviation Weather Center API
3. Open-Meteo API

### Later Weather Sources

1. Tomorrow.io
2. Meteomatics
3. WeatherFlow Tempest
4. Davis WeatherLink
5. Ambient Weather
6. Kestrel CSV/manual reading

### Weather Snapshot Table

```text
id
organization_id
mission_id
source
request_latitude
request_longitude
requested_for_time
observation_time
forecast_time
temperature_c
humidity_percent
pressure_hpa
wind_speed_mps
wind_gust_mps
wind_direction_deg
visibility_m
precipitation_mm
cloud_cover_percent
weather_code
alert_status
raw_response_json
created_at
```

### Weather Risk Rules

Implement basic rules:

```text
if wind_gust_mps > drone.max_wind_resistance_mps:
    high weather risk

if wind_speed_mps > 0.8 * drone.max_wind_resistance_mps:
    medium weather risk

if precipitation_mm > 0 and drone.ip_rating is weak/unknown:
    medium/high weather risk

if temperature_c < drone.operating_temp_min_c:
    high weather risk

if temperature_c > drone.operating_temp_max_c:
    high weather risk

if visibility_m < threshold:
    warning

if active severe weather alert:
    high weather risk
```

### Acceptance Criteria

- System fetches weather for mission location/time.
- Weather snapshot is stored.
- Weather data is shown in mission risk panel.
- Weather risk contributes to overall risk score.
- Raw weather response is retained for audit.

---

## 3.8 Airspace and Compliance Integration

### MVP Airspace Sources

| Source | Use |
|---|---|
| FAA UAS Data Delivery System | Airspace and UAS geospatial data |
| FAA UAS Facility Maps | Max altitude around airports |
| FAA NASR / airport data | Airport proximity and context |
| FAA Remote ID DOC database | Drone/module Remote ID compliance reference |
| eCFR Part 107 | Rule reference |
| FAA Remote ID page | Compliance reference |

### Airspace Snapshot Table

```text
id
organization_id
mission_id
source
route_id
controlled_airspace_intersection
uas_facility_map_max_altitude_m
planned_altitude_m
airport_distance_km
nearest_airport_id
nearest_airport_name
laanc_required
laanc_status
remote_id_required
remote_id_status
night_operation_flag
bvlos_flag
operations_over_people_flag
raw_response_json
created_at
```

### Compliance Checklist Table

```text
id
organization_id
mission_id
checklist_template_id
item_key
item_label
item_description
required
completed
completed_by
completed_at
evidence_url
notes
created_at
updated_at
```

### Default Compliance Checklist Items

```text
pilot_certificate_confirmed
drone_registration_confirmed
remote_id_confirmed
battery_health_reviewed
maintenance_status_reviewed
weather_reviewed
airspace_reviewed
laanc_authorization_confirmed_if_required
route_reviewed
site_permission_confirmed
emergency_procedure_reviewed
visual_observer_assigned_if_required
night_operation_requirements_reviewed_if_applicable
operations_over_people_reviewed_if_applicable
human_operator_final_review
```

### Acceptance Criteria

- System stores airspace snapshot per mission.
- System flags controlled airspace / altitude issue.
- System flags missing LAANC if required.
- System flags Remote ID missing/unknown.
- System generates checklist items based on mission type.
- Checklist completion is included in PDF binder.

---

## 3.9 Risk Engine

### MVP Principle

Start rule-based. Do not start with complex ML.

The risk engine should be:

- Transparent
- Explainable
- Auditable
- Easy to override by human operator
- Modular

### Risk Categories

```text
weather_risk
airspace_risk
compliance_risk
battery_risk
drone_health_risk
pilot_readiness_risk
maintenance_risk
mission_complexity_risk
```

### Risk Score Table

```text
id
organization_id
mission_id
overall_score
overall_level
weather_score
airspace_score
compliance_score
battery_score
drone_health_score
pilot_readiness_score
maintenance_score
mission_complexity_score
model_version
scoring_method
human_override_status
human_override_reason
created_at
updated_at
```

### Risk Reason Table

```text
id
risk_score_id
category
severity
reason_code
reason_text
data_source
recommended_action
confidence_level
created_at
```

### Score Levels

```text
0-24 = LOW
25-49 = MODERATE
50-74 = HIGH
75-100 = CRITICAL
```

### MVP Weighting

```text
overall_score =
  0.25 * battery_score
+ 0.20 * drone_health_score
+ 0.20 * weather_score
+ 0.15 * airspace_score
+ 0.10 * compliance_score
+ 0.05 * maintenance_score
+ 0.05 * pilot_readiness_score
```

### Example Battery Risk Rules

```text
+20 if cycle_count > configured_threshold
+25 if voltage_sag_detected
+15 if max_temperature_recent_c > threshold
+15 if battery used in recent anomalous flight
+20 if cell_imbalance_mv > threshold
+30 if unresolved maintenance record exists
+20 if battery status != READY
```

### Example Drone Health Rules

```text
+25 if recent GPS instability detected
+25 if recent IMU anomaly detected
+20 if repeated warning codes in last N flights
+30 if unresolved high-severity maintenance issue
+15 if abnormal vibration/oscillation detected
+20 if hard landing detected in recent logs
```

### Example Weather Risk Rules

```text
+35 if gust exceeds drone wind resistance
+25 if sustained wind > 80% of drone wind resistance
+20 if precipitation and drone IP rating insufficient
+20 if temperature outside drone operating range
+15 if visibility below threshold
+30 if severe weather alert active
```

### Example Airspace Risk Rules

```text
+35 if controlled airspace and LAANC missing
+25 if planned altitude exceeds UAS Facility Map altitude
+20 if route near airport
+30 if BVLOS flag true and no BVLOS authorization record
+20 if night operation and night requirements unchecked
```

### Example Compliance Risk Rules

```text
+25 if Remote ID status unknown/missing
+20 if pilot certificate not current
+20 if drone registration missing
+20 if required checklist incomplete
+15 if maintenance review incomplete
+15 if weather snapshot missing
+15 if airspace snapshot missing
```

### Required Risk Output Format

Every risk output must include:

```text
overall risk score
risk level
risk category breakdown
plain-English explanation
recommended actions
data sources used
timestamp
human review status
```

### Acceptance Criteria

- System calculates risk for every mission.
- Risk is broken down by category.
- Every nonzero risk has explanation.
- Every high/critical risk has recommended action.
- Human can approve/override with reason.
- Risk output can be exported into PDF.

---

## 3.10 Recommendation Engine

### Recommendation Table

```text
id
organization_id
mission_id
risk_score_id
priority
category
recommendation_text
related_asset_type
related_asset_id
status
created_at
updated_at
```

### Recommendation Status

```text
OPEN
ACCEPTED
DISMISSED
COMPLETED
SUPERSEDED
```

### Example Recommendations

| Trigger | Recommendation |
|---|---|
| Battery voltage sag | Use a different battery or inspect before mission |
| Wind gust too high | Delay mission or reduce route length |
| Controlled airspace | Complete LAANC authorization before mission |
| Drone GPS instability | Use another drone or avoid long-range mission |
| Missing Remote ID | Confirm Remote ID before takeoff |
| Pilot certificate expired | Reassign pilot |
| Maintenance issue unresolved | Ground asset until issue resolved |
| Route too long | Use fresh battery or split mission into segments |

### Acceptance Criteria

- Every high-risk factor creates a recommendation.
- User can accept/dismiss/complete recommendations.
- Recommendation status affects mission readiness.
- Recommendations are included in compliance binder.

---

## 3.11 Post-Flight Anomaly Detection

### MVP Anomalies

Implement simple rule-based detectors:

```text
battery_voltage_sag
battery_temperature_high
gps_signal_drop
gps_satellite_count_low
route_deviation
altitude_exceeded_plan
hard_landing
excessive_roll_pitch
speed_spike
communication_dropout
warning_code_detected
unexpected_return_to_home
```

### Anomaly Table

```text
id
organization_id
flight_log_id
mission_id
category
severity
start_time
end_time
timestamp
description
detected_value
threshold_value
recommended_action
created_at
```

### Example Anomaly Logic

```text
battery_voltage_sag:
    detect steep voltage drop over short time window

gps_signal_drop:
    gps_satellites below threshold or gps_fix_type degraded

route_deviation:
    telemetry point distance from planned route > threshold

hard_landing:
    high vertical acceleration near landing timestamp

excessive_roll_pitch:
    abs(roll) or abs(pitch) above configured threshold

altitude_exceeded_plan:
    altitude_m > mission.max_altitude_m
```

### Acceptance Criteria

- Uploaded flight logs are scanned for anomalies.
- Anomalies appear on flight detail page.
- Anomalies are attached to post-flight report.
- High-severity anomalies can create maintenance recommendations.

---

## 3.12 Compliance Binder PDF

### PDF Sections

1. Cover page
2. Mission summary
3. Assigned pilot/drone/battery
4. Route/map snapshot
5. Weather snapshot
6. Airspace snapshot
7. Remote ID / registration evidence
8. Pre-flight checklist
9. Risk score and explanations
10. Recommendations and operator decisions
11. Maintenance status
12. Post-flight anomaly report
13. Human approval / override notes
14. Appendix: raw data source references

### PDF Report Table

```text
id
organization_id
mission_id
report_type
status
file_url
generated_by
generated_at
created_at
updated_at
```

### Suggested Libraries

| Language | Library |
|---|---|
| Python | WeasyPrint |
| Python | ReportLab |
| Node | Playwright PDF |
| Node | Puppeteer |

### Acceptance Criteria

- User can generate PDF for mission.
- PDF includes all mission evidence.
- PDF has professional formatting.
- PDF can be downloaded.
- PDF is stored with mission record.

---

## 4. Data Sources and APIs

## 4.1 Public Flight / Anomaly Datasets

| Resource | URL | Use |
|---|---|---|
| PX4 Flight Review / logs | https://docs.px4.io/main/en/dev_log/flight_log_analysis_statistical | Public flight logs and telemetry examples |
| PX4 ULog format | https://docs.px4.io/main/en/dev_log/ulog_file_format | Log format reference |
| pyulog | https://github.com/PX4/pyulog | Parse PX4 ULog |
| RflyMAD | https://rfly-openha.github.io/documents/4_resources/dataset.html | Multicopter anomaly/fault data |
| ALFA UAV fault dataset | https://kilthub.cmu.edu/articles/dataset/ALFA_A_Dataset_for_UAV_Fault_and_Anomaly_Detection/12707963 | Labeled UAV fault data |
| MAVLink | https://mavlink.io/en/ | Drone telemetry protocol |
| ArduPilot logs | https://ardupilot.org/dev/docs/common-logs.html | ArduPilot log reference |
| pymavlink | https://github.com/ArduPilot/pymavlink | MAVLink parsing |

## 4.2 Weather APIs

| Resource | URL | Use |
|---|---|---|
| NOAA Aviation Weather Center API | https://aviationweather.gov/data/api/ | METAR, TAF, aviation weather |
| National Weather Service API | https://www.weather.gov/documentation/services-web-api | Forecasts, alerts, observations |
| Open-Meteo | https://open-meteo.com/en/docs | Forecast and historical weather |
| Tomorrow.io | https://www.tomorrow.io/weather-api/ | Commercial hyperlocal weather |
| Meteomatics | https://www.meteomatics.com/en/api/ | Enterprise weather API |

## 4.3 Airspace / FAA / Compliance

| Resource | URL | Use |
|---|---|---|
| FAA UAS Data Delivery System | https://udds-faa.opendata.arcgis.com/ | UAS data in CSV/JSON/KML/Shapefile |
| FAA UAS Facility Maps | https://www.faa.gov/uas/commercial_operators/uas_facility_maps | UAS altitude authorization context |
| FAA Remote ID | https://www.faa.gov/uas/getting_started/remote_id | Remote ID compliance reference |
| FAA DroneZone | https://faadronezone-access.faa.gov/ | Registration / drone services portal |
| eCFR Part 107 | https://www.ecfr.gov/current/title-14/chapter-I/subchapter-F/part-107 | Commercial UAS rule reference |
| FAA NASR Subscription | https://www.faa.gov/air_traffic/flight_info/aeronav/aero_data/NASR_Subscription/ | Airports, airspace, navaids |
| FAA Digital Products | https://www.faa.gov/air_traffic/flight_info/aeronav/digital_products | Charts and digital data |
| Federal Register BVLOS Part 108 NPRM | https://www.federalregister.gov/documents/2025/08/07/2025-14992/normalizing-unmanned-aircraft-systems-beyond-visual-line-of-sight-operations | Future BVLOS rule direction |
| FAA BVLOS page | https://www.faa.gov/newsroom/beyond-visual-line-sight-bvlos | BVLOS overview |

## 4.4 Safety / Incident Data

| Resource | URL | Use |
|---|---|---|
| NASA ASRS UAS Safety | https://asrs.arc.nasa.gov/uassafety.html | UAS safety reports |
| FAA UAS Sightings | https://www.faa.gov/uas/resources/public_records/uas_sightings_report | UAS sightings near airports |
| NTSB Aviation Investigation Search | https://www.ntsb.gov/Pages/AviationQuery.aspx | Aviation accident/incident research |
| FAA AIDS | https://www.asias.faa.gov/apex/f?p=100:12 | FAA incident data |

## 4.5 Battery Data

| Resource | URL | Use |
|---|---|---|
| NASA Li-ion Battery Aging Dataset | https://data.nasa.gov/dataset/li-ion-battery-aging-datasets | Battery aging / RUL reference |
| CALCE Battery Data | https://calce.umd.edu/battery-data | Battery SOH/SOC data |
| BatteryML | https://github.com/microsoft/BatteryML | Battery ML modeling framework |
| Battery Archive | https://www.batteryarchive.org/ | Battery research data |

## 4.6 Commercial Drone / Fleet APIs

| Platform | URL | Use |
|---|---|---|
| DJI Developer | https://developer.dji.com/ | DJI SDKs/APIs |
| DJI Mobile SDK Android | https://github.com/dji-sdk/Mobile-SDK-Android | DJI mobile integration |
| DJI Cloud API | https://developer.dji.com/cloud-api/ | DJI enterprise cloud integration |
| Skydio Cloud | https://support.skydio.com/ | Skydio fleet/cloud integration |
| AirData API | https://app.airdata.com/docs/api/ | Import customer flight/battery data |
| DroneLogbook | https://www.dronelogbook.com/ | Compliance/fleet workflow integration |
| DroneDeploy API | https://developer.dronedeploy.com/ | Enterprise drone operations integration |
| Aloft | https://www.aloft.ai/ | Airspace/LAANC/fleet integration |

## 4.7 Map / Terrain / Geospatial

| Resource | URL | Use |
|---|---|---|
| Mapbox API | https://docs.mapbox.com/api/ | Maps, geocoding, route UI |
| OpenStreetMap | https://www.openstreetmap.org/ | Open map data |
| Overpass API | https://wiki.openstreetmap.org/wiki/Overpass_API | Query OSM features |
| Cesium ion | https://cesium.com/learn/ion/rest-api/ | 3D maps/terrain later |
| USGS National Map | https://www.usgs.gov/programs/national-geospatial-program/national-map | Elevation/terrain data |
| PostGIS | https://postgis.net/ | Geospatial database |

## 4.8 Local Weather Hardware APIs

| Resource | URL | Use |
|---|---|---|
| WeatherFlow Tempest API | https://weatherflow.github.io/Tempest/ | Local weather station data |
| Davis WeatherLink API | https://weatherlink.github.io/v2-api/ | Professional weather station data |
| Ambient Weather API | https://ambientweather.docs.apiary.io/ | Weather station data |
| Kestrel LiNK | https://kestrelinstruments.com/kestrel-link | Field wind meter data/manual import |
| Gill WindSonic | https://gillinstruments.com/ | Rugged ultrasonic anemometer reference |

---

## 5. Drone Specs Starter Database

Create `drone_specs_seed.csv` with these models.

### 5.1 Required Spec Fields

```text
manufacturer
model
category
best_use_case
max_flight_time_min
max_hover_time_min
max_wind_resistance_mps
max_gust_resistance_mps
operating_temp_min_c
operating_temp_max_c
ip_rating
max_payload_kg
battery_model
rtk_supported
obstacle_sensing
transmission_range_km
max_takeoff_altitude_m
remote_id_capable
source_url
```

### 5.2 Seed Drone Models

| Drone | Category | Best Use Case |
|---|---|---|
| DJI Matrice 350 RTK | Enterprise multirotor | Heavy inspection, LiDAR, thermal |
| DJI Matrice 4 Series | Enterprise compact multirotor | Public safety, inspection |
| DJI Mavic 3 Enterprise | Compact enterprise multirotor | Mapping, inspection |
| Skydio X10 / X10D | Enterprise autonomy drone | Public safety, infrastructure |
| Autel EVO Max 4T | Enterprise multirotor | Thermal/public safety/inspection |
| WingtraOne | VTOL fixed-wing | Surveying and large-area mapping |
| senseFly / AgEagle eBee X | Fixed-wing | Agriculture, mapping, large sites |
| Freefly Astro Max | Enterprise payload drone | NDAA-friendly mapping/inspection |

### 5.3 Spec Source URLs

```text
DJI Matrice 350 RTK:
https://enterprise.dji.com/matrice-350-rtk/specs

DJI Mavic 3 Enterprise:
https://enterprise.dji.com/mavic-3-enterprise/specs

Skydio X10:
https://www.skydio.com/x10/technical-specs

WingtraOne:
https://wingtra.com/mapping-drone-wingtraone/technical-specifications/

Autel EVO Max 4T:
https://www.autelrobotics.com/productdetail/evo-max-4t.html

Freefly Astro:
https://freeflysystems.com/astro
```

Important: drone specifications may change by region, firmware, payload, battery condition, or operating mode. Store source URL and timestamp for every seeded spec.

---

## 6. Database Schema Overview

Use PostgreSQL + PostGIS.

### Core Tables

```text
users
organizations
organization_members
drones
batteries
pilots
payloads
maintenance_records
missions
routes
weather_snapshots
airspace_snapshots
compliance_checklists
flight_logs
telemetry_points
anomalies
risk_scores
risk_reasons
recommendations
post_flight_reports
pdf_reports
audit_events
```

### Audit Events Table

```text
id
organization_id
user_id
entity_type
entity_id
action
before_json
after_json
created_at
```

Use this to preserve compliance traceability.

---

## 7. Suggested API Endpoints

Use REST for MVP.

### Auth / Org

```text
POST /api/auth/login
POST /api/auth/logout
GET  /api/me
GET  /api/orgs/current
POST /api/orgs
POST /api/orgs/{org_id}/invite
```

### Assets

```text
GET    /api/drones
POST   /api/drones
GET    /api/drones/{id}
PATCH  /api/drones/{id}
DELETE /api/drones/{id}

GET    /api/batteries
POST   /api/batteries
GET    /api/batteries/{id}
PATCH  /api/batteries/{id}
DELETE /api/batteries/{id}

GET    /api/pilots
POST   /api/pilots
GET    /api/pilots/{id}
PATCH  /api/pilots/{id}
DELETE /api/pilots/{id}
```

### Missions

```text
GET   /api/missions
POST  /api/missions
GET   /api/missions/{id}
PATCH /api/missions/{id}
POST  /api/missions/{id}/calculate-risk
POST  /api/missions/{id}/generate-binder
POST  /api/missions/{id}/approve
POST  /api/missions/{id}/override-risk
```

### Flight Logs

```text
GET  /api/flight-logs
POST /api/flight-logs/upload
GET  /api/flight-logs/{id}
GET  /api/flight-logs/{id}/telemetry
GET  /api/flight-logs/{id}/anomalies
POST /api/flight-logs/{id}/reparse
```

### Weather / Airspace

```text
POST /api/weather/snapshot
GET  /api/missions/{id}/weather

POST /api/airspace/check
GET  /api/missions/{id}/airspace
```

### Reports

```text
GET  /api/reports
GET  /api/reports/{id}
POST /api/reports/{id}/download-url
```

---

## 8. Frontend Pages

### 8.1 Dashboard

Route:

```text
/app/dashboard
```

Show:

- Total missions
- Missions needing review
- Drones ready/warning/grounded
- Batteries ready/warning/grounded
- Recent high-risk findings
- Recent uploaded logs
- Quick action: Create mission
- Quick action: Upload log

### 8.2 Fleet Readiness

Route:

```text
/app/fleet
```

Tabs:

- Drones
- Batteries
- Pilots
- Maintenance

Each row should show:

```text
Asset name
Status
Last used
Risk warnings
Open maintenance
Recommended action
```

### 8.3 Mission List

Route:

```text
/app/missions
```

Columns:

```text
Mission name
Site
Planned time
Mission type
Assigned drone
Assigned pilot
Risk score
Status
Actions
```

### 8.4 Mission Detail

Route:

```text
/app/missions/[id]
```

Sections:

1. Mission summary
2. Route map
3. Assigned assets
4. Weather panel
5. Airspace panel
6. Risk score panel
7. Risk explanations
8. Recommendations
9. Checklist
10. Flight logs
11. Reports

### 8.5 Flight Log Detail

Route:

```text
/app/flight-logs/[id]
```

Sections:

1. Summary metrics
2. Route replay map
3. Telemetry charts
4. Battery chart
5. GPS quality chart
6. Anomaly list
7. Raw parser metadata

### 8.6 Reports

Route:

```text
/app/reports
```

Show:

- Compliance binders
- Post-flight anomaly reports
- Fleet risk reports
- Download PDF buttons

---

## 9. UI Design Requirements

Style:

- Clean SaaS dashboard
- Aviation command-center feel
- Minimal but information-dense
- Red/yellow/green status badges
- Professional PDF output
- Clear explanations, not black-box AI language

Status colors:

```text
LOW risk: green
MODERATE risk: yellow
HIGH risk: orange
CRITICAL risk: red
UNKNOWN: gray
```

Key UI principle:

> User should understand “what can fly today” within 10 seconds.

---

## 10. Suggested Repository Structure

```text
aeroguard/
  apps/
    web/
      app/
      components/
      lib/
      styles/
    api/
      app/
      routers/
      services/
      models/
      db/
      jobs/
      parsers/
      risk/
      reports/
  packages/
    shared/
      types/
      constants/
      schemas/
  scripts/
    seed_demo_data.py
    import_px4_logs.py
    import_ardupilot_logs.py
    import_airspace_data.py
    import_drone_specs.py
  data/
    seed/
      drone_specs_seed.csv
      demo_drones.csv
      demo_batteries.csv
      demo_pilots.csv
      demo_missions.csv
    samples/
      telemetry_csv/
      px4_logs/
      ardupilot_logs/
  docs/
    api_sources.md
    risk_engine.md
    compliance_binder.md
    deployment.md
  docker-compose.yml
  README.md
```

---

## 11. Environment Variables

```bash
DATABASE_URL=
POSTGIS_DATABASE_URL=
REDIS_URL=
S3_BUCKET=
S3_REGION=
S3_ACCESS_KEY_ID=
S3_SECRET_ACCESS_KEY=

NWS_USER_AGENT=
OPEN_METEO_BASE_URL=https://api.open-meteo.com
NOAA_AVIATION_WEATHER_BASE_URL=https://aviationweather.gov/api/data

MAPBOX_ACCESS_TOKEN=

AUTH_SECRET=
CLERK_SECRET_KEY=
CLERK_PUBLISHABLE_KEY=

PDF_STORAGE_BUCKET=
```

---

## 12. MVP Build Milestones

## Milestone 1: Foundation

Build:

- Auth
- Organizations
- Database schema
- Asset CRUD
- Mission CRUD
- Seed data

Acceptance:

- Demo org exists.
- User can create drone/battery/pilot.
- User can create mission.

## Milestone 2: Log Ingestion

Build:

- CSV log upload
- Basic parser
- Normalized telemetry
- Flight log detail page
- Route map

Acceptance:

- User uploads log.
- System displays route and metrics.
- System detects basic anomalies.

## Milestone 3: Weather and Airspace

Build:

- Weather snapshot service
- Airspace check service
- Weather and airspace panels

Acceptance:

- Mission gets weather snapshot.
- Mission gets airspace warning if applicable.
- Snapshots are stored for audit.

## Milestone 4: Risk Engine

Build:

- Rule-based scoring
- Risk reasons
- Recommendations
- Mission risk panel

Acceptance:

- Every mission has score.
- Score has explanations.
- High risks create recommendations.

## Milestone 5: Compliance Binder

Build:

- Checklist
- PDF generation
- Report storage
- Download button

Acceptance:

- User can generate PDF binder.
- Binder includes mission, weather, airspace, risk, checklist, recommendations.

## Milestone 6: Fleet Readiness

Build:

- Fleet dashboard
- Asset readiness calculations
- Maintenance integration

Acceptance:

- Drones/batteries/pilots show Ready/Warning/Grounded.
- Unresolved maintenance changes status.
- Dashboard shows “what can fly today.”

---

## 13. Testing Plan

### Unit Tests

Test:

- CSV parser
- Risk rules
- Weather scoring
- Airspace intersection
- Recommendation creation
- PDF data assembly

### Integration Tests

Test:

- Create mission -> fetch weather -> check airspace -> calculate risk
- Upload log -> parse -> detect anomaly -> create report
- Add maintenance issue -> asset becomes warning/grounded
- Complete checklist -> compliance risk decreases

### End-to-End Tests

Test:

1. User logs in.
2. Creates mission.
3. Assigns drone/battery/pilot.
4. Runs risk score.
5. Completes checklist.
6. Generates PDF.
7. Uploads post-flight log.
8. Views anomaly report.

---

## 14. Demo Script

Use this demo story:

```text
A solar inspection operator has a mission scheduled today.

They select:
- Drone D-07
- Battery B-22
- Pilot P-03

AeroGuard flags:
- Wind gusts near drone limit
- Battery B-22 voltage sag
- Drone D-07 recent GPS instability
- Remote ID confirmation missing

AeroGuard recommends:
- Use Battery B-14
- Use Drone D-03
- Confirm Remote ID
- Delay mission by 2 hours or shorten route

User accepts recommendations.

Risk score drops.

AeroGuard generates a compliance binder PDF.

After flight, user uploads log.

AeroGuard detects:
- Small route deviation
- GPS weakening near waypoint 6
- Battery temperature elevated but within threshold

AeroGuard generates post-flight anomaly report.
```

---

## 15. Legal and Safety Notes for Product Copy

Use this product language:

```text
AeroGuard provides mission risk advisory and compliance documentation support.
Operators remain responsible for final flight decisions.
AeroGuard does not replace pilot judgment, FAA authorization, LAANC approval, manufacturer instructions, or legal advice.
```

Avoid:

```text
AeroGuard guarantees safe flights.
AeroGuard approves FAA-compliant missions.
AeroGuard prevents crashes.
AeroGuard replaces human review.
```

---

## 16. Final Implementation Priority

Build in this order:

1. Database schema
2. Seed/demo data
3. Asset CRUD
4. Mission CRUD
5. CSV telemetry upload
6. Telemetry normalization
7. Weather snapshot
8. Airspace check
9. Rule-based risk engine
10. Risk explanations
11. Recommendations
12. Compliance checklist
13. PDF binder
14. Fleet readiness dashboard
15. PX4/ArduPilot parser support
16. Public dataset loaders
17. Commercial integrations later

---

## 17. One-Sentence Product Summary

AeroGuard is a mission-risk and compliance intelligence platform that helps commercial drone operators decide which missions are too risky before takeoff and generate audit-ready evidence showing how the decision was made.
