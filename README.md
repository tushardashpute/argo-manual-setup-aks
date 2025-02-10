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
    targetRevision: master
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

Once the application is successfully deployed, check the running pods:
```bash
kubectl get pods -n test
```

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

