# AWS

- [AWS](#aws)
  - [S3](#s3)
  - [RDS](#rds)
    - [Support stored procedures](#support-stored-procedures)

## S3

Bucket public URL format: https://mybucket.s3.amazonaws.com

When versioning is enabled, uploaded files have `VersionId` set; it's null otherwise.

When the storage class of an object is changed, a new version is created (even if versioning is disabled), with updated `LastModified`, `IsLatest` = true (and according `VersionId`).

## RDS

### Support stored procedures

RDS provides some SPs to perform jobs not allowed to non-root users:

```sql
SELECT routine_name
FROM information_schema.routines
WHERE routine_type = 'PROCEDURE'
      -- AND routine_schema = @database_name
      AND routine_name LIKE 'rds\_%';

-- +-----------------------------------+
-- | rds_innodb_buffer_pool_dump_now   |
-- | rds_innodb_buffer_pool_load_abort |
-- | rds_innodb_buffer_pool_load_now   |
-- | rds_kill                          |
-- | rds_kill_query                    |
-- | rds_next_master_log               |
-- | rds_rotate_general_log            |
-- | rds_rotate_global_status_history  |
-- | rds_rotate_slow_log               |
-- | rds_set_configuration             |
-- | rds_set_fk_checks_off             |
-- | rds_set_fk_checks_on              |
-- | rds_show_configuration            |
-- | rds_skip_repl_error               |
-- | rds_start_replication             |
-- | rds_stop_replication              |
-- | ...                               |
-- +-----------------------------------+
```
