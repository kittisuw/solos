# Install and config ceph-ansible centos7
### 1.Install git,virtualenv and add user ansible
```
rpm -qa |grep python3
sudo yum install git
pip3 install virtualenv
useradd -m -d /home/ansible -s /bin/bash -c "ansible" ansible
```
### 2.Clone repo ceph-ansible 
```
su -i -u ansible
git clone https://github.com/ceph/ceph-ansible.git
virtualenv .ceph-ansible
source .ceph-ansible/bin/activate
cd ceph-ansible
pip install -r requirements.txt
cp site.yml.sample site.yml
cp group_vars/all.yml.sample group_vars/all.yml
cp group_vars/mons.yml.sample group_vars/mons.yml
cp group_vars/osds.yml.sample group_vars/osds.yml
```
### 3. Create hosts file
```
sudu -i -u ubuntu
sudo -i -u root
vi /etc/ansible/hosts

[mons]
10.233.254.11
10.233.254.12
10.233.254.13

[osds]
10.233.254.11
10.233.254.12
10.233.254.13

[mgrs]
10.233.254.11

[grafana-server]
192.168.2.11
```

### 4.Exchange key between ansible and velonica (user ubuntu)
```
su -i -u ansible
cd .ssh
ssh-keygen -b 4096
ssh-copy-id ubuntu@10.233.254.11
ssh-copy-id ubuntu@10.233.254.12
ssh-copy-id ubuntu@10.233.254.13
```
Test connect without password
```
ssh ubuntu@10.233.254.11
ssh ubuntu@10.233.254.12
ssh ubuntu@10.233.254.13
```
### 4.Test connection by ansible
```
ansible --list-host all
ansible -m ping all
```





### end 
ansible-playbook -i hosts playbook.yml


#### Ref: https://kruschecompany.com/ceph-ansible/


