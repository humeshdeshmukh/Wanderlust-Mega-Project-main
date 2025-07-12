#
- <b>Go to <mark> Manage Jenkins --> System</mark> and search for SonarQube installations:</b>
![image](https://github.com/user-attachments/assets/ae866185-cb2b-4e83-825b-a125ec97243a)
#
- <b>Now again, Go to <mark> Manage Jenkins --> System</mark> and search for Global Trusted Pipeline Libraries:</b
![image](https://github.com/user-attachments/assets/874b2e03-49b9-4c26-9b0f-bd07ce70c0f1)
![image](https://github.com/user-attachments/assets/1ca83b43-ce85-4970-941d-9a819ce4ecfd)
#
- <b>Login to SonarQube server, go to <mark>Administration --> Webhook</mark> and click on create </b>
![image](https://github.com/user-attachments/assets/16527e72-6691-4fdf-a8d2-83dd27a085cb)
![image](https://github.com/user-attachments/assets/a8b45948-766a-49a4-b779-91ac3ce0443c)
#
- <b>Now, go to github repository and under <mark>Automations</mark> directory update the <mark>instance-id</mark> field on both the <mark>updatefrontendnew.sh updatebackendnew.sh</mark> with the k8s worker's instance id</b>
![image](https://github.com/user-attachments/assets/3cb044b4-df88-4d68-bf7c-775cf78d5bf2)
#
- <b>Navigate to <mark> Manage Jenkins --> credentials</mark> and add credentials for docker login to push docker image:</b>
![image](https://github.com/user-attachments/assets/1a8287fc-b205-4156-8342-3f660f15e8fa)
#
- <b>Create a <mark>Wanderlust-CI</mark> pipeline</b>
![image](https://github.com/user-attachments/assets/55c7b611-3c20-445f-a49c-7d779894e232)

#
- <b>Create one more pipeline <mark>Wanderlust-CD</mark></b>
![image](https://github.com/user-attachments/assets/23f84a93-901b-45e3-b4e8-a12cbed13986)
![image](https://github.com/user-attachments/assets/ac79f7e6-c02c-4431-bb3b-5c7489a93a63)
![image](https://github.com/user-attachments/assets/46a5937f-e06e-4265-ac0f-42543576a5cd)
#
- <b>Provide permission to docker socket so that docker build and push command do not fail (Jenkins Worker)</b>
```bash
chmod 777 /var/run/docker.sock
```
![image](https://github.com/user-attachments/assets/e231c62a-7adb-4335-b67e-480758713dbf)
#
- <b> Go to Master VM and add our own AKS cluster to argocd for application deployment using cli</b>
  - <b>Login to argoCD from CLI</b>
  ```bash
   argocd login <ARGOCD_PUBLIC_IP>:<PORT> --username admin
  ```
> [!Tip]
> <ARGOCD_PUBLIC_IP>:<PORT> --> This should be your argocd url

  ![image](https://github.com/user-attachments/assets/7d05e5ca-1a16-4054-a321-b99270ca0bf9)

  - <b>Check how many clusters are available in argocd </b>
  ```bash
  argocd cluster list
  ```
  ![image](https://github.com/user-attachments/assets/76fe7a45-e05c-422d-9652-bdaee02d630f)
  - <b>Get your cluster name</b>
  ```bash
  kubectl config get-contexts
  ```
  ![image](https://github.com/user-attachments/assets/4cab99aa-cef3-45f6-9150-05004c2f09f8)
  - <b>Add your cluster to argocd</b>
  ```bash
  argocd cluster add <azure-user>@wanderlust-aks --name wanderlust-aks-cluster
  ```
  > [!Tip]
  > <azure-user>@wanderlust-aks --> This should be your AKS Cluster Name.

  ![image](https://github.com/user-attachments/assets/0f36aafd-bab9-4ef8-ba5d-3eb56d850604)
  - <b> Once your cluster is added to argocd, go to argocd console <mark>Settings --> Clusters</mark> and verify it</b>
  ![image](https://github.com/user-attachments/assets/4490b632-19fd-4499-a341-fabf8488d13c)
#
- <b>Go to <mark>Settings --> Repositories</mark> and click on <mark>Connect repo</mark> </b>
![image](https://github.com/user-attachments/assets/cc8728e5-546b-4c46-bd4c-538f4cd6a63d)
![image](https://github.com/user-attachments/assets/eb3646e2-db84-4439-a11a-d4168080d9cc)
![image](https://github.com/user-attachments/assets/a07f8703-5ef3-4524-aaa7-39a139335eb7)
> [!Note]
> Connection should be successful

- <b>Now, go to <mark>Applications</mark> and click on <mark>New App</mark></b>

![image](https://github.com/user-attachments/assets/ec2d7a51-d78f-4947-a90b-258944ad59a2)

> [!Important]
> Make sure to click on the <mark>Auto-Create Namespace</mark> option while creating argocd application

![image](https://github.com/user-attachments/assets/55dcd3c2-5424-4efb-9bee-1c12bbf7f158)
![image](https://github.com/user-attachments/assets/3e2468ff-8cb2-4bda-a8cc-0742cd6d0cae)

- <b>Congratulations, your application is deployed on Azure AKS Cluster</b>
![image](https://github.com/user-attachments/assets/bc2d9680-fe00-49f9-81bf-93c5595c20cc)
![image](https://github.com/user-attachments/assets/1ea9d486-656e-40f1-804d-2651efb54cf6)
- <b>Open port 31000 and 31100 on worker VM and Access it on browser</b>
```bash
<worker-public-ip>:31000
```
![image](https://github.com/user-attachments/assets/a4b2a4b4-e1aa-4b22-ac6b-f40003d0723a)
![image](https://github.com/user-attachments/assets/06f9f1c8-094d-4d9f-a9d8-256fb18a9ae4)
![image](https://github.com/user-attachments/assets/64394f90-8610-44c0-9f63-c3a21eb78f55)
- <b>Email Notification</b>
![image](https://github.com/user-attachments/assets/0ab1ef47-f939-4618-8651-6aa9274721f4)

#
## How to monitor AKS cluster, kubernetes components and workloads using prometheus and grafana via HELM (On Master VM)
- <p id="Monitor">Install Helm Chart</p>
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```
```bash
chmod 700 get_helm.sh
```
```bash
./get_helm.sh
```

#
-  Add Helm Stable Charts for Your Local Client
```bash
helm repo add stable https://charts.helm.sh/stable
```

#
- Add Prometheus Helm Repository
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

#
- Create Prometheus Namespace
```bash
kubectl create namespace prometheus
```
```bash
kubectl get ns
```

#
- Install Prometheus using Helm
```bash
helm install stable prometheus-community/kube-prometheus-stack -n prometheus
```

#
- Verify prometheus installation
```bash
kubectl get pods -n prometheus
```

#
- Check the services file (svc) of the Prometheus
```bash
kubectl get svc -n prometheus
```

#
- Expose Prometheus and Grafana to the external world through Node Port
> [!Important]
> change it from Cluster IP to NodePort after changing make sure you save the file and open the assigned nodeport to the service.

```bash
kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus
```
![image](https://github.com/user-attachments/assets/90f5dc11-23de-457d-bbcb-944da350152e)
![image](https://github.com/user-attachments/assets/ed94f40f-c1f9-4f50-a340-a68594856cc7)

#
- Verify service
```bash
kubectl get svc -n prometheus
```

#
- Now,let’s change the SVC file of the Grafana and expose it to the outer world
```bash
kubectl edit svc stable-grafana -n prometheus
```
![image](https://github.com/user-attachments/assets/4a2afc1f-deba-48da-831e-49a63e1a8fb6)

#
- Check grafana service
```bash
kubectl get svc -n prometheus
```

#
- Get a password for grafana
```bash
kubectl get secret --namespace prometheus stable-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```
> [!Note]
> Username: admin

#
- Now, view the Dashboard in Grafana
![image](https://github.com/user-attachments/assets/d2e7ff2f-059d-48c4-92bb-9711943819c4)
![image](https://github.com/user-attachments/assets/3d6652d0-7795-4fe9-8919-f33eac88db73)
![image](https://github.com/user-attachments/assets/13321ee5-5d7b-4976-b409-25d3b865a42a)
![image](https://github.com/user-attachments/assets/75a22e4b-ae81-4cad-9c92-21dd90d126a8)

#
## Clean Up
- <b id="Clean">Delete AKS cluster</b>
```bash
az aks delete --name wanderlust-aks --resource-group wanderlust-rg --yes
```

# 