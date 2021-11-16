# Git Server

A git server to hold the manifests for using flux. Here we are assuming a deployment into default namespace. Consuming `jkarlos/git-server-docker` docker image

## Setting it up

### Requirements

- access to kubeconfig

### Steps

- deploy the service

    ```bash
    kubectl apply -f git-server-deployment/
    ```

- copy the keys and repo

    ```bash
    kubectl cp keys/ $(kubectl get pods --selector=app=git-server -o json | jq -r '.items[].metadata.name'):/git-server/
    kubectl cp repos/ $(kubectl get pods --selector=app=git-server -o json | jq -r '.items[].metadata.name'):/git-server/
    ```

- Since the keys are read at bootstrap of the container. Reboot the pod for reading the keys.

    ```bash
    kubectl delete pod $(kubectl get pods --selector=app=git-server -o json | jq -r '.items[].metadata.name')
    ```

- Clone the repo locally

    ```bash

    ```

## Steps for creating a new repo

### Create repo

```bash
mkdir new-repo
cd new-repo
git init --shared=true
echo "# kube-git-server" >> README.md
git add README.md
git commit -m "first commit"
cd ..
git clone --bare new-repo new-repo.git
mv new-repo.git repos/
kubectl cp new-repo.git $(kubectl get pods --selector=app=git-server -o json | jq -r '.items[].metadata.name'):/git-server/repos
sudo rm -r new-repo
kubectl cp repos/ $(kubectl get pods --selector=app=git-server -o json | jq -r '.items[].metadata.name'):/git-server/
```

### Clone repo

Remember to include your public ssh key into the keys folder and after copying reboot (delete pod/or scale to 0 and then 1) the service.

```bash
git clone ssh://git@$(kubectl get service --selector=app=git-server -o json | jq -r '.items[].status.loadBalancer.ingress[].ip'):2222/git-server/repos/flux-git-server.git
```
