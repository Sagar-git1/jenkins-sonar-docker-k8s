# Ansible-Jenkins-Sonar-Docker-k8s

## _This project will use java code_

First step is to create a jenkins master and jenkins agent using ansible

## _Ansible setup_

- First create a ubuntu ec2 instance and install ansible in it.

```
   sudo apt-add-repository ppa:ansible/ansible
   sudo apt update
   sudo apt install ansible
```

- Now create a new folder and add hosts and ansible.cfg files in it(this will override the
  ansible.cfg and hosts file under /root/ansible folder which comes as part of installation).

#### ansible.cfg file

```
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
```

#### hosts

```
[jenkins_master]
44.212.69.249
[jenkins_slave]
44.203.252.101
[jenkins_slave:vars]
agent=44.203.252.101
jenkinsmaster=44.212.69.249
```

Once the ansible setup is done create playbooks for making one server as Jenkins controller and one server as jenkins agent

#### jenkins-master-install.yaml

```yaml
- name: Install Jenkins on Ubuntu 20.04
  hosts: jenkins_master

  tasks:
    - name: Update apt package index
      apt:
        update_cache: yes

    - name: Install Java (Jenkins requirement)
      apt:
        name: openjdk-17-jdk
        state: present

    - name: Add Jenkins Debian repository key to apt
      apt_key:
        url: https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
        state: present

    - name: Add Jenkins repository to apt sources
      apt_repository:
        repo: deb https://pkg.jenkins.io/debian-stable binary/
        state: present

    - name: Update apt cache after adding Jenkins repo
      apt:
        update_cache: yes

    - name: Install Jenkins
      apt:
        name: jenkins
        state: present

    - name: Start and enable Jenkins service
      systemd:
        name: jenkins
        enabled: yes
        state: started

    - name: Open port 8080 for Jenkins
      ufw:
        rule: allow
        port: "8080"
        proto: tcp
```

The above playbook is used to install jenkins and required dependencies onto server which we decided to make it as jenkins master  
And the below playbook is used to make a server as jenkins agent

#### jenkins-agent.yaml

```yaml
- name: Install and configure Jenkins Agents
  hosts: jenkins_slave
  tasks:
    - name: Update apt package index
      apt:
        update_cache: yes

    - name: Install Java
      apt:
        name: openjdk-17-jdk
        state: present

    - name: Download Jenkins Agent JAR
      get_url:
        url: http://{{ jenkinsmaster }}:8080/jnlpJars/agent.jar
        dest: /home/ubuntu/agent.jar
        mode: 0755

    - name: Create jenkins agent service file from templates
      template:
        src: jenkins-agent.service.j2
        dest: /etc/systemd/system/jenkins-agent.service

    - name: Reload systemd to recoginze the new service
      systemd:
        daemon_reload: yes

    - name: Enable and start jenkins agnt service
      systemd:
        name: jenkins-agent
        enabled: yes
        state: started
```
