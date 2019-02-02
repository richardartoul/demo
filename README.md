# M3 Demo

This repository contains a docker-compose file which can be used to setup a demo of the M3 stack. It runs the following containers:

1. M3DB
2. M3Coordinator
3. Prometheus
4. Grafana

## Setup

```bash
# Start all the containers.
docker-compose up

# Create the metrics_raw namespace to store the unaggregated data.
curl -X POST http://localhost:7201/api/v1/database/create -d '{
  "type": "local",
  "namespaceName": "metrics_raw",
  "retentionTime": "48h"
}'

# Create the aggregated namespace to store the aggregated data.
curl -vvvsSf -X POST localhost:7201/api/v1/namespace -d '{
  "name": "metrics_5s_4320h",
  "options": {
    "bootstrapEnabled": true,
    "flushEnabled": true,
    "writesToCommitLog": true,
    "cleanupEnabled": true,
    "snapshotEnabled": true,
    "repairEnabled": false,
    "retentionOptions": {
      "retentionPeriodDuration": "4320h",
      "blockSizeDuration": "24h",
      "bufferFutureDuration": "5m",
      "bufferPastDuration": "5m",
      "blockDataExpiry": true,
      "blockDataExpiryAfterNotAccessPeriodDuration": "5m"
    },
    "indexOptions": {
      "enabled": true,
      "blockSizeDuration": "10m"
    }
  }
```