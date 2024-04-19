# Kubernetes Operations Cheat Sheet

This README serves as a comprehensive cheat sheet for various Kubernetes operations, including node management, upgrades, user management, configuration, and troubleshooting.

## Node Management

### Node Draining

```bash
kubectl drain <node_name>
kubectl drain <node_name> --ignore-daemonsets
kubectl drain <node_name> --force
kubectl uncordon <node_name>
```

## Upgrades

### Master Node

```bash
kubectl get nodes
kubectl drain <node-to-drain> --ignore-daemonsets
sudo apt-get update
sudo apt-get install -y --allow-change-held-packages kubeadm=1.21.1-00
kubeadm version
sudo kubeadm upgrade plan v1.21.1-00
sudo kubeadm upgrade apply v1.21.1
sudo apt-get install -y --allow-change-held-packages kubelet=1.21.1-00 kubectl=1.21.1-00
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl uncordon <node-to-uncordon>
kubectl get nodes
```

### Worker Node

```bash
kubectl drain <node-to-drain> --ignore-daemonsets --force
# on worker node terminal
sudo apt-get update
sudo apt-get install -y --allow-change-held-packages kubeadm=1.21.1-00
sudo kubeadm upgrade node
sudo apt-get install -y --allow-change-held-packages kubelet=1.21.1-00 kubectl=1.21.1-00
sudo systemctl daemon-reload
sudo systemctl restart kubelet
# back to master
kubectl uncordon <node-to-uncordon>
kubectl get nodes
```

## User Management

### RBAC for User

```bash
sudo apt install openssl
kubectl create namespace development

# Generate private key and certificate request
cd ${HOME}/.kube
sudo openssl genrsa -out DevUser.key 2048
sudo openssl req -new -key DevUser.key -out DevUser.csr -subj "/CN=DevUser/O=development"
sudo openssl x509 -req -in DevUser.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out DevUser.crt -days 45

# Add user to Kubeconfig
kubectl config set-credentials DevUser --client-certificate ${HOME}/.kube/DevUser.crt --client-key ${HOME}/.kube/DevUser.key
kubectl config set-context DevUser-context --cluster=<cluster_name> --namespace=development --user=DevUser

# Test access
kubectl get pods --context=DevUser-context

# Create role and role binding
kubectl apply -f pod-reader-role.yml
kubectl apply -f pod-reader-rolebinding.yml

# Test access
kubectl get pods --context=DevUser-context
kubectl run nginx --image=nginx --context=DevUser-context
```

## Service Accounts

### Service Account Creation

Create role, service account, rolebinding, use `serviceAccountName` in pod's definition.

## Dry Run

Test command execution without actually running it.

```bash
kubectl delete -f custom-resources.yaml --dry-run=client
```

## Base64 Conversion

Convert text to base64.

```bash
echo -n '<text>' | base64
```

## Secrets and ConfigMaps

### Create Secret from File

```bash
kubectl create secret generic <secret_name> --from-file <file_of_secret>
```

### Create ConfigMap from File

```bash
kubectl create configmap <confmap_name> --from-file <conf_file>
```

## Scheduling and Labeling

### Labeling Nodes and Pods

```bash
kubectl label nodes <node_name> <key>=<value>
kubectl get nodes --show-labels
```

## Daemon Sets and Static Pods

### Daemon Sets

```bash
kubectl get daemonsets
```

### Static Pods

```bash
cd /etc/kubernetes/manifests/
# create the pod's yaml file there
# wait to deploy, or restart kublet service to deploy it faster
sudo systemctl restart kubelet
```

## Scaling and Replication Controllers/Sets

### Scaling

```bash
kubectl scale --replicas=6 <replication_(controller/set)_name>
```

## Rolling Updates in Deployments

```bash
kubectl set image deployment <deployment_name> <cont_name>=<new_image>:<new_tag> --record
kubectl set resource deployment <deployment_name> -c <cont_name> --limits='cpu=100m,memory=250Mi' --record
kubectl rollout status deployment <deployment_name>
kubectl rollout history deployment <deployment_name>
kubectl rollout undo deployment <deployment_name>
kubectl rollout undo deployment <deployment_name> --to-revision=<revision_number>
kubectl rollout pause deployment <deployment_name>
kubectl rollout resume deployment <deployment_name>
kubectl scale deployment <deployment_name> --replicas=6
```

## Network Policy

```bash
kubectl label namespace network-policy <key>=<value>
```

## Additional Commands

### Curl to Specific IP and URL

```bash
curl <ip> -H 'Host: <url>'
```

### kubectl Autocompletion

```bash
vim /root/.bashrc
# Add or uncomment:
if [ -f /etc/bash_completion ]; then
  . /etc/bash_completion
fi
```

### Watching Pods

```bash
kubectl get pod <pod_name> --watch
```

### Getting Logs

```bash
sudo journalctl -u kubelet -f
```

### Deleting Stuck Pods

```bash
kubectl get pods --all-namespaces | grep Terminating | while read line; do
  pod_name=$(echo $line | awk '{print $2}' ) \
  name_space=$(echo $line | awk '{print $1}' ); \
  kubectl delete pods $pod_name -n $name_space --grace-period=0 --force
done
```

### Connecting kubectl to EKS Cluster

```bash
# Install AWS CLI
aws configure
# Update kubeconfig for EKS cluster
aws eks --region us-east-2 update-kubeconfig --name <eks_cluster_name>
```

### etcd Backup (on Master Node)

```bash
# Install etcdctl
git clone -b v3.4.28 https://github.com/etcd-io/etcd.git
cd etcd
./build
export PATH="$PATH:`pwd`/bin"
etcd --version

# Backup
etcdctl snapshot save --endpoints=127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
/home/ubuntu/myk8scluster.db

# Restore
etcdctl snapshot restore --data-dir=/var/lib/etcd-new-data2 /home/ubuntu/myk8scluster2.db
vim /var/lib/etcd
# Change /var/lib/etcd to /var/lib/etcd-new-data2
kubectl get pods
```

Feel free to use and share this cheat sheet for your Kubernetes operations!