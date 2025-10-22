---
title: 在 Kubernetes 中使用 Docker 加固镜像
linktitle: 在 Kubernetes 中使用镜像
description: 学习如何在 Kubernetes 部署中使用 Docker 加固镜像。
keywords: 使用加固镜像, kubernetes, k8s
weight: 35
---

{{< summary-bar feature_name="Docker Hardened Images" >}}

## 身份认证

若要在 Kubernetes 中使用 Docker 加固镜像，需先创建用于从镜像仓库或内部仓库拉取镜像的 Secret。

> [!NOTE]
>
> 请在每个使用 DHI 的 Kubernetes 命名空间中分别创建该 Secret。

使用个人访问令牌（PAT）创建 Secret。确保令牌至少拥有私有仓库的只读权限。若使用 Docker Hub，请将 `<registry server>` 替换为 `docker.io`。

```console
$ kubectl create -n <kubernetes namespace> secret docker-registry <secret name> --docker-server=<registry server> \
        --docker-username=<registry user> --docker-password=<access token> \
        --docker-email=<registry email>
```

使用以下命令测试 Secret 是否生效：

```console
kubectl apply --wait -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: dhi-test
  namespace: <kubernetes namespace>
spec:
  containers:
  - name: test
    image: <your-namespace>/dhi-bash:5
    command: [ "sh", "-c", "echo 'Hello from DHI in Kubernetes!'" ]
  imagePullSecrets:
  - name: <secret name>
EOF
```

查看 Pod 状态：

```console
$ kubectl get -n <kubernetes namespace> pods/dhi-test
```

若返回如下结果，说明镜像拉取并运行成功：

```console
NAME       READY   STATUS      RESTARTS     AGE
dhi-test   0/1     Completed   ...          ...
```

若看到以下状态，则表明 Secret 可能配置有误：

```console
NAME       READY   STATUS         RESTARTS   AGE
dhi-test   0/1     ErrImagePull   0          ...
```

确认 Pod 输出，应返回 `Hello from DHI in Kubernetes!`：

```console
kubectl logs -n <kubernetes namespace> pods/dhi-test
```

测试完成后，可删除该测试 Pod：

```console
$ kubectl delete -n <kubernetes namespace> pods/dhi-test
```
