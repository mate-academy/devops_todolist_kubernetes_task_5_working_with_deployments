# Kubernetes Deployment for Django ToDo App

## 🗂 Deployment Description

This Kubernetes setup includes:
- A `Deployment` with RollingUpdate strategy
- A `HorizontalPodAutoscaler` triggered by CPU and Memory usage
- Everything scoped under the `mateapp` namespace

---

## 🔧 Why these configurations?

### 🚀 Deployment Strategy

```yaml
rollingUpdate:
  maxSurge: 1
  maxUnavailable: 1
