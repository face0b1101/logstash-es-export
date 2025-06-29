# Logstash Elasticsearch Export

A project demonstrating how to export data from Elasticsearch using Logstash.

## Overview

This project provides a dockerised Logstash setup that connects to an Elasticsearch instance (local or Elastic Cloud) and exports index data to NDJSON files.

## Components

- **Docker Compose**: Orchestrates the Logstash container with appropriate volumes and environment variables
- **Logstash Pipeline**: Configured to extract data from Elasticsearch and export it to NDJSON files
- **Export Directory**: Stores the resulting NDJSON files for further processing or import

## Prerequisites

- Docker and Docker Compose installed
- Access to an Elasticsearch cluster (local or Elastic Cloud)
- Elasticsearch API key or credentials

## Configuration

1. Create a `.env` file in the project root with the following variables:

```env
STACK_VERSION=8.18.2        # Or your preferred Elastic Stack version
ELASTIC_CLOUD_ID=           # Your Elastic Cloud ID (if using Elastic Cloud)
ELASTICSEARCH_HOST=         # Your Elasticsearch host (if not using Cloud ID)
ELASTICSEARCH_API_KEY=      # Your Elasticsearch API key
ELASTICSEARCH_INDEX=        # Index to export
```

1. Customise the Logstash pipeline in `logstash/pipeline/export.conf` if needed (e.g., to modify the query). By default, the pipeline will extract the entire index specified in the `.env` file.

## Usage

### Using Docker Compose

Start the export process with Docker Compose:

```shell
docker compose up
```

The exported data will be saved to the `export` directory as NDJSON files, named after the source index.

### Using Docker Directly

If you prefer to run without Docker Compose, you can use the standard Docker commands:

```shell
# Create a network for the container
docker network create logstash-es-export

# Run Logstash container with appropriate mounts and environment variables
docker run --name logstash-geo-export \
  --network logstash-es-export \
  -e LS_JAVA_OPTS="-Xmx1g -Xms1g" \
  -e PIPELINE_WORKERS="2" \
  -e ELASTIC_CLOUD_ID="$ELASTIC_CLOUD_ID" \
  -e ELASTICSEARCH_HOST="$ELASTICSEARCH_HOST" \
  -e ELASTICSEARCH_API_KEY="$ELASTICSEARCH_API_KEY" \
  -e ELASTICSEARCH_INDEX="$ELASTICSEARCH_INDEX" \
  --env-file .env \
  -v "$(pwd)/logstash/pipeline:/usr/share/logstash/pipeline" \
  -v "$(pwd)/logstash/config:/usr/share/logstash/config" \
  -v "$(pwd)/export:/usr/share/logstash/export" \
  -p 5044:5044 -p 9600:9600 \
  docker.elastic.co/logstash/logstash:$STACK_VERSION
```

Note: Make sure your environment variables are properly set or replace them with actual values in the command above.

### Worth Noting...

If your dataset does not contain a `@timestamp` field, Logstash will likely add one with the time of extraction.

If you want to remove the field, try using `jq`:

```shell
cp you-file.ndjson you-file.ndjson.bak;
cat you-file.ndjson | jq -c 'del(.["@timestamp"])' > you-file.ndjson.tmp && mv you-file.ndjson.tmp you-file.ndjson
```

## License

This project is licensed under GNU AGPL v3, as specified in the LICENSE file.
