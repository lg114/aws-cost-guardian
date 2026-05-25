# EKS — Cost Analysis Module

## Cost Signals

- Node instance types and count
- Pod resource requests vs actual usage
- Namespace resource quotas
- Persistent volume claims
- Load balancer count

## Common Waste Patterns

1. **Over-provisioned Nodes** — Node CPU/memory utilization <40%
2. **Idle Namespaces** — Namespaces with no running pods
3. **Unbound PVCs** — PersistentVolumeClaims not attached to pods
4. **Excessive Node Groups** — Multiple node groups for similar workloads
5. **No Cluster Autoscaler** — Fixed node count regardless of demand

## Optimization Playbook

### Node Right-sizing
- Use Karpenter or Cluster Autoscaler for dynamic scaling
- Right-size node instance types based on pod resource requests
- Consider Graviton instances (ARM) for cost savings

### Pod Optimization
- Set resource requests based on actual usage (not guesses)
- Use Vertical Pod Autoscaler (VPA) for right-sizing
- Use Horizontal Pod Autoscaler (HPA) for scaling

### Namespace Management
- Set resource quotas per namespace
- Clean up unused namespaces
- Use LimitRanges to set default resource limits

## Remediation Commands

```bash
# List nodes with utilization
kubectl top nodes

# List pods by resource usage
kubectl top pods --all-namespaces --sort-by=cpu

# Find unbound PVCs
kubectl get pvc --all-namespaces | grep -v Bound

# Scale down unused deployment
kubectl scale deployment DEPLOYMENT_NAME --replicas=0 -n NAMESPACE

# Delete unused namespace (after confirmation)
kubectl delete namespace NAMESPACE_NAME
```
