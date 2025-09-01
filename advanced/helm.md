# HELM

## Install 
```
sudo snap install helm --classic
```
or
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
or
```
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add –
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

### Helm Key Components
* Helm CLI
  
The Helm command-line utility runs on your local machine, enabling you to install charts, upgrade releases, and roll back changes, among other operations.

* Charts
  
Charts are packages comprised of files that include all the instructions Helm needs to create the Kubernetes objects required by an application. They serve as reusable deployment packages and are available publicly from various repositories.

* Releases
  
A release is created when a chart is deployed to your cluster. It represents a single installation of an application based on a Helm chart. Each time you perform an action—such as an upgrade or configuration change—a new revision (or snapshot) is generated, enabling independent management of multiple application versions.

* Metadata
  
Helm stores release metadata, including chart details and revision history, as secrets within your Kubernetes cluster. This ensures that the deployment history remains accessible to everyone working on the cluster. The metadata is stored as a secret in k8s ckuster.

## Helm Charts
![image](https://github.com/user-attachments/assets/1bfb123d-e2e7-470d-930e-1ab196235ee0)

![image](https://github.com/user-attachments/assets/9dc1e9b4-62b4-4943-b716-19ede2776ef0)

![image](https://github.com/user-attachments/assets/ad17b439-e0fb-49e6-b3ce-e96101840a12)

### Charts.yaml

 This file contains the metadata of that helm chart . But it also contains any dependencies if required to be installed for that application .

 <img width="906" height="735" alt="image" src="https://github.com/user-attachments/assets/8c95a100-4087-4fd4-804a-3c5fa141321e" />

* apiVersion: Indicates the chart API version. Helm 3 charts use v2 and Helm 2 charts use v1.
* appVersion: Specifies the version of the packaged application (e.g., WordPress version 5.8.1).
* version: Represents the chart version and helps track changes independently of the application version.
* name, description, and type: Define the chart's name (WordPress), provide a brief description, and indicate that it is an "application" type chart.
* dependencies: Lists any dependent charts. In this example, the WordPress chart depends on a MariaDB chart, managing its deployment separately.
* keywords, maintainers, home, and icon: Provide additional metadata useful for chart discovery and reference.

**A typical Helm chart directory structure includes:**

A **templates** directory containing all templated resource manifests.
A **values.yaml** file that defines configuration parameters.
A **Chart.yaml** file holding chart metadata.
Optionally, a **charts directory** for dependencies and files such as a README or license.

search locally or in the hub
```
helm search wordpress
helm search hub wordpress
```
add repo 
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```
install repo
```
helm install my release bitnami wordpress
```
list the repos installed
```
helm list
```
uninstall the repo
```
helm uninstall my release
```
You can override the existing variables of values.yml file using the --set caluse 
![image](https://github.com/user-attachments/assets/2e51be8f-ce32-4dd4-a86f-72a1290f96b6)
You can also provide different values using the custom-values.yml file
![image](https://github.com/user-attachments/assets/49293f46-f4cf-498e-846e-4994b14c4dd1)
Using helm pull cmd you can download the repo and then untar it. You can then make the required changes and install it.
![image](https://github.com/user-attachments/assets/873d671f-6b72-405f-87fc-ab70bd7f3822)
you can install the same repo by changing the release name
![image](https://github.com/user-attachments/assets/2bdc97cf-9f95-4fd6-b109-9842b1206e04)

upgrade using helm upgrade and history cmd gives detail of all releases
![image](https://github.com/user-attachments/assets/2a7fc52b-be6d-4436-8836-76b24f4419fe) 
![image](https://github.com/user-attachments/assets/85b8f596-4fc6-46ad-8913-7e9bc5a401af)

Rollback cmd to rollback the changes made
![image](https://github.com/user-attachments/assets/ef19e330-4bf4-474b-ae35-0c28fa5f183a)

Creating a helm chart

![image](https://github.com/user-attachments/assets/18989f1d-0138-4a03-b3e1-ec84a1ea18bf)

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
spec:
  replicas: 2
  selector:
    matchLabels:
    app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
        - name: hello-world
          image: nginx
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
```
```
apiVersion: v1
kind: Service
metadata:
  name: hello-world
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app: hello-world
```
You can templatize the release name using {{ .Relaese.Name }} built-in variable
![image](https://github.com/user-attachments/assets/33895765-3397-411e-9a92-a42938b35762)

Default built-in variables
![image](https://github.com/user-attachments/assets/f109567a-4d65-4bf7-84d5-984fd074131a)

this is how we templatize
![image](https://github.com/user-attachments/assets/de68f067-5301-4af8-b4f9-d553a11b7414)

![image](https://github.com/user-attachments/assets/f0336032-7d96-4bc4-9108-4375ba568548)

lint cmd is to check if the syntax and variables are proper

![image](https://github.com/user-attachments/assets/c845b24e-5c69-49ed-be32-e2aa04937c9a)

Using template command you can substitute the values and check. There is also a --debug option available with the template cmd to debug

![image](https://github.com/user-attachments/assets/a411dec3-1c5e-470e-8d6f-93b26feed487)

You can also do a dry-run using --dry-run cmd

![image](https://github.com/user-attachments/assets/aba20818-65d4-4826-b044-3dc88aa63926)

different string functions available

![image](https://github.com/user-attachments/assets/eb71074b-c683-4a2b-9881-7eeeb3f5d290)

functions can be further enhanced using | pipe operator

![image](https://github.com/user-attachments/assets/baf7692d-869e-45e6-8530-9fa2da05e5ff)

![image](https://github.com/user-attachments/assets/f25ae7fe-49a2-4607-befb-d457b5569009)

you can add conditions using the if clause

![image](https://github.com/user-attachments/assets/bfe0c3f8-e865-4175-aacb-1f4ed19afcad)

But there is a problem when if is added, the block is not visible in the resulting file generated and hence we add a - before if 

![image](https://github.com/user-attachments/assets/d91a99d4-9e00-491a-a971-3c594d0c5544)

![image](https://github.com/user-attachments/assets/4ba51d07-0dd3-4ff0-8f5e-16f55dd31e4b)

you can also add the else block to the if condition

![image](https://github.com/user-attachments/assets/08bf509e-bf61-4912-966f-7962a3b23c7b)

here an else block is added to the start of serviceaccount.yml file if the create is true

![image](https://github.com/user-attachments/assets/64e2db62-0471-49e5-bdca-9c54469be1df)

setting scope using the with clause

![image](https://github.com/user-attachments/assets/de2d24b2-2528-4350-80ca-b982c7ab384f)

![image](https://github.com/user-attachments/assets/5f0c9f73-418d-41bf-865b-c6cb6a55cb5e)

![image](https://github.com/user-attachments/assets/fe1e7e70-6b1f-4ddc-a72c-1585bfea7a32)

![image](https://github.com/user-attachments/assets/f4505eac-43dc-425e-beb5-c4804343099f)

use the $ operator within the scope to go to the root level

![image](https://github.com/user-attachments/assets/bc99fc8b-ef04-4f8b-93fe-90b029c0eb4b)

you can use range operator to traverse through a list

![image](https://github.com/user-attachments/assets/2fe14c87-73c4-4c5d-a472-372adf9a0548)

if there are similar values you can use the named templates to replace those values, the values need to be written in _helper.tpl file. You can write it in any file that starts with underscore .

![image](https://github.com/user-attachments/assets/2d30ea03-4a23-4d0b-9d0f-c5af5be3cd78)

![image](https://github.com/user-attachments/assets/00d8446f-02f2-4e64-9d19-f44b3a5137cf)

template is an action so you cannot use it with | pipe operator using other functions. Use include to use it with other functions

![image](https://github.com/user-attachments/assets/44460570-1980-443b-b95d-0ec8b3b2120f)

chart hooks are used if you want to add one more process of your own . like taking a backup of the db before the upgrade . You can also specify a weight to change the precedence of file execution .

![image](https://github.com/user-attachments/assets/496f41d8-cd07-4364-819b-e82a084b02a0)

![image](https://github.com/user-attachments/assets/5e0235fc-4526-41a5-8b21-9982f0c737df)

this is how you create gpg keys and package the chart

![image](https://github.com/user-attachments/assets/2c304651-4f4c-4a4e-98d1-0c1b7bd95999)

![image](https://github.com/user-attachments/assets/d65bb782-e0d0-482a-91af-86ed704d2a9e)

verify the signature

![image](https://github.com/user-attachments/assets/75fa33a5-bc02-4698-a31b-5a4c7c4f693d)

uploading the charts

![image](https://github.com/user-attachments/assets/7a2cfcf2-3226-4503-9e45-f86ffc13b29f)

installing the same uploaded chart

![image](https://github.com/user-attachments/assets/357595bb-e53c-49d2-a5f4-8e3a9f9af953)

this is what we learned today

![image](https://github.com/user-attachments/assets/76159d49-450c-4ab3-87c8-946a0f7dd2ab)
