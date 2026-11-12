# Dwell Time Analytics

Snowflake Native App dashboard for camera metadata, detections, zone occupancy,
visitor dwell time, and behaviour categories.

Provider detection data is delivered through the Snowflake Native App, while
camera and zone metadata are supplied by the consumer through Native App table
references.

## Marketplace Installation

The Dwell Time Analytics application is installed directly from Snowflake
Marketplace.

To install the application:

1. Sign in to Snowsight.
2. Open **Snowflake Marketplace**.
3. Locate the **Dwell Time Analytics** application.
4. Select **Get**.
5. Follow the Marketplace installation prompts.
6. After installation, open the application from **Data Products > Apps**.
7. Configure the required table references as described below.
8. Open the Streamlit application.

No local Python environment, Snowflake credentials, application package
creation, or manual Native App deployment is required by the consumer.

## Native App Configuration

The application requires access to two consumer-owned Snowflake tables:

- `CV_CAMERAS`
- `CV_ZONES`

These tables can exist in any database and schema accessible to the consumer.
For example:

```text
<your_database>.<your_schema>.CV_CAMERAS
<your_database>.<your_schema>.CV_ZONES
```

After installing the application:

1. In **Snowsight > Data Products > Apps**, open **Dwell Time Analytics**.

2. Grant the application access to a warehouse. The consumer role launching
   Streamlit also requires `USAGE` on that warehouse.

3. Open the application's configuration/references page.

4. For **Cameras Table**, select the consumer-owned:

   ```text
   <your_database>.<your_schema>.CV_CAMERAS
   ```

   and approve `SELECT`.

5. For **Zones Table**, select the consumer-owned:

   ```text
   <your_database>.<your_schema>.CV_ZONES
   ```

   and approve `SELECT`.

6. Open the application's Streamlit interface.

Once both references are configured, the dashboard should display the selected
camera, four KPIs, people-in-frame and zone-occupancy charts, average dwell
time, behaviour mix, and a longest-dwell table.

The reference names and requested privileges are:

| Reference | Bind to | Required privilege |
|---|---|---|
| Zones Table (`zone_table`) | `<your_database>.<your_schema>.CV_ZONES` | `SELECT` |
| Cameras Table (`camera_table`) | `<your_database>.<your_schema>.CV_CAMERAS` | `SELECT` |

The Streamlit application uses the `snowflake.permissions` SDK to request
references that have not yet been bound and to read the selected associations.

Snowflake invokes the application's `UPDATE_REFERENCE` callback when a
reference binding changes.

The consumer role needs the `CV_APP_V2.app_instance_role` application role to
access the application's Streamlit interface and application objects. This
application role can be granted to additional account roles when access to the
application needs to be shared within the consumer account.

## Required Privileges

| Principal | Required access | Purpose |
|---|---|---|
| Consumer account role | `CV_APP_V2.app_instance_role` | Use the application's Streamlit interface, view, UDF, and procedures. |
| Consumer account role | `USAGE` on the selected warehouse | Run the Streamlit application and its queries. |
| App reference `zone_table` | `SELECT` on the consumer's `CV_ZONES` table | Read zone definitions and join detections to zones. |
| App reference `camera_table` | `SELECT` on the consumer's `CV_CAMERAS` table | Read camera metadata used by the dashboard. |

The application references request only `SELECT` access. They do not grant the
application access to other consumer tables.

## Consumer Table Schemas

The consumer must provide two tables in their own Snowflake database and
schema:

- `CV_CAMERAS`
- `CV_ZONES`

The database and schema names are determined by the consumer.

For example:

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

The consumer can create the table using:

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

Example SQL for inserting this row:

```sql
INSERT INTO <your_database>.<your_schema>.CV_CAMERAS
    (CAMERA_ID, CAMERA_NAME, LOCATION, FPS, WIDTH, HEIGHT)
VALUES
    ('CAM_01', 'Store Entrance Cam', 'Main Floor', 1, 1280, 720);
```

### CV_ZONES

The `CV_ZONES` table defines the retail zones used by the dwell-time
application.

| Column | Data Type |
|---|---|
| `ZONE_ID` | `VARCHAR(10)` |
| `ZONE_NAME` | `VARCHAR(30)` |
| `X_MIN` | `NUMBER(38,0)` |
| `Y_MIN` | `NUMBER(38,0)` |
| `X_MAX` | `NUMBER(38,0)` |
| `Y_MAX` | `NUMBER(38,0)` |

The consumer can create the table using:

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

Example SQL for inserting these rows:

```sql
INSERT INTO <your_database>.<your_schema>.CV_ZONES
    (ZONE_ID, ZONE_NAME, X_MIN, Y_MIN, X_MAX, Y_MAX)
VALUES
    ('Z1', 'ENTRANCE', 0,   260, 320,  720),
    ('Z2', 'AISLE',    320, 0,   900,  720),
    ('Z3', 'CHECKOUT', 900, 200, 1280, 720);
```

## Provider Detection Data

The application uses provider-supplied detection data exposed through the
Native App as the `CV_DETECTIONS` view.

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

The sample provider dataset contains 5,700 rows, 40 tracked people, one camera,
and three zones.

This provider data is exposed to the installed application through the
`CV_DETECTIONS` view. It is not a consumer table reference and the consumer
does not need to create this dataset.

## Streamlit Usage

After installing the application and binding both consumer table references:

1. Open **Snowsight > Data Products > Apps**.
2. Open the installed **Dwell Time Analytics** application.
3. Open the application's Streamlit interface.
4. Confirm that the required `CV_CAMERAS` and `CV_ZONES` references are
   configured.
5. View the dwell-time analytics dashboard.

Once configured, the dashboard displays:

- Bound camera information
- Visitor and detection KPIs
- People-in-frame analytics
- Zone occupancy
- Average dwell time
- Behaviour categories
- Longest dwell visits

The installed Native App currently provides the analytics workflow. Video
upload and frame extraction are not enabled in the packaged Marketplace
application.

## Functions, Procedures, and Views

The following objects are available under:

```text
CV_APP_V2.app_instance_schema
```

| Object | Type | Purpose |
|---|---|---|
| `CV_DETECTIONS` | View | Provides access to the provider's shared detection data. |
| `DWELL_CATEGORY(INT)` | Python UDF | Maps dwell duration to `PASSING`, `BROWSING`, `ENGAGED`, or `LOITERING`; `NULL` maps to `UNKNOWN`. |
| `BILLING_EVENT(INT)` | Python procedure | Example billing hook that returns a confirmation string. |
| `UPDATE_REFERENCE(STRING, STRING, STRING)` | SQL procedure | Native App callback for adding, removing, or clearing a bound reference. |
| `STREAMLIT` | Streamlit object | Provides the default dwell-time analytics dashboard. |

## Example SQL

Run the following examples using a role that has been granted:

```text
CV_APP_V2.app_instance_role
```

### Query detection data

```sql
SELECT CAMERA_ID,
       TS_SECOND,
       TRACK_ID,
       CLASS_NAME,
       CONFIDENCE,
       ZONE_ID
FROM CV_APP_V2.app_instance_schema.CV_DETECTIONS
LIMIT 10;
```

### Calculate example behaviour categories

```sql
SELECT
    CV_APP_V2.app_instance_schema.DWELL_CATEGORY(7) AS SHORT_VISIT,
    CV_APP_V2.app_instance_schema.DWELL_CATEGORY(60) AS LONGER_VISIT;
```

### Calculate dwell visits

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
    FROM CV_APP_V2.app_instance_schema.CV_DETECTIONS
),
visits AS (
    SELECT
        TRACK_ID,
        ZONE_ID,
        visit_group,
        COUNT(*) AS DWELL_SECONDS
    FROM sequenced
    GROUP BY TRACK_ID, ZONE_ID, visit_group
)
SELECT
    TRACK_ID,
    ZONE_ID,
    DWELL_SECONDS,
    CV_APP_V2.app_instance_schema.DWELL_CATEGORY(DWELL_SECONDS) AS BEHAVIOUR
FROM visits
ORDER BY DWELL_SECONDS DESC;
```

### Example procedure call

```sql
CALL CV_APP_V2.app_instance_schema*BILLING_EVENT(570*);
```

### Verify the consumer ca*era table

```sql
SELECT *
FROM <y*ur_database>.<your_schema>.CV_CAME*AS;
```

### Verify the consumer*zone table

```sql
SELECT *
FROM <*our_database>.<your_schema>.CV_ZON*S;
```

## Expected*Result After Configuration

After *nstallation*and configuration are complete:

*. `CV_CAMERAS` is*bound to the application's `camera*table` reference*
2. `CV_ZONES` is bound to the app*ication's `zone_table` reference.
*. The application can read the con*umer's camera and*zone metadata using
   `SELECT` ac*ess.
4. Provider detection data is*available through the application'*
   `CV_DETECTIONS` view.
5. The S*reamlit dashboard displays the con*igured camera and dwell-time
   an*lytics.

No local Python environme*t, local Streamlit server, `.env` *ile, Snowflake
username/password, *r manual application package insta*lation is required.
