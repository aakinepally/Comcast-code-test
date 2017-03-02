# Comcast-code-test
Assumptions:
1. Jumpbox with openstack-clients and ansible installed.
2. Tenant RC file is already created and sourced in Jumpbox.
3. Tenant space has image, private and public networks, router created, interfaces added, security rules added.

Commands:
ansible-playbook -i inventory playbook1.yml
ansible-playbook -i inventory1 solution2.yml
ansible-playbook -i inventory1 solution3.yml
ansible-playbook -i inventory2 solution4.yml
ansible-playbook -i inventory1 solution5.yml


