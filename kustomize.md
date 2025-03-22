# Kustomize

It is used to customize the k8s manifest file while keeping the same base file and making changes as per the environment. eg: there will be 1 base file and overlays(file) as per the environment .

![image](https://github.com/user-attachments/assets/0a8ce6f8-aee9-4711-80bf-970238ff05a3)

![image](https://github.com/user-attachments/assets/0c7e3f86-797b-4e5f-9c51-25bd52020d5c)

![image](https://github.com/user-attachments/assets/4472be58-7c02-44c6-91ae-7939feb8dfde)

![image](https://github.com/user-attachments/assets/8de9f378-81af-4e7c-8d09-c1df2b50d64c)

![image](https://github.com/user-attachments/assets/e8acbe9b-57bf-4084-b1d9-d07b0ab0ad1b)

#### Install kustomize
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

