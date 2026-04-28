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
mkdir -p ~/.ssh
echo "PASTE_MASTER_PUB_KEY_HERE" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
Global SSH Trust: On the Master, edit /etc/ansible/ansible.cfg and set:
host_key_checking = False

## Phase 2: Ansible Configuration (On Master)
This defines who to target and how to deploy.

1. The Inventory (/opt/ansible/hosts):
[agent_nodes]
slave_node ansible_host=172.31.47.211 ansible_user=jenkins ansible_ssh_private_key_file=/home/jenkins/.ssh/id_rsa



2. The Playbook (/opt/ansible/deploy-docker.yml):
---
- name: Deploy WAR to Docker on Agent
  hosts: agent_nodes
  tasks:
    - name: Copy WAR from Master to Slave
      copy:
        src: "{{ war_path }}"
        dest: "/tmp/PollApp.war"
        mode: '0755'

    - name: Push WAR into Tomcat Container
      shell: "docker cp /tmp/PollApp.war tomcat-prod:/usr/local/tomcat/webapps/PollApp.war"

    - name: Refresh Tomcat
      shell: "docker exec tomcat-prod /usr/local/tomcat/bin/catalina.sh stop && docker exec tomcat-prod /usr/local/tomcat/bin/catalina.sh start"
      ignore_errors: yes
	  
	  
## Phase 3: The Final Jenkins Pipeline
This is the logic that coordinates the build on the Slave and the push from the Master.

pipeline {
    agent none 

    stages {
        stage('Build & Package') {
            agent { label 'Slave-01' } 
            steps {
                git branch: 'main', url: 'https://github.com/knowledgemateit/Project-Maven-Tomcat-Nexus-Sonar-Setup-PollApp.git'
                sh 'mvn clean package -DskipTests'
                
                // Save the WAR file to Jenkins controller memory
                stash includes: 'target/*.war', name: 'app-war'
            }
            post {
                always { cleanWs() } // Clean slave workspace
            }
        }

        stage('Ansible Deploy') {
            agent { label 'built-in' } // Run Ansible from Master
            environment {
                ANSIBLE_HOST_KEY_CHECKING = 'False'
            }
            steps {
                // Pull the WAR file onto the Master's local workspace
                unstash 'app-war'
                
                ansiblePlaybook(
                    playbook: '/opt/ansible/deploy-docker.yml',
                    inventory: '/opt/ansible/hosts',
                    installation: 'ansible-latest',
                    colorized: true,
                    extraVars: [
                        war_path: "${WORKSPACE}/target/PollApp.war"
                    ]
                )
            }
            post {
                always { cleanWs() } // Clean master workspace
            }
        }
    }
}
## Phase 4: Why This Specific Setup Worked
Stash/Unstash: Solved the "No such file or directory" error by physically moving the build artifact from the Slave to the Master.

extraVars: Solved the "Absolute Path" issue by telling Ansible exactly where Jenkins stored the file in the current workspace.

Agent none + Labels: Saved your 4GB Master from crashing by ensuring the heavy Maven build stayed on the Slave.

Docker Hybrid: By running Tomcat as a container on the Slave, you utilized existing resources without needing a 3rd server.
