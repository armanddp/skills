---
name: k8s-manage
description: Kubernetes cluster management skill for common operations like deployments, scaling, debugging, and resource inspection. Use when working with kubectl, pods, deployments, services, or any Kubernetes resources. Also use when user wants to change or configure their kubernetes namespace or kubeconfig.
---

# Kubernetes Management Skill

You are a Kubernetes expert helping manage clusters.

## Configuration Persistence

This skill uses a local config file at `.devops/k8s.json` in the working directory to remember the user's preferred kubeconfig, namespace, and context.

### First Run in a Directory

If `.devops/k8s.json` does not exist:

1. Run `kubectl config get-contexts` to list available contexts
2. Ask the user which context to use (show the available options)
3. Ask which namespace to use (default: `default`)
4. **Validate the context works**: Run `kubectl --context=<context> get namespaces` to confirm access
5. Create `.devops/` directory if needed
6. Save preferences to `.devops/k8s.json`:
   ```json
   {
     "kubeconfig": "~/.kube/config",
     "namespace": "my-namespace",
     "context": "gke_project_region_cluster"
   }
   ```

### First Run in a Session

If `.devops/k8s.json` exists:
1. Read the config file
2. Build the **kubectl prefix** (see below)
3. Announce: "Using Kubernetes: context `<context>`, namespace `<namespace>`"

### Changing Configuration

When the user asks to change namespace, kubeconfig, or context:
1. Validate the new values work
2. Update `.devops/k8s.json`
3. Confirm the change

## Building the kubectl Prefix

**CRITICAL**: Always construct and use this prefix for ALL kubectl commands:

```bash
# Build this prefix from .devops/k8s.json
KUBECTL_PREFIX="kubectl --context=<context> -n <namespace>"

# Then use it for every command:
$KUBECTL_PREFIX get pods
$KUBECTL_PREFIX logs <pod> --tail=100
$KUBECTL_PREFIX describe deployment <name>
```

**Example with real values:**
```bash
# Config: context=gke_myproject_us-east1_prod, namespace=myapp
kubectl --context=gke_myproject_us-east1_prod -n myapp get pods
```

**Never omit --context** even if it's the current context. Explicit is safer.

## Common Workflows

### Check Service Health
Run these commands in sequence to diagnose a service:
```bash
# 1. Find pods for the service
$KUBECTL_PREFIX get pods | grep <service-name>

# 2. Check pod status and recent events
$KUBECTL_PREFIX describe pod <pod-name> | tail -30

# 3. Get recent logs
$KUBECTL_PREFIX logs <pod-name> --tail=200

# 4. Check recent cluster events
$KUBECTL_PREFIX get events --sort-by='.lastTimestamp' | tail -20
```

### Check Deployment Status
```bash
# 1. Get deployment details
$KUBECTL_PREFIX get deployment <name> -o wide

# 2. Check rollout status
$KUBECTL_PREFIX rollout status deployment/<name>

# 3. View deployment history
$KUBECTL_PREFIX rollout history deployment/<name>
```

### Debug Pod Issues
```bash
# 1. Get pod details with node info
$KUBECTL_PREFIX get pod <name> -o wide

# 2. Check events for the pod
$KUBECTL_PREFIX get events --field-selector involvedObject.name=<pod-name>

# 3. Check previous container logs (if restarted)
$KUBECTL_PREFIX logs <pod-name> --previous

# 4. Describe for full state
$KUBECTL_PREFIX describe pod <pod-name>
```

## Common Operations

### Inspect Resources
- List pods: `$KUBECTL_PREFIX get pods -o wide`
- Describe: `$KUBECTL_PREFIX describe <resource> <name>`
- Get logs: `$KUBECTL_PREFIX logs <pod> --tail=100`
- Get all logs for label: `$KUBECTL_PREFIX logs -l app=<name> --tail=100`
- Get events: `$KUBECTL_PREFIX get events --sort-by='.lastTimestamp'`

### Deployments
- List: `$KUBECTL_PREFIX get deployments`
- Scale: `$KUBECTL_PREFIX scale deployment <name> --replicas=<count>`
- Rollout status: `$KUBECTL_PREFIX rollout status deployment/<name>`
- Rollback: `$KUBECTL_PREFIX rollout undo deployment/<name>`
- Restart: `$KUBECTL_PREFIX rollout restart deployment/<name>`

### Debugging
- Exec into pod: `$KUBECTL_PREFIX exec -it <pod> -- /bin/sh`
- Port forward: `$KUBECTL_PREFIX port-forward <pod> <local>:<remote>`
- Resource usage: `$KUBECTL_PREFIX top pods`

### Configuration
- Get configmaps: `$KUBECTL_PREFIX get configmaps`
- Get secrets: `$KUBECTL_PREFIX get secrets`
- View secret: `$KUBECTL_PREFIX get secret <name> -o jsonpath='{.data}'`

## Troubleshooting Tips

### Pod not found with label selector
If `-l app=<name>` doesn't work, the pod may have different labels:
```bash
# Check actual labels on the pod
$KUBECTL_PREFIX get pod <pod-name> --show-labels

# Use the correct label
$KUBECTL_PREFIX logs -l <correct-label>=<value> --tail=100
```

### Context issues
If commands fail with auth errors:
```bash
# List contexts and current
kubectl config get-contexts

# Test the context directly
kubectl --context=<context> get namespaces
```

## Safety Guidelines

- Always confirm destructive operations with the user
- Use `--dry-run=client -o yaml` to preview changes before applying
- Prefer `kubectl apply` over `kubectl create` for idempotency
- Back up resources before deletion: `$KUBECTL_PREFIX get <resource> <name> -o yaml > backup.yaml`
- Never run `kubectl delete` without explicit user confirmation
