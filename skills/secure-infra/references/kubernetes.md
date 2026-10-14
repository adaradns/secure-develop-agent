# Kubernetes & Helm — secure checklist

Format: **What to check** · **Why** · **Fix**. Verify against the real manifests/Helm values.

## 1. Pod securityContext
- **Why:** containers default to permissive settings; one escape → node compromise.
- **Check / Fix:** per container/pod set:
  ```yaml
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    allowPrivilegeEscalation: false
    readOnlyRootFilesystem: true
    capabilities:
      drop: ["ALL"]
  ```

## 2. No privileged / hostPath / hostNetwork
- **Check:** no `privileged: true`, no `hostNetwork`/`hostPID`/`hostIPC`, no `hostPath` volumes
  mounting sensitive host paths. Flag any of these for justification.

## 3. Secrets handling
- **Check:** secrets come from `Secret` objects or an external manager (External Secrets, Vault,
  cloud secret store) — **not** hardcoded in `env:` values or committed Helm values.
- **Why:** Kubernetes `Secret`s are base64 (not encrypted) at rest unless etcd encryption is on —
  prefer a real secrets manager for sensitive values.

## 4. Resource limits
- **Why:** missing limits enable noisy-neighbor and resource-exhaustion DoS.
- **Check:** `resources.requests` and `resources.limits` set for CPU and memory.

## 5. Network policy & RBAC
- **Check:** `NetworkPolicy` restricts pod-to-pod traffic (default-deny ingress). ServiceAccounts
  use least-privilege `Role`/`RoleBinding`; don't bind `cluster-admin` to app workloads.
- **Check:** `automountServiceAccountToken: false` where the pod doesn't call the API.

## 6. Image pull policy & provenance
- **Check:** pinned image digests; private registry with pull secrets; optionally admission
  control (signed images / Kyverno / OPA Gatekeeper).

> TODO: expand with cluster-specific policies (PodSecurity admission level, service mesh mTLS)
> as the project's platform standards are defined.
