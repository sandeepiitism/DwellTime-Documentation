# Dwell Time Analytics V1.0

Dwell Time Analytics V1.0 is a Snowflake Native App that provides an interactive Streamlit dashboard for camera metadata, detections, zone occupancy, visitor dwell time, and behaviour categories.

The current Marketplace version uses provider-supplied, pre-processed detection data. Camera and zone metadata are supplied by the consumer through Native App table references.

Consumers are not required to upload or process video.

## Marketplace Release Information

- Listing name: `DWELL_TIME_ANALYTICS_V1.0`
- Marketplace title: `Dwell Time Analytics V1.0`
- Published application version: `V2`
- Published patch: `0`

The application package is managed by the provider. Consumers do not create, configure, or deploy an application package.

## Marketplace Installation

Dwell Time Analytics V1.0 is installed directly from Snowflake Marketplace.

To install the application:

1. Sign in to Snowsight.
2. Open Snowflake Marketplace.
3. Locate Dwell Time Analytics V1.0.
4. Select Get.
5. Follow the Marketplace installation prompts.
6. After installation, open the application from Data Products > Apps.
7. Configure the required table references as described below.
8. Open the Streamlit application.

No local Python environment, Snowflake credentials, application package creation, or manual Native App deployment is required by the consumer.

## Native App Configuration

The application requires access to two consumer-owned Snowflake tables:

- `CV_CAMERAS`
- `CV_ZONES`

These tables can exist in a database and schema owned and controlled by the consumer.

For example:

```text
<your_database>.<your_schema>.CV_CAMERAS
<your_database>.<your_schema>.CV_ZONES
```

After installing the application:

1. In Snowsight > Data Products > Apps, open the installed Dwell Time Analytics V1.0 application.

2. Grant the application access to a warehouse. The consumer role launching Streamlit also requires `USAGE` on that warehouse.

3. Open the application's configuration/references page.

4. For Cameras Table, select the consumer-owned table:

```text
<your_database>.<your_schema>.CV_CAMERAS
```

Approve `SELECT` access.

5. For Zones Table, select the consumer-owned table:

```text
<your_database>.<your_schema>.CV_ZONES
```

Approve `SELECT` access.

6. Open the application's Streamlit interface.

Once both references are configured, the dashboard displays camera information, KPIs, people-in-frame analytics, zone occupancy, average dwell time, behaviour categories, and longest-dwell visits.

## Table References

| Reference | Consumer Table | Required Privilege |
|---|---|---|
| Zones Table (`zone_table`) | `<your_database>.<your_schema>.CV_ZONES` | `SELECT` |
| Cameras Table (`camera_table`) | `<your_database>.<your_schema>.CV_CAMERAS` | `SELECT` |

The Streamlit application uses the `snowflake.permissions` SDK to request references that have not yet been bound and to read the selected associations.

Snowflake invokes the application's `UPDATE_REFERENCE` callback when a reference binding changes.

## Required Privileges

| Principal | Required Access | Purpose |
|---|---|---|
| Consumer account role | Application role `app_instance_role` | Access the application's Streamlit interface and application objects |
| Consumer account role | `USAGE` on the selected warehouse | Run the Streamlit application and its queries |
| App reference `zone_table` | `SELECT` on the consumer's `CV_ZONES` table | Read zone definitions |
| App reference `camera_table` | `SELECT` on the consumer's `CV_CAMERAS` table | Read camera metadata |

The application references request only `SELECT` access to the two selected consumer tables. They do not provide access to other consumer tables.

## Consumer Table Schemas

The consumer must provide the following two tables in their own Snowflake database and schema:

```text
<your_database>.<your_schema>.CV_CAMERAS
<your_database>.<your_schema>.CV_ZONES
```

### CV_CAMERAS

The `CV_CAMERAS` table contains camera metadata used by the application.

| Column | Data Type |
|---|---|
| `CAMERA_ID` | `VARCHAR(20)` |
| `CAMERA_NAME` | `VARCHAR(60)` |
| `LOCATION` | `VARCHAR(60)` |
| `FPS` | `NUMBER(38,0)` |
| `WIDTH` | `NUMBER(38,0)` |
| `HEIGHT` | `NUMBER(38,0)` |

Create the table using:

```sql
CREATE TABLE <your_database>.<your_schema>.CV_CAMERAS (
    CAMERA_ID   VARCHAR(20),
    CAMERA_NAME VARCHAR(60),
    LOCATION    VARCHAR(60),
    FPS         NUMBER(38,0),
    WIDTH       NUMBER(38,0),
    HEIGHT      NUMBER(38,0)
);
```

Example row:

```text
CAM_01, Store Entrance Cam, Main Floor, 1, 1280, 720
```

Example insert:

```sql
INSERT INTO <your_database>.<your_schema>.CV_CAMERAS
    (CAMERA_ID, CAMERA_NAME, LOCATION, FPS, WIDTH, HEIGHT)
VALUES
    ('CAM_01', 'Store Entrance Cam', 'Main Floor', 1, 1280, 720);
```

### CV_ZONES

The `CV_ZONES` table defines the retail zones used by the dwell-time application.

| Column | Data Type |
|---|---|
| `ZONE_ID` | `VARCHAR(10)` |
| `ZONE_NAME` | `VARCHAR(30)` |
| `X_MIN` | `NUMBER(38,0)` |
| `Y_MIN` | `NUMBER(38,0)` |
| `X_MAX` | `NUMBER(38,0)` |
| `Y_MAX` | `NUMBER(38,0)` |

Create the table using:

```sql
CREATE TABLE <your_database>.<your_schema>.CV_ZONES (
    ZONE_ID   VARCHAR(10),
    ZONE_NAME VARCHAR(30),
    X_MIN     NUMBER(38,0),
    Y_MIN     NUMBER(38,0),
    X_MAX     NUMBER(38,0),
    Y_MAX     NUMBER(38,0)
);
```

Example rows:

```text
Z1, ENTRANCE, 0, 260, 320, 720
Z2, AISLE, 320, 0, 900, 720
Z3, CHECKOUT, 900, 200, 1280, 720
```

Example insert:

```sql
INSERT INTO <your_database>.<your_schema>.CV_ZONES
    (ZONE_ID, ZONE_NAME, X_MIN, Y_MIN, X_MAX, Y_MAX)
VALUES
    ('Z1', 'ENTRANCE', 0, 260, 320, 720),
    ('Z2', 'AISLE', 320, 0, 900, 720),
    ('Z3', 'CHECKOUT', 900, 200, 1280, 720);
```

## Provider Detection Data

The current application uses provider-supplied, pre-processed detection data that is exposed to the Native App through the `CV_DETECTIONS` view.

The detection data contains:

| Column | Data Type |
|---|---|
| `CAMERA_ID` | `VARCHAR(20)` |
| `FRAME_NUMBER` | `NUMBER(38,0)` |
| `TS_SECOND` | `NUMBER(38,0)` |
| `TRACK_ID` | `VARCHAR(10)` |
| `CLASS_NAME` | `VARCHAR(30)` |
| `CONFIDENCE` | `FLOAT` |
| `X` | `NUMBER(38,0)` |
| `Y` | `NUMBER(38,0)` |
| `W` | `NUMBER(38,0)` |
| `H` | `NUMBER(38,0)` |
| `CX` | `NUMBER(38,0)` |
| `CY` | `NUMBER(38,0)` |
| `ZONE_ID` | `VARCHAR(10)` |

The sample provider dataset contains 5,700 rows, 40 tracked people, one camera, and three zones.

This provider data is exposed through the application's `CV_DETECTIONS` view. It is not a consumer table reference, and the consumer does not need to create or load this detection dataset.

## Streamlit Usage

After installing Dwell Time Analytics V1.0 and binding both consumer table references:

1. Open Snowsight > Data Products > Apps.
2. Open the installed Dwell Time Analytics V1.0 application.
3. Open the Streamlit interface.
4. Confirm that `CV_CAMERAS` and `CV_ZONES` are configured.
5. View the dwell-time analytics dashboard.

Once configured, the dashboard displays:

- Camera information
- Visitor and detection KPIs
- People-in-frame analytics
- Zone occupancy
- Average dwell time
- Behaviour categories
- Longest dwell visits

The current Marketplace version provides analytics using pre-processed detection data.

Video upload, frame extraction, video processing, conversational analytics, and natural-language interaction are not enabled in this version.

## Functions, Procedures, and Views

The application exposes its objects through the installed application's `app_instance_schema`.

In the examples below:

```text
<installed_app_name>
```

represents the application name assigned during Marketplace installation.

The following objects are available:

| Object | Type | Purpose |
|---|---|---|
| `CV_DETECTIONS` | View | Provides access to provider-supplied detection data |
| `DWELL_CATEGORY(INT)` | Python UDF | Maps dwell duration to `PASSING`, `BROWSING`, `ENGAGED`, or `LOITERING`. `NULL` maps to `UNKNOWN` |
| `BILLING_EVENT(INT)` | Python procedure | Returns a confirmation string for the application's billing-event example |
| `UPDATE_REFERENCE(STRING, STRING, STRING)` | SQL procedure | Callback used when adding, removing, or clearing a table reference |
| `STREAMLIT` | Streamlit object | Provides the dwell-time analytics dashboard |

## Example SQL

Use `<installed_app_name>` below as the name of the Native App installed from Snowflake Marketplace.

### Query Detection Data

```sql
SELECT
    CAMERA_ID,
    TS_SECOND,
    TRACK_ID,
    CLASS_NAME,
    CONFIDENCE,
    ZONE_ID
FROM <installed_app_name>.app_instance_schema.CV_DETECTIONS
LIMIT 10;
```

### Calculate Behaviour Categories

```sql
SELECT
    <installed_app_name>.app_instance_schema.DWELL_CATEGORY(7)
        AS SHORT_VISIT,
    <installed_app_name>.app_instance_schema.DWELL_CATEGORY(60)
        AS LONGER_VISIT;
```

### Calculate Dwell Visits

```sql
WITH sequenced AS (
    SELECT
        TRACK_ID,
        ZONE_ID,
        TS_SECOND,
        TS_SECOND - ROW_NUMBER() OVER (
            PARTITION BY TRACK_ID, ZONE_ID
            ORDER BY TS_SECOND
        ) AS visit_group
    FROM <installed_app_name>.app_instance_schema.CV_DETECTIONS
),
visits AS (
    SELECT
        TRACK_ID,
        ZONE_ID,
        visit_group,
        COUNT(*) AS DWELL_SECONDS
    FROM seque*ced
    GROUP BY TRACK_ID, ZONE_ID* visit_group
)
SELECT
    TRACK_ID
    ZONE_ID,
    DWELL_SECONDS,
 <installed_app_name>.app_instance_schema.DWELL_CATEGORY(DWELL_SECONDS)
        AS BEHAVIOUR
FROM visit*
ORDER BY DWELL_SECONDS DESC;
```
### Example Procedure Call

```sql
CALL <installed_app_name>.app_instance_schema.BILLING_EVENT(5700);
```

### Verify Consumer Camera Data
```sql
SELECT *
FROM <your_database>.<your_schema>.CV_CAMERAS;
```

### Verify Consumer Zone Data

```sql
SELECT *
FROM <your_database>.<your_schema>.CV_ZONES;
```

