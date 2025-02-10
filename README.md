# Deploy ArgoCD to AKS and Deploy a Sample Spring Boot Application

This guide provides step-by-step instructions to deploy **ArgoCD** to an **Azure Kubernetes Service (AKS)** cluster and then use ArgoCD to deploy a sample **Spring Boot** application.

## **Prerequisites**
Ensure you have the following installed:

- **Azure CLI**: [Install Guide](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
- **Kubectl**: [Install Guide](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- **ArgoCD CLI**: [Install Guide](https://argo-cd.readthedocs.io/en/stable/getting_started/)

## **Step 1: Set Up AKS Cluster**

Create an AKS cluster if you don’t already have one:

```bash
az login
az group create --name argo-rg --location eastus
az aks create --resource-group argo-rg --name argo-aks --node-count 2 --enable-managed-identity --generate-ssh-keys
```

Retrieve the AKS credentials:
```bash
az aks get-credentials --resource-group argo-rg --name argo-aks
```

Verify the cluster connection:
```bash
kubectl get nodes
```

---

## **Step 2: Install ArgoCD on AKS**

### **Install ArgoCD using Helm**
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd --namespace argocd --create-namespace
```

### **Expose ArgoCD UI via LoadBalancer**

Patch the ArgoCD service to use LoadBalancer type:
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

Retrieve the external IP:
```bash
kubectl get svc argocd-server -n argocd
```

Once the LoadBalancer is ready, access ArgoCD at:
```bash
https://<EXTERNAL-IP>
```

![image](https://github.com/user-attachments/assets/dd47a094-5915-43a1-b031-205a901b75c3)


Retrieve the ArgoCD admin password:
```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
```

Login to ArgoCD:
```bash
argocd login <EXTERNAL-IP> --username admin --password <retrieved-password>
```

Change the admin password (optional):
```bash
argocd account update-password
```

---

## **Step 3: Deploy the Spring Boot Application**

### **Create an ArgoCD Application**

Apply the following YAML configuration to create an ArgoCD application in the `test` namespace:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: springboot-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/tushardashpute/argo-manual-setup-aks.git
    path: springboot
    targetRevision: main
  destination:
    server: https://kubernetes.default.svc
    namespace: test
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Apply the configuration:
```bash
kubectl apply -f springboot-app.yaml
```

### **Verify the Deployment**
```bash
argocd app list
argocd app get springboot-app
```

    $ argocd app list
    NAME                   CLUSTER                         NAMESPACE  PROJECT  STATUS  HEALTH       SYNCPOLICY  CONDITIONS  REPO                                                         PATH        TARGET
    argocd/springboot-app  https://kubernetes.default.svc  test       default  Synced  Progressing  Auto-Prune  <none>      https://github.com/tushardashpute/argo-manual-setup-aks.git  springboot  main
    
    $ argocd app get springboot-app
    Name:               argocd/springboot-app
    Project:            default
    Server:             https://kubernetes.default.svc
    Namespace:          test
    URL:                https://48.216.193.10/applications/springboot-app
    Source:
    - Repo:             https://github.com/tushardashpute/argo-manual-setup-aks.git
      Target:           main
      Path:             springboot
    SyncWindow:         Sync Allowed
    Sync Policy:        Automated (Prune)
    Sync Status:        Synced to main (5e9b0f7)
    Health Status:      Healthy
    
    GROUP  KIND        NAMESPACE  NAME        STATUS  HEALTH   HOOK  MESSAGE
           Service     test       springboot  Synced  Healthy        service/springboot created
    apps   Deployment  test       springboot  Synced  Healthy        deployment.apps/springboot created

We can check the same in the UI as well:

![image](https://github.com/user-attachments/assets/14148d0b-2513-49a0-b800-8494d154f909)

![image](https://github.com/user-attachments/assets/05fe2827-9c09-47a8-9f53-206cd2a44a06)


Once the application is successfully deployed, check the running pods:
```bash
kubectl get pods -n test
```

    $ k get pods -n test
    NAME                          READY   STATUS    RESTARTS   AGE
    springboot-56b59b8d5b-nw6h7   1/1     Running   0          33s
    springboot-56b59b8d5b-pd2bz   1/1     Running   0          33s
    springboot-56b59b8d5b-sj6v2   1/1     Running   0          33s

---

## **Step 4: Access the Spring Boot Application**

Retrieve the service details:
```bash
kubectl get svc -n test
```

If the application is exposed via a LoadBalancer, fetch the external IP:
```bash
kubectl get svc springboot-app -n test
```

Access the application in your browser:
```bash
http://<EXTERNAL-IP>:<PORT>
```

---

## **Conclusion**
You have successfully deployed **ArgoCD** to an **AKS cluster** and used it to manage a **Spring Boot application** in the `test` namespace. You can now leverage ArgoCD’s automated deployment capabilities to manage further updates to your application.

For more details, visit the [ArgoCD documentation](https://argo-cd.readthedocs.io/en/stable/).

