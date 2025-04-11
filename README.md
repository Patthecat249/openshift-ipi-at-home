# openshift-ipi-at-home
This Ansible-Playbook should spin up a quick OCP-Cluster within Minutes in my own vSphere Environment

# Workflow
This is the common workflow to deploy an OpenShift-Cluster in my vSphere-Environment

- build the container
- run and exec into the container
- execute the ansible-playbook

## Prerequisites
This screnario is tested with:
- vSphere 8.0.x
- vCenter 8.0.x
- VMware
  - Tag (openshift) und Categorie (openshift) in vCenter (find out the uuid of the tags with GOVC: `govc tags.info openshift`)
- podman
- DNS-Entries must set up (look at the troubleshooting section)

## Getting started

### Clone the Github-Repo

```bash
MYPATH=$PWD
# Create a folder on your host, where the install-files are copied to
# this folder will contain the kubeadmin-password and kubeconfig file
# from your ocp installation and git clone
mkdir -p $MYPATH/git $MYPATH/ocp && cd $MYPATH/git && git clone https://github.com/Patthecat249/openshift-ipi-at-home.git
```

### Build the container
By building the container-image, all necessary tools will be downloaded and installed,
which are needed to install openshift in that version

```bash
# Install podman
dnf install -y podman

# Build the container
export OPENSHIFT_VERSION=4.16.24
podman build -t ipi-installer:${OPENSHIFT_VERSION} -f $MYPATH/git/openshift-ipi-at-home/containerfile/containerfile --build-arg OPENSHIFT_VERSION=${OPENSHIFT_VERSION}
```

### Run the Container and execute the Ansible-Playbook

```bash
# Build the container
export OPENSHIFT_VERSION=4.16.24
podman run --rm -it -v $MYPATH/ocp:/root/ocp/pod-to-host-mount localhost/ipi-installer:${OPENSHIFT_VERSION} bash
```


## Prepare OpenShift-Installation Environment
This create a folder-structure and prepares the install-config.yaml. After this step, you can do the openshift-cluster creation.

### Create a password-file for your ansible-vault password
This file contains your ansible-vault password to run the ansible-playbook without asking for vault-password
```bash
echo "Test1234" > /root/ocp/pod-to-host-mount/password.txt
```

## Create a ansible-vault with your vcenter-credentials
This file is needed to connect to your vcenter. Remember your password. You will paste it in a file in a few steps.
```bash
# Create a vcenter credentials file
ansible-vault create /root/ocp/pod-to-host-mount/vcenter_credentials.yaml --vault-password-file /root/ocp/pod-to-host-mount/password.txt
vcenter_username: "<vcenter-username>"
vcenter_password: "<vcenter-password>"
```

## Create Red Hat pull-secret file
This file contains your Red Hat pull-secret. Please Download from and paste it into the file:
<https://console.redhat.com/openshift/downloads>
```bash
ansible-vault create /root/ocp/pod-to-host-mount/pull-secret --vault-password-file /root/ocp/pod-to-host-mount/password.txt
# Paste your Red Hat pull-secret here
```


## Run this playbook to create your install-config.yaml for your OpenShift-Installation
You can choose one of the parameters to prepare your install-config file.

### EXAMPLE: Install with Defaults from vars/main.yaml
```bash
# ansible-playbook 01-playbook.yaml --ask-vault-pass
ansible-playbook /root/ocp/git/openshift-ipi-at-home/01-playbook.yaml --vault-password-file /root/ocp/pod-to-host-mount/password.txt
```
### EXAMPLE: Customize Clustername
```bash
ansible-playbook $MYPATH/git/openshift-ipi-at-home/01-playbook.yaml --vault-password-file $MYPATH/password.txt -e "openshift_clustername=ipi"
```

### EXAMPLE: Customize Clustername and OpenShift-Version
```bash
ansible-playbook $MYPATH/git/openshift-ipi-at-home/01-playbook.yaml --vault-password-file $MYPATH/password.txt -e "openshift_clustername=patrick" -e "openshift_version=4.16.20"
```

### EXAMPLE: Customize Clustername, OpenShift-Version and Worker-Node Sizing and API+Inress-VIP
```bash
ansible-playbook $MYPATH/git/openshift-ipi-at-home/01-playbook.yaml --vault-password-file $MYPATH/password.txt -e "openshift_clustername=patrick" -e "openshift_version=4.16.20" -e "worker_node_count=4" -e "worker_cpu=8" -e "worker_memory=16384" -e "worker_disksize=200" -e "openshift_api_vip=172.16.249.245" -e "openshift_ingress_vip=172.16.249.246"
```

# Install OpenShift
Now it's time install your OpenShift-Cluster!!!

`openshift-install create cluster --dir /root/ocp/openshift-installations/<openshift-clustername>/`

## Install the Cluster ipi.home.local
```bash
openshift-install create cluster --dir /root/ocp/openshift-installations/ipi/
```
## Install the Cluster patrick.home.local
```bash
openshift-install create cluster --dir /root/ocp/openshift-installations/patrick/
```

# Troubleshooting - Section
## Approve CSR
Sometimes the OpenShift-Installation Routine does not approve the Certificate-Signing-Requests for the worker-nodes.
```bash
# Approve CSR
oc adm certificate approve $(oc get csr | grep -i pending | awk '{print $1}')
```

## DNS
The following DNS-Entries are set in the /etc/dnsmasq.d/dns.conf

```bash
# Cluster IPI
*.apps.ipi.home.local  172.16.249.241
api.ipi.home.local 172.16.249.242
api-int.ipi.home.local 172.16.249.242

# Cluster PATRICK
*.apps.patrick.home.local  172.16.249.245
api.patrick.home.local 172.16.249.246
api-int.patrick.home.local 172.16.249.246
```
