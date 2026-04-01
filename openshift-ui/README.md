# ActiveMQ Artemis Operator UI Wireframes

This directory contains UI wireframes for the ActiveMQ Artemis Operator's OpenShift Console dynamic plugin. These wireframes demonstrate the split-CRD architecture with `BrokerService` (infrastructure) and `BrokerApp` (application intent) resources.

## Architecture Overview

The ActiveMQ Artemis Operator uses a split-CRD design:
- **BrokerService**: Managed by cluster operators (kubeadmin), defines the messaging infrastructure (broker cluster, storage, HA, PKI)
- **BrokerApp**: Managed by application developers (developer), defines application messaging intent and binds to a BrokerService

### User Personas
- **Cluster Operator** (kubeadmin): Full cluster access via Administrator perspective, manages BrokerServices
- **Developer** (developer): Limited namespace access via Developer perspective, manages BrokerApps only in Workloads section

## Wireframe Files

### BrokerService Wireframes (Cluster Operator Persona - kubeadmin)

These wireframes show the Administrator perspective with full cluster access.

#### 1. wireframe-service-creation.html
**Purpose**: Day 1 creation form for BrokerService resources
**Target User**: Cluster operators (kubeadmin)
**Key Features**:
- **Configure Via** toggle: Switch between Form View and YAML View
- **Form View**: General Details section (Name, Namespace), Infrastructure & Capacity section (Memory/RAM allocation)
- **YAML View**: Direct YAML editor with toolbar (download, copy, expand)
- OpenShift Console styling with project selector bar

#### 2. wireframe-service-list.html
**Purpose**: List view of all BrokerService instances
**Target User**: Cluster operators
**Key Features**:
- Data table with 8 columns: Name, Status, Apps Loaded, Message Count, Consumer Count, Delivering Count, Memory Usage, CPU Usage
- Progress bars for resource utilization (Memory, CPU)
- Filter/search functionality
- "Create BrokerService" action button

#### 3. wireframe-broker-details-overview.html
**Purpose**: Day 2 details page for a BrokerService (Overview tab)
**Target User**: Cluster operators
**Key Features**:
- Breadcrumb navigation and status badge
- Details section (Labels, Annotations)
- Metrics section with 6 charts:
  - Total Memory Usage
  - Total CPU Usage
  - Memory Usage per App (multi-line chart)
  - Message Count per App (multi-line chart)
  - Consumer Count per App (multi-line chart)
  - Delivering Count per App (multi-line chart)
- Loaded Apps table showing provisioned BrokerApps (App Name, Status, Consumer Count)
- Conditions table showing resource status

#### 4. wireframe-broker-details-resources.html
**Purpose**: Day 2 details page for a BrokerService (Resources tab)
**Target User**: Cluster operators
**Key Features**:
- Resources table listing Kubernetes objects managed by this BrokerService
- StatefulSet and Service resources
- Control Plane PKI certificates (broker-cert, operator-manager-ca) with cert-manager CertificateRequests
- Data Plane PKI certificates (data-plane-broker-cert, data-plane-ca) with cert-manager CertificateRequests
- App credentials (app1-credentials, app2-credentials)
- Broker configuration (ex-aao-props)
- Filter/search functionality

### BrokerApp Wireframes (Application Developer Persona - developer)

These wireframes show the Developer perspective with limited namespace access. Developers can only create BrokerApps in the Workloads section.

#### 5. wirefram-app-creation.html
**Purpose**: Day 1 creation form for BrokerApp resources
**Target User**: Application developers (developer)
**Key Features**:
- **Configure Via** toggle: Switch between Form View and YAML View
- **Form View**:
  - Application Details section (Name, Namespace, Application Role for RBAC)
  - Messaging Capabilities section with three subsections:
    - **Produces To** (blue border): Queues/topics the app will send messages to
    - **Consumes From** (green border): Queues the app will read from
    - **Subscribes To** (purple border): Topics the app will subscribe to
  - Dynamic add/remove functionality for messaging addresses
- **YAML View**: Direct YAML editor with toolbar (download, copy, expand)
- OpenShift Console styling

#### 6. wireframe-app-list.html
**Purpose**: List view of all BrokerApp instances
**Target User**: Application developers
**Key Features**:
- Data table with 5 columns: Name, Status, Provisioned Service, Message Count, Consumer Count
- Shows provisioning status (Provisioned, Pending)
- Links to provisioned BrokerService
- Filter/search functionality
- "Create BrokerApp" action button

#### 7. wireframe-app-details-overview.html
**Purpose**: Day 2 details page for a BrokerApp (Overview tab)
**Target User**: Application developers
**Key Features**:
- Breadcrumb navigation, status badge, and provisioned service indicator
- Details section (Application Role, Labels, Annotations)
- App-specific metrics (2 charts):
  - Message Count
  - Consumer Count
- Connectivity Tester card with "Run Connectivity Test" button and mock success result showing TLS validation steps
- Messaging Capabilities section displaying configured addresses (Produces To, Consumes From, Subscribes To)

#### 8. wireframe-app-details-resources.html
**Purpose**: Day 2 details page for a BrokerApp (Resources tab)
**Target User**: Application developers
**Key Features**:
- Resources table listing credentials and configuration for this app
- **Critical credential resources**:
  - `my-payment-app-client-cert` (Secret): TLS client certificate for broker authentication
  - `my-payment-app-prometheus-client-cert` (Secret): TLS certificate for Prometheus metrics
  - `data-plane-ca` (ConfigMap): CA certificate for TLS trust
- cert-manager CertificateRequests showing automated certificate lifecycle management
- Additional resources: PEMCFG ConfigMap, service binding secret, role binding
- Info box explaining how to use credentials
- Filter/search functionality

## Design Principles

### Visual Styling
- All wireframes use **Tailwind CSS via CDN** for styling
- Mimic **Red Hat PatternFly** design system
- Match **OpenShift Console** visual language
- Consistent color scheme:
  - Primary blue: `#0066cc`
  - Success green: `#3e8635`
  - Warning/error red: `#a30000`
  - Dark backgrounds: `#212427` (sidebar), `#292929` (header)

### User Experience
- **Project selector bar**: Allows namespace/project selection
- **Breadcrumb navigation**: Shows resource hierarchy
- **Status badges**: Clear visual indicators (Running, Provisioned, Pending, etc.)
- **Tab navigation**: Separates Overview, YAML, and Resources views
- **Filter/search**: Every list view includes search functionality
- **Responsive metrics**: Charts use SVG for mock visualizations

### Role-Based Design
- **BrokerService views**: Focused on infrastructure, capacity planning, and multi-app monitoring
- **BrokerApp views**: Focused on application connectivity, credentials, and messaging capabilities

## PKI Architecture

The wireframes demonstrate a dual-PKI architecture:
- **Control Plane PKI**: For operator-to-broker communication (broker-cert, operator-manager-ca)
- **Data Plane PKI**: For app-to-broker communication (data-plane-broker-cert, data-plane-ca, client certs)
- **cert-manager integration**: All certificates show corresponding CertificateRequest resources

## Artemis Metrics

The wireframes use three specific ActiveMQ Artemis queue metrics that are accurately tracked by the broker:

- **Message Count** (`getMessageCount`): Total number of messages currently in the queue
- **Consumer Count** (`getConsumerCount`): Number of active consumers attached to the queue
- **Delivering Count** (`getDeliveringCount`): Number of messages currently being delivered to consumers

These metrics are displayed throughout the wireframes:
- **BrokerService List**: Shows aggregated metrics across all queues
- **BrokerService Overview**: Shows per-app metrics in individual charts
- **BrokerApp List**: Shows metrics for queues used by each app
- **BrokerApp Overview**: Shows metrics specific to the app's queues

## Technology Stack
- HTML5
- Tailwind CSS (via CDN)
- Inline SVG for icons and mock charts
- Self-contained files (no external dependencies except Tailwind CDN)

## Usage

Each HTML file can be opened directly in a web browser. No build process or server required.

```bash
# Open any wireframe in your browser
firefox wireframe-service-list.html
# or
chromium wireframe-app-details-overview.html
```

## Notes

- File naming: `wirefram-app-creation.html` contains a typo (missing 'e') but was kept as requested
- All timestamps use mock date: March 30, 2026
- All data values are placeholder/mock data for demonstration purposes
- Charts are static SVG mockups, not interactive
