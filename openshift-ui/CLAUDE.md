# ActiveMQ Artemis Operator Wireframes - AI Assistant Guide

This document provides instructions for AI assistants (Claude or others) working on this wireframes project.

## Project Context

This project contains UI wireframes for an OpenShift Console dynamic plugin for the ActiveMQ Artemis Operator. The operator uses a **split-CRD architecture**:
- **BrokerService**: Infrastructure managed by cluster operators (messaging cluster, storage, PKI)
- **BrokerApp**: Application intent managed by developers (bind to service, define messaging capabilities)

### Key Architectural Concepts
1. **Dual-PKI Architecture**: Separate Control Plane PKI (operator↔broker) and Data Plane PKI (app↔broker)
2. **cert-manager Integration**: All certificates are provisioned via cert-manager CertificateRequests
3. **Role-Based Design**: Cluster operators manage infrastructure, developers manage apps
4. **RBAC for Messaging**: Application roles define authorization for produce/consume/subscribe capabilities

### User Personas and Permissions
- **Cluster Operator** (kubeadmin): Full cluster access, manages BrokerServices
  - Can view and manage all namespaces
  - Uses "Administrator" perspective in OpenShift Console
  - Creates infrastructure resources (StatefulSets, Services, PKI)

- **Developer** (developer): Limited access, manages BrokerApps only
  - Can only access assigned namespaces/projects
  - Uses "Developer" perspective in OpenShift Console
  - Can only create BrokerApps in Workloads section
  - Cannot create or modify BrokerServices
  - Cannot access cluster-wide configuration

## Core Design Principles

### Always Maintain These Standards

1. **OpenShift Console Styling**
   - Use Tailwind CSS via CDN (`<script src="https://cdn.tailwindcss.com"></script>`)
   - Follow Red Hat PatternFly design patterns
   - Maintain consistent color palette (defined in tailwind.config)
   - All wireframes must be self-contained HTML files (no external dependencies except Tailwind)

2. **Standard Page Structure** (Every wireframe must include)
   ```
   - Warning banner (OpenShift Local dev cluster warning)
   - Top navigation bar (okd logo, notifications, settings, user dropdown)
   - Project selector bar (namespace selection)
   - Left sidebar navigation (perspective switcher, nav items)
   - Main content area
   ```

3. **Consistent Navigation Patterns**
   - Breadcrumbs on detail pages: `Parent > Resource Name`
   - Tab navigation for detail views: Overview, YAML, Resources (and Pods for BrokerService)
   - Active tab uses `border-b-2 border-pf-blue text-pf-blue`
   - Filter/search bar on all list views

4. **Status Indicators**
   - Use badges with appropriate colors:
     - Green: Running, Bound, Created, Approved
     - Yellow: Pending
     - Red: Error, Failed
   - Format: `<span class="bg-green-100 text-green-800 text-xs font-medium px-2 py-1 rounded border border-green-300">Status</span>`

## File Organization

### Current Wireframe Files
```
BrokerService (Cluster Operator):
├── wireframe-service-creation.html       # Day 1: Create form (Form View + YAML View toggle)
├── wireframe-service-list.html          # Day 2: List view
├── wireframe-broker-details-overview.html # Day 2: Details - Overview tab
└── wireframe-broker-details-resources.html # Day 2: Details - Resources tab

BrokerApp (Developer):
├── wirefram-app-creation.html           # Day 1: Create form (Form View + YAML View toggle, note: typo in name)
├── wireframe-app-list.html              # Day 2: List view
├── wireframe-app-details-overview.html  # Day 2: Details - Overview tab
└── wireframe-app-details-resources.html # Day 2: Details - Resources tab
```

## How to Update Wireframes

### When Making Changes

1. **Always Read First**: Use the Read tool to view the current wireframe before editing
2. **Use Edit Tool**: For targeted changes, use the Edit tool (not Write) to preserve file structure
3. **Maintain Consistency**: Check other wireframes to ensure visual and structural consistency
4. **Test Patterns**: Ensure new patterns work across all relevant wireframes

### Common Update Scenarios

#### Adding a New Field to a Form
```html
<!-- Pattern for form fields -->
<div class="mb-4">
    <label class="block text-sm font-medium text-gray-700 mb-1">
        Field Label
        <span class="text-gray-500 text-xs ml-1">(Optional help text)</span>
    </label>
    <input
        type="text"
        placeholder="Placeholder text"
        class="w-full px-3 py-2 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-pf-blue focus:border-pf-blue"
    />
</div>
```

#### Adding a Column to a Table
1. Add `<th>` in the `<thead>` section
2. Add corresponding `<td>` in each `<tr>` in `<tbody>`
3. Maintain text size classes: `text-xs font-medium text-gray-600` for headers, `text-sm text-gray-900` for data

#### Adding a Metric Chart
```html
<!-- Pattern for metric cards with charts -->
<div class="border border-gray-200 rounded p-4 bg-gray-50">
    <h3 class="text-sm font-medium text-gray-900 mb-4">Chart Title ⓘ</h3>
    <div class="relative h-48">
        <!-- Y-axis labels -->
        <div class="absolute left-0 top-0 h-full flex flex-col justify-between text-xs text-gray-500">
            <span>Max</span>
            <span>Mid</span>
            <span>0</span>
        </div>
        <!-- Chart area with axes -->
        <div class="ml-16 h-full border-l border-b border-gray-300 relative">
            <svg class="w-full h-full" viewBox="0 0 100 100" preserveAspectRatio="none">
                <polyline fill="none" stroke="#0066cc" stroke-width="1.5" points="0,60 20,55 40,50 60,45 80,40 100,35"/>
            </svg>
        </div>
        <!-- X-axis labels -->
        <div class="ml-16 mt-2 flex justify-between text-xs text-gray-500">
            <span>Start</span>
            <span>End</span>
        </div>
    </div>
</div>
```

#### Adding Resources to Resources Tab

**Table Container Pattern** (MUST use for all resources tables):
```html
<div class="border border-gray-200 rounded overflow-hidden">
    <table class="min-w-full">
        <thead class="bg-gray-50 border-b border-gray-200">
            <tr>
                <th class="px-6 py-3 text-left text-xs font-medium text-gray-600">Name</th>
                <th class="px-6 py-3 text-left text-xs font-medium text-gray-600">Kind</th>
                <th class="px-6 py-3 text-left text-xs font-medium text-gray-600">Status</th>
                <th class="px-6 py-3 text-left text-xs font-medium text-gray-600">Created or Description</th>
            </tr>
        </thead>
        <tbody class="bg-white">
            <!-- Resource rows here -->
        </tbody>
    </table>
</div>
```

**Resource Row Pattern**:
```html
<tr class="border-b border-gray-100 hover:bg-gray-50">
    <td class="px-6 py-3 whitespace-nowrap">
        <div class="flex items-center">
            <!-- Icon (color indicates resource type) -->
            <svg class="w-5 h-5 mr-2 text-orange-500" fill="currentColor" viewBox="0 0 20 20">
                <circle cx="10" cy="10" r="8"/>
            </svg>
            <a href="#" class="text-pf-blue hover:underline">resource-name</a>
        </div>
    </td>
    <td class="px-6 py-3 whitespace-nowrap text-sm text-gray-900">Kind</td>
    <td class="px-6 py-3 whitespace-nowrap">
        <span class="inline-flex items-center text-sm text-green-700">
            <svg class="w-4 h-4 mr-1" fill="currentColor" viewBox="0 0 20 20">
                <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/>
            </svg>
            Created
        </span>
    </td>
    <td class="px-6 py-3 text-sm text-gray-600">Description or timestamp with icon</td>
</tr>
```

**Resource Icon Colors**:
- Blue (`text-blue-500`): StatefulSet, Deployment
- Green (`text-green-500`): Service
- Orange (`text-orange-500`): Secret, ConfigMap
- Purple (`text-purple-500`): CertificateRequest
- Yellow (`text-yellow-500`): RoleBinding, Role

**IMPORTANT**: Resources tables MUST have:
- Bordered and rounded container: `border border-gray-200 rounded`
- Gray background on header: `bg-gray-50`
- Colored circle icons before resource names
- Status with checkmark icon (NOT badge pills)

## Connecting Wireframes to User Stories

### When Given a User Story

1. **Identify the Persona**: Cluster operator or application developer?
2. **Determine the Workflow**: Day 1 (creation) or Day 2 (operations)?
3. **Map to Wireframes**:
   ```
   Cluster Operator + Creation → wireframe-service-creation.html
   Cluster Operator + Listing → wireframe-service-list.html
   Cluster Operator + Details → wireframe-broker-details-*.html
   Developer + Creation → wirefram-app-creation.html
   Developer + Listing → wireframe-app-list.html
   Developer + Details → wireframe-app-details-*.html
   ```

4. **Update Relevant Wireframes**: Add/modify UI elements to support the user story
5. **Maintain Consistency**: If the change affects multiple wireframes, update all of them

### User Story Template Analysis

When receiving a user story like:
```
As a [cluster operator|developer]
I want to [action]
So that [benefit]
```

**Ask yourself**:
- What UI elements are needed? (form fields, buttons, tables, charts)
- Which wireframe(s) need updates?
- What data needs to be displayed?
- Are there status indicators or validation messages needed?
- Does this introduce new navigation or workflow?

### Example: Adding a New Feature

**User Story**: "As a cluster operator, I want to see the total number of messages processed per hour in the BrokerService overview, so that I can monitor throughput."

**Implementation**:
1. Open `wireframe-broker-details-overview.html`
2. Add a new metric chart in the Metrics section
3. Use the metric card pattern with an SVG line chart
4. Label it "Messages Processed per Hour ⓘ"
5. Add mock data showing an upward trend
6. Update README.md to document the new chart

## Creating New Wireframes

### When Adding a New View

1. **Copy an Existing Wireframe**: Start with the most similar existing file
2. **Update the `<title>` Tag**: Reflect the new view
3. **Update Breadcrumbs**: Show correct navigation hierarchy
4. **Update Active Navigation**: Highlight the correct sidebar item
5. **Update Active Tab**: If it's a detail view with tabs
6. **Replace Content**: Add the new content while maintaining the page structure
7. **Test Consistency**: Compare with other wireframes for visual consistency
8. **Update README.md**: Document the new wireframe

### Naming Conventions
- Use lowercase with hyphens: `wireframe-resource-view.html`
- Format: `wireframe-[resource]-[view].html`
- Examples: `wireframe-broker-details-yaml.html`, `wireframe-app-creation-wizard.html`

## Quality Checklist

Before completing any wireframe update, verify:

- [ ] File is self-contained HTML (only Tailwind CDN dependency)
- [ ] Warning banner is present
- [ ] Top navigation bar matches other wireframes
- [ ] **User persona is correct**: `kubeadmin` for BrokerService, `developer` for BrokerApp
- [ ] **Creation pages have Form View / YAML View toggle** (wireframe-service-creation.html and wirefram-app-creation.html)
- [ ] Project selector bar is present
- [ ] Left sidebar navigation is correct for persona (Administrator vs Developer)
- [ ] **Workloads menu includes both Brokers AND BrokerApps** (both personas can see both, but developers can only create BrokerApps)
- [ ] **Perspective switcher matches persona**: Administrator icon for BrokerService, Developer icon for BrokerApp
- [ ] Breadcrumbs are accurate
- [ ] Tab navigation (if applicable) has correct active state
- [ ] All links use `text-pf-blue hover:underline`
- [ ] Status badges use appropriate colors
- [ ] Tables have consistent column styling
- [ ] **Resources tables use consistent styling**: bordered container, gray header background, colored icons, status with checkmarks (not badge pills)
- [ ] Forms use consistent input styling with focus states
- [ ] Charts/metrics use the established SVG pattern
- [ ] Mock data is realistic and consistent across wireframes
- [ ] No broken HTML or missing closing tags
- [ ] Color palette uses custom Tailwind colors (pf-blue, os-dark, etc.)

## CRITICAL: Persona Distinction

**Never mix personas in wireframes!** This is one of the most important aspects of this project.

### BrokerService Wireframes (Cluster Operator)
- **Username**: `kubeadmin`
- **Perspective**: Administrator (shield icon)
- **Permissions**: Full cluster access
- **Sidebar Navigation**: Shows all cluster resources (StatefulSets, Storage, etc.)
  - **MUST include both Brokers AND BrokerApps** in the Workloads menu
- **Files**: All wireframes with `service` or `broker-details` in the name

### BrokerApp Wireframes (Developer)
- **Username**: `developer`
- **Perspective**: Developer (code brackets icon)
- **Permissions**: Limited to namespace/project, can only create BrokerApps in Workloads
- **Sidebar Navigation**: Shows only developer-accessible resources (limited view)
- **Files**: All wireframes with `app` in the name (wirefram-app-*, wireframe-app-*)

### When Creating/Updating Wireframes
1. **Always check which persona the wireframe is for**
2. **Set the correct username** in the top navigation dropdown
3. **Use the correct perspective icon** in the sidebar
4. **Show appropriate navigation items** based on permissions
5. **Never show cluster-admin features** in developer wireframes

## Important Conventions to Remember

### Color Palette (from tailwind.config)
```javascript
'pf-blue': '#0066cc',         // Primary actions, links
'pf-blue-hover': '#004080',   // Hover state for blue
'pf-gray': '#f0f0f0',         // Page background
'pf-border': '#d2d2d2',       // Border color
'os-dark': '#1f1f1f',         // Dark background
'os-header': '#292929',       // Header background
'os-sidebar': '#212427',      // Sidebar background
'os-nav-hover': '#3c3f42',    // Sidebar hover state
'os-warning': '#a30000',      // Warning banner
```

### Messaging Capability Colors
- **Produces To**: Blue border (`border-blue-200`, `bg-blue-50`)
- **Consumes From**: Green border (`border-green-200`, `bg-green-50`)
- **Subscribes To**: Purple border (`border-purple-200`, `bg-purple-50`)

### Mock Data Consistency
- BrokerService name: `ex-aao`
- BrokerApp examples: `order-processor-app`, `payment-gateway`, `my-payment-app`, `notification-service`
- Project/namespace: `my-application` (developer), `default` (admin)
- Users:
  - BrokerService wireframes: `kubeadmin` (cluster operator persona)
  - BrokerApp wireframes: `developer` (developer persona with limited rights)
- Timestamp format: `Mar 30, 2026, 4:24 PM`

### cert-manager Resource Naming
- Certificate Secret: `[name]-cert`
- CertificateRequest: `[name]-cert-[random-suffix]` (e.g., `broker-cert-xyz12`)
- CertificateRequest status: Always "Approved" (green badge)

### Artemis Metrics

The wireframes use three specific ActiveMQ Artemis queue metrics that are accurately tracked:

- **Message Count** (`getMessageCount`): Total number of messages in queue - displayed in tables and charts
- **Consumer Count** (`getConsumerCount`): Number of active consumers - displayed in tables and charts
- **Delivering Count** (`getDeliveringCount`): Number of messages being delivered - displayed in tables and charts

**IMPORTANT**: Only use these three metrics in all wireframes. Do not add metrics for producers or other values that are difficult to track accurately across different protocols.

Where metrics appear:
- **BrokerService List**: Table columns for Message Count, Consumer Count, Delivering Count
- **BrokerService Overview**: Charts for each metric per app (6 charts total)
- **BrokerService Loaded Apps Table**: Consumer Count column
- **BrokerApp List**: Table columns for Message Count, Consumer Count
- **BrokerApp Overview**: Charts for Message Count and Consumer Count (2 charts total)

## Common Patterns

### Form View / YAML View Toggle (Creation Pages)
Both creation wireframes support switching between Form View and YAML View:

```html
<!-- Configure Via Radio Buttons -->
<div class="mb-6">
    <label class="text-sm font-medium text-gray-900 mb-3 block">Configure Via</label>
    <div class="flex gap-6">
        <label class="flex items-center cursor-pointer">
            <input type="radio" name="configure-via" value="form" checked
                   class="w-4 h-4 text-pf-blue border-gray-300 focus:ring-pf-blue"
                   onclick="document.getElementById('form-view').style.display='block'; document.getElementById('yaml-view').style.display='none';">
            <span class="ml-2 text-sm text-gray-900">Form View</span>
        </label>
        <label class="flex items-center cursor-pointer">
            <input type="radio" name="configure-via" value="yaml"
                   class="w-4 h-4 text-pf-blue border-gray-300 focus:ring-pf-blue"
                   onclick="document.getElementById('form-view').style.display='none'; document.getElementById('yaml-view').style.display='block';">
            <span class="ml-2 text-sm text-gray-900">YAML View</span>
        </label>
    </div>
</div>

<!-- Form View -->
<div id="form-view">
    <!-- Form content here -->
</div>

<!-- YAML View -->
<div id="yaml-view" style="display: none;">
    <div class="bg-white border border-gray-200 rounded">
        <!-- Editor Toolbar -->
        <div class="border-b border-gray-200 px-4 py-2 flex items-center gap-2 bg-gray-50">
            <button class="p-1 hover:bg-gray-200 rounded" title="Download">
                <!-- Download icon SVG -->
            </button>
            <button class="p-1 hover:bg-gray-200 rounded" title="Copy">
                <!-- Copy icon SVG -->
            </button>
            <button class="p-1 hover:bg-gray-200 rounded" title="Expand">
                <!-- Expand icon SVG -->
            </button>
            <div class="ml-auto text-xs text-gray-500">
                <a href="#" class="text-pf-blue hover:underline">Shortcuts</a>
            </div>
        </div>
        <!-- YAML Editor -->
        <textarea class="w-full font-mono text-sm p-4 bg-white border-0 focus:outline-none focus:ring-0 resize-none"
                  rows="20" spellcheck="false">
apiVersion: broker.amq.io/v1beta1
kind: BrokerService
# ... YAML content
        </textarea>
    </div>
    <!-- Action buttons -->
</div>
```

### Link Pattern
```html
<a href="#" class="text-pf-blue hover:underline">Link Text</a>
```

### Button Pattern (Primary)
```html
<button class="px-4 py-1.5 bg-pf-blue text-white text-sm font-normal rounded hover:bg-pf-blue-hover focus:outline-none transition">
    Button Text
</button>
```

### Info/Alert Box Pattern
```html
<div class="p-4 bg-blue-50 border border-blue-200 rounded">
    <div class="flex items-start">
        <svg class="w-5 h-5 text-blue-600 mr-2 mt-0.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/>
        </svg>
        <div class="flex-1">
            <h4 class="text-sm font-medium text-blue-900 mb-1">Title</h4>
            <p class="text-sm text-blue-700">Message content</p>
        </div>
    </div>
</div>
```

### Multi-line Chart with Legend
```html
<svg class="w-full h-full" viewBox="0 0 100 100" preserveAspectRatio="none">
    <!-- First line (blue) -->
    <polyline fill="none" stroke="#0066cc" stroke-width="1.5" points="0,60 20,58 40,55 60,52 80,50 100,48"/>
    <!-- Second line (green) -->
    <polyline fill="none" stroke="#3e8635" stroke-width="1.5" points="0,75 20,73 40,70 60,68 80,67 100,65"/>
</svg>
<!-- Legend -->
<div class="ml-16 mt-2 flex gap-4 text-xs">
    <div class="flex items-center gap-1">
        <div class="w-3 h-3 bg-blue-600"></div>
        <span>Series 1</span>
    </div>
    <div class="flex items-center gap-1">
        <div class="w-3 h-3 bg-green-600"></div>
        <span>Series 2</span>
    </div>
</div>
```

## Workflow for Typical Tasks

### Task: Update a Form Field
1. `Read` the creation wireframe file
2. Locate the form section
3. Use `Edit` to add/modify the field using the form field pattern
4. Verify the change maintains visual consistency

### Task: Add a Resource to Resources Tab
1. `Read` the resources wireframe file
2. Locate the `<tbody>` section
3. Use `Edit` to add a new `<tr>` using the resource row pattern
4. Choose appropriate icon color based on resource type
5. If it's a certificate, add a corresponding CertificateRequest row

### Task: Add a Metric to Overview
1. `Read` the overview wireframe file
2. Locate the Metrics section
3. Determine grid layout (current is 3x2 for BrokerService, 1x3 for BrokerApp)
4. Use `Edit` to add new chart card using the metric pattern
5. Generate realistic mock SVG polyline data

### Task: Create a New Wireframe from User Story
1. Analyze the user story to determine persona and view type
2. Copy the most similar existing wireframe
3. Update all metadata (title, breadcrumbs, navigation)
4. Replace content with new UI elements
5. Update README.md with description
6. Run through quality checklist

## Troubleshooting

### Issue: Wireframe Doesn't Match OpenShift Style
- Check that Tailwind CDN is included
- Verify custom colors are defined in tailwind.config
- Compare with existing wireframes for structural patterns
- Ensure proper spacing classes (px-4, py-2, mb-4, etc.)

### Issue: Inconsistent Appearance Across Wireframes
- Read multiple wireframes to find the established pattern
- Copy exact class strings from working examples
- Check color values match the defined palette

### Issue: User Story Doesn't Map to Existing Wireframes
- Consider if a new wireframe is needed
- Check if the story requires updates to multiple wireframes
- Verify the persona and workflow stage (Day 1 vs Day 2)

## Future Enhancements

When adding new features, consider:
- **Accessibility**: Add ARIA labels for screen readers
- **Responsive Design**: Test at different viewport sizes
- **Interactivity**: Document any JavaScript needed for dynamic behavior
- **Validation**: Add error states and validation messaging patterns
- **Loading States**: Add skeleton screens or spinners for async operations
- **Empty States**: Add "no data" or "no results" messaging

## Additional Resources

- Red Hat PatternFly: https://www.patternfly.org/
- OpenShift Console: Study actual console screenshots provided by user
- Tailwind CSS: https://tailwindcss.com/docs
- cert-manager: https://cert-manager.io/ (for understanding certificate lifecycle)

## Getting Help

If you're unsure about:
- **Design decisions**: Ask the user for screenshots or examples from OpenShift Console
- **Data model**: Ask about the CRD structure and relationships
- **User workflows**: Request user stories or use cases
- **Terminology**: Ask for clarification on Artemis/messaging concepts

## Notes

- The typo in `wirefram-app-creation.html` (missing 'e') is intentional - keep it as is
- All mock dates use March 30, 2026
- cert-manager is assumed to be installed and managing all certificates
- The dual-PKI architecture is a critical concept - Control Plane vs Data Plane
- Always preserve the OpenShift Console look and feel
