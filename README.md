# openshift-ipi-at-home
This Ansible-Playbook should spin up a quick OCP-Cluster within Minutes in my own vSphere Environment

# Prerequisites
- vSphere 8.0.x
- vCenter 8.0.x
- Tag (openshift) und Kategorie (openshift) im vCenter (mit GOVC die UUID herausfinden "govc tags.info openshift")
- VM or WSL with Linux RHEL, Fedora, Rocky or CentOS
  - Tested with **Rocky Linux 9.4 (Blue Onyx)** on WSL-2.0
  - Ansible
- DNS-Entry for:
  - api.ipi.home.local
  - api-int.ipi.home.local
  - *.apps.ipi.home.local

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
```bash
# git clone
MYPATH=$PWD
mkdir $MYPATH/git && cd $MYPATH/git && git clone https://github.com/Patthecat249/openshift-ipi-at-home.git

# Create a vcenter credentials file
ansible-vault create $MYPATH/git/openshift-ipi-at-home/oneclick-ocp/vars/vcenter_credentials.yaml

# Create Red Hat pull-secret file
ansible-vault create $MYPATH/git/openshift-ipi-at-home/oneclick-ocp/vars/pull-secret

# Run this playbook
# ansible-playbook 01-playbook.yaml --ask-vault-pass
ansible-playbook 01-playbook.yaml --vault-password-file password.txt

# Customize Clustername
ansible-playbook 01-playbook.yaml --vault-password-file password.txt -e "openshift_clustername=ipi"

# Customize Clustername and OpenShift-Version
ansible-playbook 01-playbook.yaml --vault-password-file password.txt -e "openshift_clustername=patrick" -e "openshift_version=4.16.20"

# Customize Clustername, OpenShift-Version and Worker-Node Sizing and API+Inress-VIP
ansible-playbook 01-playbook.yaml --vault-password-file password.txt -e "openshift_clustername=patrick" -e "openshift_version=4.16.20" -e "worker_node_count=4" -e "worker_cpu=8" -e "worker_memory=16384" -e "worker_disksize=200" -e "openshift_api_vip=10.0.249.245" -e "openshift_ingress_vip=10.0.249.246"
```
# Install OpenShift
Now it's time install your OpenShift-Cluster!!!

```bash
# openshift-install create cluster --dir /root/ocp/openshift-installations/<openshift-clustername>/

# For example
openshift-install create cluster --dir /root/ocp/openshift-installations/ipi/
openshift-install create cluster --dir /root/ocp/openshift-installations/patrick/
```

# Approve CSR
```bash
# Approve CSR
oc adm certificate approve $(oc get csr | grep -i pending | awk '{print $1}')
```