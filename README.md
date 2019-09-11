# carsalesEC2patching
carsalesEC2patching using Ansible Playbook

Below are the steps to be followed in implementing the EC2 patching using AWS SSM and Ansible Playbook.

UseCase: Two EC2 machines with different linux versions ( Redhat, Ubuntu ) running on the AWS account. Requirement is to patch both the
EC2 instance and reboot at different timings based on its application criticality after business hours if required. The application type is been tagged on EC2 console mentioning Critical or Non Critical.

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
      when:  EC2facts.tag.AppType == "Critical"

    - name: Reboot all the non critical application after patching.
      command: shutdown -r +1 “Rebooting System After Patching”
      when: EC2facts.tag.AppType == "NonCritical"
      
Step 3: Select the AWS service system manager "Run Command on the left"
Select the instance to patch based on the tags or groups or manual selection.

Step 4: Using the Run command, AWS CLI execute the below command by placing the above playbook in s3 bucket.
aws ssm send-command --document-name "AWS-RunAnsible" --instance-ids "Redhat AppA id " "Ubuntu AppB id" "" --max-errors 1 --parameters '{"extravars":["SSM=True"],"check":["False"],"playbook":["s3://carsaless3bucket/EC2patching.yml"]}' --timeout-seconds 600 --region ap-southeast-2a

Step 5: Run Command shows the progress of the OS patching by running the playbook and the result of success. The logs are being logged to the s3 bucket choosen to log the results.
      
      
