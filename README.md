![](images/diagram.png)


# Benefits of ansible
- Efficient: Because you don’t need to install any extra software, there’s more room for application resources on your server.
- Agentless: You don’t need to install any other software or firewall ports on the client systems you want to automate. You also don’t have to set up a separate management structure.
- Flexible: You can orchestrate the entire application environment no matter where it’s deployed. You can also customize it based on your needs.
- Powerful: Ansible lets you model even highly complex IT workflows. 

# Ansible controller and agent nodes set up guide
- Clone this repo and run `vagrant up`
- `(double check syntax/intendation)`

## We will use 18.04 ubuntu for ansible controller and agent nodes set up 
### Please ensure to refer back to your vagrant documentation

- **You may need to reinstall plugins or dependencies required depending on the OS you are using.**

```vagrant 
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what

# MULTI SERVER/VMs environment 
#
Vagrant.configure("2") do |config|

# creating first VM called web  
  config.vm.define "web" do |web|
    
    web.vm.box = "bento/ubuntu-18.04"
   # downloading ubuntu 18.04 image

    web.vm.hostname = 'web'
    # assigning host name to the VM
    
    web.vm.network :private_network, ip: "192.168.33.10"
    #   assigning private IP
    
    config.hostsupdater.aliases = ["development.web"]
    # creating a link called development.web so we can access web page with this link instread of an IP   
        
  end
  
# creating second VM called db
  config.vm.define "db" do |db|
    
    db.vm.box = "bento/ubuntu-18.04"
    
    db.vm.hostname = 'db'
    
    db.vm.network :private_network, ip: "192.168.33.11"
    
    config.hostsupdater.aliases = ["development.db"]     
  end

 # creating are Ansible controller
  config.vm.define "controller" do |controller|
    
    controller.vm.box = "bento/ubuntu-18.04"
    
    controller.vm.hostname = 'controller'
    
    controller.vm.network :private_network, ip: "192.168.33.12"
    
    config.hostsupdater.aliases = ["development.controller"] 
    
  end

end
```
## Adding machines to the hosts
- navigate to `cd /etc/ansible`
- edit the hosts file using `sudo nano hosts`
- add the IPs of the machines to the controller with the type of connection username and password as the parameters `192.168.33.10 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=`
### Running commands from the controller
- to execute `uname` in all the machines confirgured in the hosts file is `ansible all -a "uname -a"`
- to execute the same command on a specific machine such as the web it is `ansible web -a "uname -a"`
Here

```
-m – represents the module

-a – additional parameter to the reboot module to set the timeout to 3600 seconds

-u – remote SSH user

-i – inventory file

-b – to instruct ansible to become root user before executing the task

Here is the execution output of this ad hoc command. we have three commands in this screenshot, First is to check the status and second to reboot and third one is to check the uptime
```
## Ansible playbook
- Written in yaml
- Instructions to talk to servers
- Multiple playbooks for multiple servers
- What is IAC: No manual handling, run 1 command and everything is done for us
- Playbook are list of tasks that allow us to automate tasks enforces IAC

```
# This is a playbook to install and set up Nginx in our web server (192.168.33.10)
# This playbook is written in YAML and YAML starts with three dashes (front matter)

---
# name of the hosts - hosts is to define the name of your host of all
- hosts: web

# find the facts about the host
  gather_facts: yes

# admin access
  become: true

# instructions using task module in ansible
  tasks:
  - name: Install Nginx

# install nginx
    apt: pkg=nginx state=present update_cache=yes

# ensure it's running/active
# update cache
# restart nginx if reverse proxy is implemented or if needed
    notify:
      - restart nginx
  - name: Allow all access to tcp port 80
    ufw:
        rule: allow
        port: '80'
        proto: tcp

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```
# Task 
### 
