# rhpds.loadtester

Ansible collection for Zero Touch (ZT) load testing in RHDP labs.

Triggers solve + validation on all lab modules simultaneously via the
zerotouch-automation runner API — used to verify the grading engine works
end-to-end after provisioning.

## Roles

| Role | Lab type | Description |
|---|---|---|
| `zt_load_test_ocp` | OCP tenant labs | Triggers solve + validate on all modules, reports via `agnosticd_user_info` |
| `zt_load_test_rhel` | RHEL/VM labs | Same for VM-based showrooms |

## How runner deployment works

The runner container (`zerotouch-automation`) is deployed **natively by the
`showroom/zerotouch` helm chart** when you set:

```yaml
ocp4_workload_showroom_deployer_chart_name: zerotouch
ocp4_workload_showroom_deployer_chart_version: "1.9.19"
```

To use a custom runner image (e.g. with `kubernetes.core` pre-installed for
OCP-native labs), set the image via the showroom role var
(requires agnosticd/showroom PR #52):

```yaml
ocp4_workload_showroom_runtime_automation_image: "quay.io/rhpds/zt-runner:v1.0.0"
```

## Usage in AgV

```yaml
requirements_content:
  collections:
  - name: https://github.com/agnosticd/showroom.git
    type: git
    version: v1.6.x          # or zt-runtime-automation-image branch until PR #52 merges
  - name: https://github.com/rhpds/rhpds-loadtester.git
    type: git
    version: "{{ tag }}"

workloads: >-
  {{
    [
      'agnosticd.namespaced_workloads.ocp4_workload_tenant_keycloak_user',
      'agnosticd.namespaced_workloads.ocp4_workload_tenant_namespace',
      'agnosticd.showroom.ocp4_workload_showroom'
    ] +
    (
      ['rhpds.loadtester.zt_load_test_ocp']
      if zt_load_testing_enabled | default(false) | bool
      else []
    )
  }}

# zerotouch chart — runner deployed natively
ocp4_workload_showroom_deployer_chart_name: zerotouch
ocp4_workload_showroom_deployer_chart_version: "1.9.19"
ocp4_workload_showroom_runtime_automation_image: "quay.io/rhpds/zt-runner:v1.0.0"

# Load test checkbox
# __meta__.catalog.parameters:
# - name: zt_load_testing_enabled
#   formLabel: "Run ZT Load Test"
#   openAPIV3Schema: {type: boolean, default: false}
```
