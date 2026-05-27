# Case Study - Telemetry Car System

## Overview

A company that manufactures autonomous systems for vehicles needs a new computer system to reliably receive telemetry from cars and display data about them.

**Current Scale:**
- 10,000+ vehicles on the roads
- Expected to grow to 200,000+ vehicles by end of year

---

## Functional Requirements

**Web-based system** with the following capabilities:

### Telemetry Collection
- Receive telemetry data from cars (location, speed, breakdowns, etc.)

### Data Storage
- Store telemetry in a persistent data store

### Data Visualization
- Display dashboards summarizing the data

### Data Analysis
- Perform analysis on the collected telemetry data

---

## Non-Functional Requirements

### System Characteristics
- **Data intensive system**: Large volume of incoming data
- **User base**: Not a lot of users (mainly data analysts)
- **Data volume**: Very large amount of data
- **Performance**: Real-time data processing required (no stale data)

### Technical Specifications
- **Message throughput**: 7,000 messages per second
- **Operational database size**: Maximum 4TB
- **Data retention**: Must define retention period to prevent database from growing infinitely and maintain query performance

### Data Characteristics (from customer)
- **Message schema**: Schema-less (no definite structure, messages can differ from each other)
- **Message loss tolerance**: Some message loss is acceptable

---

## Key Considerations

### Data Retention
- Defines how long records are kept in the database
- Prevents database from exploding in size
- Improves query performance by limiting data volume

---

<img width="3172" height="1460" alt="Blank diagram (2)" src="https://github.com/user-attachments/assets/b68f4f69-d799-4764-843f-b8e5cb4848f7" />
<img width="4843" height="1738" alt="mermaid-diagram (1)" src="https://github.com/user-attachments/assets/ef9a15ef-6309-42f0-a282-430f5474713f" />



## Architecture Reasoning

### AWS IoT Core
- Receives telemetry data from vehicles using MQTT
- Designed for continuous IoT device communication
- Handles large numbers of connected vehicles

### Why AWS IoT Core instead of normal HTTP APIs
- HTTP creates more overhead for frequent telemetry updates
- Continuous HTTP calls from thousands of vehicles can increase latency and server load
- MQTT is lighter and better for frequent small messages from IoT devices

### Kinesis Data Stream
- Buffers and streams telemetry data
- Handles high message throughput
- Prevents traffic spikes from overwhelming the system

### AWS Lambda - Telemetry Processor
- Processes telemetry events in near real time
- Scales automatically based on incoming traffic
- Writes recent data to DynamoDB and historical data to S3

### Why AWS Lambda instead of EC2
- Telemetry processing is event-driven
- Lambda runs only when data arrives
- EC2 needs always-running servers and more manual management

### Amazon DynamoDB
- Stores recent operational telemetry data
- Supports flexible/schema-less records
- Provides fast reads and writes for monitoring dashboards

### Why DynamoDB instead of SQL
- Telemetry messages can have different structures
- DynamoDB handles high write throughput well
- SQL is better for relational data, but telemetry mainly needs fast writes and recent lookups

### Admin Monitoring / Data Visualization
- Displays summarized telemetry data for admin users
- Uses recent operational data from DynamoDB
- Good for current vehicle status, recent activity, and active alerts

### Dashboard API
- Serves telemetry data to the admin dashboard
- Handles filtering and request processing
- Prevents direct frontend access to DynamoDB

### Dashboard UI
- Provides a simple web dashboard for admins
- Shows telemetry summaries and vehicle monitoring data
- Focused on viewing and monitoring, not deep analysis

### Amazon S3 Data Lake
- Stores historical telemetry data
- Cheaper storage for long-term retention
- Used as the source for analytics and reporting

### Data Retention Strategy
- Keeps only recent data in DynamoDB
- Moves older data to S3 for long-term storage
- Prevents the operational database from growing too large

### Amazon Athena
- Runs SQL queries on telemetry data stored in S3
- Used for historical analysis and reporting
- Avoids loading all historical data into DynamoDB

### Amazon QuickSight
- Creates BI dashboards and reports
- Visualizes historical telemetry trends
- Used by analysts for insights and reporting

### Data Analysis / Reporting
- Uses historical telemetry stored in the S3 data lake
- Athena queries the historical data
- QuickSight visualizes trends, reports, and long-term insights
- Keeps only recent data in the operational store (e.g., last 30 days)
- Archives older data to Azure Storage for long-term use
- Improves query performance by limiting active dataset size
- Reduces operational costs by using appropriate storage tiers

## Current setup VS Redshift
Redshift is more suitable for heavy enterprise analytics workloads that require very fast BI queries, complex joins, high concurrency, and warehouse-level performance. For this telemetry system, the current S3 + Athena + QuickSight setup is already sufficient because it provides a simpler and more cost-effective approach for storing and analyzing large volumes of historical telemetry data
