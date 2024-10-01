# Ansible-Jenkins-Sonar-Docker-k8s
## _This project will use java code_

First step is to create a jenkins master and jenkins agent using ansible
## _Ansible setup_
- First create a ubuntu ec2 instance and install ansible in it.
`
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible
`
- Now create a new folder and add hosts and ansible.cfg files in it(this will override the 
ansible.cfg and hosts file under /root/ansible folder which comes as part of installation).
[defaults]
remote_user=ubuntu
private_key_file=./newone.pem
inventory=./hosts
host_key_checking=False
[privilege_escalation]
become=True
become_method=sudo
become_ask_pass=False
become_user=root
