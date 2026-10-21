# ACM IaC

Infrastructure-as-Code repository for bootstrapping and managing **Red Hat Advanced Cluster Management (ACM)** on OpenShift, using the GitOps app-of-apps pattern with Argo CD.

## How it works

1. Install the **OpenShift GitOps operator** (Argo CD) via Kustomize.
2. Configure the Argo CD instance (Helm plugin, repository-root references, custom health checks).
3. Two **ApplicationSets** take over from there:
   - `cluster-bootstrap-appset` — installs cluster-scoped operators and operands, driven by the `acm-placement` ConfigMap (`lab-clusterset` label), syncing in `operators → operand → configuration` order.
   - `clusters-appset` — deploys per-cluster applications from `kustomize/cluster-overlays/<cluster>` and Helm charts in `helm/charts`, with per-cluster values under `helm/cluster-overlays/<cluster>`.

## Repository layout

- `argo-applications/` — the ApplicationSet manifests (`cluster-bootstrap-appset.yaml`, `applications.yaml`).
- `helm/` — Helm charts (`charts/`) and per-cluster value overlays (`cluster-overlays/`).
- `kustomize/` — Kustomize bases (`bases/`), the GitOps operator install (`cluster-bootstrap/`), and per-cluster overlays (`cluster-overlays/`).

## Getting started

### Prerequisites

- An OpenShift 4.x cluster with cluster-admin access.
- The `oc` CLI logged in.

### Bootstrap

Install the OpenShift GitOps operator (Argo CD):

```bash
oc apply -k kustomize/cluster-bootstrap/openshift-gitops-operator
```

Configure the Argo CD instance — enables the Helm plugin, referencing charts from the repository root, and custom health checks:

```bash
oc apply -k kustomize/cluster-bootstrap/openshift-gitops-configuration
```

### Add a cluster or application

- Kustomize-based app: add an overlay under `kustomize/cluster-overlays/<cluster>/`.
- Helm-based app: add the chart under `helm/charts/` and its values under `helm/cluster-overlays/<cluster>/<chart>/values.yaml`.

The ApplicationSets pick up new directories automatically (automated sync with retry, plus self-heal on the bootstrap set).

## Notes

- Tested against OpenShift 4.x; adjust `revision`/`targetRevision` if you pin a different branch.
- Sync order is enforced via RollingSync: operators first, then operands, then configuration.
