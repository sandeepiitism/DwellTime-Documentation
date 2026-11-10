# Dwell Time Analytics

Snowflake Native App dashboard for camera metadata, detections, zone occupancy,
visitor dwell time, and behaviour categories. Provider data is shared through
the application package; camera and zone metadata are supplied by the consumer
through Native App table references.

## Marketplace Installation

For a published listing, open it in **Snowflake Marketplace**, select **Get**,
choose the target account, and install the app. A listing must be published and
available to the consumer account first. After installation, configure the app
and references as described below.

For this repository's package-based deployment, the provider creates
`CV_APP_PACKAGE_V2` with `scripts/2. create-package.sql`, stages the six app
files, and installs the app with `scripts/3. build_app.sql`. The equivalent
consumer install command is:

```sql
CREATE APPLICATION CV_APP_V2
  FROM APPLICATION PACKAGE CV_APP_PACKAGE_V2
  USING '@CV_APP_PACKAGE_V2.APP_SRC.STAGE';
```

## Native App Configuration

1. In **Snowsight > Data Products > Apps**, open **Dwell Time Analytics**.
2. Grant the app access to a warehouse. The consumer role launching Streamlit
  also needs `USAGE` on that warehouse.
3. Open the app's configuration/references page. For **Zones Table**, select
  `MLOPS_2.CV_APP_SCHEMA_V2.CV_ZONES` and approve `SELECT`; for **Cameras
  Table**, select `MLOPS_2.CV_APP_SCHEMA_V2.CV_CAMERAS` and approve `SELECT`.
4. Open the app's Streamlit interface. It should show the selected camera,
  four KPIs, people-in-frame and zone-occupancy charts, average dwell time,
  behaviour mix, and a longest-dwell table.

The reference names and requested privileges are:

| Reference | Bind to | Required privilege |
|---|---|---|
| Zones Table (`zone_table`) | `MLOPS_2.CV_APP_SCHEMA_V2.CV_ZONES` | `SELECT` |
| Cameras Table (`camera_table`) | `MLOPS_2.CV_APP_SCHEMA_V2.CV_CAMERAS` | `SELECT` |

The contract is defined in `app/src/manifest.yml`. The Streamlit app uses the
`snowflake.permissions` SDK to request references that have not yet been bound
and read the selected associations. Snowflake invokes the app's
`UPDATE_REFERENCE` callback when a binding changes. The consumer role needs the
`CV_APP_V2.app_instance_role` application role; grant it to additional account
roles when sharing the app within the account.

## Required Privileges

| Principal | Required access | Purpose |
|---|---|---|
| Consumer account role | `CV_APP_V2.app_instance_role` | Use the app's Streamlit, view, UDF, and procedures. |
| Consumer account role | `USAGE` on the selected warehouse | Run the Streamlit app and its queries. |
| App reference `zone_table` | `SELECT` on `CV_ZONES` | Read zone names and join detections to zones. |
| App reference `camera_table` | `SELECT` on `CV_CAMERAS` | Read the camera details shown in the dashboard. |

The references request only `SELECT`; they do not grant the app access to other
consumer tables.

## Consumer Table Schemas

Create and load these tables in `MLOPS_2.CV_APP_SCHEMA_V2`. The schemas and seed
data are in `1. prepare_data.sql` and `app/data/`.

| Table | Columns |
|---|---|
| `CV_CAMERAS` | `CAMERA_ID VARCHAR(20)`, `CAMERA_NAME VARCHAR(60)`, `LOCATION VARCHAR(60)`, `FPS NUMBER(38,0)`, `WIDTH NUMBER(38,0)`, `HEIGHT NUMBER(38,0)` |
| `CV_ZONES` | `ZONE_ID VARCHAR(10)`, `ZONE_NAME VARCHAR(30)`, `X_MIN NUMBER(38,0)`, `Y_MIN NUMBER(38,0)`, `X_MAX NUMBER(38,0)`, `Y_MAX NUMBER(38,0)` |

Example camera row:

```text
CAM_01, Store Entrance Cam, Main Floor, 1, 1280, 720
```

Example zone rows:

```text
Z1, ENTRANCE, 0, 260, 320, 720
Z2, AISLE, 320, 0, 900, 720
Z3, CHECKOUT, 900, 200, 1280, 720
```

The provider detection table `MLOPS_2.CV_APP_SCHEMA_V2.CV_DETECTIONS` contains
`CAMERA_ID VARCHAR(20)`, `FRAME_NUMBER NUMBER(38,0)`, `TS_SECOND NUMBER(38,0)`,
`TRACK_ID VARCHAR(10)`, `CLASS_NAME VARCHAR(30)`, `CONFIDENCE FLOAT`, bounding
box fields `X`, `Y`, `W`, `H` and center fields `CX`, `CY` (all
`NUMBER(38,0)`), and `ZONE_ID VARCHAR(10)`. The sample dataset has 5,700 rows,
40 tracked people, one camera, and three zones. This provider data is exposed
to the app through the package's `shared_content_schema.CV_DETECTIONS` view; it
is not a consumer reference.

## Sample Data for Functional Review

The small, synthetic CSV bundle prepared for `SFMARKETPLACEOPS.R_PROD3` is in
[`sfmarketplaceops/`](sfmarketplaceops/README.md):
[`CV_CAMERAS.csv`](sfmarketplaceops/CV_CAMERAS.csv),
[`CV_ZONES.csv`](sfmarketplaceops/CV_ZONES.csv),
[`CV_DETECTIONS.csv`](sfmarketplaceops/CV_DETECTIONS.csv), and
[`DWELL_TIME_TEST_CASES.csv`](sfmarketplaceops/DWELL_TIME_TEST_CASES.csv).
The detections include consecutive one-second visits that produce dwell-time
results. The bundle README documents the schemas, import steps, and expected
results. For an app-level test, the detections CSV must populate the provider
source table behind the package's shared `CV_DETECTIONS` view, while the camera
and zone tables are bound as consumer references. These files are prepared for
CSV delivery; they are not automatically shared with or uploaded to the
Snowflake review account.

## Streamlit Usage

Open the installed app from **Data Products > Apps > CV_APP_V2** after binding
both references. The dashboard
shows the bound camera, visitor and detection KPIs, people-in-frame and zone
occupancy charts, average dwell time, behaviour mix, and longest dwell visits.
The installed Native App currently provides the analytics workflow; video
upload and frame extraction are not enabled in the packaged app.

## Functions, Procedures, and Views

Objects are created by `app/src/setup.sql` in `CV_APP_V2.app_instance_schema`.

| Object | Type | Purpose |
|---|---|---|
| `CV_DETECTIONS` | View | Reads the provider's shared detection data. |
| `DWELL_CATEGORY(INT)` | Python UDF | Maps duration to `PASSING`, `BROWSING`, `ENGAGED`, or `LOITERING`; `NULL` maps to `UNKNOWN`. |
| `BILLING_EVENT(INT)` | Python procedure | Example billing hook; returns a confirmation string. |
| `UPDATE_REFERENCE(STRING, STRING, STRING)` | SQL procedure | Native App callback for adding, removing, or clearing a bound reference. |
| `STREAMLIT` | Streamlit object | Default analytics dashboard. |

The separate `scripts/5. video_to_frames.sql` procedure is a provider-side
prototype; it is not installed or exposed by the Native App package.

## Example SQL

Run as a role granted `CV_APP_V2.app_instance_role`:

```sql
-- Shared detection view
SELECT CAMERA_ID, TS_SECOND, TRACK_ID, CLASS_NAME, CONFIDENCE, ZONE_ID
FROM CV_APP_V2.app_instance_schema.CV_DETECTIONS
LIMIT 10;

-- Example behaviour categories
SELECT CV_APP_V2.app_instance_schema.DWELL_CATEGORY(7)  AS SHORT_VISIT,
       CV_APP_V2.app_instance_schema.DWELL_CATEGORY(60) AS LONGER_VISIT;

-- Dwell visits from consecutive per-second detections
WITH sequenced AS (
  SELECT TRACK_ID, ZONE_ID, TS_SECOND,
       TS_SECOND - ROW_NUMBER() OVER (
         PARTITION BY TRACK_ID, ZONE_ID ORDER BY TS_SECOND
       ) AS visit_group
  FROM CV_APP_V2.app_instance_schema.CV_DETECTIONS
), visits AS (
  SELECT TRACK_ID, ZONE_ID, visit_group, COUNT(*) AS DWELL_SECONDS
  FROM sequenced
  GROUP BY TRACK_ID, ZONE_ID, visit_group
)
SELECT TRACK_ID, ZONE_ID, DWELL_SECONDS,
     CV_APP_V2.app_instance_schema.DWELL_CATEGORY(DWELL_SECONDS) AS BEHAVIOUR
FROM visits
ORDER BY DWELL_SECONDS DESC;

-- Example procedure call
CALL CV_APP_V2.app_instance_schema.BILLING_EVENT(5700);

-- Example consumer camera and zone rows
SELECT * FROM MLOPS_2.CV_APP_SCHEMA_V2.CV_CAMERAS;
SELECT * FROM MLOPS_2.CV_APP_SCHEMA_V2.CV_ZONES;
```
