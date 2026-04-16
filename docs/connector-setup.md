# Connector Setup

## Overview

This document outlines the setup and configuration for integrating the **AlienVault OTX** and **Splunk** connectors with OpenCTI. These connectors pull threat intelligence data from AlienVault OTX and allow Splunk to visualize and analyze the data ingested into OpenCTI.

### Key Steps

1. AlienVault OTX Connector Configuration
2. Splunk Connector Configuration


## 1. AlienVault OTX Connector Configuration

The AlienVault OTX connector is used to fetch threat intelligence data (observables, indicators, reports) from the AlienVault OTX platform and ingest it into OpenCTI.

### Step 1: Obtain AlienVault OTX API Key

* Register for an account on [AlienVault OTX](https://otx.alienvault.com/) if you don't already have one.
* Once logged in, go to your **profile settings** and generate a new **API key**.

### Step 2: Add API Key to `.env` File

Update the `.env` file with the AlienVault OTX API key.

```env
# AlienVault OTX API Key
ALIENVAULT_API_KEY=your_alienvault_api_key_here
```

### Step 3: Configure `docker-compose.yml`

In the `docker-compose.yml` file, add the AlienVault OTX connector service configuration under the `services` section. Ensure the configuration includes the API key and any other necessary settings like update frequency or polling intervals.

Example:

```yaml
   connector-alienvault:
    image: opencti/connector-alienvault:7.260401.0
    environment:
      - OPENCTI_URL=http://opencti:8080
      - OPENCTI_TOKEN=${OPENCTI_ADMIN_TOKEN}
      - CONNECTOR_ID=Generate new UUID
      - CONNECTOR_NAME=AlienVault
      - CONNECTOR_SCOPE=alienvault
      - CONNECTOR_LOG_LEVEL=debug
      - CONNECTOR_CONFIDENCE_LEVEL=50
      - CONNECTOR_LIVE_STREAM_ID=live
      - CONNECTOR_INITIAL_DELAY=120
      - CONNECTOR_DURATION_PERIOD=PT30M
      - ALIENVAULT_BASE_URL=https://otx.alienvault.com
      - ALIENVAULT_API_KEY=${ALIENVAULT_API_KEY}
      - ALIENVAULT_TLP=White
      - ALIENVAULT_CREATE_OBSERVABLES=true
      - ALIENVAULT_CREATE_INDICATORS=true
      - ALIENVAULT_PULSE_START_TIMESTAMP=2026-04-01T00:00:00
      - ALIENVAULT_REPORT_TYPE=threat-report
      - ALIENVAULT_REPORT_STATUS=New
      - ALIENVAULT_FILTER_INDICATORS=false
    restart: always
```

### Step 4: Restart the Docker Containers

After adding the AlienVault connector configuration, restart the Docker containers to apply the changes:

```bash id="s9k93n"
docker compose up -d
```

### Step 5: Verify AlienVault OTX Data in OpenCTI

* Access the OpenCTI UI (`http://<VM-IP>:8080`).
* Go to **Data → Ingestion**.
* Check the **AlienVault OTX** connector status. It should show as active and ingesting data.
* Confirm that the AlienVault threat intelligence data (indicators, observables, and reports) has been ingested into OpenCTI.

<img src="/docs/screenshots/AlienVault_Connector.png">

## 2. Splunk Connector Configuration

The Splunk connector enables the visualization of CTI data within Splunk by linking it to the OpenCTI platform.

### Step 1: Obtain Splunk Token

* Log in to your Splunk enterprise.
* Navigate to **Settings → Users and Authentications → Token**.
* Create a new a token with necessary details.

### Step 2: Add Token to `.env` File

Update the `.env` file with your Splunk token and URL:

```env
# Splunk Configuration
SPLUNK_TOKEN=your_splunk_token_here
```

### Step 3: Configure `docker-compose.yml`

In the `docker-compose.yml` file, configure the Splunk connector by adding the following under `services`. This configuration allows OpenCTI to send data to Splunk through the HEC endpoint.

Example configuration for the Splunk connector:

```yaml
   connector-splunk:
    image: opencti/connector-splunk:latest
    environment:
      - OPENCTI_URL=http://opencti:8080
      - OPENCTI_TOKEN=${OPENCTI_ADMIN_TOKEN}
      - CONNECTOR_ID=Generate new UUID
      - CONNECTOR_LIVE_STREAM_ID=live # ID of the live stream created in the OpenCTI UI
      - CONNECTOR_LIVE_STREAM_LISTEN_DELETE=true
      - CONNECTOR_LIVE_STREAM_NO_DEPENDENCIES=true
      - CONNECTOR_NAME=OpenCTI_Splunk_Connector
      - CONNECTOR_SCOPE=splunk
      - CONNECTOR_CONFIDENCE_LEVEL=80 # From 0 (Unknown) to 100 (Fully trusted)
      - CONNECTOR_LOG_LEVEL=error
      - SPLUNK_URL=https://Your_Splunk_IP:8089
      - SPLUNK_TOKEN=${SPLUNK_TOKEN}
      - SPLUNK_AUTH_TYPE=Bearer
      - SPLUNK_OWNER=nobody
      - SPLUNK_SSL_VERIFY=false
      - SPLUNK_APP=search
      - SPLUNK_KV_STORE_NAME=opencti
      - SPLUNK_IGNORE_TYPES="attack-pattern,campaign,course-of-action,data-component,data-source,external-reference,identity,intrusion-set,kill-chain-phase,label,location,malware,marking-definition,relationship,threat-actor,tool,vocabulary,vulnerability"
    restart: always
```

### Step 4: Create a new lookup definition

Under settings select Lookups and create a new lookup definition as shown below.

<img src="/docs/screenshots/Lookup.png">


### Step 5: Restart Docker Containers

After adding the Splunk connector configuration, restart the containers to apply the changes:

```bash id="gbq74r"
docker compose up -d
```

### Step 6: Verify Splunk Integration

* In Splunk, verify that the OpenCTI data is being received by searching for logs or events.
* Use a search query in Splunk's Search and Reporting such as:

```spl
|inputlookup opencti_splunk
```

* Check for relevant CTI data, including indicators, observables, and reports that have been ingested from OpenCTI.

<img src="/docs/screenshots/Splunk_Search_Query.png">

---

## Troubleshooting

### AlienVault OTX Connector Issues

* **No data ingested**: Check if the API key is correctly added in the `.env` file. Verify that the connector is running using `docker-compose ps`.
* **Connector not starting**: Ensure that the OpenCTI platform is running and the `ALIENVAULT_API_KEY` is correctly set in the `.env` file.

### Splunk Connector Issues

* **No data in Splunk**: Double-check that the token is correctly set in the `.env` file. Ensure that the Splunk lookup is enabled and configured correctly.
* **Data format errors**: Ensure that the data from OpenCTI is correctly formatted for Splunk. If issues persist, check the logs for any errors related to token authentication or data formatting.
