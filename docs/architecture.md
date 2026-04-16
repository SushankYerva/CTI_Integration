# Architecture Overview

## System Components

The CTI Integration Pipeline involves several core components that work together to ingest, process, and visualize threat intelligence data. The architecture utilizes **OpenCTI**, **AlienVault OTX**, and **Splunk**, all orchestrated through **Docker Compose** on an **Ubuntu VM**.

### Key Components

1. **Ubuntu Virtual Machine**
   The infrastructure is provisioned on an Ubuntu-based virtual machine (VM) that hosts all services, including Docker and Docker Compose, for container orchestration.

2. **Docker & Docker Compose**
   Docker is used to containerize each service, ensuring that dependencies are correctly isolated and easy to manage. Docker Compose defines and manages multi-container setups, including OpenCTI, Elasticsearch, Redis, RabbitMQ, MinIO, AlienVault OTX connector, and Splunk.

3. **OpenCTI**
   The core of the pipeline, OpenCTI is an open-source threat intelligence platform used to normalize and correlate incoming threat data. It serves as the centralized platform for receiving and storing CTI data.

4. **AlienVault OTX Connector**
   AlienVault OTX is a widely used threat intelligence feed. This connector pulls threat data (including observables, indicators, and reports) from AlienVault OTX and ingests it into OpenCTI.

5. **Splunk Connector**
   Splunk is a SIEM (Security Information and Event Management) tool used to visualize and analyze the CTI data ingested by OpenCTI. The Splunk connector pulls the relevant CTI data from OpenCTI and makes it available in Splunk through token-based authentication.

6. **Supporting Services**

   * **Elasticsearch**: Used by OpenCTI to index and search threat data.
   * **Redis**: A caching and message broker service used by OpenCTI for data queueing.
   * **RabbitMQ**: A message queue service used by OpenCTI to handle asynchronous tasks.
   * **MinIO**: A scalable object storage service used for storing data, such as documents or reports.



## Data Flow Diagram

```plaintext
+-----------------------+             +---------------------+             +-------------------+
|     AlienVault        |             |      OpenCTI        |             |                   |
|      OTX Feed         |             |     (Core Platform) |             |     Splunk        |
|  (Threat Intelligence)|  -->  -->   |                     |  -->  -->   | (Visualization &  |
|    (API Integration)  |             |  - Normalize Data   |             |   Analysis)       |
|  - Observables        |             |  - Correlate Data   |             |                   |
|  - Indicators         |             |  - Store in Elastic |             |                   |
|  - Reports            |             |  - Message Queues   |             |                   |
+-----------------------+             +---------------------+             +-------------------+
                                    
                              
```



## Component Interactions

1. **AlienVault OTX → OpenCTI**
   The AlienVault OTX connector continuously pulls threat intelligence data, including indicators, observables, and reports, from the AlienVault API. The data is ingested into OpenCTI, where it is normalized and correlated to provide actionable intelligence.

2. **OpenCTI → Elasticsearch**
   Once ingested, the data is stored in **Elasticsearch**. Elasticsearch provides fast indexing and searching capabilities, enabling OpenCTI to efficiently search through the threat data.

3. **OpenCTI → Redis / RabbitMQ**
   Redis and RabbitMQ are used for internal communication within OpenCTI. Redis acts as a caching layer, while RabbitMQ handles asynchronous tasks and message queuing, allowing for decoupling between services.

4. **OpenCTI → MinIO**
   MinIO is used for storing large binary files, such as documents, reports, or any large objects related to threat intelligence. It provides high scalability and redundancy.

5. **OpenCTI → Splunk**
   After data normalization and storage, **Splunk** is used for advanced data analysis and visualization. The Splunk connector retrieves relevant data from OpenCTI, and the data is made available for queries, dashboards, and alerts within Splunk.



## Service Dependencies

1. **OpenCTI** depends on **Elasticsearch** for data indexing, **Redis** and **RabbitMQ** for message queuing and caching, and **MinIO** for object storage.
2. The **AlienVault OTX Connector** depends on the correct configuration of the AlienVault API key and OpenCTI’s connector settings to pull threat intelligence data.
3. The **Splunk Connector** requires token-based authentication for connecting OpenCTI and Splunk, ensuring that data flows securely between the two platforms.


## Key Takeaways

* **Modular Architecture**: The system is highly modular, allowing you to swap or upgrade individual services like OpenCTI, AlienVault, and Splunk.
* **Containerization**: Docker and Docker Compose ensure the platform is easy to set up, manage, and scale.
* **Real-time Data Flow**: The threat data flows seamlessly from AlienVault OTX to OpenCTI and is then made available for visualization and analysis in Splunk.
* **Scalable and Reproducible**: The architecture is designed for scalability, with components like Elasticsearch and MinIO capable of handling large datasets efficiently.



## Future Improvements

* Add additional CTI sources such as MISP and IBM X-Force
* Enhance the Splunk integration by creating pre-configured dashboards for threat analysis
* Automate connector health checks and notifications
* Implement real-time alerts in OpenCTI based on specific indicator thresholds



This document serves as an architectural overview of the CTI integration pipeline, providing insight into how each component interacts within the overall system.

---
