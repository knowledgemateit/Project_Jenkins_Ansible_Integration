# Project_Jenkins_Ansible_Integration

## Phase 1: Infrastructure & SSH Setup
This phase ensures the Master can "talk" to the Slave as the jenkins user without passwords or security prompts.

On the Master (as root): Create and set permissions for the jenkins user's keys.

mkdir -p /home/jenkins/.ssh

ssh-keygen -t rsa -b 4096 -f /home/jenkins/.ssh/id_rsa -N ""

chown -R jenkins:jenkins /home/jenkins/.ssh

chmod 700 /home/jenkins/.ssh

chmod 600 /home/jenkins/.ssh/id_rsa


On the Slave (as jenkins): Authorize the Master's key.
# Copy the output of 'cat /home/jenkins/.ssh/id_rsa.pub' from Master
# Then on Slave:
sudo dnf install java-21-amazon-corretto-devel maven -y

sudo useradd -m jenkins

sudo su - jenkins

mkdir -p ~/.ssh && chmod 700 ~/.ssh

echo "PASTE_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys

chmod 600 ~/.ssh/authorized_keys

ssh -i /home/jenkins/.ssh/id_rsa jenkins@3.64.127.91

Global SSH Trust: On the Master, edit /etc/ansible/ansible.cfg and set:

host_key_checking = False
