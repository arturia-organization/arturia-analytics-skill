# ASC Model Stats (Software Usage Telemetry)

**Table**: `arturia-bi-405110.dataset.asc__model_stats`
**Purpose**: Arturia Software Center usage session telemetry data
**Size**: 152M+ rows, 70+ GB
**Primary Key**: `session_uuid`

## Key Fields

### Session Identification

| Field | Type | Description | Analysis Use |
|-------|------|-------------|--------------|
| `session_uuid` | STRING (26 chars) | Plugin instance identifier (ULID format) | Plugin-level tracking, deduplication |
| `original_session_timestamp` | TIMESTAMP | **✅ RELIABLE** - Actual session start time (UTC) | Session grouping, temporal analysis |
| `session_timestamp` | INTEGER | **❌ UNRELIABLE** - Hybrid field, do not use | Never use for analytics |
| `session_timestamp_offset` | INTEGER | Delivery delay in seconds (0 = real-time) | Data quality, offline usage analysis |
| `session_created_on` | TIMESTAMP | **⚠️ Local user time, NOT UTC** | User-local temporal analysis only |
| `message_timestamp` | TIMESTAMP | Server receipt time (UTC) | Server load analysis |
| `session_duration` | INTEGER | Session length in seconds | ⚠️ Unreliable - recalculate from events |
| `session_events_count` | INTEGER | Number of events in session | Engagement metrics |
| `first_event_ts` | TIMESTAMP | First event timestamp | Session start verification |
| `last_event_ts` | TIMESTAMP | Last event timestamp | Session end verification |

### User & Machine

| Field | Type | Description | Analysis Use |
|-------|------|-------------|--------------|
| `user_id` | INTEGER | User identifier (links to webstore) | Cross-system user analysis |
| `machine_id` | INTEGER | Machine identifier | Multi-device tracking |
| `machine_name` | STRING | Machine name | Device identification |
| `new_machine` | BOOLEAN | First-time machine flag | New device adoption |
| `machine_os` | STRING | Operating system | OS distribution analysis |
| `machine_cpu` | STRING | CPU model | Hardware profile |

### Product & Application

| Field | Type | Description | Analysis Use |
|-------|------|-------------|--------------|
| `session_product_id` | INTEGER | Product identifier | Product usage tracking |
| `session_app_name` | STRING | Application name | Product identification |
| `session_version` | STRING | Software version | Version adoption |
| `session_host` | STRING | DAW/host application | DAW preference analysis |
| `session_wrapper` | STRING | Plugin format (VST, AU, AAX, etc.) | Format distribution |
| `session_controller` | STRING | MIDI controller info | Hardware integration |

### License Information

| Field | Type | Description | Analysis Use |
|-------|------|-------------|--------------|
| `session_licence_type` | STRING | License type | License distribution |
| `session_is_demo` | BOOLEAN | Demo mode flag | Demo vs licensed usage |
| `session_activation_state` | INTEGER | Activation status | License health |

### Geographic & Network

| Field | Type | Description | Analysis Use |
|-------|------|-------------|--------------|
| `requester_ip` | STRING | IP address | Network analysis |
| `continent` | STRING | Continent | Regional analysis |
| `country` | STRING | Country | Geographic distribution |
| `city` | STRING | City | Local market analysis |
| `latitude`, `longitude` | FLOAT | Coordinates | Geo-mapping |

### Hardware Specifications

| Field | Type | Description | Analysis Use |
|-------|------|-------------|--------------|
| `session_cpu_name` | STRING | CPU name | Performance profiling |
| `session_cpu_logical_core` | INTEGER | Logical cores | System capability |
| `session_cpu_physical_core` | INTEGER | Physical cores | Hardware analysis |
| `session_ram` | INTEGER | RAM amount | Memory requirements |
| `session_cpu_clock` | INTEGER | CPU clock speed | Performance analysis |

## Standard Filter

```sql
WHERE user_id > 0
  AND session_duration > 0
  AND session_product_id IS NOT NULL
```

## Critical Warnings

### 1. `session_timestamp` is UNRELIABLE
- For 86.8% of records: Calculated as receive_time - duration_s
- For 13.2% of records: Message receive time, NOT session time
- **DO NOT USE** for analytics - use `original_session_timestamp` instead

### 2. `session_created_on` is LOCAL TIME
- Stores timestamps in local user time, not UTC
- No timezone offset stored
- ✅ Valid: "What time of day do users in Germany work?"
- ❌ Invalid: Cross-timezone aggregations, global hourly analysis

### 3. Session Duration Errors
- 43.4% of records have incorrect `session_duration`
- Recalculate: `TIMESTAMP_DIFF(last_event_ts, first_event_ts, SECOND)`

### 4. Offline Queuing
- 13.2% of sessions sent hours/days after occurrence
- Check `session_timestamp_offset` for delay
- Max observed delay: 691 days

### 5. Load Balancer IP Issue
- 15.3% of June-Oct 2025 data shows IP 51.38.57.105 (France)
- This masks real user location
- Fixed after October 15, 2025

## DAW Session Grouping

`session_uuid` is per-plugin instance, not per DAW session. Use this for DAW sessions:

```sql
CONCAT(CAST(user_id AS STRING), '_',
       CAST(machine_id AS STRING), '_',
       CAST(UNIX_SECONDS(original_session_timestamp) AS STRING)) as daw_session_id
```

Average: 2.96 plugin instances per DAW session.

## Common Queries

### Daily Active Users
```sql
SELECT
  DATE(original_session_timestamp) as date_utc,
  COUNT(DISTINCT user_id) as dau
FROM `arturia-bi-405110.dataset.asc__model_stats`
WHERE user_id > 0
GROUP BY date_utc
ORDER BY date_utc;
```

### DAW Distribution
```sql
SELECT
  session_host,
  COUNT(DISTINCT CONCAT(CAST(user_id AS STRING), '_',
                        CAST(machine_id AS STRING), '_',
                        CAST(UNIX_SECONDS(original_session_timestamp) AS STRING))) as daw_sessions
FROM `arturia-bi-405110.dataset.asc__model_stats`
WHERE session_host IS NOT NULL AND user_id > 0
GROUP BY session_host
ORDER BY daw_sessions DESC;
```

### Demo vs Licensed Usage
```sql
SELECT
  session_is_demo,
  COUNT(DISTINCT user_id) as users,
  ROUND(AVG(TIMESTAMP_DIFF(last_event_ts, first_event_ts, SECOND)), 2) as avg_duration_sec
FROM `arturia-bi-405110.dataset.asc__model_stats`
WHERE user_id > 0
  AND first_event_ts IS NOT NULL
  AND last_event_ts IS NOT NULL
GROUP BY session_is_demo;
```
