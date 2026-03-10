# pg_cron: Scheduled Hub Content Metrics

Schedules `sp_generate_hub_content_metrics()` to run automatically every Sunday at 23:00 UTC, snapshotting the week's `hub_content_metrics` from `view_current_hub_content`. Without this, metrics must be generated manually.

**Depends on:** RDS running (Step 6) and database schema deployed (Step 7a — `sp_generate_hub_content_metrics` must exist). This is Step 7b in the deployment guide.

---

## Step 1: Enable pg_cron in the RDS Parameter Group

Two parameters must be set

| Parameter | Value | Why |
|---|---|---|
| `shared_preload_libraries` | `pg_cron` | Loads the pg_cron background worker at startup |
| `cron.database_name` | `${PROJECT_NAME}_${ENV}` (e.g. `redwood_dev`) | Tells pg_cron which database to read jobs from. Defaults to `postgres` — without this, `CREATE EXTENSION` will fail with: *"can only create extension in database postgres"* |

```bash
# Find your RDS parameter group name
aws rds describe-db-instances \
  --region ${AWS_REGION} \
  --profile ${AWS_PROFILE} \
  --query "DBInstances[?contains(DBInstanceIdentifier, '${PROJECT_NAME}')].{ID:DBInstanceIdentifier, PG:DBParameterGroups[0].DBParameterGroupName}" \
  --output table

# Set BOTH parameters in one command
aws rds modify-db-parameter-group \
  --region ${AWS_REGION} \
  --profile ${AWS_PROFILE} \
  --db-parameter-group-name <your-parameter-group-name> \
  --parameters \
    "ParameterName=shared_preload_libraries,ParameterValue=pg_cron,ApplyMethod=pending-reboot" \
    "ParameterName=cron.database_name,ParameterValue=${PROJECT_NAME}_${ENV},ApplyMethod=pending-reboot"

# Reboot the RDS instance to apply both changes
aws rds reboot-db-instance \
  --region ${AWS_REGION} \
  --profile ${AWS_PROFILE} \
  --db-instance-identifier <your-db-instance-id>

# Wait for the instance to come back online (~5 minutes)
aws rds wait db-instance-available \
  --region ${AWS_REGION} \
  --profile ${AWS_PROFILE} \
  --db-instance-identifier <your-db-instance-id>
```

> ⚠️ **Note:** If your RDS instance uses a **default** parameter group (e.g. `default.postgres15`), you cannot modify it directly. Create a custom one first:
> ```bash
> aws rds create-db-parameter-group \
>   --db-parameter-group-name ${PROJECT_NAME}-pg-${ENV} \
>   --db-parameter-group-family postgres15 \
>   --description "${PROJECT_NAME} custom parameter group for ${ENV}" \
>   --profile ${AWS_PROFILE}
>
> aws rds modify-db-instance \
>   --db-instance-identifier <your-db-instance-id> \
>   --db-parameter-group-name ${PROJECT_NAME}-pg-${ENV} \
>   --apply-immediately \
>   --profile ${AWS_PROFILE}
> ```
> Then apply both parameters and reboot as above.

---

## Step 2: Install the Extension and Schedule the Job

Connect to your database and run the following SQL. This only needs to be done **once per environment**.

```bash
ENDPOINT=$(aws rds describe-db-instances \
  --db-instance-identifier <your-db-instance-id> \
  --region ${AWS_REGION} --profile ${AWS_PROFILE} \
  --query "DBInstances[0].Endpoint.Address" --output text)

psql -h ${ENDPOINT} -U datahubpostgres${ENV} -d ${PROJECT_NAME}_${ENV}
```

Then in the psql session:

```sql
-- 1. Enable the pg_cron extension (requires superuser / rds_superuser)
CREATE EXTENSION IF NOT EXISTS pg_cron;

-- 2. Schedule: every Sunday at 23:00 UTC
--    Cron syntax: minute  hour  day-of-month  month  day-of-week (0=Sunday)
SELECT cron.schedule(
    'weekly-hub-content-metrics',
    '0 23 * * 0',
    'CALL sp_generate_hub_content_metrics()'
);
```

> 💡 **Time zone:** pg_cron uses **UTC**. 23:00 UTC Sunday equals:
> - 7:00 PM US Eastern (EDT, UTC-4) / 6:00 PM EST (UTC-5)
> - 4:00 PM US Pacific (PDT, UTC-7) / 3:00 PM PST (UTC-8)
>
> Adjust the schedule if needed — e.g. `'0 5 * * 1'` = Monday 05:00 UTC (Sunday midnight US Eastern).

---

## Step 3: Verify the Schedule

```sql
SELECT jobid, jobname, schedule, command, active
FROM cron.job;
```

Expected output:
```
 jobid |          jobname           | schedule  |                       command                        | active
-------+----------------------------+-----------+------------------------------------------------------+--------
     1 | weekly-hub-content-metrics | 0 23 * * 0 | CALL sp_generate_hub_content_metrics()              | t
```

---

## Step 4: Run Immediately to Validate

Don't wait until Sunday — trigger the procedure manually to confirm it works end-to-end:

```sql
-- Run the stored procedure now
CALL sp_generate_hub_content_metrics();

-- Verify a row was inserted into metrics_report
SELECT id, report_date, type_id FROM metrics_report ORDER BY report_date DESC LIMIT 5;

-- Verify hub_content_metrics was populated
SELECT report_id, center, study_id, study_title, data_file_count
FROM hub_content_metrics
ORDER BY report_id DESC
LIMIT 10;
```

---

## Ongoing: Check Execution History

After the first scheduled run, use this query to confirm successful execution and diagnose any failures:

```sql
SELECT
    j.jobname,
    d.start_time,
    d.end_time,
    d.status,
    d.return_message
FROM cron.job_run_details d
JOIN cron.job j ON j.jobid = d.jobid
WHERE j.jobname = 'weekly-hub-content-metrics'
ORDER BY d.start_time DESC
LIMIT 10;
```

| `status` | Meaning |
|---|---|
| `succeeded` | Procedure ran and committed successfully |
| `failed` | Check `return_message` for the error |

---

## Managing the Schedule

```sql
-- Pause the job without deleting it
SELECT cron.unschedule('weekly-hub-content-metrics');

-- Re-add it
SELECT cron.schedule('weekly-hub-content-metrics', '0 23 * * 0',
    'CALL sp_generate_hub_content_metrics()');

-- Change the schedule (e.g. Saturday night instead)
SELECT cron.alter_job(
    job_id := (SELECT jobid FROM cron.job WHERE jobname = 'weekly-hub-content-metrics'),
    schedule := '0 23 * * 6'
);
```
