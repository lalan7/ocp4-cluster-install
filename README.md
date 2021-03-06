# OCP4 AWS (IPI) Installation Playbooks

### Bastion Provisioning

Run these commands on a RHEL8 AMI to setup a new bastion host that will be used as a base of operations:

```
ls /etc/yum.repos.d/redhat-rhui*.repo | xargs -I{} sudo mv {} {}.disabled # Disable RHUI
sudo subscription-manager register # Red Hat Developer/RHEL Subscription Required
sudo subscription-manager attach --pool=YOURPOOL
sudo subscription-manager repos --enable=rhel-8-for-x86_64-baseos-rpms \
  --enable=rhel-8-for-x86_64-appstream-rpms --enable=ansible-2.8-for-rhel-8-x86_64-rpms 

sudo yum -y install git ansible wget unzip vim tmux
sudo ln -nfs /usr/share/zoneinfo/Australia/Sydney  /etc/localtime # Or whatever timezone you are in
sudo yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum -y install python3-kubernetes python3-openshift python3-psutil.x86_64 yamllint platform-python-pip
pip3 install ansible-lint

# EPEL should be kept off by default to avoid a 'yum update' operation jumping to EPEL's newer version of Ansible RPM,
# which is not covered by Red Hat support. Turn this on (and subsequently off) for EPEL package installs.
sed -i 's/enabled=1/enabled=0/g' /etc/yum.repos.d/epel.repo # Disable epel
git config --global user.name $(hostname)
```

### Setup Deploy Key

```
echo '<deploy-key>' > ~/.ssh/id_rsa && chmod 400 ~/.ssh/id_rsa
```

### Install Repository
```
git clone origin git@github.com:mikishapiro/ocp4-cluster-install.git

```

### Ansible Vault
Ensure your group_vars (containing sensitive stuff) are set correctly.
```
ansible-vault edit --vault-id=YOURVAULTPASSWORDFILE group_vars/nonprod.yml
```

### Deploying the cluster
```
ansible-playbook -i <inventory> deploy.yml -e var_cluster_instruction=create 
```
#### Example
```
ansible-playbook -i inventory_nonprod deploy.yml -e var_cluster_instruction=create 
```

You can also use 'var_cluster_instruction=modify' instead of var_cluster_instruction=create to re-run the post-installer roles or 'var_cluster_instruction=destroy' to destroy the cluster (requires the post-build artfecacts)
