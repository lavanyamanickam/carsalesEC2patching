# carsalesEC2patching
carsalesEC2patching using Ansible Playbook

Below are the steps to be followed in implementing the EC2 patching using AWS SSM and Ansible Playbook.

UseCase: Two EC2 machines with different linux versions ( Redhat, Ubuntu ) running on the AWS account. Requirement is to patch both the
EC2 instance and reboot at differnt timings based on its application criticality after business hours if required. The application type is been tagged on EC2 mentioning Critical or Non Critical.

Step 1: Install Ansible on both the EC2 while creating the EC2 instances using the bootstrap command.
( For Ubuntu )
sudo apt-get install ansible -y 
( For Redhat 7 )
sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum -y install ansible

Step 2: Below is the Ansible playbook for EC2 patching.

- hosts: all
    become: true

    tasks:
    - name: gather ec2 facts
      action: ec2_facts
      register: EC2facts

    - name: Patching on redhat or centos instances
      yum: name="*" state=latest
      when: ansible_os_family == "RedHat"

    - name: Patching  on debian or ubuntu instances
      apt: upgrade=yes update_cache=yes
      when: ansible_os_family == "Debian"

    - name: Reboot the critical application during non business hours.
      command: echo "/sbin/shutdown -h now" |at 20:00 today
      ignore_errors: true
      when: "tag:AppType" == "Critical"

    - name: Reboot all the non critical application after patching.
      command: shutdown -r +1 “Rebooting System After Patching”
      when: "tag:AppType" == "NonCritical"
      
      
      
