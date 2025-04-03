# Grafana Deployment on OpenShift

> ⚠️ **Warning**  
If your OpenShift cluster already includes a pre-installed Grafana instance, contact your cluster administrator **before continuing**.  
In that case, you only need to create **folders, dashboards, and ConfigMaps**—**not** a new Grafana instance.

## 1. Install Grafana Operator from OperatorHub

Install the **Grafana Operator** from the **OperatorHub** in the **`openshift-operators`** namespace:

1. Navigate to **OperatorHub** in the OpenShift Web Console.
2. Search for **Grafana Operator**.
3. Select it and click **Install**.
4. In the **Installation Mode**, choose **All namespaces on the cluster (default)**.
5. Set the **Installed Namespace** to `openshift-operators`.
6. Click **Install**.

Wait until the Operator is in the `Succeeded` state and ready to use.

## 2. Create a Namespace for the Grafana Instance

```bash
oc new-project grafana
```

## 3. Prepare Credentials

By default, Grafana instances are created with `root`/`start` credentials.

You can either:

- Edit [`./basic/grafana-instance.yaml`](./basic/grafana-instance.yaml) to define custom credentials.
- Or inject credentials using a Kubernetes Secret as documented [here](https://grafana.github.io/grafana-operator/docs/examples/credential_secret/readme).

## 4. Deploy the Grafana Instance

```bash
oc create -f ./basic/grafana-instance.yaml
```

## 5. Assign Required Permissions

Grant Grafana service account the `cluster-monitoring-view` role:

```bash
oc adm policy add-cluster-role-to-user cluster-monitoring-view -z grafana-sa -n grafana
```

## 6. Create a Long-Lived Service Account Token

```bash
BEARER_TOKEN=$(oc create token grafana-sa --duration=157788000s -n grafana)
```

Create a secret with the token:

```bash
oc create secret generic grafana-sa-token \
  --from-literal=PROMETHEUS_TOKEN="${BEARER_TOKEN}" \
  -n grafana
```

---

## 7. Create the Prometheus Data Source

```bash
oc create -f ./basic/grafana-datasource.yaml
```

## 8. Create Folder for Morpheus Dashboards

```bash
oc create -f ./basic/grafana-morpheus-folder.yaml
```

## 9. Deploy Morpheus Dashboards

```bash
oc create -f ./basic/grafana-morpheus-dashboard-configmap.yaml
oc create -f ./basic/grafana-morpheus-dashboard.yaml
```

## 10. (Optional) Deploy vLLM Dashboards

```bash
oc create -f ./basic/grafana-vllm-folder.yaml
oc create -f ./basic/grafana-vllm-dashboard-configmap.yaml
oc create -f ./basic/grafana-vllm-dashboard.yaml
```
