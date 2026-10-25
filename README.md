# ocp-virt-platform

Git-ready manifests for **OpenShift Virtualization on OCP 4.22** (disconnected, agent-based install) and
**VMware ESXi → OpenShift migration with MTV 2.12**. The authoritative procedure is the runbook in
`docs/OpenShift-Virtualization-ESXi-Migration-Runbook-SOP.docx`; this repository holds the manifests it references.

| Directory | Runbook phase | Apply |
|---|---|---|
| `00-disconnected/` | Phase 0 – mirror registry, oc-mirror v2 image set, agent-based install-config / agent-config | consumed by `oc-mirror` and `openshift-install agent create image` |
| `01-virtualization/` | Phase 1 – OpenShift Virtualization operator + HyperConverged | `oc apply -f` (approve InstallPlan between files) |
| `02-operators/` | Phase 2 – NMState, ODF, NHC/SNR, OADP, MetalLB, Descheduler; observability: cluster monitoring (60–62, alertmanager.yaml), Logging 6.x + Loki (70–72), COO (73–74), Network Observability (75–76) | `oc apply -f` in numeric order |
| `03-mtv/` | Phase 3 – MTV operator, ForkliftController, vSphere Provider, ESXi Host CRs, VDDK Containerfile | `oc apply -f` |
| `04-waves/` | Phases 4–6 – NetworkMap/StorageMap, Plans, Migration CRs per wave | `oc apply -f` maps+plan, `oc create -f` migration |
| `docs/` | Runbook / SOP (Word) | — |

## Before applying
1. Replace every `<placeholder>` (see runbook Appendix A – Site Variables).
2. **Disconnected clusters:** `sed -i 's/source: redhat-operators/source: cs-redhat-operator-index-v4-22/' */*.yaml`
3. Pin `startingCSV` values to what the client catalog actually offers:
   `oc get packagemanifest <name> -n openshift-marketplace -o jsonpath='{.status.channels[*].currentCSV}'`
4. Secrets (`52-provider-vsphere.yaml`, `53-hosts/*`, OADP credentials, `alertmanager.yaml` routing keys, SIEM CA) must be sealed/externalised — never commit real values.
5. `alertmanager.yaml` is not applied with `oc apply`; load it with
   `oc -n openshift-monitoring create secret generic alertmanager-main --from-file=alertmanager.yaml --dry-run=client -o yaml | oc replace -f -`

## Workflow
```
git init -b main && git add . && git commit -m "Baseline platform manifests"
git remote add origin <repo-url> && git push -u origin main
# one branch + merge request per change; validate with: oc apply --dry-run=server -f <dir>/
```
