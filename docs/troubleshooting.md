# Troubleshooting Guide

## Overview

This troubleshooting guide provides solutions to common issues encountered during the deployment and operation of the **CTI Integration Pipeline** with **OpenCTI**, **AlienVault OTX**, and **Splunk**. The issues typically relate to Docker container failures, connector misconfigurations, or issues with data ingestion and visualization.

---

## 1. General Docker Issues

### Docker Containers Not Starting

**Problem**: Containers fail to start or exit unexpectedly.

**Possible Causes**:

* Insufficient system resources (RAM, CPU).
* Incorrect configuration in `.env` or `docker-compose.yml`.
* Missing Docker images or corrupted containers.

**Solutions**:

1. **Check Logs**:
   Use the following command to get detailed logs for each service:

   ```bash
   docker-compose logs <service_name>
   ```

   Replace `<service_name>` with the container name (e.g., `opencti`, `alienvault_otx`, `splunk_connector`).

2. **Verify Resource Allocation**:
   Ensure the VM has enough memory (minimum 8GB recommended) and CPU resources. Increase Docker’s memory allocation if needed.

3. **Rebuild Docker Containers**:
   Rebuild the containers if the images are corrupted or outdated:

   ```bash
   docker-compose down
   docker-compose up --build
   ```

4. **Check Docker Version**:
   Ensure you are running a compatible version of Docker. Use the following commands to verify:

   ```bash
   docker --version
   docker-compose version
   ```

---

## 2. AlienVault OTX Connector Issues

### No Data Being Ingested from AlienVault OTX

**Problem**: The AlienVault OTX connector is not fetching or ingesting data into OpenCTI.

**Possible Causes**:

* Incorrect API key or configuration.
* Connectivity issues with AlienVault OTX API.
* Insufficient permissions in AlienVault OTX account.

**Solutions**:

1. **Verify API Key**:
   Ensure the AlienVault OTX API key in the `.env` file is correct. If needed, regenerate a new API key from the [AlienVault OTX portal](https://otx.alienvault.com/).

2. **Check Connector Logs**:
   View the logs for the AlienVault connector:

   ```bash
   docker logs <alienvault_otx_connector_container_name>
   ```

   Look for errors like authentication failures or connection timeouts.

3. **Test API Connectivity**:
   Try accessing the AlienVault OTX API using a tool like `curl` to ensure that the system is able to connect:

   ```bash
   curl -H "X-OTX-API-KEY: <your_api_key>" https://otx.alienvault.com/api/v1/pulses/subscribed
   ```

4. **Check Polling Interval**:
   AlienVault OTX connector pulls data at a defined interval. It may take some time before data is ingested, depending on the configuration.

---

### AlienVault Connector Not Showing in OpenCTI

**Problem**: The AlienVault OTX connector is not visible or active in OpenCTI’s UI.

**Possible Causes**:

* The connector is not running.
* Misconfiguration in `docker-compose.yml`.
* OpenCTI service issues.

**Solutions**:

1. **Check Docker Status**:
   Ensure the connector service is running by listing all containers:

   ```bash
   docker-compose ps
   ```

2. **Verify Connector Configuration**:
   Double-check the `docker-compose.yml` file to ensure that the `opencti_url` and `OTX_API_KEY` are correctly set.

3. **Restart OpenCTI**:
   Sometimes, restarting OpenCTI may resolve the issue:

   ```bash
   docker-compose restart opencti
   ```

---

## 3. Splunk Connector Issues

### Splunk Connector Not Receiving Data

**Problem**: The Splunk connector is not receiving any data from OpenCTI.

**Possible Causes**:

* Incorrect Splunk token or endpoint configuration.
* Splunk lookup definition is not enabled.
* Network issues between OpenCTI and Splunk.

**Solutions**:

1. **Verify Token and URL**:
   Double-check the **SPLUNK_TOKEN** and **SPLUNK_URL** in the `.env` file. Ensure that the token has the necessary permissions for writing data to Splunk.

2. **Ensure lookup is Enabled in Splunk**:

   * In Splunk, go to **Settings → Lookups -> Lookup definitons**.
   * Verify that it is enabled and proper permissions are granted and make sure the collection name is same as SPLUNK_KV_STORE_NAME under docker-compose.yml.

3. **Check Network Connectivity**:
   Ensure that the Ubuntu VM running OpenCTI can reach the Splunk instance. Try testing connectivity with `curl`:

   ```bash
   curl -k -H "Authorization: Splunk <your_splunk_token>" <splunk_url>:8088
   ```

4. **View Splunk Connector Logs**:
   Check the logs for any error messages related to the Splunk connector:

   ```bash
   docker logs <splunk_connector_container_name>
   ```

5. **Test Data Flow**:

   * Search for data in Splunk using queries like:

     ```spl
     index=opencti sourcetype="opencti_events"
     ```

---

## 4. Elasticsearch Issues

### Elasticsearch Shard Unavailable

**Problem**: Elasticsearch shows shard unavailability or errors.

**Possible Causes**:

* Insufficient system resources (memory or disk).
* Elasticsearch node failed to start.

**Solutions**:

1. **Check Elasticsearch Logs**:
   View the Elasticsearch logs to identify the specific issue:

   ```bash
   docker logs <elasticsearch_container_name>
   ```

2. **Increase Memory Allocation**:
   Elasticsearch requires significant memory for performance. Increase the memory allocation in the `docker-compose.yml` for Elasticsearch if needed:

   ```yaml
   services:
     elasticsearch:
       environment:
         - ES_JAVA_OPTS=-Xms4g -Xmx4g
   ```

3. **Restart Elasticsearch**:
   Restart the Elasticsearch container to recover from shard issues:

   ```bash
   docker-compose restart elasticsearch
   ```

---

## 5. Redis / RabbitMQ Issues

### Redis or RabbitMQ Failures

**Problem**: Redis or RabbitMQ is not functioning correctly, causing delays or failures in data processing.

**Possible Causes**:

* Insufficient resources (CPU, memory).
* Incorrect Docker network configuration.
* Service misconfiguration.

**Solutions**:

1. **Check Service Status**:
   Ensure that the services are running:

   ```bash
   docker-compose ps
   ```

2. **View Logs**:
   View the logs for Redis or RabbitMQ:

   ```bash
   docker logs <redis_container_name>
   docker logs <rabbitmq_container_name>
   ```

3. **Verify Configuration**:
   Double-check the configuration in `docker-compose.yml` to ensure network settings and dependencies are correctly defined.

4. **Restart Services**:
   Restart Redis and RabbitMQ if needed:

   ```bash
   docker-compose restart redis
   docker-compose restart rabbitmq
   ```

---

## 6. MinIO Issues

### MinIO Not Accessible

**Problem**: MinIO is not accessible or data is not being stored.

**Possible Causes**:

* Incorrect MinIO credentials or endpoint.
* Network issues between containers.

**Solutions**:

1. **Check MinIO Logs**:

   ```bash
   docker logs <minio_container_name>
   ```

2. **Verify Configuration**:
   Check the MinIO configuration in `.env` and `docker-compose.yml`.

3. **Restart MinIO**:
   Restart the MinIO container if it’s not starting properly:

   ```bash
   docker-compose restart minio
   ```

---

## 7. Performance Issues

### Slow Data Ingestion or Search Performance

**Problem**: Data ingestion or search queries are slower than expected.

**Possible Causes**:

* Elasticsearch is not optimized.
* Insufficient system resources.

**Solutions**:

1. **Increase Elasticsearch Resources**:
   Ensure Elasticsearch has enough memory allocated in `docker-compose.yml`:

   ```yaml
   environment:
     - ES_JAVA_OPTS=-Xms4g -Xmx4g
   ```

2. **Optimize Elasticsearch Indexing**:
   Review Elasticsearch settings for better performance, such as adjusting the number of shards and replicas for the indices.

3. **Scale Docker Resources**:
   Consider increasing the CPU and memory allocation for Docker containers, especially for Elasticsearch and OpenCTI.

