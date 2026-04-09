# ActiveMQ Artemis Operator UI Implementation Roadmap

This roadmap outlines the tasks required to implement the OpenShift Console dynamic plugin for the ActiveMQ Artemis Operator.

## Executive Summary

**Goal**: Deliver a basic working UI for BrokerService and BrokerApp CRUD operations  
**User**: Cluster administrators only (kubeadmin)  
**Timeline**: ~18 weeks with 2-3 developers  
**Philosophy**: Ship the simplest thing that works, iterate later

## Scope

### ✅ In Scope (MVP)
- BrokerService and BrokerApp creation forms (Form View + YAML View)
- List views with basic filtering
- Detail views with Overview, YAML, and Resources tabs
- Basic metrics (Memory, CPU, Queue Depth, Consumer Count)
- Certificate generation script (temporary, until operator handles it)
- Prometheus ServiceMonitor setup

### ❌ Out of Scope (Future)
- Multi-tenancy (no developer persona)
- Namespace-based authorization or filtering
- Label suggestions (ConfigMaps, localStorage, drag-and-drop)
- RBAC and approval workflows
- Advanced metrics (per-app breakdowns, aggregations)
- Connectivity testing
- Inline editing of resources

## Implementation Phases

1. **Phase 1: Scripts & Setup** (2 weeks) - Get cluster ready
2. **Phase 2: Core CRUD** (8 weeks) - Creation and detail views
3. **Phase 3: Lists** (3 weeks) - List views and navigation
4. **Phase 4: Metrics** (3 weeks) - Charts and monitoring
5. **Phase 5: Polish** (2 weeks) - Testing and docs

**Total**: ~18 weeks

## Wireframe Reference

| View | Wireframe File | Purpose |
|------|----------------|---------|
| **BrokerService Creation** | `wireframe-service-creation.html` | Create form (kubeadmin) |
| **BrokerService List** | `wireframe-service-list.html` | List all services |
| **BrokerService Details** | `wireframe-broker-details-overview.html` | Overview with metrics |
| **BrokerService Resources** | `wireframe-broker-details-resources.html` | Related K8s resources |
| **BrokerApp Creation** | `wirefram-app-creation.html` | Create form (kubeadmin) |
| **BrokerApp List** | `wireframe-app-list.html` | List all apps |
| **BrokerApp Details** | `wireframe-app-details-overview.html` | Overview with connection info |
| **BrokerApp Resources** | `wireframe-app-details-resources.html` | Credentials and certs |

## Technology Stack

- **Framework**: React + TypeScript
- **UI Library**: PatternFly 6
- **State Management**: React Reducers
- **Platform**: OpenShift Console Dynamic Plugin SDK
- **Metrics**: Prometheus integration via OpenShift monitoring
- **Build**: Webpack

---

## Phase 1: Scripts & Setup (2 weeks)

**Goal**: Get a development cluster ready to test the UI

### 1.1 Certificate Generation Script
**Effort**: 3-4 days  
**Owner**: Backend/DevOps engineer

**What**: Script to generate all PKI certificates needed for broker communication

**Why**: Operator doesn't handle cert-manager yet, so we need a workaround

**Tasks**:
```bash
# Inspiration: activemq-artemis-self-provisioning-plugin/scripts/chain-of-trust.js
scripts/
  generate-certs.sh (or .js)
    └─ Generates:
       - Control Plane PKI: broker-cert, operator-manager-ca
       - Data Plane PKI: data-plane-broker-cert, data-plane-ca  
       - App certs: {app-name}-client-cert, {app-name}-prometheus-cert
    └─ Stores as Kubernetes Secrets
```

**Usage**: `./generate-certs.sh my-broker my-app`

**Deliverables**:
- [ ] Script file with inline comments
- [ ] README section explaining usage
- [ ] Example invocation in docs

---

### 1.2 Monitoring Setup
**Effort**: 1 day  
**Owner**: DevOps engineer

**What**: Enable Prometheus user workload monitoring in OpenShift

**Tasks**:
```bash
# 1. Apply ConfigMap
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-monitoring-config
  namespace: openshift-monitoring
data:
  config.yaml: |
    enableUserWorkload: true
EOF

# 2. Verify
kubectl -n openshift-user-workload-monitoring get pods
```

**Deliverables**:
- [ ] Script or doc with exact commands
- [ ] Verification steps documented

---

### 1.3 ServiceMonitor Configuration
**Effort**: 2-3 days  
**Owner**: DevOps engineer  
**Depends on**: 1.1, 1.2

**What**: Configure Prometheus to scrape broker metrics

**Tasks**:
- Create ServiceMonitor YAML for broker pods
- Configure TLS using Data Plane CA from 1.1
- Test with sample PromQL queries (see "Metrics Queries" section below)

**Deliverables**:
- [ ] `scripts/setup-servicemonitor.sh`
- [ ] ServiceMonitor template YAML
- [ ] Test query examples

---

### 1.4 Development Environment
**Effort**: 1-2 days  
**Owner**: All developers

**What**: Set up local dev environment for plugin development

**Tasks**:
- Install OpenShift Local (CRC) or connect to dev cluster
- Install `oc` CLI and `kubectl`
- Clone plugin repo and install dependencies
- Run plugin in dev mode (`npm run dev`)
- Verify plugin loads in OpenShift Console

**Deliverables**:
- [ ] Setup guide in README
- [ ] Troubleshooting section
- [ ] Screenshot of working local setup

---

## Phase 2: Core CRUD (8 weeks)

**Goal**: Users can create and view BrokerServices and BrokerApps

### 2.1 BrokerService Creation Form
**Effort**: 5-6 days  
**Owner**: Frontend developer  
**Wireframe**: `wireframe-service-creation.html`

**What**: Form to create a BrokerService

**Features**:
- [ ] Form View / YAML View toggle (radio buttons)
- [ ] Name input with DNS-1123 validation
- [ ] Namespace display (read-only from project selector)
- [ ] Labels input: Simple key-value text fields
  - Each label is 2 text inputs (key, value) + remove button
  - "Add Label" button to add another row
  - No ConfigMaps, no localStorage, no drag-and-drop
- [ ] Memory input with unit dropdown (Mi/Gi)
- [ ] YAML editor with syntax highlighting (CodeEditor component)
- [ ] Create button submits to K8s API
- [ ] Cancel button goes back

**Tech Stack**:
- React component: `src/components/BrokerServiceForm/BrokerServiceForm.tsx`
- PatternFly: `Form`, `FormGroup`, `TextInput`, `Select`, `CodeEditor`
- State: Simple `useState` (no reducer needed for MVP)
- Validation: Basic required field checks

**Acceptance Criteria**:
- [ ] Can create BrokerService via form view
- [ ] Can create BrokerService via YAML view
- [ ] Form validates required fields
- [ ] Labels add/remove works correctly
- [ ] Resource appears in `kubectl get brokerservices`
- [ ] Error messages display if creation fails

---

### 2.2 BrokerService Details View
**Effort**: 4-5 days  
**Owner**: Frontend developer  
**Wireframes**: `wireframe-broker-details-overview.html`, `wireframe-broker-details-resources.html`

**What**: Multi-tab detail view for a BrokerService

**Features**:
- [ ] **Overview tab**:
  - Status badge (Running/Pending/Error)
  - Labels display (read-only)
  - Conditions table
  - Placeholder for metrics (filled in Phase 4)
- [ ] **YAML tab**: CodeEditor showing resource YAML
- [ ] **Resources tab**: Table of related resources
  - StatefulSet, Services, Secrets, ConfigMaps
  - Resource name (clickable link), Kind, Status, Created timestamp
  - Colored icons per resource type
- [ ] **Pods tab**: Table of broker pods
  - Pod name, Status, Restarts, Age
- [ ] Breadcrumb: `Brokers > {name}`
- [ ] Action menu: Edit, Delete (future)

**Tech Stack**:
- Component: `src/components/BrokerServiceDetails/`
- PatternFly: `Tabs`, `DescriptionList`, `Table`, `Label`
- K8s API: `useK8sWatchResource` hook

**Acceptance Criteria**:
- [ ] All tabs render correctly
- [ ] Clicking resource name navigates to resource detail
- [ ] Status badge reflects actual resource status
- [ ] Conditions table shows operator status

---

### 2.3 BrokerApp Creation Form
**Effort**: 6-7 days  
**Owner**: Frontend developer  
**Wireframe**: `wirefram-app-creation.html`

**What**: Form to create a BrokerApp (binds to BrokerService)

**Features**:
- [ ] Form View / YAML View toggle
- [ ] Name input
- [ ] Namespace display (read-only)
- [ ] Application Role input (text field)
- [ ] **Service Selector matchLabels**: Simple key-value pairs
  - Same pattern as BrokerService labels
  - User types key, value, clicks "Add Match Label"
- [ ] **Messaging Capabilities**:
  - Produces To: array of address names
  - Consumes From: array of address names
  - Subscribes To: array of address names
  - Each has: text input + "Add" button + list with remove buttons
- [ ] YAML editor
- [ ] Create/Cancel buttons

**Tech Stack**:
- Component: `src/components/BrokerAppForm/BrokerAppForm.tsx`
- PatternFly: `Form`, `FormGroup`, `TextInput`, `Button`
- State: Array management for addresses and labels

**Acceptance Criteria**:
- [ ] Can create BrokerApp via form
- [ ] Can add/remove addresses for each capability
- [ ] Can add/remove selector labels
- [ ] Operator provisions app to matching BrokerService
- [ ] Credentials are created

---

### 2.4 BrokerApp Details View
**Effort**: 5-6 days  
**Owner**: Frontend developer  
**Wireframes**: `wireframe-app-details-overview.html`, `wireframe-app-details-resources.html`

**What**: Multi-tab detail view for a BrokerApp

**Features**:
- [ ] **Overview tab**:
  - Status badge (Provisioned/Pending/Error)
  - Provisioned Service (clickable link to BrokerService)
  - Application Role
  - **Connection Information** section:
    - Broker Host (with copy button)
    - Acceptor Port (with copy button)
    - Connection Secret (link to secret)
  - **Messaging Capabilities** section:
    - Lists for Produces To, Consumes From, Subscribes To
  - Conditions table
  - Placeholder for metrics (Phase 4)
- [ ] **YAML tab**: CodeEditor
- [ ] **Resources tab**: Table of resources
  - Secrets, ConfigMaps, CertificateRequests, RoleBindings
- [ ] Breadcrumb: `BrokerApps > {name}`

**Tech Stack**:
- Component: `src/components/BrokerAppDetails/`
- PatternFly: `Tabs`, `DescriptionList`, `ClipboardCopy`
- Copy-to-clipboard functionality

**Acceptance Criteria**:
- [ ] Connection info displays correctly
- [ ] Copy buttons work
- [ ] Link to provisioned service navigates correctly
- [ ] Messaging capabilities display all addresses

---

## Phase 3: List Views (3 weeks)

**Goal**: Users can browse and navigate BrokerServices and BrokerApps

### 3.1 BrokerService List
**Effort**: 4-5 days  
**Owner**: Frontend developer  
**Wireframe**: `wireframe-service-list.html`

**What**: List view for all BrokerServices in the cluster

**Features**:
- [ ] Table with columns: Name, Namespace, Status, Labels, Memory, Created
- [ ] Click on name to navigate to details
- [ ] Filter/search box for name search
- [ ] "Create BrokerService" button
- [ ] Pagination controls

**Tech Stack**:
- Component: `src/components/BrokerServiceList/BrokerServiceList.tsx`
- PatternFly: `Table`, `Pagination`, `SearchInput`
- K8s API: `useK8sWatchResources` hook for list with watch

**Acceptance Criteria**:
- [ ] Table displays all BrokerServices from all namespaces
- [ ] Clicking service name navigates to detail view
- [ ] Search box filters by name
- [ ] Create button navigates to creation form
- [ ] List updates automatically when resources change

---

### 3.2 BrokerApp List
**Effort**: 4-5 days  
**Owner**: Frontend developer  
**Wireframe**: `wireframe-app-list.html`

**What**: List view for all BrokerApps in the cluster

**Features**:
- [ ] Table with columns: Name, Namespace, Status, Provisioned To, Created
- [ ] Click on name to navigate to details
- [ ] Click on "Provisioned To" service name to navigate to service details
- [ ] Filter/search box for name search
- [ ] "Create BrokerApp" button
- [ ] Pagination controls

**Tech Stack**:
- Component: `src/components/BrokerAppList/BrokerAppList.tsx`
- PatternFly: `Table`, `Pagination`, `SearchInput`
- K8s API: `useK8sWatchResources` hook for list with watch

**Acceptance Criteria**:
- [ ] Table displays all BrokerApps from all namespaces
- [ ] Clicking app name navigates to detail view
- [ ] Clicking provisioned service navigates to service detail
- [ ] Search box filters by name
- [ ] Create button navigates to creation form
- [ ] List updates automatically when resources change

---

### 3.3 Navigation Integration
**Effort**: 2-3 days  
**Owner**: Frontend developer  
**Dependencies**: 3.1, 3.2

**What**: Integrate list and detail views into OpenShift Console navigation

**Features**:
- [ ] Add "Brokers" menu item under Workloads (points to list)
- [ ] Add "BrokerApps" menu item under Workloads (points to list)
- [ ] Configure routes for all views:
  - `/k8s/cluster/broker.amq.io~v1beta1~BrokerService` → List
  - `/k8s/cluster/broker.amq.io~v1beta1~BrokerService/~new` → Create
  - `/k8s/ns/:ns/broker.amq.io~v1beta1~BrokerService/:name` → Details
  - Same pattern for BrokerApp
- [ ] Breadcrumb navigation on all pages

**Tech Stack**:
- File: `console-extensions.json`
- OpenShift Console SDK: `NavItem`, `RoutePage` extensions

**Acceptance Criteria**:
- [ ] Brokers menu item appears in sidebar under Workloads
- [ ] BrokerApps menu item appears in sidebar under Workloads
- [ ] All routes navigate correctly
- [ ] Breadcrumbs show correct hierarchy

---

## Phase 4: Metrics Integration (3 weeks)

**Goal**: Display performance metrics for BrokerServices and BrokerApps

### 4.1 Metrics Foundation
**Effort**: 4-5 days  
**Owner**: Frontend developer  
**Dependencies**: 1.3 (ServiceMonitor setup)

**What**: Build reusable infrastructure for displaying Prometheus metrics

**Features**:
- [ ] Create reusable `MetricsChart` component
- [ ] Integrate with OpenShift monitoring API
- [ ] Time range selector (1h, 6h, 24h)
- [ ] SVG-based chart rendering
- [ ] Loading and error states
- [ ] No data state

**Tech Stack**:
- Component: `src/components/MetricsChart/MetricsChart.tsx`
- OpenShift Console SDK: `usePrometheusPoll` hook
- PatternFly: `Chart`, `ChartAxis`, `ChartLine`
- PromQL query builder utilities

**Acceptance Criteria**:
- [ ] Chart component accepts PromQL query and renders line chart
- [ ] Time range selector updates query and refreshes data
- [ ] Loading spinner shows while fetching
- [ ] Error message displays if query fails
- [ ] "No data" message shows if query returns empty

---

### 4.2 BrokerService Metrics
**Effort**: 3-4 days  
**Owner**: Frontend developer  
**Dependencies**: 4.1, 2.2 (Details view)

**What**: Add metrics charts to BrokerService overview tab

**Features**:
- [ ] Memory Usage chart
- [ ] CPU Usage chart
- [ ] Shared time range selector (applies to both charts)
- [ ] Charts display in grid layout

**Tech Stack**:
- Update: `src/components/BrokerServiceDetails/OverviewTab.tsx`
- Use `MetricsChart` component from 4.1
- PromQL queries with broker name substitution

**PromQL Queries**:
```promql
# Memory Usage
container_memory_usage_bytes{pod=~"<broker-name>-ss-.*"}

# CPU Usage (rate over 5 minutes)
rate(container_cpu_usage_seconds_total{pod=~"<broker-name>-ss-.*"}[5m])
```

**Acceptance Criteria**:
- [ ] Memory chart shows container memory usage over time
- [ ] CPU chart shows CPU rate over time
- [ ] Time range selector updates both charts
- [ ] Charts show "No data" if broker has no pods
- [ ] Broker name is correctly substituted in queries

---

### 4.3 BrokerApp Metrics
**Effort**: 3-4 days  
**Owner**: Frontend developer  
**Dependencies**: 4.1, 2.4 (Details view)

**What**: Add metrics charts to BrokerApp overview tab

**Features**:
- [ ] Queue Depth chart (derived from Message Count - Delivering Count)
- [ ] Consumer Count chart
- [ ] Shared time range selector (applies to both charts)
- [ ] Charts display in grid layout

**Tech Stack**:
- Update: `src/components/BrokerAppDetails/OverviewTab.tsx`
- Use `MetricsChart` component from 4.1
- PromQL queries with app role substitution

**PromQL Queries**:
```promql
# Queue Depth (messages waiting to be processed)
artemis_message_count{role="<app-role>"} - artemis_delivering_count{role="<app-role>"}

# Consumer Count (active consumers for this app)
artemis_consumer_count{role="<app-role>"}
```

**Acceptance Criteria**:
- [ ] Queue Depth chart shows waiting messages over time
- [ ] Consumer Count chart shows active consumers over time
- [ ] Time range selector updates both charts
- [ ] Charts show "No data" if app has no activity
- [ ] App role is correctly substituted in queries

---

## Phase 5: Polish & Testing (2 weeks)

**Goal**: Validate all features and document the system

### 5.1 Integration Testing
**Effort**: 5-7 days  
**Owner**: All developers  
**Dependencies**: All previous phases

**What**: End-to-end testing of all user workflows

**Test Scenarios**:
- [ ] **BrokerService Workflow**:
  - Create BrokerService via form view
  - Create BrokerService via YAML view
  - Verify StatefulSet, Services, Secrets created
  - View service in list
  - View service details (Overview, YAML, Resources, Pods tabs)
  - Verify metrics charts display correctly
- [ ] **BrokerApp Workflow**:
  - Create BrokerApp via form view
  - Create BrokerApp via YAML view
  - Verify app provisions to matching service
  - Verify credentials are created
  - View app in list
  - View app details (Overview, YAML, Resources tabs)
  - Verify metrics charts display correctly
  - Verify connection information is accessible
- [ ] **Navigation**:
  - All breadcrumbs navigate correctly
  - All links between resources work
  - Sidebar navigation highlights active item
- [ ] **Error Handling**:
  - Invalid form input shows validation errors
  - Failed API calls show error messages
  - Metrics with no data show "No data" state

**Deliverables**:
- [ ] Test checklist (completed)
- [ ] Bug list (if any issues found)
- [ ] Video recording of happy path workflow

---

### 5.2 Documentation
**Effort**: 3-4 days  
**Owner**: Technical writer + Frontend developer  
**Dependencies**: 5.1

**What**: User-facing and developer documentation

**Documentation Sections**:
- [ ] **Installation Guide**:
  - Prerequisites (OpenShift version, Operator installation)
  - Plugin deployment steps
  - Certificate generation script usage
  - Prometheus setup verification
- [ ] **User Guide**:
  - Creating a BrokerService (with screenshots)
  - Creating a BrokerApp (with screenshots)
  - Understanding connection information
  - Reading metrics charts
- [ ] **Troubleshooting Guide**:
  - Common errors and solutions
  - Certificate issues
  - Metrics not appearing
  - App not provisioning to service
- [ ] **Developer Guide**:
  - Local development setup
  - Running plugin in dev mode
  - Building and deploying
  - Contributing guidelines

**Deliverables**:
- [ ] README.md with installation steps
- [ ] docs/USER_GUIDE.md with screenshots
- [ ] docs/TROUBLESHOOTING.md
- [ ] docs/DEVELOPMENT.md

---

## Timeline Summary

```mermaid
gantt
    title Simplified Implementation Timeline (18 Weeks)
    dateFormat YYYY-MM-DD

    section Phase 1 Scripts
    Certificate Script       :p1_1, 2026-04-07, 4d
    Monitoring Config        :p1_2, 2026-04-07, 1d
    ServiceMonitor Setup     :p1_3, 2026-04-09, 3d
    Dev Environment          :p1_4, 2026-04-14, 2d

    section Phase 2 CRUD
    BS Creation Form         :p2_1, 2026-04-21, 6d
    BS Details               :p2_2, 2026-04-29, 5d
    BA Creation Form         :p2_3, 2026-05-06, 7d
    BA Details               :p2_4, 2026-05-15, 6d

    section Phase 3 Lists
    BS List                  :p3_1, 2026-05-27, 5d
    BA List                  :p3_2, 2026-06-03, 5d
    Navigation               :p3_3, 2026-06-10, 3d

    section Phase 4 Metrics
    Metrics Foundation       :p4_1, 2026-06-17, 5d
    BS Metrics               :p4_2, 2026-06-24, 4d
    BA Metrics               :p4_3, 2026-07-01, 4d

    section Phase 5 Polish
    Integration Testing      :p5_1, 2026-07-08, 7d
    Documentation            :p5_2, 2026-07-17, 4d
```

**Total Duration**: ~18 weeks (vs. 30 weeks for full feature set)

---

## Key Simplifications

Compared to the full roadmap, this simplified version removes:

1. **No Multi-Tenancy**:
   - No developer persona
   - No namespace isolation
   - Everything as kubeadmin

2. **No Authorization**:
   - No namespace filter labels
   - No ConfigMap management
   - No RBAC setup

3. **No Label Suggestions**:
   - No ConfigMap loading
   - No previously used labels (localStorage)
   - Simple manual text input only

4. **No Drag-and-Drop**:
   - Simple form inputs instead
   - Reduces complexity significantly

5. **Minimal Metrics**:
   - Only basic charts
   - No per-app breakdowns
   - No advanced aggregations

---

## Future Enhancements (Post-MVP)

After the basic implementation is complete and stable, consider adding:

- Multi-tenancy with developer persona
- Namespace-based authorization
- Label suggestion system with ConfigMaps
- Drag-and-drop label selectors
- Advanced metrics and monitoring
- Approval workflows
- Connectivity testing
- Selector change warnings
