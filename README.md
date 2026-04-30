# Project_Jenkins_Ansible_Integration

Create two server (4gb Ram,Amazon Linux,30GB Disk)

**Master:**

sudo su

hostname master

sudo dnf install java-21-amazon-corretto-devel maven tree git ansible -y

sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo

sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

sudo dnf install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins

Temp folder issue fix:
sudo mkdir -p /etc/systemd/system/jenkins.service.d/

sudo tee /etc/systemd/system/jenkins.service.d/override.conf <<EOF
[Service]
Environment="JAVA_OPTS=-Djenkins.model.Nodes.minSpaceThreshold=104857600 -Djava.io.tmpdir=/var/lib/jenkins/tmp"
Environment="JENKINS_PORT=8090"
EOF

sudo mkdir -p /var/lib/jenkins/tmp
sudo chown -R jenkins:jenkins /var/lib/jenkins/tmp

sudo systemctl daemon-reload
sudo systemctl reset-failed jenkins
sudo systemctl restart jenkins

---------------------------------------------------

**Slave:**

sudo su
hostname agent


sudo dnf install java-21-amazon-corretto-devel maven git -y

sudo dnf install docker -y

sudo systemctl start docker

sudo systemctl enable docker

-----------------------------------------------------------

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

sudo useradd -m jenkins

sudo su - jenkins

mkdir -p ~/.ssh && chmod 700 ~/.ssh

cat /home/jenkins/.ssh/id_rsa.pub   -> Take from Master
echo "PASTE_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys

chmod 600 ~/.ssh/authorized_keys

ssh -i /home/jenkins/.ssh/id_rsa jenkins@3.70.175.45

------------------------------------------

### Phase 3: Jenkins UI Configuration (Web Dashboard)

**Add Credentials**: Manage Jenkins > Credentials > Global > Add Credentials

Kind: SSH Username with private key

ID: slave-ssh-key

Username: jenkins

Private Key: Paste the content of /home/jenkins/.ssh/id_rsa from the Master.

**Create the Node**: Manage Jenkins > Nodes > New Node

Name: Slave-01

Remote root directory: /home/jenkins

Launch Method: Launch agents via SSH

Host: 172.31.47.211

Credentials: slave-ssh-key

Host Key Verification: Non-verifying Verification Strategy.

JVM settings: -Djava.io.tmpdir=/home/jenkins/tmp

mkdir /home/jenkins/tmp

sudo chown jenkins:jenkins /home/jenkins/tmp

chmod 755 /home/jenkins/tmp

Navigate to Nodes > Configure Monitors.

Change Free Disk Space and Free Temp Space thresholds from 1GiB to 200MiB. This prevents Jenkins from marking the node as "Offline" due to low disk space common in cloud environments.

-------------------
Install Ansible Plugin in Jenkins master

Add that as tool in ansible-latest
**Name  **ansible-latest
**Path to ansible executables directory **  : /usr/bin



