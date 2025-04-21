 # Google Kubernetes Engine

## Architecture
It also has a Cloud Controller manager { CCM }

Depending on the implementation of the cloud controller manager, these are the controllers that are provided by the cloud controller manager:

- Node Controller: This controller is responsible for checking if a particular node is present in the cloud or not and updating the node labels with appropriate labels and annotations.
- Route Controller: This controller as the name suggests is responsible for configuring routes in the cloud environment so that the nodes remain reachable.
- Service Controller: Whenever a service of the type load balancer is deployed on the kubernetes cluster then this controller comes into the picture. This controller is responsible for creating a load balancer in the cloud environment.

https://medium.com/@murtazavasi.dev/demystifying-cloud-controller-manager-0ba2d509603c 

![image](https://github.com/user-attachments/assets/0d1e4d7a-abf1-4d52-bdc5-22962923ab32)

## There are two modes of operation available

![image](https://github.com/user-attachments/assets/5184c3cb-2cc4-42d7-82ea-35b04620b307)

![image](https://github.com/user-attachments/assets/88fbe175-af9d-4e19-8478-01e9ce2bab5d)

### Standard mode

Standard mode can be configured as regional or zonal mode

![image](https://github.com/user-attachments/assets/a7ed3d0e-2e98-4ce4-af1e-11f15b2f2093)

Zonal cluster is deployed in a single zone and has only 1 controlplane in that zone .Single zone cluster all infra in 1 zone .

![image](https://github.com/user-attachments/assets/639139be-fb7f-447d-9590-a718ecb07430)

In Multi-zone cluster there is still single control plane but the worker nodes are distributed in multiple zones .

![image](https://github.com/user-attachments/assets/c05bd670-6d5e-4fcf-89e1-a9d5c3208210)

### regional mode 
Regional mode has multiple control planes in multiple zones . a node pool by default is copied across 3 zones. this value can be changed later .

![image](https://github.com/user-attachments/assets/72714d0e-4503-4865-a953-b5c451ef7ca2)



