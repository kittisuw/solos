#Install and config ceph-ansible centos7
##How to
1.Install git,virtualenv and add user ansible

rpm -qa |grep python3
sudo yum install git
pip3 install virtualenv
useradd -m -d /home/ansible -s /bin/bash -c "ansible" ansible
----
2.Exchange key between ansible and velonica (user ubuntu)


----
3. add host and test

vi /etc/ansible/hosts
ansible --list-host all
ansible -m ping all



end. 
ansible-playbook -i hosts playbook.yml