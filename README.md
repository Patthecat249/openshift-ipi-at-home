# openshift-ipi-at-home
This Ansible-Playbook should spin up a quick OCP-Cluster within Minutes in my own vSphere Environment

# Prerequisites
- vSphere 8.0.x
- vCenter 8.0.x
- Tag (openshift) und Kategorie (openshift) im vCenter (mit GOVC die UUID herausfinden "govc tags.info openshift")
- VM or WSL with Linux RHEL, Fedora, Rocky or CentOS
  - Tested with **Rocky Linux 9.4 (Blue Onyx)** on WSL-2.0
  - Ansible
- Make sure, that DNS-Entry on PI-DNSMASQ for these clusters are set. Use nslookup
  - *.apps.ipi.home.local  10.0.249.241
  - api.ipi.home.local 10.0.249.242
  - api-int.ipi.home.local 10.0.249.242
  
  - *.apps.patrick.home.local  10.0.249.245
  - api.patrick.home.local 10.0.249.246
  - api-int.patrick.home.local 10.0.249.246
  

```bash
# Ansible --version
ansible [core 2.14.14]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.9/site-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.9.18 (main, Oct  4 2024, 00:00:00) [GCC 11.4.1 20231218 (Red Hat 11.4.1-3)] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = True
```

# Prepare OpenShift-Installation Environment
This create a folder-structure and prepares the install-config.yaml. After this step, you can do the openshift-cluster creation.

## Clone the Github-Repo
```bash
# git clone
MYPATH=$PWD
mkdir -p $MYPATH/git && cd $MYPATH/git && git clone https://github.com/Patthecat249/openshift-ipi-at-home.git
```

## Create a ansible-vault with your vcenter-credentials
This file is needed to connect to your vcenter. Remember your password. You will paste it in a file in a few steps.
```bash
# Create a vcenter credentials file
ansible-vault create $MYPATH/git/openshift-ipi-at-home/oneclick-ocp/vars/vcenter_credentials.yaml
vcenter_username: "<vcenter-username>"
vcenter_password: "<vcenter-password>"
```

## Create Red Hat pull-secret file
This file contains your Red Hat pull-secret. Please Download from and paste it into the file:
<https://console.redhat.com/openshift/downloads>
```bash
ansible-vault create $MYPATH/git/openshift-ipi-at-home/oneclick-ocp/vars/pull-secret
# Paste your Red Hat pull-secret here
```

## Create a password-file for your ansible-vault password
This file contains your ansible-vault password to run the ansible-playbook without asking for vault-password
```bash
echo "Test1234" > $MYPATH/password.txt
```

## Run this playbook to create your install-config.yaml for your OpenShift-Installation
You can choose one of the parameters to prepare your install-config file.

### Install with Defaults from vars/main.yaml
```bash
# ansible-playbook 01-playbook.yaml --ask-vault-pass
ansible-playbook $MYPATH/git/openshift-ipi-at-home/01-playbook.yaml --vault-password-file $MYPATH/password.txt
```
### Customize Clustername
```bash
ansible-playbook $MYPATH/git/openshift-ipi-at-home/01-playbook.yaml --vault-password-file $MYPATH/password.txt -e "openshift_clustername=ipi"
```

### Customize Clustername and OpenShift-Version
```bash
ansible-playbook $MYPATH/git/openshift-ipi-at-home/01-playbook.yaml --vault-password-file $MYPATH/password.txt -e "openshift_clustername=patrick" -e "openshift_version=4.16.20"
```

### Customize Clustername, OpenShift-Version and Worker-Node Sizing and API+Inress-VIP
```bash
ansible-playbook $MYPATH/git/openshift-ipi-at-home/01-playbook.yaml --vault-password-file $MYPATH/password.txt -e "openshift_clustername=patrick" -e "openshift_version=4.16.20" -e "worker_node_count=4" -e "worker_cpu=8" -e "worker_memory=16384" -e "worker_disksize=200" -e "openshift_api_vip=10.0.249.245" -e "openshift_ingress_vip=10.0.249.246"
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