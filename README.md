# Installing K8s Cluster on Ubuntu 20.04 in 9 Easy Steps

Deploy a K8s cluster with 1 control & 2 worker nodes with KubeVirt Hyper Converged Operator (HCO), Flannel, Multus, and Linux Bridge CNIs complete with CDI, Ingress NGINX, Kubernetes Dashboard, and OKD Web Console.

Installation steps:

## 1. Prepare 4x Ubuntu 20.04 servers:

- 1x NFS server (min 1 CPU, 2 GB RAM)
- 1x control (min 2 CPUs, 4 GB RAM)
- 2x workers (min 2 CPUs, 4 GB RAM, )

## 2. Assign static IP addresses to all 4 servers, such as the following:

- nfs - 172.31.254.9/24
- control - 172.31.254.1/24
- worker1 - 172.31.254.11/24
- worker2 - 172.31.254.12/24

## 3. Install NFS server on the NFS server node:

Replace `{password}` with your sudo password.

```
echo -e "{password}" | sudo -S su
wget --no-cache https://raw.githubusercontent.com/rkrisman/k8s-1c2w-kubevirt-hco/main/00-nfs-server-install
chmod +x 00-nfs-server-install
./00-nfs-server-install -h nfs -i {nfs-node-ip} -p {nfs-share} -n {allowed-cidr}
exit
```

Example:
```
echo -e "{password}" | sudo -S su
wget --no-cache https://raw.githubusercontent.com/rkrisman/k8s-1c2w-kubevirt-hco/main/00-nfs-server-install
chmod +x 00-nfs-server-install
./00-nfs-server-install -h nfs -i 172.31.254.9 -p /data/nfs1 -n 172.31.254.0/24
exit
```

## 4. Install K8s cluster on the Control node:

Replace `{password}` with your sudo password.

```
echo -e "{password}" | sudo -S su
wget --no-cache https://raw.githubusercontent.com/rkrisman/k8s-1c2w-kubevirt-hco/main/01-control-cluster-install
chmod +x 01-control-cluster-install
./01-control-cluster-install -h control -i 172.31.254.1 -r 1.25.3 -n 172.30.0.0/16
exit
```

Example:
```
echo -e "{password}" | sudo -S su
wget --no-cache https://raw.githubusercontent.com/rkrisman/k8s-1c2w-kubevirt-hco/main/01-control-cluster-install
chmod +x 01-control-cluster-install
./01-control-cluster-install -h control -i 172.31.254.1 -r 1.25.3 -n 172.30.0.0/16
exit
```

## 5. Install K8s on the Worker nodes:

Replace `{password}` with your sudo password.

```
echo -e "{password}" | sudo -S su
wget --no-cache https://raw.githubusercontent.com/rkrisman/k8s-1c2w-kubevirt-hco/main/02-worker-node-install
chmod +x 02-worker-node-install
./02-worker-node-install -h {worker-node-hostname} -i {worker-node-ip} -r {k8s-release} -s {nfs-server-ip}
exit
```

worker1 example:
```
echo -e "{password}" | sudo -S su
wget --no-cache https://raw.githubusercontent.com/rkrisman/k8s-1c2w-kubevirt-hco/main/02-worker-node-install
chmod +x 02-worker-node-install
./02-worker-node-install -h worker1 -i 172.31.254.11 -r 1.25.3 -s 172.31.254.9
exit
```

worker2 example:
```
echo -e "{password}" | sudo -S su
wget --no-cache https://raw.githubusercontent.com/rkrisman/k8s-1c2w-kubevirt-hco/main/02-worker-node-install
chmod +x 02-worker-node-install
./02-worker-node-install -h worker2 -i 172.31.254.12 -r 1.25.3 -s 172.31.254.9
exit
```

## 6. Join K8s workers to cluster:

Look for the output at the end of the K8s cluster node installation for an instruction to join the cluster. Ensure to run the command using sudo.

```
sudo kubeadm join {control-node-ip}:6443 --token xxxxx \
    --discovery-token-ca-cert-hash xxxxx
```

Example:
```
sudo kubeadm join 172.31.254.1:6443 --token e9jsaq.m6ctbxe0gznirlpf \
    --discovery-token-ca-cert-hash sha256:754ea651bbb2cdd0a5a3639d700a4af0f3418f8f3c1804156fd0573b9338eedc
```

## 7. Install KubeVirt Hyperper Converged Operator (HCO) Addons on the Control node:

Replace `{password}` with your sudo password.

```
echo -e "{password}" | sudo -S su
wget --no-cache https://raw.githubusercontent.com/rkrisman/k8s-1c2w-kubevirt-hco/main/03-hco-addons-install
chmod +x 03-hco-addons-install
./03-hco-addons-install -s {nfs-node-ip} -p {nfs-share} -i {metallb-ippool-range}
```

Example:
```
echo -e "{password}" | sudo -S su
wget --no-cache https://raw.githubusercontent.com/rkrisman/k8s-1c2w-kubevirt-hco/main/03-hco-addons-install
chmod +x 03-hco-addons-install
./03-hco-addons-install -s 172.31.254.9 -p /data/nfs1 -i 172.31.254.201-172.31.254.249
```

## 8. Configure Linux bridges for ingress, HA, and egress ports on the Control node:

Replace `{password}` with your sudo password.
```
echo -e "{password}" | sudo -S su
wget --no-cache https://raw.githubusercontent.com/rkrisman/k8s-1c2w-kubevirt-hco/main/04-linux-bridges-install
chmod +x 04-linux-bridges-install
./04-linux-bridges-install -a {worker1-hostname} -b {worker2-hostname} -i {ingress-port} -h {ha-port} -e {egress-port}
```

Example:
```
echo -e "{password}" | sudo -S su
wget --no-cache https://raw.githubusercontent.com/rkrisman/k8s-1c2w-kubevirt-hco/main/04-linux-bridges-install
chmod +x 04-linux-bridges-install
./04-linux-bridges-install -a worker1 -b worker2 -i ens10 -h ens11 -e ens12
```

## 9. Configure static host mapping for the K8s Dashboard and OKD Web Console:

Edit your local host file (/etc/hosts in Linux or C:\Windows\System32\drivers\etc\hosts in Windows) to include the following entry:

```
{control-node-ip} control.k8s.lab dashboard.k8s.lab okd.k8s.lab
```

Example:
```
172.31.254.1 control.k8s.lab dashboard.k8s.lab okd.k8s.lab
```

Then, access the K8s Dashboard by browsing using HTTPS on port 30443, [https://dashboard.k8s.lab:30443](https://dashboard.k8s.lab:30443)
Access the OKD Web Console by browsing using HTTPS pon port 30036, [http://okd.lab.k8s:30036](http://okd.lab.k8s:30036)
