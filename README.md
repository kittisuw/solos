# Install and config ceph-ansible centos7
### 1.Install git,virtualenv and add user ansible
```
rpm -qa |grep python3
sudo yum install git
pip3 install virtualenv
useradd -m -d /home/ansible -s /bin/bash -c "ansible" ansible
```
### 2.Exchange key between ansible and velonica (user ubuntu)
```
su -i -u ansible
cd .ssh
ssh-keygen -b 4096
ssh-copy-id ubuntu@10.233.254.11
ssh-copy-id ubuntu@10.233.254.12
ssh-copy-id ubuntu@10.233.254.13
```
### 3. add host and test
```
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
192.168.2.1
```
```
ansible --list-host all
ansible -m ping all
```

### end 
ansible-playbook -i hosts playbook.yml