# 🔐 Kubernetes Network Isolation with Calico

This repository demonstrates how to implement **namespace isolation** in a Kubernetes cluster using [Calico](https://projectcalico.docs.tigera.io/), a powerful networking and network security solution.

---

## 📁 Repository Structure

```
.
├── README.md                      # This file
├── global-policy.yaml             # GlobalNetworkPolicy to deny all ingress traffic
├── namespaced-policies.yaml       # Namespace-specific allow policy (ns-a → ns-b)
└── namespaces-pods-services.yaml  # Creates 3 namespaces, pods, and services
```

---

## 🚀 Purpose of the Demo

By default, Kubernetes allows unrestricted communication between all pods.  
**This demo shows how to restrict pod-to-pod traffic using Calico policies.**

We will:

1. Deploy three pods in three different namespaces.
2. Apply a **GlobalNetworkPolicy** that denies all ingress traffic.
3. Apply a **namespaced NetworkPolicy** that explicitly allows traffic **from `ns-a` to `ns-b` only**.
4. Test communication via `wget` inside pods.

---

## 🧱 Setup Instructions

### 1. Start Minikube with Calico

```bash
minikube start --cni=calico
```

> This ensures Calico is properly installed with CRDs and CNI plugin.

---

### 2. Deploy Namespaces, Pods, and Services

```bash
kubectl apply -f namespaces-pods-services.yaml
```

> Each pod runs `nginx:alpine` with `ping`, `wget`, and `curl` pre-installed.

---

### 3. Apply Global Ingress Deny Policy

```bash
kubectl apply -f global-policy.yaml
```

> This policy blocks all incoming traffic to pods, except Calico’s internal control traffic.

---

### 4. Apply Namespaced Allow Policy (ns-a → ns-b)

```bash
kubectl apply -f namespaced-policies.yaml
```

This policy allows only:
- **Ingress** into `ns-b` from:
  - `ns-a`
  - Pods with a specific `subnets` label (optional)

---

## 🧪 Demo: What to Expect

| Source Pod    | Target Service                   | Expected Result |
|---------------|----------------------------------|-----------------|
| `pod-a` (ns-a) | `svc-b.ns-b.svc.cluster.local`   | ✅ Allowed       |
| `pod-a` (ns-a) | `svc-c.ns-c.svc.cluster.local`   | ❌ Blocked       |
| `pod-c` (ns-c) | `svc-b.ns-b.svc.cluster.local`   | ❌ Blocked       |

### Example Test Command:

```bash
kubectl exec -n ns-a pod-a -it -- wget --spider svc-b.ns-b.svc.cluster.local
```

---

## 📌 Notes

- The GlobalNetworkPolicy uses `namespaceSelector` to avoid interfering with `kube-system` and `calico-system`.
- DNS (port 53) is not blocked since we only deny **Ingress**.

---

## ✅ Summary

This project demonstrates:
- How to enforce **zero-trust networking** with Calico
- The use of **default-deny** with **opt-in allow** policies
- A modular setup to easily extend to more namespaces or policies

---

## 📚 References

- [Calico Official Docs](https://projectcalico.docs.tigera.io/)
- [Kubernetes Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)