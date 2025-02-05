---
title: "安装指南"
description: "了解如何安装或卸载 Kapacity"
weight: 12

---

本文档主要介绍环境准备、如何快速安装部署Kapacity。

## 环境准备

- Kubernetes 1.19+
- Helm 3+
- Cert-Manager 1.11+
- Prometheus

## 安装流程

### 安装 CertManager

使用 helm 安装 Cert-Manager

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --set installCRDs=true \
  --create-namespace \
  --version v1.12.0 
```

### 安装 Prometheus

可以使用 helm 安装 Prometheus

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install prometheus  prometheus-community/prometheus -n prometheus  \
   --create-namespace \
   --set pushgateway.enabled=false \
   --set alertmanager.enabled=false \
   --set prometheus-node-exporter.hostRootFsMount.enabled=false
```

查看 Prometheus Server 的 ClusterIp 和 端口

```bash
kubectl -nprometheus get svc

NAME                                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
prometheus-kube-state-metrics         ClusterIP   10.97.190.94     <none>        8080/TCP   2d2h
prometheus-prometheus-node-exporter   ClusterIP   10.100.121.134   <none>        9100/TCP   2d2h
prometheus-server                     ClusterIP   10.104.214.48    <none>        80/TCP     2d2h
```

### 安装 Kapacity

使用 Helm 安装 Kapacity

- 添加Helm仓库地址

```bash
helm repo add kapacity https://traas-stack.github.io/kapacity-charts
```

- 更新 Helm 仓库到最新版本

```bash
helm repo update
```

- 使用 Helm Charts 安装 Kapacity

安装 Kapacity 时 prometheus-address 参数可以通过 [安装 Prometheus](#安装-prometheus) 步骤获取

```bash
helm install kapacity-manager kapacity -n kapacity-system \
  --create-namespace \
  --set prometheus.address=http://<prometheus-server-clusterip>:<port> 
```

- 验证 Kapacity 安装是否成功

```bash
kubectl get deploy -n kapacity-system

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
kapacity-controller-manager   1/1     1            1           3d23h
```

## 卸载安装

```bash
helm uninstall kapacity -n kapacity-system
helm uninstall prometheus -n prometheus
helm uninstall cert-manager -n cert-manager
```