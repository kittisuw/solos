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
### 5.Test connection by ansible
```
ansible --list-host all
ansible -m ping all
```
### 6.Edit file site.yml
```
---
# Defines deployment design and assigns role to server groups
 
- hosts:
  - mons
  - osds
 
  gather_facts: false
  any_errors_fatal: true
  become: true
 
  tags: always
 
  vars:
    delegate_facts_host: True
 
  pre_tasks:
    # If we can't get python2 installed before any module is used we will fail
    # so just try what we can to get it installed
 
    - import_tasks: raw_install_python.yml
 
    - name: gather facts
      setup:
      when: not delegate_facts_host | bool
 
    - name: gather and delegate facts
      setup:
      delegate_to: "{{ item }}"
      delegate_facts: True
      with_items: "{{ groups['all'] }}"
      run_once: true
      when: delegate_facts_host | bool
 
    - name: install required packages for fedora &gt; 23
      raw: sudo dnf -y install python2-dnf libselinux-python ntp
      register: result
      when:
        - ansible_distribution == 'Fedora'
        - ansible_distribution_major_version|int &gt;= 23
      until: result is succeeded
 
    - name: check if it is atomic host
      stat:
        path: /run/ostree-booted
      register: stat_ostree
      tags: always
 
    - name: set_fact is_atomic
      set_fact:
        is_atomic: '{{ stat_ostree.stat.exists }}'
      tags: always
 
  tasks:
    - import_role:
        name: ceph-defaults
    - import_role:
        name: ceph-facts
    - import_role:
        name: ceph-validate
    - import_role:
        name: ceph-infra
 
- hosts: mons
  gather_facts: false
  become: True
  any_errors_fatal: true
  pre_tasks:
    - name: set ceph monitor install 'In Progress'
      run_once: true
      set_stats:
        data:
          installer_phase_ceph_mon:
            status: "In Progress"
            start: "{{ lookup('pipe', 'date +%Y%m%d%H%M%SZ') }}"
 
  tasks:
    - import_role:
        name: ceph-defaults
      tags: ['ceph_update_config']
    - import_role:
        name: ceph-facts
      tags: ['ceph_update_config']
    - import_role:
        name: ceph-handler
    - import_role:
        name: ceph-common
    - import_role:
        name: ceph-config
      tags: ['ceph_update_config']
    - import_role:
        name: ceph-mon
    - import_role:
        name: ceph-mgr
      when: groups.get(mgr_group_name, []) | length == 0
 
  post_tasks:
    - name: set ceph monitor install 'Complete'
      run_once: true
      set_stats:
        data:
          installer_phase_ceph_mon:
            status: "Complete"
            end: "{{ lookup('pipe', 'date +%Y%m%d%H%M%SZ') }}"
 
- hosts: mgrs
  gather_facts: false
  become: True
  any_errors_fatal: true
  pre_tasks:
    - name: set ceph manager install 'In Progress'
      run_once: true
      set_stats:
        data:
          installer_phase_ceph_mgr:
            status: "In Progress"
            start: "{{ lookup('pipe', 'date +%Y%m%d%H%M%SZ') }}"
 
  tasks:
    - import_role:
        name: ceph-defaults
      tags: ['ceph_update_config']
    - import_role:
        name: ceph-facts
      tags: ['ceph_update_config']
    - import_role:
        name: ceph-handler
    - import_role:
        name: ceph-common
    - import_role:
        name: ceph-config
      tags: ['ceph_update_config']
    - import_role:
        name: ceph-mgr
 
  post_tasks:
    - name: set ceph manager install 'Complete'
      run_once: true
      set_stats:
        data:
          installer_phase_ceph_mgr:
            status: "Complete"
            end: "{{ lookup('pipe', 'date +%Y%m%d%H%M%SZ') }}"
 
- hosts: osds
  gather_facts: false
  become: True
  any_errors_fatal: true
  pre_tasks:
    - name: set ceph osd install 'In Progress'
      run_once: true
      set_stats:
        data:
          installer_phase_ceph_osd:
            status: "In Progress"
            start: "{{ lookup('pipe', 'date +%Y%m%d%H%M%SZ') }}"
 
  tasks:
    - import_role:
        name: ceph-defaults
      tags: ['ceph_update_config']
    - import_role:
        name: ceph-facts
      tags: ['ceph_update_config']
    - import_role:
        name: ceph-handler
    - import_role:
        name: ceph-common
    - import_role:
        name: ceph-config
      tags: ['ceph_update_config']
    - import_role:
        name: ceph-osd
 
  post_tasks:
    - name: set ceph osd install 'Complete'
      run_once: true
      set_stats:
        data:
          installer_phase_ceph_osd:
            status: "Complete"
            end: "{{ lookup('pipe', 'date +%Y%m%d%H%M%SZ') }}"
 
- hosts: mons
  gather_facts: false
  become: True
  any_errors_fatal: true
  tasks:
    - import_role:
        name: ceph-defaults
    - name: get ceph status from the first monitor
      command: ceph --cluster {{ cluster }} -s
      register: ceph_status
      changed_when: false
      delegate_to: "{{ groups[mon_group_name][0] }}"
      run_once: true
 
    - name: "show ceph status for cluster {{ cluster }}"
      debug:
        msg: "{{ ceph_status.stdout_lines }}"
      delegate_to: "{{ groups[mon_group_name][0] }}"
      run_once: true
      when: not ceph_status.failed
 
- import_playbook: infrastructure-playbooks/dashboard.yml
  when:
    - dashboard_enabled | bool
    - groups.get(grafana_server_group_name, []) | length &gt; 0
    - ansible_os_family in ['RedHat', 'Suse']
```

### 7.Edit file site.yml group_vars/all.yml for internal network
```
ceph_origin: repository
ceph_repository: community
ceph_stable_release: nautilus
monitor_interface: eth0
journal_size: 5120
#public_network: 0.0.0.0/0 - leave commented
cluster_network: 10.0.1.0/24 #specify the network for internal traffic
```

### 7.Edit file group_vars/osds.yml for internal network




### end 
ansible-playbook playbook.yml


#### Ref: https://kruschecompany.com/ceph-ansible/
#### Ref2: https://www.server-world.info/en/note?os=CentOS_7&p=ceph14&f=1

