# argo-manual-setup-aks

Pre-requisites:
--------------
1. Running AKS cluster


Installation Options:
---------------------
You need to create a namespace for argocd.

    kubectl create ns argocd

and then choose one of the below options :

**1. Non-HA:**

    a. cluster-admin privileges: https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    
    b. namespace level privileges: https://github.com/argoproj/argo-cd/raw/stable/manifests/namespace-install.yaml

**2. HA:**

    a. cluster-admin privileges: https://github.com/argoproj/argo-cd/raw/stable/manifests/ha/install.yaml
    
    b. namespace level privileges: https://github.com/argoproj/argo-cd/raw/stable/manifests/ha/namespace-install.yaml

**3. Light installation "Core"**

    https://github.com/argoproj/argo-cd/raw/stable/manifests/core-install.yaml

**4. Helm chart:**

   https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd

I am using N-n-HA setup with cluster-admin-privileges:

    kubectl apply -n argocd -f  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Get the admin password:

    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

ArgoCD CLI installation:

    https://argo-cd.readthedocs.io/en/stable/cli_installation/

    VERSION=$(curl -L -s https://raw.githubusercontent.com/argoproj/argo-cd/stable/VERSION)
    curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/download/v$VERSION/argocd-linux-amd64
    sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
    rm argocd-linux-amd64

Login to argocd using CLI:

    argocd login WEB_SERVER_URI

Enter admin as user and enter the password that you got previously.

    ex.  argocd login 172.30.1.2:32073 --grpc-web --plaintext
        Username: admin 
        Password: 
        'admin:login' logged in successfully
        Context '172.30.1.2:32073' updated

After successful login you can start using other commands such as listing apps or clusters.

    argocd cluster list --grpc-web

Deploy Application
------------------
