# RHACM & OPA Gatekeeper Integration Overview

Red Hat Advanced Cluster Management (RHACM) integrates with **OPA Gatekeeper** to provide robust admission control, audit capabilities, and policy enforcement across a multi-cluster fleet. It functions as a core component of RHACM’s "defense-in-depth" governance framework.


## 1. Automated Installation and Lifecycle Management
RHACM simplifies the deployment and maintenance of Gatekeeper infrastructure across all managed clusters.

* **Operator Policy:** Deploy the Gatekeeper Operator and CR instances using `OperatorPolicy` or `ConfigurationPolicy`.
* **Version Control:** Manage subscription channels and choose between **Automatic** or **Manual** approval strategies for updates.
* **Granular Configuration:** Control low-level settings like audit intervals, resource limits, and mutation webhook toggles via the ACM Hub.

## 2. Policy Distribution (Constraints & Templates)
RHACM serves as the delivery engine for OPA-based rules, moving logic from the Hub to the Spokes.

* **Hub-to-Spoke Propagation:** Define `ConstraintTemplates` (logic) and `Constraints` (parameters) within an ACM Policy template.
* **Policy Generator Integration:** Use the Policy Generator to pull raw Gatekeeper manifests from Git.
    * **Key Flag:** `informGatekeeperPolicies` determines if violations report back to the Hub.

## 3. Dynamic Enforcement Mapping
RHACM intelligently synchronizes its remediation intent with Gatekeeper’s internal enforcement actions.

| ACM Remediation Action | Gatekeeper Enforcement Action | Resulting Behavior |
| :--- | :--- | :--- |
| **Inform** | `warn` | Violations are logged/reported but not blocked. |
| **Enforce** | `deny` | Non-compliant API requests are actively blocked. |


## 4. Centralized Observability and Reporting
RHACM aggregates distributed Gatekeeper data into a "single pane of glass" for compliance monitoring.

* **Audit Result Aggregation:** The Gatekeeper sync controller watches for local audit results and syncs them as compliance events to the **Governance Dashboard**.
* **Admission Event Monitoring:** Deploy `policy-gatekeeper-admission` to monitor real-time Events created when Gatekeeper denies a request.
* **Drill-Down Details:** Directly view which specific namespace or deployment caused a violation from the central console.

## 5. Advanced Rule Engines (Rego & CEL)
The integration supports both traditional and modern Kubernetes policy standards.

* **Rego Support:** Full compatibility with standard OPA Rego for complex logic.
* **CEL & VAP Support:** Starting with Gatekeeper 3.17+, users can utilize **Common Expression Language (CEL)**, aligning with Kubernetes Validating Admission Policies (VAPs).
* **Mutation:** Support for Mutating Webhooks allows policies to modify requests (e.g., auto-injecting labels) in addition to validating them.

## 6. Fail-Open Architecture
To prevent cluster "lock-out" or downtime during failures, the RHACM-managed Gatekeeper defaults to a stable failure mode.

> **Note on Failure Policy:** The webhook is set to `failurePolicy: Ignore`. If Gatekeeper components become unavailable, the API server will continue to admit resources rather than freezing the cluster, ensuring operational continuity.
