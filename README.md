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


### Phase 3: Jenkins UI Configuration (Web Dashboard)

Add Credentials: Manage Jenkins > Credentials > Global > Add Credentials

Kind: SSH Username with private key

ID: slave-ssh-key

Username: jenkins

Private Key: Paste the content of ~/.ssh/id_rsa_jenkins from the Master.

Create the Node: Manage Jenkins > Nodes > New Node

Name: Slave-01

Remote root directory: /home/jenkins

Launch Method: Launch agents via SSH

Host: 172.31.47.211

Credentials: slave-ssh-key

Host Key Verification: Non-verifying Verification Strategy.

JVM settings: -Djava.io.tmpdir=/home/jenkins/tmp


SRE Optimization (Crucial for t2.micro/Small Disks):

Navigate to Nodes > Configure Monitors.

Change Free Disk Space and Free Temp Space thresholds from 1GiB to 200MiB. This prevents Jenkins from marking the node as "Offline" due to low disk space common in cloud environments.

mkdir /home/jenkins/tmp

sudo chown jenkins:jenkins /home/jenkins/tmp

chmod 755 /home/jenkins/tmp
