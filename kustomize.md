# Kustomize

It is used to customize the k8s manifest file while keeping the same base file and making changes as per the environment. eg: there will be 1 base file and overlays(file) as per the environment .

![image](https://github.com/user-attachments/assets/0a8ce6f8-aee9-4711-80bf-970238ff05a3)

![image](https://github.com/user-attachments/assets/0c7e3f86-797b-4e5f-9c51-25bd52020d5c)

![image](https://github.com/user-attachments/assets/4472be58-7c02-44c6-91ae-7939feb8dfde)

![image](https://github.com/user-attachments/assets/8de9f378-81af-4e7c-8d09-c1df2b50d64c)

![image](https://github.com/user-attachments/assets/e8acbe9b-57bf-4084-b1d9-d07b0ab0ad1b)

#### Install kustomize client

Kustomize is already inbuilt in the kubernetes cluster. You can do " kubectl kustomize --help " to check that. 

```
curl -s "https://raw.githubusercontent.com/kubernetessigs/kustomize/master/hack/install_kustomize.sh" | bash
```

##### Ubuntu
```
sudo apt update
sudo apt install snapd
sudo snap install kustomize
kustomize version
kustomize --help
```

#### kustomization.yml file

![image](https://github.com/user-attachments/assets/4dfe196c-03bf-489b-86de-fc3c23fb2b1b)

![image](https://github.com/user-attachments/assets/57619e33-23d5-48bc-9fe0-758ee6094643)

![image](https://github.com/user-attachments/assets/757fc9e5-3d1d-448e-b3cf-f96e489bc0fd)

#### apply kustomize config

![image](https://github.com/user-attachments/assets/cb42d86a-694f-4fc6-9179-0e172c218cc5)

![image](https://github.com/user-attachments/assets/04708921-b5a6-4d64-b53f-2e773801cb7e)

### directory structure

![image](https://github.com/user-attachments/assets/aa450eea-ca04-4530-8a44-fbd4142b780f)

or create kustomization.yaml file in each directory

![image](https://github.com/user-attachments/assets/968085f1-8a1b-40da-a784-9898dfc69eab)

#### Transformers
It is used to replace the value of labels. 

![image](https://github.com/user-attachments/assets/613f2c86-c6f6-4751-b92b-a976d6ab82ff)

![image](https://github.com/user-attachments/assets/6c951b20-67e6-46bd-814f-f7c0fc547b32)

![image](https://github.com/user-attachments/assets/1e38e0fc-f925-4c97-9e70-261ef7668632)

![image](https://github.com/user-attachments/assets/984b9c19-9f70-4726-b60e-489430fa61e6)

![image](https://github.com/user-attachments/assets/63c1676e-4c25-4610-8711-38e1694d7161)

![image](https://github.com/user-attachments/assets/438dc360-4002-4bb7-87aa-20115b690e86)

![image](https://github.com/user-attachments/assets/d4081cf7-2923-40ff-babe-d83099b8c378)

#### Patches
Using transformer you can make changes to the labels or images but you can't modify, add or delete other things such as names. For that you have to use patches . Using patches you can pin-point a specific section.

![image](https://github.com/user-attachments/assets/a0f9341c-a183-458e-84c9-f059d699a35c)

In the below case a name is replaced from api-development to web-development

![image](https://github.com/user-attachments/assets/697e17c2-6a0f-4d0e-bbe3-b6bbb609783b)

you can also replace the replicas as follows

![image](https://github.com/user-attachments/assets/d66d92c8-b14c-49e8-8bf6-d930b85075e6)

There are two different types of patches
 JSON 6902 patch
 strategic merge patch

 ![image](https://github.com/user-attachments/assets/38195656-0317-4769-8cd6-6abc6f938e9c)

 These are further divided into inline and seperate file

 ![image](https://github.com/user-attachments/assets/b78109d1-5758-4ca8-b03f-7ff0fe9c78da)

 ![image](https://github.com/user-attachments/assets/b7e8f550-8282-4fb9-b0b0-942283b5145e)

### Using Dictionary

 Replacing a Dictionary using JSON6902

 ![image](https://github.com/user-attachments/assets/a71ac7b8-3f55-4051-b850-1338cfa488b3)

 Replacing a Dictionary using strategic merge patch 

 ![image](https://github.com/user-attachments/assets/11defbe2-c9c3-4b95-8051-1cb1602a3f87)
 
Add a Dictionary using JSON6902

![image](https://github.com/user-attachments/assets/c60151f9-ee77-428d-9352-c7321a01257f)

Add a Dictionary using strategic merge patch

![image](https://github.com/user-attachments/assets/0db26eb2-4b44-4983-bb48-5883c7721bf7)

Remove a Dictionary using JSON6902

![image](https://github.com/user-attachments/assets/71128ef9-7795-47c2-bc8d-e09747e389a7)

Remove a Dictionary using strategic merge patch

![image](https://github.com/user-attachments/assets/faccb906-7fd4-47c4-94f5-1f0671033ad1)

### Using List

Replacing a List using JSON6902

![image](https://github.com/user-attachments/assets/5f460576-9d05-4cc2-b387-0e387407dc49)

Replacing a List using strategic merge patch

![image](https://github.com/user-attachments/assets/abf09d93-1f1c-420d-83e0-57008e5fb845)

Add a List using JSON6902

![image](https://github.com/user-attachments/assets/04cff382-b2a9-414d-a20d-7a8785335c1c)

Add a List using strategic merge patch

![image](https://github.com/user-attachments/assets/b463d3ff-4e50-4d02-a8a4-c8f0801b4276)

Delete a List using JSON6902

![image](https://github.com/user-attachments/assets/6fbf100a-67cd-4a4e-af79-f23f4950fb5e)

Delete a List using strategic merge patch

![image](https://github.com/user-attachments/assets/c5308e27-7a94-44b9-b691-8c9c0356c58d)

### Overlays

If you want to apply the changes in only one environment

kubectl apply -k /root/code/k8s/overlays/QA

Environment specific config that add or modify base config

![image](https://github.com/user-attachments/assets/4cb849e6-362b-4cc7-abb0-854360b3badc)

You can add a new deployment specific to the environment. As shown below, grafana is installed only on prod

![image](https://github.com/user-attachments/assets/80d4c705-49e7-43ca-bcb6-f6e5ceac911a)

A generalised structure shown below

![image](https://github.com/user-attachments/assets/f979df26-0240-470c-94fc-5524e44d2067)

### Components

In short if we want to make similar changes in only a subset i.e QA and Prod but not in dev . We make those changes under components and apply it to only QA & prod .

![image](https://github.com/user-attachments/assets/7274dfae-8a91-474c-bbf6-c63ea754360e)

![image](https://github.com/user-attachments/assets/9393591c-09f2-4f20-af89-8281eb4390dc)

![image](https://github.com/user-attachments/assets/4aab3f65-0ce5-4a18-afbf-3fc19a36d392)

### Generators

In a deployment when secrets are changed it will only change the comfigMap but it will not change the deployment. Hence generators are used which will change the underlying deployment also by adding a suffix to the name automatically.

![image](https://github.com/user-attachments/assets/5dfc4ffc-0534-4164-9879-c0d6c3e0a0d5)

![image](https://github.com/user-attachments/assets/50613b84-862f-41ed-94ef-bfd8a483fb8c)

![image](https://github.com/user-attachments/assets/38d64138-e0da-4706-84da-c998d6355474)

### Imperative commands
using these command line commands we can make changes directly to the kustomization.yaml file. We can also make the same changes by editing the kustomization.yaml file.

![image](https://github.com/user-attachments/assets/7bcd7f0e-c624-4a59-b487-ec595796cd33)

```
kustomize edit --help

kustomize edit set namespace production  

kustomize edit set label role:admin org:kodekloud

kustomize edit set image nginx=memcached

kustomize edit set replicas api-deployment=8

kustomize edit add configmap domain-config --from-literal=domain=kodekloud.com --from-literal=subdomain=api

kustomize edit add secret my-secret --from-literal=username=root --from-literal=password=rootpassword123

kustomize edit add resource prometheus-depl.yaml
```

![image](https://github.com/user-attachments/assets/8768cc23-4890-4f92-b24e-382cf016726c)

![image](https://github.com/user-attachments/assets/6df2de12-5f39-4c3f-a06b-1f6c52e7a69e)







