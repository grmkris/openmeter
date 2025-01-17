# Quickstart

## Prerequisites

- Docker (with Compose)
- curl
- jq

Clone the repository:

```sh
git clone git@github.com:openmeterio/openmeter.git
cd openmeter/quickstart
```

## 1. Launch OpenMeter

Launch OpenMeter and its dependencies via:

```sh
docker compose up -d
```

## 2. Ingest usage event(s)

Ingest usage events in [CloudEvents](https://cloudevents.io/) format:

```sh
curl -X POST http://localhost:8888/api/v1/events \
-H 'Content-Type: application/cloudevents+json' \
--data-raw '
{
  "specversion" : "1.0",
  "type": "api-calls",
  "id": "00001",
  "time": "2023-01-01T00:00:00.001Z",
  "source": "service-0",
  "subject": "customer-1",
  "data": {
    "duration_ms": "1",
    "method": "GET",
    "path": "/hello"
  }
}
'
```

Note how ID is different:

```sh
curl -X POST http://localhost:8888/api/v1/events \
-H 'Content-Type: application/cloudevents+json' \
--data-raw '
{
  "specversion" : "1.0",
  "type": "api-calls",
  "id": "00002",
  "time": "2023-01-01T00:00:00.001Z",
  "source": "service-0",
  "subject": "customer-1",
  "data": {
    "duration_ms": "1",
    "method": "GET",
    "path": "/hello"
  }
}
'
```

Note how ID and time are different:

```sh
curl -X POST http://localhost:8888/api/v1/events \
-H 'Content-Type: application/cloudevents+json' \
--data-raw '
{
  "specversion" : "1.0",
  "type": "api-calls",
  "id": "00003",
  "time": "2023-01-02T00:00:00.001Z",
  "source": "service-0",
  "subject": "customer-1",
  "data": {
    "duration_ms": "1",
    "method": "GET",
    "path": "/hello"
  }
}
'
```

## 3. Query Usage

Query the usage hourly:

```sh
curl 'http://localhost:8888/api/v1/meters/m1/query?windowSize=HOUR&groupBy=method&groupBy=path' | jq
```

```json
{
  "windowSize": "HOUR",
  "data": [
    {
      "value": 2,
      "windowStart": "2023-01-01T00:00:00Z",
      "windowEnd": "2023-01-01T01:00:00Z",
      "subject": null,
      "groupBy": {
        "method": "GET",
        "path": "/hello"
      }
    },
    {
      "value": 1,
      "windowStart": "2023-01-02T00:00:00Z",
      "windowEnd": "2023-01-02T01:00:00Z",
      "subject": null,
      "groupBy": {
        "method": "GET",
        "path": "/hello"
      }
    }
  ]
}
```

Query the total usage for `customer-1`:

```sh
curl 'http://localhost:8888/api/v1/meters/m1/query?subject=customer-1' | jq
```

```json
{
  "data": [
    {
      "value": 3,
      "windowStart": "2023-01-01T00:00:00Z",
      "windowEnd": "2023-01-02T00:01:00Z",
      "subject": "customer-1",
      "groupBy": {}
    }
  ]
}
```

## 4. Configure additional meter(s) _(optional)_

Configure how OpenMeter should process your usage events.
In this example we will meter the execution duration per API invocation, groupped by method and path.
You can think about it how AWS Lambda [charges](https://aws.amazon.com/lambda/pricing/) by execution duration on a millisecond level.

```yaml
# ...

meters:
  - slug: m1
    eventType: api-calls
    valueProperty: $.duration_ms
    aggregation: SUM
    groupBy:
      method: $.method
      path: $.path
```

## Cleanup

Once you are done, stop any running instances:

```sh
docker compose down -v
```
