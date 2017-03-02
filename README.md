1.
Two ways we can achive this:
	1. Call Ansible playbook, which invoke Heat and Cloud-init(cloud-init will have userdata to configure LDAP)
	ansible -i inventory playbook1.yml
	
	2. Call 2 ansibel playbooks. one to deploy and other to configure LDAP 
==============================================================================================
cat playbook1.yml
- hosts: localhost
  sudo: true
  gather_facts: no
  tasks:
  - name: create stack
    ignore_errors: True
    register: stack_create
    os_stack:
     auth:
       auth_url: http://XXXXXXX:5000/v2.0
       username: user1
       password: xxxxx123
       project_name: qwerty
     name: ansible_stack
     state: present
     template: "/root/heat/techops.yml"
     environment:
     - /root/heat/techops.env 
=================================================================================================
cat techops.yml 
heat_template_version: 2015-04-30
description: Heat to deploy VM
  
parameters:
  image:
    type: string
    label: Image name or ID
    description: Image to be used for compute instance
    default: d94455cf-a261-476a-8130-1d564e8c5041
  flavor:
    type: string
    label: Flavor
    description: Type of instance (flavor) to be used 
    default: m1.small

  private_network:
    type: string
    label: Private network name or ID
    description: Network to attach instance to.
    default: 231121c7-e9ec-4735-9db6-7676f0527e86

  public_network:
    type: string
    label: Private network name or ID
    description: Network to attach instance to.
    default: 231121c7-e9ec-4735-9db6-7676f0527e86	

resources:
  vm1:
    type: OS::Nova::Server
    properties:
      image: { get_param: image }
      flavor: { get_param: flavor }
      networks:
        - network: { get_param: private_network }
      config_drive: True
      user_data:
        get_file: cloudinit.txt
      user_data_format: RAW
	  
server1_floating_ip:
    type: OS::Neutron::FloatingIP
    properties:
      floating_network: { get_param: public_net }
      port_id: { get_param: private_network }	  
=================================================================================================	  
cat cloudinit.txt
#!/bin/bash 
runcmd:
- yum install *openldap* -y
- chkconfig --levels 235 ldap on
- service ldap start

=============================================================================
2.This can be achived using cloud-init or ansible 
ansible -i inventory2 playbook1.yml
====================================================================
cat inventory2
[remote]
centos-openclient.localdomain ansible_ssh_host=<floating ip of host1> ansible_connection=ssh ansible_ssh_user=root ansible_ssh_pass=root 
	 
===============================================================================
cat solution2.yml 
- hosts: <floatingIP>
  sudo: true
  gather_facts: no
  tasks:
  - name: add group
    shell: groupadd techops_dba
	
  - name: Adding sudoers
    shell: echo "%techops_dba ALL=(ALL)   ALL " >> /etc/sudoers/
  
  - name: Adding to access.conf
    shell: echo "+ : @techops_dba user : ALL" >> /etc/security/access.conf

========================================
3. 
cat solution3.yml 
- hosts: <floatingIP>
  sudo: true
  gather_facts: no
  tasks:
  - name: Install  NTP
    shell: yum install -y ntp
	
  - name: add the network range you allow to receive requests
    shell: echo "restrict 10.0.0.0 mask 255.255.255.0 nomodify notrap" >> /etc/ntp.conf 

  - name: configure ntp
    shell: echo "server ntp1.jst.mfeed.ad.jp iburst" >> /etc/ntp.conf
	
========================================
4. 	Execute these commands in host 2 
    Ip tables rules: Set these rules in host 2
	sudo iptables -A INPUT -p tcp -s 15.15.15.0/24 --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
    sudo iptables -A OUTPUT -p tcp --sport 22 -m conntrack --ctstate ESTABLISHED -j ACCEPT
=====================================================	
5. ansible -i inventory2 playbook1.yml
cat solution2.yml 
- hosts: floatingIP
  sudo: true
  gather_facts: no
  tasks:
  - name: 1-Verify LDAPs
    shell: ldapsearch -x -LLL -h host.example.com -D user -w password -b"dc=ad,dc=example,dc=com" -s sub "(objectClass=user)" givenName
  
  - name: 2-verify /etc/sudoers file 
    shell: cat /etc/sudoers | grep techops_dba
    register: sudoers_file
  - debug: var=sudoers_file 
  
  - name: 2-verify /etc/security/access.conf
    shell: cat /etc/security/access.conf
	register: access_conf
  - debug: var=access_conf
  
  - name: 3-get NTP startum value 
    shell: ntpdc -c sysinfo 
	register: result
  - debug: var=result 
   
  - name: 4-ping
    shell: ping -c 10 xxxxxx (second host IP)
    register: ping 
  - debug: var=ping 
	
===============================================================	
6. From git bash:
git clone ****
git init .  (git init /path_to_dir)
ls -al
git status           
git add .  
git commit -m "this will add files into repo"
git push origin master
========================================================================
