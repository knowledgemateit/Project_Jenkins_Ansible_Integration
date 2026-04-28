pipeline {
    agent none 

    environment {
        // Centralizing paths makes the script easier to maintain
        ANSIBLE_PROJECT_DIR = "ansible"
        PLAYBOOK_PATH       = "ansible/playbooks/deploy-docker.yml"
        INVENTORY_PATH      = "ansible/inventory/hosts.ini"
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }

    stages {
        stage('Build & Package') {
            agent { label 'Slave-01' } 
            steps {
                echo '--- Executing Build on Slave Agent ---'
                git branch: 'main', url: 'https://github.com/knowledgemateit/Project-Maven-Tomcat-Nexus-Sonar-Setup-PollApp.git'
                
                sh 'mvn clean package -DskipTests'
                
                // Save the artifact to Jenkins controller memory
                stash includes: 'target/*.war', name: 'app-war'
            }
            post {
                always {
                    cleanWs() // Clear Slave workspace immediately after stash
                }
            }
        }

        stage('Ansible Deploy') {
            agent { label 'built-in' } // Runs on the Master
            steps {
                echo '--- Executing Ansible Deployment from Master ---'
                
                // Pull the repository on the Master to get the Playbook and Inventory
                git branch: 'main', url: 'https://github.com/knowledgemateit/Project_Jenkins_Ansible_Integration.git'
                
                // Pull the WAR file from the previous stage
                unstash 'app-war'
                
                ansiblePlaybook(
                    playbook: "${PLAYBOOK_PATH}",
                    inventory: "${INVENTORY_PATH}",
                    installation: 'ansible-latest',
                    colorized: true,
                    extraVars: [
                        // Dynamically pass the absolute path of the WAR file to Ansible
                        war_path: "${WORKSPACE}/target/PollApp.war"
                    ]
                )
            }
            post {
                success {
                    echo "SUCCESS: PollApp deployed to http://172.31.47.211:8080/PollApp"
                }
                always {
                    cleanWs() // Clear Master workspace
                }
            }
        }
    }
}
