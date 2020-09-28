
## 0.7.0

### Function execution

Transponder is now able to execute functions (https://webhookrelay.com/v1/guide/functions) that can transform requests and return dynamic responses. This feature is enabled by default. Functions by default are stored in the same SQL database but can be stored externally in GCP, AWS, Azure object storage providers or just filesystem.

Storage can be controlled with environment variables:

- FUNCTIONS_BUCKET_URL (default 'sql://', change it to 'file:///data/functions' to use filesystem, 'mem://' for in-memory storage or cloud specific bucket URLs)

Since Lua functions are tiny, it makes sense to keep them in the SQL database. Switch to object storage once WASM functions are released.

### Responses from webhooks

Now you can select whether caller should receive the response from either the first destination response or from the first successful response.

### Web dashboard is now in-built into the Transponder container

As web dashboard is now inbuilt into the server, deployment is simplified and there's no need to run a separate container.

### New database configuration options

These environment variables will allow configuring database connections to let you adapt to cases when you either run very small database servers or very large:

- DB_MAX_OPEN_CONNS (default 40)
- DB_MAX_IDLE_CONNS (default 10)
- DB_CONN_MAX_LIFETIME (default 1m)

### New webhook ingestion accelerator

To help speed up the webhook ingestion, there is now a local disk-based accelerator included that can dramatically reduce load on your database. Configuration is done through environment variables:

- ACCELERATOR_ENABLED (default true)
- ACCELERATOR_STORAGE_TYPE (default is "default" but for testing you can use "memory", however it's not crash safe)
- ACCELERATOR_PATH (default is ${XDG_DATA_HOME}, it will create a new file 'accelerator_db')
- ACCELERATOR_SYNC_WRITES (default 'true', this ensures that writes are synced to the disk, disabling it will improve performance but introduces a risk on crash)
- ACCELERATOR_COMPRESSION_LEVEL (default '1', uses Snappy compression algorithm)
- ACCELERATOR_BATCH_SIZE (default 100, defines a maximum batch size to use. Depends on your database server size and webhook payload size. All webhooks will be inserted from the accelerator database anyway, they will just appear in the dashboard later)
- ACCELERATOR_COLLECT_FREQUENCY (default '5s', you can use time like '1s', '50s' or minutes like '1m'. There's no point in having this more than 30s)

### Logs retention

Webhook Relay SaaS uses separate services to clean up database. Transponder will now be able to remove old webhook log entries as well with several settings:

- LOGS_RETENTION_DAYS (default 30, logs older than 30 days are removed)
- LOGS_RETENTION_JOB_SCHEDULE (default '@midnight', when to run the job). 

There's a table for schedule configuration:

Entry                  | Description                                | Equivalent To
-----                  | -----------                                | -------------
@yearly (or @annually) | Run once a year, midnight, Jan. 1st        | 0 0 0 1 1 *
@monthly               | Run once a month, midnight, first of month | 0 0 0 1 * *
@weekly                | Run once a week, midnight on Sunday        | 0 0 0 * * 0
@daily (or @midnight)  | Run once a day, midnight                   | 0 0 0 * * *
@hourly                | Run once an hour, beginning of hour        | 0 0 * * * *

### Misc improvements

- Shutdown now correctly drains remaining webhooks and persist them before the server exists.
- Updated Let's Encrypt library to work with the latest API.
- API caching - most API endpoints are now cached meaning that it's much more robust when you are using the API directly.
- Logs statistics completely overhauled, prepares daily statistics for previous days so it doesn't have to make the database calculate everything for each API request.