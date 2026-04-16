# CTI Integration Pipeline with OpenCTI

## Overview

This project demonstrates the deployment and integration of an end-to-end Cyber Threat Intelligence (CTI) pipeline using OpenCTI, AlienVault OTX, and Splunk on an Ubuntu virtual machine. The platform is deployed using Docker Compose, configured through environment variables and service definitions, and extended with external connectors to ingest and visualize threat intelligence data.

The focus of this project is on **deployment, connector integration, data ingestion, and validation of CTI workflows**.

<img src="/docs/screenshots/OpenCTI_Dashboard.png">

## Objectives

* Deploy OpenCTI on an Ubuntu VM using Docker Compose
* Configure platform services using `.env` and `docker-compose.yml`
* Integrate AlienVault OTX as a threat intelligence source
* Integrate Splunk for downstream visualization
* Validate ingestion and data flow across the pipeline
* Document setup, architecture, and troubleshooting steps

## Tech Stack

* Ubuntu Virtual Machine
* Docker & Docker Compose
* OpenCTI
* AlienVault OTX Connector
* Splunk
* Elasticsearch
* Redis
* RabbitMQ
* MinIO

## Architecture Overview

The system is designed as a modular CTI pipeline:

1. **AlienVault OTX** acts as the external threat intelligence source
2. **OpenCTI** ingests, normalizes, and correlates threat intelligence
3. **Splunk** consumes and visualizes CTI data for analysis

A full architecture diagram is available in the `assets/` directory.


## Project Workflow

1. Provision an Ubuntu virtual machine
2. Install Docker and Docker Compose
3. Deploy OpenCTI core services
4. Configure environment variables in `.env`
5. Modify `docker-compose.yml` to include AlienVault and Splunk connectors
6. Start the platform using Docker Compose
7. Allow AlienVault OTX connector to ingest threat intelligence into OpenCTI
8. Configure Splunk integration using token-based authentication
9. Validate visibility of CTI data in both OpenCTI and Splunk

## Repository Structure

```bash
cti-integration-pipeline/
├── README.md
├── .env.example
├── docker-compose.yml
├── docs/
    ├── architecture.md
    ├── deployment-steps.md
    ├── connector-setup.md
    ├── troubleshooting.md
    └── screenshots/
```
## Screenshots

Available in `docs/screenshots/`:

* OpenCTI dashboard
* Connector status (AlienVault & Splunk)
* Threat intelligence ingestion (indicators/observables)
* Splunk dashboards and queries
* Docker container status

## Troubleshooting Highlights

* Adjusted environment configurations to align connector settings with OpenCTI
* Modified Docker Compose to correctly include and initialize connectors
* Verified service dependencies across Redis, RabbitMQ, and Elasticsearch
* Observed ingestion delays due to connector polling intervals
* Validated Splunk token-based integration for downstream visualization

## Future Improvements

* Integrate additional CTI sources such as MISP
* Automate connector health monitoring
* Add prebuilt Splunk dashboards
* Introduce sample STIX datasets for testing
* Implement CI checks for configuration validation

## Notes

* This project is intended as a hands-on CTI integration exercise
* All sensitive credentials and keys have been removed
* `.env.example` is provided for reproducibility

## License

This project is for educational and demonstration purposes.
