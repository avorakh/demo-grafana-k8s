# demo-grafana-k8s

---

## Tools

- [Grafana](https://grafana.com/docs/)
- [Prometheus](https://prometheus.io/docs/introduction/overview/)
- [Kubernetes](https://kubernetes.io/)
- [helm](https://helm.sh/)
- [Jenkins](https://www.jenkins.io/)

---

### Prerequisites

- The Kubernetes cluster should be created.
- The Jenkins should be installed. - [link](https://github.com/avorakh/k8s-jenkins-helm/tree/task-7)
- The Prometheus should be installed. - [link](https://github.com/avorakh/demo-prometheus-k8s/tree/develop)

---
## Install Grafana

0. **Create/Update Kubernetes Secrets**

**Create/Update the Secret for Admin Password**

   Check if the `grafana-admin-password` secret exists:
   ```bash
   kubectl get secret grafana-admin-password -n demo-metrics
   ```

   Create/update the secret:
   >  Replace `'secure-password'` with a secure password of your choice.
   ```bash
   kubectl create secret generic grafana-admin-password \
     --from-literal=password='secure-password' \
     -n demo-metrics --dry-run=client -o yaml | kubectl apply -f -
   ```
**Create Kubernetes Secrets for SMTP Credentials**

   Check if the `grafana-smtp-secret` secret exists:
   ```bash
   kubectl get secret grafana-smtp-secret -n demo-metrics
   ```

   Create/update the secret: 
   > Replace `your-email@example.com` and `your-password` with your actual  email and password. Please use the https://www.mailersend.com/ as a mail service.

   ```bash     
   kubectl create secret generic grafana-smtp-secret \
     --from-literal=smtp-user=your-email@example.com \
     --from-literal=smtp-password=your-password \
     -n demo-metrics --dry-run=client -o yaml | kubectl apply -f -
   ```

1. **Add the Bitnami Repository:**

   Run to add the Bitnami charts repository to the Helm client:

   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   ```

2. **Update Helm Repositories:**

   After adding the repository, Run to update the local Helm chart repository cache to ensure that the latest version of the charts is used:

   ```bash
   helm repo update
   ```

3. **Install Grafana:**

   Run to install Grafana using the Bitnami Helm chart:

   ```bash
   helm upgrade --install grafana bitnami/grafana -n demo-metrics -f values.yaml --create-namespace
   ```
4. **Verify the Installation:**

   Use the following command to list all Helm releases to check if Grafana is installed and running:

   ```bash
   helm list -n demo-metrics
   ```

   Also, use the following command to check the status of the pods to ensure they are running correctly:

   ```bash
   kubectl get pods -l app.kubernetes.io/instance=grafana -n demo-metrics
   ```

## Configure Grafana - Add a new data source

   After Grafana is installed, configure Prometheus as a data source. You can do this by editing the `values.yaml` file to include the Prometheus data source configuration or by using the Grafana UI.
   Configure Prometheus as a data source using the Grafana UI. - [Link](https://grafana.com/docs/grafana/latest/datasources/)
   > Replace `<prometheus-service-name>` and `<namespace>` with the appropriate values for the Prometheus installation.
   ```bash
   http://<prometheus-service-name>.<namespace>.svc.cluster.local:9090
   ```
   
   > The Prometheus data source is configured in `values.yaml`
   ```yaml
   datasources:
     secretDefinition:
       apiVersion: 1
       datasources:
       - name: Prometheus
         type: prometheus
         url: http://<prometheus-service-name>.<namespace>.svc.cluster.local:9090
         access: proxy
         isDefault: true
   ```

## Create a Dashboard

The project contains the dashboard file and this file can be imported. - [Link](k8s-cluster-dashboard.json)

Import dashboards with using Documentation. - [Link](https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/import-dashboards/)

Create a dashboard with using Documentation. - [Link](https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/create-dashboard/)

---
---
## Jenkins Pipeline Configuration

### Prerequisites
1. Kubernetes Cluster: The Jenkins instance should have access to the Kubernetes cluster.
> Please use the ['k8s-jenkins-helm'](https://github.com/avorakh/k8s-jenkins-helm/tree/task-7) repository to create cluster. 
2. Jenkins Configuration:
- The jenkins service account should have sufficient permissions in the cluster. PLease use the updated service account. [Link](https://github.com/avorakh/k8s-jenkins-helm/blob/task-7/jenkins-ci/templates/serviceaccount.yaml)
- [Slack Notification plugin](https://plugins.jenkins.io/slack/) should be installed.  Slack credentials (SLACK_CICD_CHANNEL and SLACK_TOKEN) must be set up in Jenkins.
3. Prometheus should be installed. - [Link](https://github.com/avorakh/demo-prometheus-k8s/tree/develop)

### Add Pipeline to Jenkins

1. Navigate to Jenkins Dashboard.
2. Create a new multibranch pipeline project.
3. Link the repository containing the `Jenkinsfile`.

---