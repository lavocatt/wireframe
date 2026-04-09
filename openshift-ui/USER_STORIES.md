# User Stories - ActiveMQ Artemis Operator UI

This document contains user stories for the OpenShift Console dynamic plugin for the ActiveMQ Artemis Operator.

## Persona

- **Cluster Administrator (kubeadmin)**: Manages all infrastructure and applications, creates BrokerServices and BrokerApps

**Note**: Multi-tenancy and authorization features are deferred to future phases. Initial implementation focuses on basic CRUD operations.

---

## Epic 1: BrokerService Management

### Story 1.1: Create BrokerService

**As a** cluster administrator  
**I want to** create a BrokerService with basic configuration  
**So that** messaging infrastructure is available for applications

**Acceptance Criteria:**
- Navigate to Workloads > Brokers in Administrator perspective
- Click "Create BrokerService"
- Fill in form:
  - Name: `my-broker` (text input)
  - Namespace: `default` (read-only, from project selector)
  - Labels: Type key-value pairs manually (e.g., `tier: "production"`, `forWorkQueue: "true"`)
  - Memory: 2Gi (text input with unit dropdown)
- Submit form
- Verify BrokerService is created
- Verify StatefulSet, Services, and Secrets are created by operator

**Wireframes**: `wireframe-service-creation.html`

---

### Story 1.2: View BrokerService List

**As a** cluster administrator  
**I want to** see a list of all BrokerServices  
**So that** I can monitor messaging infrastructure

**Acceptance Criteria:**
- Navigate to Workloads > Brokers
- View table with columns:
  - Name (clickable link)
  - Namespace
  - Status (Running, Pending, Error)
  - Labels (first label + "+N more")
  - Memory Usage
  - Created (timestamp)
- Click on service name to view details

**Wireframes**: `wireframe-service-list.html`

---

### Story 1.3: View BrokerService Details

**As a** cluster administrator  
**I want to** view detailed information about a BrokerService  
**So that** I can monitor its health and configuration

**Acceptance Criteria:**
- Click on a BrokerService from list view
- View Overview tab showing:
  - Status badge
  - Labels and annotations
  - Conditions table (Ready, Available, etc.)
  - Basic metrics (Memory, CPU)
- Navigate to YAML tab to view/edit YAML
- Navigate to Resources tab showing:
  - StatefulSet
  - Services
  - Secrets
  - ConfigMaps
- Navigate to Pods tab showing broker pods

**Wireframes**: `wireframe-broker-details-overview.html`, `wireframe-broker-details-resources.html`

---

## Epic 2: BrokerApp Management

### Story 2.1: Create BrokerApp

**As a** cluster administrator  
**I want to** create a BrokerApp that binds to a BrokerService  
**So that** applications can send and receive messages

**Acceptance Criteria:**
- Navigate to Workloads > BrokerApps
- Click "Create BrokerApp"
- Fill in form:
  - Name: `my-app` (text input)
  - Namespace: `default` (read-only, from project selector)
  - Application Role: `my-app-sa` (text input)
  - Service Selector matchLabels: Type key-value pairs manually (e.g., `tier: "production"`)
  - Messaging Capabilities:
    - Produces To: `orders.created` (add multiple via input + button)
    - Consumes From: `orders.pending` (add multiple)
    - Subscribes To: `notifications.global` (add multiple)
- Submit form
- Verify BrokerApp is created
- Verify operator provisions app to matching BrokerService
- Verify credentials are created

**Wireframes**: `wirefram-app-creation.html`

---

### Story 2.2: View BrokerApp List

**As a** cluster administrator  
**I want to** see a list of all BrokerApps  
**So that** I can monitor application bindings

**Acceptance Criteria:**
- Navigate to Workloads > BrokerApps
- View table with columns:
  - Name (clickable link)
  - Namespace
  - Status (Provisioned, Pending, Error)
  - Provisioned To (BrokerService name, clickable)
  - Created (timestamp)
- Click on app name to view details

**Wireframes**: `wireframe-app-list.html`

---

### Story 2.3: View BrokerApp Details

**As a** cluster administrator  
**I want to** view detailed information about a BrokerApp  
**So that** I can get connection information and monitor status

**Acceptance Criteria:**
- Click on a BrokerApp from list view
- View Overview tab showing:
  - Status badge
  - Provisioned Service (link to BrokerService)
  - Application Role
  - Connection Information:
    - Broker Host (copyable)
    - Acceptor Port (copyable)
    - Connection Secret (link)
  - Messaging Capabilities:
    - Produces To (list)
    - Consumes From (list)
    - Subscribes To (list)
  - Conditions table
  - Labels and annotations
- Navigate to YAML tab
- Navigate to Resources tab showing:
  - Secrets (credentials)
  - ConfigMaps
  - CertificateRequests
  - RoleBindings

**Wireframes**: `wireframe-app-details-overview.html`, `wireframe-app-details-resources.html`

---

## Epic 3: Metrics & Monitoring

### Story 3.1: View BrokerService Metrics

**As a** cluster administrator  
**I want to** see metrics for a BrokerService  
**So that** I can monitor performance and capacity

**Acceptance Criteria:**
- Open BrokerService details page
- View metrics section with charts:
  - Memory Usage
  - CPU Usage
- Select time range (1h, 6h, 24h)
- Charts update based on Prometheus queries

**Wireframes**: `wireframe-broker-details-overview.html`

---

### Story 3.2: View BrokerApp Metrics

**As a** cluster administrator  
**I want to** see metrics for a BrokerApp  
**So that** I can monitor message throughput

**Acceptance Criteria:**
- Open BrokerApp details page
- View metrics section with charts:
  - Queue Depth (Message Count - Delivering Count)
  - Consumer Count
- Select time range (1h, 6h, 24h)
- Charts update based on Prometheus queries

**Wireframes**: `wireframe-app-details-overview.html`

---

## Future Enhancements (Deferred)

These features are out of scope for the initial implementation:

- **Multi-tenancy**: Developer persona, namespace isolation
- **Authorization**: Namespace-based filtering, RBAC
- **Label suggestions**: ConfigMaps, previously used labels
- **Approval workflows**: Manual approval for sensitive services
- **Advanced metrics**: Per-app breakdowns, aggregations
- **Connectivity testing**: Built-in connection validator
- **Selector changes**: Reprovisioning, data migration warnings
