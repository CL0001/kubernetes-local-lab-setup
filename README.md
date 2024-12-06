# Setup Local K8s Lab
This quick guide uses **Oracle VirtualBox** with an **Ubuntu Server Image** setup.
Download the setup scripts using curl and run them for an easier lab setup. Alternatively, you can follow the official Kubernetes documentation [here](https://kubernetes.io/docs/setup/)

## Step 1: Install Dependencies

**For Docker:** Docker Engine and Docker CRI
```bash
curl -fsSL https://raw.githubusercontent.com/CL0001/kubernetes-local-lab-setup/refs/heads/main/get-docker.sh -o get-docker.sh
sudo sh get-docker.sh
```

#

**For Kubernetes Tools:** kubelet, kubeadm, and kubectl
```bash
curl -fsSL https://raw.githubusercontent.com/CL0001/kubernetes-local-lab-setup/refs/heads/main/get-k8s.sh -o get-k8s.sh
sudo sh get-k8s.sh
```

## Step 2: Initialize the Kubernetes Cluster
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket=unix:///var/run/cri-dockerd.sock
```

## Step 3: Set Up the kubectl Configuration
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Step 4: Install the Pod Network Add-on

We will use the **Calico** pod network add-on. You can find the official documentation [here](https://www.tigera.io/project-calico/) and the quickstart guide [here](https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart)
```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.1/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.1/manifests/custom-resources.yaml
watch kubectl get pods -n calico-system
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

## Step 5: Join worker nodes
After initializing the cluster, a join command with a token should be displayed. Run this command on worker nodes to join them to the cluster.

If the token is forgotten or expires (after 24 hours), you can generate a new token:
```bash
kubeadm token generate
kubeadm token create <generated-token> --print-join-command --ttl=0
```
