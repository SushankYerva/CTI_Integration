# Deployment Steps

## Overview

This document outlines the step-by-step process to deploy the CTI Integration Pipeline using OpenCTI, AlienVault OTX, and Splunk on an Ubuntu virtual machine using Docker Compose.



## 1. Provision Ubuntu VM

* Create an Ubuntu VM (recommended: Ubuntu 22.04 LTS)
* Minimum specs:

  * 4 vCPU
  * 8 GB RAM (OpenCTI stack is heavy)
  * 50 GB storage

Update system packages:

```bash
sudo apt update && sudo apt upgrade -y
```



## 2. Install Docker and Docker Compose

### Install Docker Compose

```bash
sudo apt install -y docker-compose
```

Verify installation:

```bash
docker --version
docker compose version
```



## 3. Clone Repository

```bash
git clone https://github.com/OpenCTI-Platform/docker.git
cd docker
```



## 4. Configure Environment Variables

Create `.env` file:

```bash
cp .env.example .env
```

Update required values:

* OpenCTI admin email and password
* OpenCTI base URL
* AlienVault OTX API key
* Splunk token and endpoint
* Connector configuration values

**Important:** Do not commit `.env` to GitHub.

**Note:** Use the OpenCTI official website for details related to .env (https://docs.opencti.io/latest/deployment/installation/#using-docker). 



## 5. Edit and Review docker-compose.yml

Add the yml code for AlienVault and Slunk connectors into docker-compose.yml which can be found in OpenCTI GitHub repo. [Connectors](https://github.com/OpenCTI-Platform/connectors/tree/master/external-import)

Key changes:

* OPENCTI_URL
* OPENCTI_TOKEN
* CONNECTOR_ID
* API_KEY

Use UUID Generator to create UUIDs for CONNECTOR_ID.

Ensure the following services are present:

* OpenCTI core platform
* Elasticsearch
* Redis
* RabbitMQ
* MinIO
* AlienVault OTX connector
* Splunk connector

Key checks:

* Environment variables are correctly referenced
* Connector services include proper configuration
* Ports are correctly mapped
* Dependencies are defined properly



## 6. Start the CTI Platform

Run:

```bash
docker-compose up -d
```

This will start all services in detached mode.

---

## 7. Verify Container Status

```bash
docker-compose ps
```

Ensure all services are running:

* opencti
* elasticsearch
* redis
* rabbitmq
* minio
* connectors

<img src="/docs/screenshots/Docker_ps.png">

## 8. Access OpenCTI

Open browser:

```
http://<ip-of-your-vm>:8080
```

Login using credentials defined in `.env`.

---

## 9. Verify Connector Health

In OpenCTI UI:

* Navigate to **Data → Ingestion**
* Confirm:

  * AlienVault connector is active
  * Splunk connector is active
  * No error states

<img src="/docs/screenshots/Data_Ingestion.png">

## 10. Validate AlienVault OTX Ingestion

* Wait for initial connector sync (can take time depending on configuration)
* Check:

  * Indicators
  * Observables
  * Reports

Expected result:

* AlienVault pulses start appearing in OpenCTI


## 11. Configure and Validate Splunk Integration

* Ensure Splunk token is correctly configured
* Verify connection between OpenCTI and Splunk
* In Splunk:

  * Search for ingested CTI data
  * Validate logs, indicators, or events

Expected result:

* CTI data from OpenCTI is visible in Splunk



## 12. Troubleshooting

Check logs for troubleshooting:

```bash
docker logs opencti
docker logs <connector-container-name>
```

For full stack logs:

```bash
docker compose logs -f
```



## 13. Stop the Platform

```bash
docker compose down
```



## Notes

* Initial ingestion from AlienVault may take time depending on:

  * connector configuration
  * polling intervals
* Ensure sufficient system memory; Elasticsearch may fail if resources are low
* Splunk visibility depends on correct token and endpoint configuration



## Common Issues

### 1. Containers not starting

* Check logs using `docker compose logs`
* Verify `.env` values

### 2. No data ingestion

* Wait for connector polling cycle
* Check connector status in OpenCTI

### 3. Elasticsearch issues

* Ensure sufficient memory allocation
* Verify container health

### 4. Splunk not receiving data

* Validate token and endpoint configuration
* Check network connectivity between OpenCTI and Splunk
