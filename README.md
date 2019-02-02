# M3 Demo

This repository contains a docker-compose file which can be used to setup a demo of the M3 stack. It runs the following containers:

1. M3DB
2. M3Coordinator
3. Prometheus
4. Grafana

## Setup

### Container Setup and Database Initialization

Start all the containers. Note that if you'd like to reuse the underlying storage between runs (I.E so that data and topology / namespaces are retained) then remove the --force-renew-anon-volumes argument from the command below.

```bash
docker-compose up --force-renew-anon-volumes
```

Create the initial placement.

```bash
curl -X POST localhost:7201/api/v1/placement/init -d '{
    "num_shards": 64,
    "replication_factor": 1,
    "instances": [
        {
            "id": "m3db_seed",
            "isolation_group": "embedded",
            "zone": "embedded",
            "weight": 1024,
            "endpoint": "m3db_seed:9000",
            "hostname": "m3db_seed",
            "port": 9000
        }
    ]
}' | jq  .
```

Create the unaggregated namespace to store the raw data.

```bash
curl -X POST localhost:7201/api/v1/namespace -d '{
  "name": "raw_metrics",
  "options": {
    "bootstrapEnabled": true,
    "flushEnabled": true,
    "writesToCommitLog": true,
    "cleanupEnabled": true,
    "snapshotEnabled": true,
    "repairEnabled": false,
    "retentionOptions": {
      "retentionPeriodDuration": "48h",
      "blockSizeDuration": "2h",
      "bufferFutureDuration": "5m",
      "bufferPastDuration": "5m",
      "blockDataExpiry": true,
      "blockDataExpiryAfterNotAccessPeriodDuration": "5m"
    },
    "indexOptions": {
      "enabled": true,
      "blockSizeDuration": "2h"
    }
  }
}' | jq .
```

Create the aggregated namespace to store the aggregated data.

```bash
curl -X POST localhost:7201/api/v1/namespace -d '{
  "name": "metrics_5s_168h",
  "options": {
    "bootstrapEnabled": true,
    "flushEnabled": true,
    "writesToCommitLog": true,
    "cleanupEnabled": true,
    "snapshotEnabled": true,
    "repairEnabled": false,
    "retentionOptions": {
      "retentionPeriodDuration": "168h",
      "blockSizeDuration": "24h",
      "bufferFutureDuration": "5m",
      "bufferPastDuration": "5m",
      "blockDataExpiry": true,
      "blockDataExpiryAfterNotAccessPeriodDuration": "5m"
    },
    "indexOptions": {
      "enabled": true,
      "blockSizeDuration": "24h"
    }
  }
}' | jq .
```

### Graphite
Navigate to `http://localhost:3000` to open Grafana.

Add a new Graphite data source with the URL: `http://m3coordinator01:7201/api/v1/graphite`

Write data to the carbon ingestion port

```bash
(export now=$(date +%s) && echo "stats.sum.bar 10 $now" | nc 0.0.0.0 7204)

(export now=$(date +%s) && echo "stats.mean.bar 10 $now" | nc 0.0.0.0 7204)
```

Create a new panel to visualize the results with the following query: `transformNull(stats.*.bar)`

### Prometheus
Add a new Prometheus data source with the URL: `http://m3coordinator01:7201`

Data is already being scraped by Prometheus and written to M3, so add a new panel and visualize it with this sample query which shows the rate at which M3DB is writing to its commitlog: `sum(rate(commitlog_writes_success{}[30s]))`







