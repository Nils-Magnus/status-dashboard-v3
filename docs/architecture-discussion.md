# Architecture Specification for Status Dashboard (SD3)

## 1. Overview

### 1.1 Purpose
The Status Dashboard (SD3) is an information system designed to display the current state of cloud services in the Open Telekom Cloud. It serves two primary user groups:
- **Customers** who want to stay informed about incidents and ongoing maintenance activities.
- **Service Managers** responsible for coordinating incident resolution and communicating progress to customers.

The third version of the Status Dashboard (SD3) aims to introduce an improved look and feel, better usability, and enhanced user experience for both use cases.

---

## 2. Architecture Design

### 2.1 High-Level Architecture
SD3 will be implemented as a client-server solution using Golang for backend services and a SQL database for persistent data storage. The frontend will consume APIs exposed by the backend to display event statuses and management options.

#### 2.1.1 Key Components
- **Frontend (Client):** Web-based user interface displaying event data.
- **Backend (Server):** RESTful API implemented in Golang to handle event processing and user interactions.
- **Database:** SQL database to store event details, incident statuses, and system updates.
- **Metric Processor:** External component responsible for detecting incidents and forwarding event notifications to SD3.

---

## 3. Core Functionalities

### 3.1 Event Processing Workflow

#### 3.1.1 Event Detection & Creation
1. The Metric Processor detects an event and sends the following data to SD3:
   - Affected Component (String Identifier)
   - Region (String Identifier)
   - Impact Level (Enum: 0 = Reserved, 1 = Minor Incident, 2 = Major Incident, 3 = Outage, 4 = Planned Maintenance)
2. SD3 checks if an identical event (same component, region, and impact) already exists in the database:
   - If a matching event exists, the new notification is ignored.
   - If the event is new, SD3 creates a new entry:
     - **Component** = "Cloud Eye"
     - **Region** = "eu-de"
     - **Impact** = 2 (Major Incident)
     - **Start Time** = NOW
     - **End Time** = NOW + SLA Reaction Time (e.g., 2 hours)
     - **Status** = 1 ("Investigating")
     - **ID** = Unique Identifier (e.g., auto-increment or SHA-1 based on start time)
     - **Title** = "Major Incident - Cloud Eye"
     - **Description** = "A major incident has been detected at {start_time} affecting {component}. Investigation is ongoing until {end_time}."

#### 3.1.2 Event Display Table
SD3 will present a table with the following attributes:
- ID
- Title
- Start Time
- End Time
- Status
- Component
- Region
- Impact (color-coded label)
- Link to event details

#### 3.1.3 Incident Management
Service Managers can manage incidents through the Admin Backend:
1. Select an event from the event table.
2. Update event details via an editable form:
   - Title
   - Description
   - Start & End Time (with time selectors and predefined shortcuts like "Now" or "Now + SLA Response Time")
3. Perform status transitions based on predefined allowed actions.
4. If the incident is resolved, archive the entry.

#### 3.1.4 Status Transitions
A **State Machine** will govern the status transitions. Instead of hardcoding the workflow, SD3 will maintain a configurable set of status transitions:

| Current Status  | Next Status       | Description |
|----------------|------------------|-------------|
| Investigating  | Identified        | "Cause identified, working on a fix." |
| Identified     | Fix in Progress   | "Fix is being implemented, estimated completion: {end_time}." |
| Fix in Progress | Resolved         | "Incident resolved, verifying stability." |
| Resolved       | Closed           | "Issue closed and archived." |

Each allowed transition will be displayed as a button in the admin interface.

---

## 4. Additional Features

### 4.1 Scheduled Downtime Management
- Service Managers can schedule planned maintenance events.
- Planned maintenance follows a simpler workflow with predefined statuses.

### 4.2 Information Banners
- Ability to post informational banners (e.g., holiday greetings) across the dashboard.
- Banners are independent of the event processing system.

---

## 5. Technical Implementation Details

### 5.1 Backend Implementation (Golang)
- RESTful API to manage event data and state transitions.
- Configuration-driven state machine implementation.
- Database interaction using an ORM for SQL.
- Authentication and role-based access control for admin functions.

### 5.2 Database Schema (SQL)
- **events_table**:
  - `id` (Primary Key, Auto-increment/Hash)
  - `component` (String)
  - `region` (String)
  - `impact` (Enum)
  - `status` (Enum)
  - `start_time` (Timestamp)
  - `end_time` (Timestamp)
  - `title` (String)
  - `description` (Text)

- **status_transitions_table**:
  - `current_status` (Enum)
  - `next_status` (Enum)
  - `description` (String)

### 5.3 Frontend Implementation
- Web UI consuming RESTful API.
- Dynamic table display for events.
- Forms for incident management.
- Status transition buttons rendered dynamically.
- Realized as a single-page application based on the React framework from Telekom Scale.

---

## 6. Conclusion
This architecture ensures a scalable and maintainable SD3 implementation by leveraging Golang for the backend, SQL for persistent storage, and a configurable state machine for workflow flexibility. It provides clear paths for incident tracking, customer transparency, and service manager usability.

