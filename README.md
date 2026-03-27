# rhpds.loadtester

Ansible collection providing Zero Touch (ZT) runner roles for RHDP labs.

Adds a `zerotouch-automation` runner sidecar to the showroom pod so that
solve/validate/setup buttons work via the nookbag UI.

## Roles

| Role | Mode | Use case |
|---|---|---|
| `zt_runner_ocp` | OCP-native | OCP labs — runner uses `kubernetes.core`, no SSH/bastion |
| `zt_runner_rhel` | SSH/bastion | RHEL/VM labs — runner SSHes to bastion to run scripts |

Both roles use the same unified runner image: `quay.io/rhpds/zt-runner:latest`

## Runner Image

`quay.io/rhpds/zt-runner:latest` includes:
- Latest `ansible-core` + `ansible-runner`
- Python: `kubernetes`, `jmespath`, `netaddr`, `hvac`, `pyOpenSSL`, `passlib`, `boto3`
- Collections: `kubernetes.core`, `ansible.posix`, `community.general`, `community.crypto`,
  `redhat.openshift`, `ansible.controller`, `community.hashi_vault`, `ansible.netcommon`,
  `community.mysql`, `community.postgresql`

## Usage (AgV common.yaml)

```yaml
requirements_content:
  collections:
  - name: https://github.com/rhpds/rhpds-loadtester.git
    type: git
    version: main

workloads: >-
  {{
    ['agnosticd.showroom.ocp4_workload_showroom'] +
    (['rhpds.loadtester.zt_runner_ocp'] if zt_runner_enabled | default(false) | bool else [])
  }}

zt_runner_ocp_lab_user: "devuser-{{ guid }}"
zt_runner_ocp_showroom_namespace: "{{ ocp4_workload_tenant_keycloak_username }}-showroom"
```
